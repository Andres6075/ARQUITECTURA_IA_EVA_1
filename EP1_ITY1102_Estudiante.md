# EP1_ITY1102_Estudiante

v

Evaluación Parcial 1

Diseño y Documentación de Arquitectura de Sistemas  IA

Encargo | Estudiante

2025

Evaluación Parcial N°1

Diseño y Documentación de Arquitectura de Sistemas IA

Encargo | Estudiante

| Sigla | Nombre Asignatura | Tiempo Asignado | % Ponderación |
| --- | --- | --- | --- |
| ITY1102 | Arquitectura de Sistemas de Inteligencia Artificial | 5 horas pedagógicas | 30% |

1. Instrucciones generales Evaluación Parcial 1

| Descripción |
| --- |
| La Evaluación Parcial 1 consiste en la primera parte del proyecto de arquitectura de sistemas IA en equipos, a partir de un caso empresarial  proporcionado por el/la docente. Los/las estudiantes identifican componentes arquitectónicos, aplican principios de diseño, especifican  infraestructura necesaria y proponen estrategias de integración y despliegue. Esta evaluación medirá los siguientes Indicadores de Logro: ▫ IL1.1 Distingue los componentes principales de una arquitectura de sistemas de IA a partir de la teoria y modelos de referencia en el  contexto del análisis de soluciones tecnológicas para organizaciones. ▫ IL1.2 Analiza los principios de diseño en arquitecturas de IA (escalabilidad, flexibilidad, seguridad, observabilidad y confiabilidad)  considerando los requerimientos de la organización y las mejores prácticas de la industria en el contexto de la toma de decisiones  arquitectónicas. ▫ IL1.3 Relaciona los elementos de infraestructura (CPU, GPU, TPU, frameworks, servicios cloud y redes de comunicación) con las necesidades de procesamiento y almacenamiento de sistemas de IA en el contexto de la implementación de soluciones  empresariales. , ▫ IL1.4 Determina estrategias de integración y despliegue de sistemas de IA (CI/CD, edge computing, modelos en producción) en función  de su eficiencia, rendimiento y confiabilidad en el contexto de la operación de sistemas de inteligencia artificial. |

2025

Página 1 de 9

• Las instrucciones se proporcionarán en la semana 5 y la entrega debe realizarse en la semana 7.

• El tiempo asignado para desarrollar esta evaluación es de 2 horas pedagógicas.

• El encargo se desarrolla de manera grupal (2 a 3 personas), e idealmente el equipo de trabajo debe mantenerse durante todo el  semestre.

• Cada equipo debe desarrollar el encargo durante su tiempo de trabajo autónomo.

Ítem I: Instrucciones específicas de la Evaluación: El encargo corresponde a un informe técnico, que incluye los siguientes apartados:

1.1 Análisis del Caso Empresarial

a. Identificación de requerimientos funcionales y no funcionales del sistema de IA, a partir del caso empresarial proporcionado. Revisar  Anexo Caso de Trabajo Sistema de recomendación de productos para E-Commerce. (IE1)

b. Identificación de componentes principales de la arquitectura: datos, modelo, API, interfaz (IE1)

c. Relación de cada componente con su función específica utilizando modelos de referencia como MLOps, arquitecturas de referencia de  AWS/Azure (IE2)

1.2 Principios de Diseño Arquitectónico

a. Cumplimiento de principios de diseño: escalabilidad, flexibilidad, seguridad, observabilidad y confiabilidad. Explicación técnica de su  aplicación en el caso específico (IE3)

b. Decisiones arquitectónicas clave, basadas en principios de observabilidad y confiabilidad (IE4)

1.3 Especificación de Infraestructura

a. Determinación de recursos computacionales: CPU, GPU, TPU según requerimientos de procesamiento del modelo (IE5) b. Selección de frameworks de desarrollo de IA apropiados (TensorFlow, PyTorch, etc.) (IE5)

c. Selección de servicios cloud (AWS, Azure, GCP) considerando almacenamiento, redes de comunicación y escalabilidad (IE6) d. Dimensionamiento inicial de recursos con estimación de costos (IE6)

2025

Página 2 de 9

| 1.4 Estrategias de Integración y Despliegue a. Selección de estrategias de despliegue: CI/CD, edge computing, modelos en producción según características del proyecto (IE7) b. Comparación de alternativas de integración evaluando eficiencia, rendimiento y confiabilidad (IE8) c. Pipeline propuesto de integración continua y despliegue continuo (IE8) d. Diagrama de flujo de despliegue (IE7) Los aspectos formales son: • Formato: Documento PDF de 10 páginas máximo, utilizando plantilla arc42 simplificada proporcionada por el/la docente. • Estructura: Debe incluir todas las secciones indicadas (1.1 a 1.4) • Referencias: Fuentes y bibliografía en formato APA (mínimo 2 fuentes académicas o técnicas) • Declaración de uso de IA: Incluir declaración explícita sobre el uso de herramientas de IA generativa durante el desarrollo del trabajo,  especificando qué herramientas se utilizaron y para qué propósitos • Entrega: A través de AVA en formato PDF, nombre del archivo: EP1_Equipo[N]_Apellidos.pdf Los materiales, herramientas o insumos que se requieren para realizar esta evaluación: • Plantilla arc42 simplificada (proporcionada por docente en AVA) • Caso empresarial asignado • Software de diagramación: Draw.io o similar • Acceso a documentación técnica de AWS • Material de referencia sobre arquitecturas de IA (disponible en AVA) |
| --- |

2025

Página 3 de 9

| Caso/Antecedentes |
| --- |
| Casos Empresariales Disponibles • Esta evaluación se desarrolla a partir de casos empresariales predefinidos que el/la docente asigna a cada equipo. Se proporcionan 3  casos que deben rotarse entre equipos para evitar duplicación. Caso 1: Sistema de Recomendación para E-commerce (Versión completa disponible en documento Anexo EFT-EP Caso de Trabajo) • Contexto: Retail mediana de tecnología, 100K usuarios activos mensuales, 50K productos. Necesita aumentar conversión de 1.5% a  3.5% mediante recomendaciones personalizadas. • Requerimientos Técnicos: Latencia <200ms, disponibilidad 99.5%, picos de 10K usuarios concurrentes, actualización semanal del  modelo. • Restricciones: Presupuesto AWS $3K/mes, integración con Shopify, cumplimiento Ley Protección Datos, equipo técnico reducido (2  developers). • Datos: 2 años historial compras (500K transacciones), clickstream, datos demográficos básicos. Caso 2: Chatbot Inteligente para Atención al Cliente • Contexto: Operador telecomunicaciones con 500K clientes. 80% llamadas son consultas repetitivas. Objetivo: automatizar 60%  consultas, reducir espera de 8 a 2 minutos. • Requerimientos Técnicos: 24/7, latencia <3 seg/respuesta, multicanal (web, WhatsApp, app), 1K conversaciones simultáneas, español  chileno. • Restricciones: Integración CRM Siebel legacy, cumplimiento GDPR y Ley 19.628, escalamiento gradual (piloto 10% usuarios). • Datos: 50K transcripciones llamadas, 200K tickets soporte (3 años), 500 artículos FAQ. |

2025

Página 4 de 9

Caso 3: Detección de Fraude Financiero

• Contexto: Fintech pagos digitales, 200K usuarios, 200K transacciones diarias. Fraude actual 0.5% ($500K USD/año), sistema con 40%  falsos positivos.

• Requerimientos Técnicos: Latencia crítica <100ms, disponibilidad 99.9%, escalabilidad 1M transacciones/día, explicabilidad para  auditoría.

• Restricciones: Cumplimiento CMF, Ley Delitos Informáticos, PCI-DSS, auditoría trimestral obligatoria. • Datos: 3 años transacciones (200M registros), perfiles usuario, listas negras, geolocalización, 0.5% labels fraudulentos.

Prompt para Generar Casos Adicionales con IA

• Si el/la docente necesita casos adicionales o actualizados:

• Genera un caso empresarial para arquitectura de sistemas IA considerando:

- Sector: [retail/finanzas/manufactura/salud/servicios]

- Problema: [recomendación/predicción/clasificación/NLP/visión]

- Escala: [startup/mediana/grande]

- Incluir: contexto organizacional, problema cuantificado, requerimientos técnicos

(latencia, disponibilidad, escalabilidad), 3 restricciones (presupuesto/

técnicas/regulatorias), datos disponibles (tipo, volumen)

- Complejidad: intermedia (estudiantes 4to-5to semestre)

- Debe permitir análisis de componentes, principios diseño, infraestructura

y estrategias despliegue

Plantilla de Entrega a Estudiantes

• CASO EMPRESARIAL: [Nombre]

CONTEXTO ORGANIZACIONAL

[Descripción empresa, sector, tamaño, usuarios]

2025

Página 5 de 9

| PROBLEMA DE NEGOCIO [Desafío actual, métricas, objetivos] REQUERIMIENTOS TÉCNICOS [Latencia, disponibilidad, volumen, escalabilidad] RESTRICCIONES [Presupuestarias, tecnológicas, regulatorias] DATOS DISPONIBLES [Tipos, volumen, calidad, fuentes] |
| --- |

2. Pauta de Evaluación (Escala de valoración)

| Categoría % logro Descripción niveles de logro |
| --- |

2025

Página 6 de 9

| Muy buen desempeño | 100% | Demuestra un desempeño destacado, evidenciando el logro de todos los aspectos evaluados en el indicador. |
| --- | --- | --- |
| Buen desempeño | 80% | Demuestra un alto desempeño del indicador, presentando pequeñas omisiones, dificultades y/o errores. |
| Desempeño aceptable | 60% | Demuestra un desempeño competente, evidenciando el logro de los elementos básicos del indicador, pero con  omisiones, dificultades o errores. |
| Desempeño incipiente | 30% | Presenta importantes omisiones, dificultades o errores en el desempeño, que no permiten evidenciar los  elementos básicos del logro del indicador, por lo que no puede ser considerado competente. |
| Desempeño no logrado | 0% | Presenta ausencia o incorrecto desempeño. |

| Indicador de Evaluación | Categorías de Respuesta | Ponderación  Indicador de  Evaluación |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
|  | Muy buen desempeño 100% | Buen desempeño 80% | Desempeño  aceptable 60% | Desempeño incipiente 30% | Desempeño no  logrado 0% |  |
| IE1 Identifica los componentes principales de una  arquitectura de IA (datos, modelo, API, interfaz) a  partir de un caso empresarial dado |  |  |  |  |  | 12% |
| IE2 Relaciona los componentes funcionales y no  funcionales de la arquitectura del proyecto con su  función específica, considerando su relación con los  requerimientos del sistema. |  |  |  |  |  | 13% |
| IE3 Determina el cumplimiento de principios de diseño  (escalabilidad, flexibilidad, seguridad) en una  arquitectura de IA propuesta, considerando las buenas  prácticas de la industria. |  |  |  |  |  | 15% |
| IE4 Selecciona los principios de diseño para la  arquitectura del proyecto basados en la observabilidad  y confiabilidad, acorde a los requerimientos técnicos y  organizacionales |  |  |  |  |  | 15% |
| IE5 Selecciona los recursos de infraestructura (CPU,  GPU, TPU) y frameworks apropiados según los  requerimientos de procesamiento de un modelo de IA  definido en el proyecto empresarial. |  |  |  |  |  | 13% |

2025

Página 7 de 9

| IE6 Explica la elección de servicios cloud considerando  las necesidades de almacenamiento y comunicación  del sistema, acorde a las necesidades del proyecto. |  |  |  |  |  | 12% |
| --- | --- | --- | --- | --- | --- | --- |
| IE7 Establece estrategias de despliegue (CI/CD, edge  computing) acorde a los requerimientos del proyecto  empresarial |  |  |  |  |  | 10% |
| IE8 Establece alternativas de integración considerando su impacto en eficiencia y rendimiento del sistema,  según las necesidades del proyecto. |  |  |  |  |  | 10% |
| Total | 100% |  |  |  |  |  |

2025

Página 8 de 9
