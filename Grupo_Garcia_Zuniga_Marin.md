# Grupo_Garcia_Zuniga_Marin

Documento de Arquitectura de Software

DuocBot: asistente de WhatsApp para estudiantes de Duoc UC

Evaluación Parcial 1

Arquitectura de Sistemas de Inteligencia Artificial

Sección 001D

Integrantes:

Javier García

Andrés Zúñiga

Leandro Marin

Fecha: 20/09/2026

Contenidos

# Objetivo y resumen del caso

Este informe documenta la arquitectura de DuocBot, un chatbot de WhatsApp para estudiantes de Duoc UC de la sede San Bernardo y después se llevará a otras sedes.  Hoy las solicitudes, sugerencias y reportes se demoran en responderse y, a veces, el estudiante ni siquiera sabe si su solicitud quedó ingresada.

La propuesta responde en tres niveles: el numero 1 un motor de reglas para lo simple y repetitivo, el nivel 2 un modelo de lenguaje o como se conoce LLM de OpenAI para lo complejo y nivel 3 un operador humano cuando el sistema no tiene permiso o control para actuar por si solo. Como el caso no entrega cifras, dejamos explícitos los supuestos con que dimensionamos el sistema; los validaremos con datos reales durante el piloto.

| Supuesto del piloto | Valor y comentario |
| --- | --- |
| Estudiantes con acceso | 5.000 (supuesto; se validará con la sede San Bernardo) |
| Adopción mensual y volumen | 40% (2.000 usuarios) × 15 mensajes/mes = 30.000 mensajes/mes (~1.000/día) |
| Pico (matrícula, cierre de notas) | 5× el volumen normal = 5.000 mensajes/día; diseñamos para 5 mensajes/s |
| Consultas que llegan al LLM | 30% = 9.000 llamadas/mes; el 70% restante lo resuelven las reglas |
| Solicitudes formales | 500/mes, con 2 notificaciones de estado cada una = 1.000 mensajes de plantilla/mes |
| Presupuesto de referencia | Techo de US$250/mes para el piloto (supuesto del equipo) |

# 1.1 Análisis del caso empresarial

## a) Requisitos funcionales y no funcionales

Cada requisito tiene un identificador, una métrica verificable y el componente que lo cumple, para poder rastrear el requisito hasta la arquitectura (ver Figura 1).

| ID | Requisito | Métrica o criterio | Componente responsable |
| --- | --- | --- | --- |
| RF1 | Responder consultas frecuentes: horarios, ramos, notas, certificados y estado de solicitudes | ≥ 70% resuelto sin IA | Clasificador, motor de reglas, sistema académico |
| RF2 | Recibir solicitudes, sugerencias y reportes desde WhatsApp | 100% queda con número de caso | Webhook/API, PostgreSQL |
| RF3 | Registrar y avisar al estudiante cada cambio de estado (falla que hoy genera molestia) | Aviso en < 1 min tras el cambio | Backend, WhatsApp (plantillas) |
| RF4 | Pasar el caso a una persona cuando la IA no puede o no debe actuar | 100% de casos sin permiso o con falla de IA, con historial | Módulo de escalamiento, backoffice |
| RF5 | Recordar el contexto de la conversación por estudiante | Últimos 10 turnos disponibles | PostgreSQL, módulo IA |
| RF6 | Verificar la identidad del estudiante antes de entregar datos personales | Código OTP al correo institucional | Módulo de identidad, PostgreSQL |
| RF7 | Permitir al operador gestionar casos desde un panel | Ver cola, historial y cerrar caso | Backoffice |
| RNF1 | Latencia: Respuesta rápida: la espera es lo que más molesta hoy | Menos de 2 segundos para respuestas por reglas y menos de 8 segundos cuando interviene la IA | Webhook, motor de reglas y optimización de llamadas al LLM. |
| RNF2 | Disponibilidad: Servicio 24/7 durante el piloto | 99,5% de tiempo en línea al mes máximo 3,6 horas de parada planificada o imprevista | Infraestructura en la nube con balanceador de carga ALB y autoescalado. |
| RNF3 | Escalabilidad: Aguantar picos sin degradarse | Ante ráfagas de carga 5 veces superiores a lo normal, la respuesta no debe ralentizarse más de un 20%. | Auto Scaling, cola de mensajes SQS |
| RNF4 | Confiabilidad: Nada se pierde ni se procesa dos veces | 0 mensajes perdidos; 0 duplicados | SQS + DLQ, idempotencia |
| RNF5 | Seguridad: Datos académicos protegidos (Ley 19.628 y Ley 21.719) | Cifrado, acceso por rol, sin datos personales hacia el LLM | Sanitizador PII, IAM, Secrets Manager |
| RNF6 | Observabilidad: Reconstruir cualquier trámite de principio a fin | 100% de interacciones con ID de correlación | CloudWatch, logs JSON |
| RNF7 | Costo: La IA se usa solo cuando hace falta | ≤ US$250 al mes | Diseño de dos niveles |
| RNF8 | Idioma: Solo español en esta versión | Prompts y reglas en español | Clasificador, módulo IA |

## b) Componentes principales: datos, modelo, API e interfaz

La arquitectura tiene las cuatro piezas de cualquier sistema de IA. La Figura 1 muestra cómo se conectan y por dónde viaja un mensaje.

Datos: PostgreSQL propio como usuarios, conversaciones, solicitudes, casos escalados y base de conocimiento con FAQ y reglamentos, con búsqueda semántica pgvector y el sistema académico de Duoc, del que se consultan horarios, ramos y notas. Supuesto a confirmar: Duoc expone una API o vista de solo lectura.

Modelo: un clasificador de intención spaCy + regresión logística, umbral de confianza 0,75, el motor de reglas y el módulo IA LLM de OpenAI con recuperación de contexto desde la base de conocimiento y guardrails. Un sanitizador quita datos personales antes de llamar al LLM.

API: WhatsApp Business Cloud API Meta, webhook y endpoints del backend en FastAPI, y una cola SQS que desacopla la recepción del procesamiento.

Interfaz: WhatsApp para el estudiante y un backoffice web para el operador.

Ejemplo 1, consulta de horario: el estudiante escribe -> el webhook encola el mensaje -> se verifica su identidad -> el clasificador reconoce la intención con confianza alta -> el motor de reglas consulta el sistema académico y responde por WhatsApp. Ejemplo 2, certificado especial: el clasificador tiene poca confianza y lo manda al módulo IA -> la IA detecta que la acción requiere una autorización fuera de su control -> el módulo de escalamiento crea el caso con el historial completo, avisa al operador y confirma al estudiante que su solicitud quedó recibida.

## c) Relación con modelos de referencia MLOps y AWS Azure

Para no inventar la arquitectura desde cero la contrastamos con MLOps y con las arquitecturas de referencia de AWS y Azure . Como no entrenamos un LLM propio, la etapa de reentrenamiento no aplica; en su lugar versionamos y evaluamos el clasificador, las reglas y los prompts.

| Referencia y pilar | Práctica recomendada | Dónde se aplica en DuocBot |
| --- | --- | --- |
| MLOps: ciclo datos -> modelo -> despliegue -> monitoreo | Versionar datos y modelos, automatizar el despliegue y monitorear en producción | BD y base de conocimiento; clasificador, reglas y LLM; CI/CD ; métricas |
| AWS ML Lens: excelencia operativa | Automatizar, versionar y tener procedimientos de recuperación | GitHub Actions, imágenes Docker con tag por commit, rollback |
| AWS ML Lens: seguridad | Mínimo privilegio, cifrado, gestión de secretos | IAM por rol, Secrets Manager, TLS, cifrado en RDS y S3 |
| AWS ML Lens: confiabilidad | Desacoplar con colas, reintentos, respaldos | SQS + DLQ, health checks del ALB, backups de RDS |
| AWS ML Lens: desempeño y costos | Dimensionar según la carga; usar servicios gestionados | Solo CPU; reglas antes que LLM; instancias 1–3 |
| Azure Well-Architected: cinco pilares (confiabilidad, seguridad, costos, excelencia operativa y eficiencia) | Mismas ideas; equivalencias de servicios: ALB ≈ Application Gateway, RDS ≈ Azure Database for PostgreSQL, SQS ≈ Service Bus, CloudWatch ≈ Azure Monitor | Diseño portable: si el proyecto migra a Azure, cambian los servicios, no la arquitectura |

# 1.2 Principios de diseño arquitectónico

## a) Cómo cumplimos los cinco principios

| Principio | Cómo lo cumplimos en DuocBot | Cómo lo comprobamos |
| --- | --- | --- |
| Escalabilidad | Backend sin estado en un Auto Scaling Group (1–3 instancias, escala con CPU > 60%). La cola SQS absorbe los picos y los workers la consumen a su ritmo. PostgreSQL crece en vertical. | Prueba de carga a 5 mensajes/s (Locust): el p95 no empeora más de 20%. |
| Flexibilidad | Puertos y adaptadores: interfaces LLMProvider y ChannelProvider. Reglas, prompts y base de conocimiento son archivos versionados en Git. Se puede cambiar de proveedor de IA o canal sin tocar el resto. | Cambiar el proveedor de LLM solo por configuración con las pruebas de regresión en verde. |
| Seguridad | Identidad: OTP al correo institucional antes de mostrar notas o certificados. Privacidad: el LLM nunca recibe RUT, nombre ni notas (el sanitizador los reemplaza por marcadores y solo el motor de reglas los inserta en la respuesta). LLM: responde solo con fragmentos de la base de conocimiento, sin permisos de escritura, y el texto del usuario se trata como dato, no como instrucción (OWASP, 2025). Base: TLS, cifrado en reposo, secretos en Secrets Manager, firma del webhook de Meta verificada, IAM de mínimo privilegio. | Pruebas de inyección de prompts; revisión trimestral de accesos; prueba automática de que los logs no contienen RUT ni notas. |
| Observabilidad | Logs JSON con ID de correlación por mensaje, métricas y alarmas centralizadas en CloudWatch (detalle en 1.2b). | Reconstruir cualquier trámite de punta a punta desde los logs. |
| Confiabilidad | Cola con reintentos con espera creciente y DLQ; idempotencia por ID de mensaje de WhatsApp (Kleppmann, 2017); timeouts en llamadas externas; si la IA falla, el caso pasa a un operador. | Bloquear la API de OpenAI en staging y confirmar que todos los mensajes llegan al operador. |

## b) Decisiones clave (observabilidad y confiabilidad)

| ID | Decisión | Alternativa descartada | Motivo y consecuencia |
| --- | --- | --- | --- |
| D1 | Reglas antes que IA | Resolver todo con el LLM | Más barato, más rápido y sin depender de un tercero para lo simple. Costo: mantener las reglas. |
| D2 | Procesamiento asíncrono con cola | Responder dentro del webhook | Meta reintenta si el webhook no responde a tiempo; con la cola se responde 200 de inmediato y no se pierden ni duplican mensajes. Costo: +50–200 ms. |
| D3 | Todo lo fuera del permiso de la IA se deriva a una persona | Dejar que la IA ejecute acciones | Nunca se ejecuta algo que no corresponde (por ejemplo, cambiar una nota); el operador recibe el historial. |
| D4 | Log estructurado con ID de correlación y sin datos personales | Logs de texto libre | Permite auditar y depurar sin exponer datos de estudiantes. |
| D5 | Degradación controlada: si el LLM falla, reintento con espera y luego escalamiento | Devolver un error al estudiante | Nadie queda sin respuesta; se activa una alarma. |
| D6 | Medir la calidad de la IA: revisión semanal de 50 conversaciones y botón “¿Te sirvió?” | Suponer que las respuestas son buenas | Detecta respuestas inventadas o que se degradan con el tiempo. |

| Métrica observable | Meta (SLO) | Alarma |
| --- | --- | --- |
| Latencia p95 respuestas por reglas / con IA | < 2 s / < 8 s | Sobre la meta durante 5 min |
| Tasa de errores 5xx | < 1% | > 5% durante 10 min (dispara rollback) |
| Fallos de la API del LLM | < 5% | > 10% durante 5 min: modo solo reglas y operador |
| Antigüedad del mensaje más antiguo en la cola | < 30 s | > 60 s |
| Mensajes en la DLQ | 0 | ≥ 1: aviso inmediato al equipo |
| Resolución sin IA | ≥ 70% | < 60% en la semana: revisar reglas |
| Costo diario del LLM (tokens por conversación) | ≤ US$2,5 por día | > US$3 por día |
| Satisfacción (“¿Te sirvió?”) | ≥ 80% | < 70% en la semana |

# 1.3 Especificación de infraestructura

## a) Recursos computacionales (CPU, GPU y TPU)

El LLM se consume por API de OpenAI, así que la inferencia pesada ocurre fuera de nuestra infraestructura. Lo que corremos nosotros (clasificador, reglas, API y workers) es liviano: al pico de 5 mensajes/s, un clasificador de regresión logística responde en milisegundos, y el tiempo de cada solicitud lo domina la espera del LLM y del sistema académico, no la CPU.

| Recurso | Decisión | Justificación |
| --- | --- | --- |
| CPU | EC2 t3.medium (2 vCPU), 1 a 3 instancias | Suficiente para 5 mensajes/s con workers asíncronos; se escala horizontalmente en picos. |
| RAM | 4 GiB por instancia | Modelo de spaCy en memoria, conversaciones activas y dos contenedores (API y worker). |
| GPU | No por ahora | El entrenamiento del clasificador toma segundos en CPU y el LLM no se aloja acá. Se evaluaría (instancia g5) solo si se hiciera fine-tuning de un modelo propio. |
| TPU | No | Solo se justifican para entrenar o servir redes grandes en TensorFlow/JAX; no es nuestro caso. |

## b) Frameworks de desarrollo

| Tecnología | Decisión | Justificación |
| --- | --- | --- |
| FastAPI (Python) | Elegido para API y backoffice | Es asíncrono (ideal para webhooks y llamadas lentas al LLM), valida datos con Pydantic y documenta la API solo. Se descarta Django por ser más pesado para un equipo pequeño; el backoffice usa plantillas Jinja2 + HTMX. |
| spaCy + scikit-learn | Elegidos para el clasificador | spaCy (es_core_news_md) limpia el texto y scikit-learn entrena la regresión logística de intenciones. Se descarta NLTK porque spaCy ya trae modelos preentrenados para español. |
| OpenAI SDK | Elegido para el LLM | Cliente oficial, detrás de la interfaz LLMProvider para poder cambiarlo. |
| SQLAlchemy + Alembic, Pytest, Docker | Elegidos | Acceso a datos con pool de conexiones y migraciones; pruebas automáticas; imágenes idénticas en pruebas y producción. |
| TensorFlow / PyTorch | No se usan por ahora | No entrenamos redes neuronales. Si más adelante se hace fine-tuning, se usaría PyTorch por su ecosistema. |

## c) Servicios cloud AWS, Azure y GCP

Elegimos AWS. Las tres nubes ofrecen servicios equivalentes (balanceador, PostgreSQL gestionado, colas, almacenamiento de objetos y monitoreo), así que la decisión pesa en los siguientes puntos:

| Criterio | AWS (elegida) | Azure | GCP |
| --- | --- | --- | --- |
| Referencia y documentación | ML Lens de Well-Architected y documentación técnica que entrega el curso | Well-Architected de Azure; buena documentación | Arquitecturas de referencia propias |
| Servicios que necesitamos | ALB, RDS, SQS, S3, CloudWatch, Secrets Manager: todo gestionado | Equivalentes (ver 1.1c) | Equivalentes |
| Cercanía y datos | sa-east-1 (São Paulo) al redactar; validar si hay región en Chile | Brazil South | Tiene región en Santiago (ventaja de residencia de datos) |
| Riesgo y mitigación | Datos personales fuera de Chile: se minimizan y solo el sanitizador decide qué sale | Igual | Menor, pero con menos referencia en el curso |

| Necesidad | Servicio AWS | Por qué |
| --- | --- | --- |
| Almacenamiento | RDS PostgreSQL 10–20 GB y S3 para logs y respaldos | Gestionado, con backups automáticos y cifrado; S3 barato y duradero. |
| Red y comunicación | VPC con subred pública (ALB y backend) y privada (base de datos), HTTPS, SQS entre webhook y workers | La base de datos nunca es accesible desde Internet; la cola desacopla y protege ante picos. |
| Escalabilidad | Auto Scaling Group tras el ALB (1–3 instancias) | Crece solo en picos y evita pagar capacidad ociosa. |
| Monitoreo y secretos | CloudWatch (logs, métricas, alarmas) y Secrets Manager | Observabilidad centralizada y claves fuera del código. |

## d) Dimensionamiento inicial y costos

Cálculo del LLM: 9.000 llamadas/mes × (1.500 tokens de entrada + 300 de salida) = 13,5 M de tokens de entrada + 2,7 M de salida. Con un modelo pequeño de referencia US$0,15 y US$0,60 por millón de tokens con  cuesta ≈ US$4; con un modelo grande (referencia US$2,50 y US$10) ≈ US$61. Los costos de AWS usan precios de referencia de us-east-1 con un recargo asumido de 40% para sa-east-1.

| Recurso | Tamaño inicial | US$/mes | Detalle |
| --- | --- | --- | --- |
| EC2 t3.medium + IP pública | 1 instancia (hasta 3 en picos) | 30 – 36 | 2 vCPU, 4 GiB |
| RDS PostgreSQL db.t4g.micro | Single-AZ, 20 GB, backups | 14 – 18 |  |
| ALB (incluye IPv4 públicas) | 1 balanceador | 22 – 30 |  |
| S3, CloudWatch, Secrets Manager, ECR, SQS | Uso bajo | 8 – 15 |  |
| Instancias extra en picos | 0–2 instancias unas horas | 0 – 10 |  |
| Subtotal AWS (referencia) |  | 74 – 109 |  |
| Recargo estimado región sa-east-1 | ~40% sobre la infraestructura AWS | 30 – 44 | Supuesto |
| API de OpenAI | 9.000 llamadas/mes | 4 – 61 | Según modelo |
| WhatsApp: plantillas de notificación | 1.000 mensajes × US$0,01–0,03 | 10 – 30 | Validar tarifa |
| Total de referencia |  | ≈ 118 – 244 | < US$250 |

Son estimaciones nuestras, no cotizaciones: los precios cambian y deben validarse en la calculadora de AWS, en la tarifa de OpenAI y en la de Meta para Chile. El costo más sensible es el LLM (depende del modelo elegido y del 30% supuesto). Además, WhatsApp permite responder libremente dentro de las 24 horas posteriores al último mensaje del estudiante (Meta, s. f.); las notificaciones fuera de esa ventana requieren plantillas aprobadas y se cobran.

# 1.4 Estrategias de integración y despliegue

## a) Estrategias de despliegue elegidas

CI/CD: cada cambio pasa por build, pruebas, evaluación de respuestas de IA, escaneo de seguridad y despliegue automático a staging; producción requiere una aprobación manual. Se libera con rolling update y health checks del ALB. No usamos canary por costo y por volumen del piloto.

Modelos en producción: el LLM se llama por API con la versión del modelo fijada en configuración. El clasificador, las reglas, los prompts y la base de conocimiento se versionan en Git y viajan dentro de una imagen Docker inmutable; volver atrás es volver a desplegar la imagen anterior.

Edge computing: no aplica. El celular solo conversa por WhatsApp y no procesa nada en el dispositivo, y la latencia objetivo se cumple desde la nube. Se reevaluaría solo si hubiera que procesar datos en la sede.

## b) Comparación de alternativas de integración

Comparamos las alternativas de canal y las de arquitectura de integración según eficiencia, rendimiento y confiabilidad.

| Canal | Eficiencia (costo y esfuerzo) | Rendimiento | Confiabilidad | Veredicto |
| --- | --- | --- | --- | --- |
| WhatsApp Cloud API (Meta) | Cobro por plantilla; cuenta por aprobar | Sin intermediarios | Depende de Meta | Elegido: canal que el estudiante ya usa |
| Twilio sobre WhatsApp | Cobra un extra por mensaje; un solo SDK multicanal | Un salto más | No suma redundancia: también depende de Meta | Descartado como respaldo; queda como opción si hay más canales |
| Telegram | Gratis y simple | Bueno | Buena, pero casi sin usuarios estudiantes | Descartado |
| App propia | Desarrollo, publicación y mantención altos | Control total | Depende de instalación | Descartado |

| Decisión de integración | Alternativas | Elegida y por qué |
| --- | --- | --- |
| Procesamiento del mensaje | Síncrono en el webhook vs. cola SQS | Cola: el webhook responde en < 1 s, se absorben los picos y no se pierden mensajes; agrega 50–200 ms, dentro del p95 de 2 s. |
| Acceso al sistema académico | Consulta en línea vs. réplica nocturna (ETL) vs. caché | Consulta en línea con caché de 5 min solo para horarios y ramos; notas y estados sin caché. El ETL nocturno se usa solo para actualizar la base de conocimiento. |
| Proveedor del LLM | API de OpenAI vs. Amazon Bedrock vs. modelo propio con GPU | OpenAI tras la interfaz LLMProvider: menor esfuerzo y pago por uso. Bedrock queda como alternativa si la privacidad exige mantener los datos en AWS; el modelo propio se descarta por GPU y equipo reducido. |

## c) Pipeline de CI/CD propuesto

La Figura 2 muestra el camino de un cambio, desde el commit hasta producción. Criterios de aprobación propuestos: pruebas de reglas y clasificador en verde, ≥ 90% de aciertos en el set de intenciones, ≥ 90% de respuestas de IA aprobadas en el set de regresión y cero secretos o dependencias críticas expuestas. Si el error 5xx supera el 5% durante 10 minutos tras el despliegue, se hace rollback automático (Sato et al., 2019). Todo se implementa con GitHub Actions.

## d) Diagrama de flujo de despliegue

La Figura 3 muestra dónde corre cada pieza. El backend queda en una subred pública sin puertos abiertos salvo desde el ALB para evitar el costo del NAT Gateway serían unos US$35 o más al mes, en el piloto, al escalar se mueve a una subred privada con NAT. La base de datos siempre está en la subred privada. El ALB y RDS exigen subredes en dos zonas de disponibilidad, aunque RDS opere en una sola.

| Etapa | Detalles |
| --- | --- |
| Base de datos | Pool de conexiones con SQLAlchemy; claves en Secrets Manager; cada operación guarda fecha, hora y origen; permisos por rol puede ser estudiante, operador, sistema . Con mínimo privilegio, datos cifrados en tránsito y en reposo, retención y archivado definidos,ETL nocturno solo para la base de conocimiento. |
| Alojamiento | AWS: EC2 en Auto Scaling Group tras el ALB, RDS en subred privada, S3 para logs y respaldos, SQS entre webhook y workers y VPC con subredes públicas y privadas. |
| Producción | GitHub Actions con aprobación manual; escalado horizontal; monitoreo con CloudWatch; equipo de 2 a 3 desarrolladores. Recuperación ante desastres: RPO ≤ 15 min (recuperación a un punto en el tiempo de RDS) y RTO ≤ 4 h (redespliegue de la imagen y restauración de la base). |

# Limitaciones y próximos pasos

Los volúmenes, costos y el 30% de consultas hacia el LLM son supuestos; lo primero será medir el uso real durante el piloto y ajustar el dimensionamiento.

Falta confirmar con Duoc la API del sistema académico, el acuerdo de tratamiento de datos con OpenAI y las tarifas de meta para Chile.

La primera versión es solo en español y solo para la sede San Bernardo. Si el piloto funciona: más sedes, revisar cuántas consultas cubren las reglas para gastar menos en LLM y, solo si hiciera falta, evaluar un modelo propio ahí sí habría que usar GPU.

# Referencias

Amazon Web Services. (2024). Machine Learning Lens: AWS Well-Architected Framework. AWS Documentation. https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/

Chile. (2024, 13 de diciembre). Ley 21.719: Regula la protección y el tratamiento de los datos personales y crea la Agencia de Protección de Datos Personales. Diario Oficial de la República de Chile.

Kleppmann, M. (2017). Designing data-intensive applications. O’Reilly Media.

Meta. (s. f.). WhatsApp Business Platform: Cloud API. Meta for Developers. https://developers.facebook.com/docs/whatsapp/cloud-api

Microsoft. (s. f.). Azure Well-Architected Framework. Microsoft Learn. https://learn.microsoft.com/azure/well-architected/

OWASP Foundation. (2025). OWASP Top 10 for LLM Applications. https://genai.owasp.org/llm-top-10/

Sato, D., Wider, A., & Windheuser, C. (2019). Continuous delivery for machine learning. martinfowler.com. https://martinfowler.com/articles/cd4ml.html

Treveil, M., Omont, N., Stenac, C., Lefevre, K., Phan, D., Zentici, J., Lavoillotte, A., Miyazaki, M., & Heidmann, L. (2020). Introducing MLOps: How to scale machine learning in the enterprise. O’Reilly Media.

# Declaración de uso de IA generativa

Usamos herramientas de IA generativa (ChatGPT y Claude, de Anthropic) como apoyo en este trabajo: para ordenar y redactar descripciones, generar borradores de tablas comparativas y de dimensionamiento, revisar el informe contra la pauta de evaluación, proponer mejoras y generar los diagramas, y revisar ortografía y consistencia. Las decisiones de arquitectura, los supuestos, la definición de requisitos y la validación técnica del contenido fueron revisados y aprobados por el equipo (Leandro Marin, Javier García y Andrés Zúñiga).
