# BORRADOR - GENERADO POR GEMINI

# Formulación de Proyecto Capstone: Análisis Exploratorio Multidimensional y Modelamiento de Rendimiento en la PAES (Procesos 2023-2026)

**Curso:** (202607)(INF074H) INTRODUCCIÓN Y FUNDAMENTOS ESTADÍSTICOS
**Profesora:** Daniela Opitz
**Integrantes:** Daniela Contreras - Emanuel Nicklichen
**Institución:** Departamento de Informática, Universidad Técnica Federico Santa María

---

### 1. El problema y la pregunta

#### Contexto y delimitación del problema
La Prueba de Acceso a la Educación Superior (PAES) constituye el principal instrumento para la selección y transición hacia la educación universitaria en Chile. Si bien el Ministerio de Educación y el DEMRE publican anualmente informes de carácter descriptivo macro, el acceso a sus microdatos nos abre un espacio idóneo para profundizar de manera particular en asimetrías de rendimiento escolar, geográfico y curricular. 

La idea de este proyecto Capstone es desarrollar un proceso de análisis exploratorio flexible y dinámico. En lugar de fijar una única línea rígida desde el inicio, planteamos que los datos y el propio proceso de análisis bivariado guiarán nuestra profundización académica. Basándonos en exploraciones de la comunidad educativa y en hallazgos empíricos iniciales, identificamos cuatro temáticas con alto potencial de investigación:

1. **Bajo rendimiento en Competencia Matemática 2 (M2):** A nivel país se observa que en la prueba de Matemáticas 2 los resultados son bajos, con un puntaje promedio nacional de 418 puntos.
2. **Asimetría Geográfica Norte-Sur:** Se observa la existencia de una tendencia preliminar en la cual los mejores puntajes de la prueba tienden a concentrarse en la zona sur del país en comparación con las zonas norte y centro.
3. **Inversión de brechas en regiones extremas:** Existen zonas del territorio nacional en las que los establecimientos particulares pagados no necesariamente registran los mejores puntajes de rendimiento (como ocurre en la Región de Arica y Parinacota).
4. **Homogeneidad bivariada de la rama Técnico-Profesional:** A nivel nacional, los colegios de modalidad técnica presentan una distribución de resultados mucho más homogénea entre las diferentes asignaturas evaluadas.

#### Pregunta de investigación y flexibilidad iterativa
En concordancia con el ciclo de vida de un proyecto de ciencia de datos, entendemos que las hipótesis preliminares son dinámicas y están sujetas a modificaciones a medida que limpiamos y exploramos las variables. Por ello, **nuestro equipo profundizará únicamente en una sola de las cuatro opciones** descritas anteriormente. Al ser un proceso iterativo, es posible que la pregunta cambie a futuro o en otros módulos del diplomado. **La definición y acotación definitiva de la pregunta a estudiar se realizará bajo la orientación de la profesora y de los ayudantes.**

A modo de punto de partida exploratorio, se propone la siguiente pregunta de investigación provisional:
* *¿Existen diferencias estadísticamente significativas en los puntajes de la prueba de Competencia Matemática 2 (M2) entre estudiantes provenientes de la rama Científico-Humanista (HC) diurna y la rama Técnico-Profesional (TP) diurna en establecimientos municipales y dependientes de Servicios Locales de Educación Pública (SLEP) para el Proceso de Admisión Regular 2026?*

#### Hipótesis de trabajo
* **Hipótesis Nula ($H_0$):** El puntaje promedio en la prueba M2 de los estudiantes que egresan de establecimientos públicos bajo la modalidad HC diurna es idéntico al puntaje promedio de los egresados de establecimientos de modalidad TP pública diurna para el Proceso de Admisión 2026.
* **Hipótesis Alternativa ($H_1$):** Los puntajes promedio en la prueba M2 para ambas ramas educativas (HC y TP) en el sector público diurno presentan diferencias significativas en el Proceso de Admisión 2026.

---

### 2. Los datos y sus metadatos

De acuerdo con lo definido originalmente en nuestra planificación técnica de entorno:
* **Trabajaremos con los datos encontrados en:** `https://datosabiertos.mineduc.cl/pruebas-de-admision-a-la-educacion-superior/`
* **Inicialmente con los datos del año 2026.**

En esta fase de formulación e inicio del Módulo 1, el análisis se limitará de manera acotada a la **prueba de verano (PAES Regular 2026)**, evaluando las asignaturas obligatorias de **Competencia Lectora** y **Competencia Matemática 1 (M1)**, utilizando **Competencia Matemática 2 (M2)** como elemento clave de contraste. 

No obstante, reconociendo que este proyecto integrador se extenderá progresivamente a lo largo de las distintas asignaturas del diplomado (incluyendo módulos de modelamiento de *Machine Learning* e Inteligencia Artificial), se contempla la posibilidad de **expandir los datos a años de admisiones anteriores (2023, 2024 y 2025)** e incorporar a los modelos analíticos otras evaluaciones electivas como las pruebas de Ciencias y de Historia y Geografía.

#### Estructura y Metadatos de la base de datos (Proceso 2026)
La información se compone originalmente de las siguientes 5 tablas relacionales provistas oficialmente por el MINEDUC y el DEMRE:
1. **PAES-2026-Inscritos-Puntajes:** Contiene los puntajes de las asignaturas rendidas por cada postulante.
2. **PAES 2026 - Socioeconómicos:** Agrupa datos de caracterización del postulante, su grupo familiar y su establecimiento educativo.
3. **PAES-2026-Postulantes:** Registra las postulaciones y preferencias de carreras universitarias seleccionadas.
4. **PAES 2026 - Matrícula:** Almacena la confirmación de la matrícula final del estudiante.
5. **PAES 2026 - Oferta Definitiva Programas:** Detalla los cupos, ponderaciones y exigencias de admisión de las universidades adscritas.

A continuación, definimos el diccionario de las variables prioritarias que guiarán la etapa exploratoria bivariada inicial:

| Origen | Variable | Descripción | Tipo de Variable | Unidad de Medida / Rango |
| :--- | :--- | :--- | :--- | :--- |
| Inscritos-Puntajes | `MRUN` | Identificador único del postulante | Cualitativa Nominal | Llave hash anonimizada |
| Inscritos-Puntajes | `CLEC_PUNTAJE` | Puntaje obtenido en la prueba de Competencia Lectora | Cuantitativa Continua | Rango entre 100 y 1000 puntos |
| Inscritos-Puntajes | `CMAT1_PUNTAJE` | Puntaje obtenido en Competencia Matemática 1 | Cuantitativa Continua | Rango entre 100 y 1000 puntos |
| Inscritos-Puntajes | `CMAT2_PUNTAJE` | Puntaje obtenido en Competencia Matemática 2 | Cuantitativa Continua | Rango entre 100 y 1000 puntos |
| Socioeconómicos | `COD_DEPE` | Dependencia administrativa del colegio de egreso | Cualitativa Nominal | Municipal, SLEP, Subvencionado, Particular |
| Socioeconómicos | `RAMA_EDUC` | Rama educativa del establecimiento | Cualitativa Nominal | HC Diurno, HC Nocturno, Técnico-Profesional |
| Socioeconómicos | `COD_REG_EST` | Región geográfica donde se ubica el establecimiento | Cualitativa Nominal | Categorías discretas del 1 al 16 |

---

### 3. Estrategia de obtención

#### Origen y frecuencia de actualización
Las fuentes de datos serán extraídas directamente en formato tabular delimitado (`.csv` o `.txt`) desde el portal oficial de **Datos Abiertos del MINEDUC**. Al tratarse de un sistema consolidado tras el cierre del proceso de postulación y matrícula universitaria nacional, su frecuencia de actualización es estrictamente **anual** (publicándose por lo general durante el primer trimestre de cada año calendario posterior a la rendición).

#### Restricciones, confidencialidad y volumen
* **Acceso y Licencia:** Las bases son públicas y gratuitas, destinadas expresamente a la investigación y toma de decisiones públicas. No existen restricciones de volumen ni de licenciamiento comercial que obstaculicen su descarga.
* **Resguardo de Confidencialidad:** Con el fin de dar cumplimiento a la Ley 19.628 de protección de datos personales en Chile, el DEMRE anonimiza los registros eliminando la identificación civil de los postulantes (RUT) y reemplazándola por una clave sintética aleatoria denominada `MRUN`.
* **Volumen:** La población anual nacional abarca más de 250.000 estudiantes inscritos. El volumen total es idóneo para su procesamiento local en DataFrames de Pandas.

---

### 4. Viabilidad

La viabilidad técnica e investigativa del proyecto para el equipo está garantizada tanto por los plazos del diplomado como por la consistencia del entorno informático definido:

> *"Considerando el entorno propuesto en clases por la profesora, se ha decidido reutilizar tanto las librerías como la versión de Python copiando directamente el `uv.lock`, `pyproject.toml` y `.python-version` e ir agregando, en medida de lo necesario, nuevas librerías."*

Para garantizar la reproducibilidad científica entre ambos compañeros de equipo, empleamos **Python 3.12** y el gestor de paquetes **uv** localmente. La limpieza y análisis de los datos se realizará mediante las herramientas fundamentales del diplomado:
* **Pandas y NumPy:** Permiten realizar la carga masiva de los tabulares, la correcta indexación de registros por `MRUN`, las operaciones de combinación de tablas (`merge`) y la depuración de valores faltantes (NaN) en tramos de variables sensibles o pruebas no rendidas.
* **Matplotlib y Seaborn:** Apoyarán la construcción de gráficos univariados e histogramas bivariados de densidad, facilitando la visualización espacial de las brechas y la comparación de cajas (*boxplots*) de medias.

Al contar con el histórico completo de los años **2023, 2024 y 2025** ya estructurados en nuestro entorno local de trabajo, el diseño de datos del Módulo 1 sienta una sólida base para las materias siguientes de visualización en Tableau/PowerBI y el modelado predictivo complejo en los cursos posteriores de *Machine Learning* y *Deep Learning*.

---

### 5. Novedad e impacto

#### Novedad metodológica
A diferencia de los anuarios y boletines agregados tradicionales del MINEDUC que reportan estadísticas planas de nivel nacional (como la brecha clásica de colegios públicos frente a particulares pagados), esta propuesta adopta un enfoque dinámico y progresivo. Nuestra novedad radica en estructurar un repositorio multianual flexible capaz de dar respuesta de forma iterativa a dinámicas poco profundizadas, tales como el comportamiento de colegios técnico-profesionales, asimetrías geográficas extremas o focos de rendimiento atípicos bajo variables de control socioeconómico locales.

#### Impacto y uso esperado de los resultados
Dado que el diplomado fomenta la transferencia tecnológica y la toma de decisiones basada en datos, los hallazgos exploratorios univariados y multivariados de este trabajo serán de utilidad directa para:
* **Directivos Escolares y Administradores Locales (SLEP y DAEM):** Quienes contarán con evidencia local detallada para planificar tutorías, focalizar horas de refuerzo académico en el área matemática o justificar la creación de preuniversitarios municipales orientados a nivelar las brechas de la prueba M2.
* **Encargados de Programas de Acceso Inclusivo (PACE):** Para ajustar políticas de ponderación y programas de inserción académica universitaria de manera temprana en los territorios identificados con mayor rezago de acceso.

---

### Declaración de uso de Inteligencia Artificial

Declaramos que para la redacción de este documento preliminar de formulación se utilizó el asistente de IA **Gemini Notebook**. Su aplicación se restringió exclusivamente al apoyo en el refinamiento del estilo de redacción académica y al formateo del diccionario de variables en Markdown, sustentándose íntegramente en las especificaciones dadas en el programa, la rúbrica institucional y el repositorio técnico del equipo.
