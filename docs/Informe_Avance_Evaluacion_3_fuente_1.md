# Quedar fuera teniendo puntaje

### Revisión del proceso de Admisión a la Educación Superior 2026

**Daniela Contreras · Emanuel Nicklichen** — Diplomado en Ciencia de Datos Aplicada, UTFSM · INF074H · Evaluación 3, informe de avance · 25 de septiembre de 2026

**Resumen.** En la Admisión 2026, 27.492 personas no quedaron seleccionadas en ninguna de sus preferencias. Este informe mide cuántas de ellas tenían puntaje suficiente para entrar a algún programa con vacantes no cubiertas, usando el registro administrativo completo del Mineduc y el DEMRE (cinco tablas, 183 variables) y una reconstrucción del puntaje ponderado validada en 99,96% contra 1,33 millones de puntajes oficiales. Entre los 22.218 habilitados sin selección, 15.772 (70,99%) superaban el corte de al menos un programa de su área de interés con cupos libres. La hipótesis de que los egresados de colegios particulares pagados tendrían más alternativas no se sostiene: 55,5% contra 76,0% de los públicos. Una regresión logística muestra que esa brecha se asocia al puntaje y cambia de signo al comparar estudiantes equiparables. El próximo paso es refinar el área de interés e incorporar los requisitos de cada programa al criterio de alcanzabilidad.

## 1. Objetivos, pregunta e hipótesis

El sistema de admisión chileno es centralizado y opera con un algoritmo de aceptación diferida (Gale & Shapley, 1962; Ríos et al., 2021): el postulante ordena hasta veinte preferencias y se le asigna la más alta que su puntaje ponderado alcanza. Quien no alcanza ninguna queda sin asignación, aunque en el mismo proceso existan programas que terminan con vacantes no cubiertas. Este proyecto mide ese desencuentro entre puntaje disponible y cupos disponibles, un fenómeno ya documentado en otros sistemas (Hoxby & Avery, 2013) y en el chileno (Larroucau et al., 2025).

**Objetivo general.** Cuantificar, sobre la población completa de no seleccionados de la Admisión 2026, qué proporción tenía puntaje suficiente para haber sido seleccionada en algún programa con vacantes no cubiertas dentro de su propia área de interés declarada.

**Objetivos específicos de esta etapa.** (1) Reconstruir y validar la fórmula de puntaje ponderado de los 2.150 programas de la oferta, de modo de evaluar a un estudiante en programas a los que no postuló. (2) Contrastar si la proporción con alternativa alcanzable difiere según la dependencia del establecimiento de egreso y según el decil de ingreso del hogar. (3) Ajustar un primer modelo que examine si esas diferencias persisten al comparar estudiantes equiparables en puntaje y nivel socioeconómico.

**Hipótesis puesta a prueba, tal como fue formulada en la Evaluación 1.** *La proporción de estudiantes con una alternativa alcanzable no cubierta es mayor entre quienes egresaron de establecimientos particulares pagados que entre quienes egresaron de establecimientos públicos.*

- **Población:** los 22.218 postulantes habilitados de la Admisión 2026 que no fueron seleccionados en ninguna de sus preferencias (sección 2).
- **Variable respuesta:** `tiene_alternativa`, binaria, definida operacionalmente en la sección 3.
- **Variable explicativa del contraste:** `DEPENDENCIA`, agrupada en Particular Pagado y Público (Corporación Municipal, Municipal y Servicios Locales de Educación Pública), según la definición escrita en la formulación. Los establecimientos particulares subvencionados y de Corporación de Administración Delegada aparecen en los descriptivos pero no forman parte del contraste.

La novedad de este trabajo no es advertir el problema, sino medirlo sobre la población completa y sin encuesta, y cruzarlo con dependencia y nivel socioeconómico (sección 8).

## 2. Los datos

Cinco bases públicas del proceso de Admisión 2026, publicadas por el Ministerio de Educación y el DEMRE, unidas por el identificador `MRUN` y, en la tabla de postulaciones, por la llave `(MRUN, ORDEN_PREF, TIPO_PREF)`. La Tabla 1 resume su tamaño y la Tabla 2 las variables usadas.

**Tabla 1.** Fuentes de datos, tamaño y contenido.

| Tabla | Contenido | Filas | Variables |
|---|---|---:|---:|
| A · Inscritos y puntajes | Puntajes PAES, NEM, ranking, establecimiento de egreso, habilitación | 320.087 | 131 |
| B · Socioeconómicos | Decil de ingreso per cápita del hogar declarado | 320.087 | 10 |
| C · Postulantes | Una fila por preferencia: programa, orden, estado, puntaje ponderado | 1.694.081 | 9 |
| D · Matrícula | Matrícula efectiva de primer año por programa | 146.873 | 10 |
| E · Oferta definitiva | Ponderaciones, vacantes y requisitos de cada programa | 2.150 | 23 |

**Tabla 2.** Variables utilizadas, con su tipo y unidad.

| Variable | Origen | Tipo | Unidad o escala |
|---|---|---|---|
| `CLEC_MAX`, `MATE1_MAX`, `MATE2_MAX`, `HCSOC_MAX`, `CIEN_MAX` | A | Numérica continua | Puntaje PAES, escala 100–1000 |
| `NEM`, `RANKING` | A | Numérica continua | Puntaje, escala 100–1000 |
| `DEPENDENCIA` | A | Categórica nominal, 6 niveles | — |
| `HABILITACION_POST` | A | Categórica, 3 niveles | — |
| `INGRESO_PERCAPITA_GRUPO_FA` | B | Categórica ordinal | Decil de ingreso per cápita, 1–10 (+ código 99) |
| `ORDEN_PREF`, `ESTADO_PREF`, `TIPO_PREF` | C | Discreta / categóricas | Orden 1–20; código de estado; vía de postulación |
| `PTJE_PREF` | C | Numérica continua | Puntaje ponderado, escala 100–1000 |
| Ponderaciones y `VAC_1ER` | E | Numéricas | % que suma 100; vacantes en personas |
| Matrícula de primer año | D | Numérica discreta | Estudiantes |
| *Derivadas:* `tiene_alternativa`, `n_oportunidades`, `ptje_obligatorias` | S6 | Binaria, conteo, continua | — / programas / puntaje 100–1000 |

**Limpieza y decisiones.** Ninguno de los problemas encontrados es un dato ausente declarado: son valores que significan algo distinto de lo que aparentan y que, tomados literalmente, sesgan el resultado (Tabla 3).

**Tabla 3.** Problemas de calidad detectados y decisión adoptada.

| Situación | Magnitud | Decisión |
|---|---|---|
| Cero en los puntajes PAES | — | Es "no rindió", no desempeño: la escala parte en 100. Verificado contra el código de cuadernillo y procesos anteriores |
| `PTJE_PREF` en blanco | 131.859 (7,78%) | Partición exacta con `ESTADO_PREF`: solo los estados 24, 25 y 26 traen puntaje; el resto son anulaciones. `ESTADO_PREF = 26` es "seleccionado en preferencia anterior" y no cuenta como selección |
| Decil igual a 99 | 27,61% | "Prefiere no responder". No es un decil: entra como categoría propia, sin imputar |
| `DEPENDENCIA` en blanco | 3.200 (1,00%) | Sin establecimiento de egreso registrado. Quedan fuera del modelo, no del conteo |
| Duplicados aparentes de postulación | 280.569 | Se postula a la misma carrera por vía regular y por cupo de género: exige `TIPO_PREF` en la llave |
| `PTJE_PREF` sobre 1000 | 25.845 | Bonificación de la vía PACE. Los cortes se calculan solo sobre `TIPO_PREF = REGULAR` |
| Ponderación Historia / Ciencias | 1.411 programas | No se suman: es un cupo único que se llena con el mejor de los dos puntajes. Corregido, quedan 10 programas que no suman 100% |

**El embudo y la población de análisis.** De 320.087 inscritos, 189.399 postularon, 161.907 quedaron seleccionados y 27.492 no quedaron en ninguna preferencia. De estos últimos, 5.274 **no estaban habilitados**: la habilitación exige un promedio de Competencia Lectora y Matemática 1 de al menos 458 puntos, o pertenecer al 10% superior de notas del establecimiento (`HABILITACION_POST = 2`). Su postulación se anula al cierre del proceso, de modo que la pregunta —si les alcanzaba en otra parte— no tiene sentido para ellos. La población de análisis son los **22.218 habilitados sin selección**, cifra que coincide con los 21.973 de la minuta oficial del Mineduc dentro de un 1,1%.

## 3. Definición operacional y reconstrucción del puntaje ponderado

**Un estudiante tiene una alternativa alcanzable** si existe al menos un programa que (i) ofrece una carrera con el mismo nombre que alguna de las que él mismo postuló —su *área de interés declarada*—, (ii) terminó con matrícula de primer año inferior a sus vacantes informadas, y (iii) cuyo **corte** su puntaje ponderado habría superado. El corte se define como el mínimo puntaje ponderado entre los seleccionados por vía regular del programa (`ESTADO_PREF = 24`), la misma convención usada en la literatura sobre este sistema (Larroucau et al., 2025).

Evaluar a un estudiante en un programa al que no postuló obliga a reconstruir la ponderación de ese programa, porque cada carrera asigna pesos propios a NEM, ranking, lectura, matemática y el electivo. La reconstrucción incorpora dos reglas no evidentes: Historia y Ciencias compiten por un mismo cupo ponderado, que se llena con el mejor de los dos puntajes; y a los extranjeros sin NEM chileno el DEMRE redistribuye el peso de NEM y ranking entre las demás pruebas, en lugar de asignarles cero.

**Validación.** La fórmula reconstruida se aplicó a las postulaciones reales y se comparó con el puntaje oficial que trae la base C. Sobre **1.328.381 preferencias regulares con puntaje informado, el 99,96% coincide con error menor a 0,5 puntos**. De las 472 discrepancias, 407 (86,2%) son carreras artísticas que exigen una prueba especial no registrada en la base de oferta; las 65 restantes se concentran en **solo 6 estudiantes** repartidos en 61 programas, lo que descarta un defecto de la fórmula. La formulación inicial reportaba 1.325.219 preferencias; la cifra vigente es la recalculada sobre la llave única, y la diferencia (0,24%) no altera el porcentaje de coincidencia. Esta validación es el activo metodológico del proyecto: sin ella, todo lo que sigue sería una estimación sin respaldo.

## 4. Resultado central

**Tabla 4.** Situación de los 27.492 no seleccionados respecto de la existencia de una alternativa alcanzable.

| Situación | Estudiantes | % del total | % de habilitados |
|---|---:|---:|---:|
| Con alternativa alcanzable | 15.772 | 57,37% | **70,99%** |
| Con candidatos, sin alcanzar ninguno | 4.028 | 14,65% | 18,13% |
| Sin ningún candidato evaluable | 2.418 | 8,80% | 10,88% |
| No habilitado para postular | 5.274 | 19,18% | — |
| **Total** | **27.492** | **100%** | (22.218) |

Siete de cada diez de quienes quedaron fuera estando habilitados superaban el corte de al menos un programa de su propia área de interés que no llenó sus cupos (Tabla 4). La formulación de la Evaluación 1 anticipaba "8 de cada 10" sobre una revisión preliminar; la diferencia se explica íntegramente por haber separado a los no habilitados.

El dato que cambia la lectura está dentro del grupo que sí tenía candidatos: quienes **no** alcanzaron ninguno promedian **741,2 puntos** de mejor ponderado, contra **634,8** de quienes sí lo lograron. Más de cien puntos por encima, y con peor resultado. El puntaje alto no protege, porque quien lo tiene apunta a carreras selectivas, y son esas las que llenan todos sus cupos. Lo que determina el resultado no es el nivel de puntaje en abstracto, sino la selectividad de las carreras a las que ese puntaje apunta. Esto motiva los dos hallazgos que siguen.

## 5. Hallazgo 1 — La relación entre dependencia y alternativa invierte la hipótesis

![**Figura 1.** Proporción de estudiantes habilitados con alternativa alcanzable, según grupo de dependencia del establecimiento de egreso. Eje horizontal: grupo de dependencia. Eje vertical: porcentaje de estudiantes con alternativa (%). Las líneas verticales son intervalos de confianza del 95% (método de Wilson).](figuras/s7_02_hipotesis_pp_publico.png){width=60%}

La Figura 1 relaciona la dependencia del establecimiento con la existencia de una alternativa alcanzable. Entre los 2.492 egresados de establecimientos particulares pagados, el **55,50%** tenía alternativa (IC 95%: [53,54 – 57,44]); entre los 6.580 de establecimientos públicos, el **75,99%** (IC 95%: [74,94 – 77,00]). La diferencia es de **−20,49 puntos porcentuales**, con una razón de odds de 0,394 (prueba z de dos proporciones: z = −19,08; p < 0,001), y los intervalos de confianza no se traslapan. **La hipótesis, tal como fue formulada, no se sostiene: el resultado corre en la dirección exactamente contraria.**

El contraste se hizo con los grupos definidos en la formulación: redefinir "público" tras conocer el resultado habría sido acomodar la prueba a la respuesta. Como la población es el registro completo y no una muestra, el intervalo de confianza no describe error de muestreo, sino cuán estable sería el resultado si el proceso que lo generó se repitiera bajo las mismas condiciones; se reporta porque permite juzgar si una diferencia es distinguible del ruido de ese proceso.

## 6. Hallazgo 2 — El nivel socioeconómico muestra el mismo patrón, y el mecanismo

![**Figura 2.** Proporción de estudiantes habilitados con alternativa alcanzable, según decil de ingreso per cápita del hogar. Eje horizontal: decil (1 = menor ingreso, 10 = mayor). Eje vertical: porcentaje con alternativa (%); no parte en cero. Barras verticales: intervalo de confianza del 95%.](figuras/s7_03_decil.png){width=64%}

La Figura 2 relaciona el decil de ingreso con la misma variable respuesta. Entre los deciles 1 y 8 la proporción oscila entre **68,3% y 74,4%** sin una dirección que la ordene por ingreso; existen diferencias reales entre algunos deciles —el 5 queda por debajo del 1 y del 8, con intervalos que no se tocan— pero no constituyen una tendencia. La caída nítida está en el extremo alto: **65,3% en el decil 9 y 59,0% en el decil 10**, este último por debajo de todos los demás. Los 6.606 estudiantes que no informan ingreso se reportan aparte (70,1%) y no se imputan.

Que dependencia y decil apunten al mismo lugar no es una confirmación independiente: son poblaciones que se solapan fuertemente, y la coincidencia es la primera señal de que lo que las une es una tercera variable.

**El mecanismo, medido.** Si la explicación fuera que los estudiantes de particulares pagados persiguen carreras con menos sustitutos, debería verse en cuántos programas candidatos tiene cada uno. Se modeló ese conteo con una **regresión de Poisson con errores estándar robustos** (HC1), necesarios porque está fuertemente sobredisperso: media 15,1 y varianza 368,1 programas.

**Tabla 5.** Modelo de Poisson sobre el número de programas candidatos. Razón de tasas de incidencia (IRR), Particular Pagado respecto de Municipal.

| Especificación | IRR | IC 95% |
|---|---|---|
| Sin ajustar | 0,559 | [0,523 – 0,598] |
| Ajustado por puntaje y amplitud de la lista | **1,074** | [1,009 – 1,144] |

Sin ajustar, un estudiante de particular pagado evalúa **10,8 programas candidatos contra 19,3** de uno municipal: cerca de la mitad. A igual puntaje y amplitud de lista, evalúa un 7% más (Tabla 5). **Salvedad:** el número de carreras distintas y el de oportunidades están ligados por construcción, porque los programas candidatos se buscan entre las carreras que el propio estudiante nombró; controlar por esa variable no aísla un confusor, sino que acota la comparación a estudiantes de igual puntaje e igual amplitud de lista.

## 7. Primer modelo — A igual puntaje, la brecha cambia de signo

Se ajustó una **regresión logística** (`statsmodels`, errores estándar robustos HC1) sobre los 21.981 estudiantes habilitados con dependencia registrada. **Explica** si el estudiante tiene o no una alternativa alcanzable (variable binaria) **a partir de** la dependencia del establecimiento, el decil de ingreso (con "no responde" como categoría propia) y el promedio de Competencia Lectora y Matemática 1. Se usa ese promedio y no el puntaje ponderado porque el ponderado depende de qué programas eligió el estudiante, que es parte de lo que el modelo busca explicar; el promedio de las obligatorias es el criterio que el propio DEMRE usa para habilitar y no depende de ninguna elección. Se eligió una logística y no una lineal porque la respuesta es binaria y una lineal podría predecir probabilidades fuera del rango [0, 1].

**Tabla 6.** Regresión logística. Razón de odds (OR) de Particular Pagado respecto de Municipal.

| Especificación | OR | IC 95% | p |
|---|---|---|---|
| Modelo 1 · sin ajustar (pseudo R² = 0,014) | 0,370 | [0,330 – 0,414] | < 0,001 |
| Modelo 2 · ajustado por decil y puntaje (pseudo R² = 0,151) | **1,336** | **[1,163 – 1,536]** | < 0,001 |

![**Figura 3.** Probabilidad de tener alternativa alcanzable, por dependencia del establecimiento. Eje horizontal: dependencia. Eje vertical: probabilidad (%). En azul, la proporción observada; en rojo, la probabilidad estimada por el Modelo 2 fijando a todos los estudiantes en el decil 3 y en el puntaje mediano (582 puntos).](figuras/s7_04_observado_ajustado.png){width=66%}

**El efecto no desaparece al ajustar: se invierte** (Tabla 6). El intervalo del Modelo 2 queda completo por sobre 1. Traducido a probabilidades, Particular Pagado pasa de la proporción observada más baja (55,5%) a la probabilidad ajustada **más alta (83,3%)**, frente a 78,9% de Municipal (Figura 3).

Lo que ocupa el lugar de la dependencia es el puntaje: **OR = 0,991 por punto**, es decir, diez puntos adicionales reducen las odds cerca de un 9%. Una vez incluido el puntaje, ningún decil se distingue del decil 1, y el pseudo R² sube de 0,014 a 0,151: casi todo el poder explicativo lo aporta el puntaje. La magnitud detrás de ese coeficiente es grande: los egresados de particular pagado promedian **732 puntos** en las pruebas obligatorias, contra **576** de los municipales. En el modelo ajustado subsisten además dos diferencias menores: SLEP (OR 0,820; p = 0,006) y Corporación de Administración Delegada (OR 0,799; p = 0,043) quedan por debajo de Municipal.

**Qué permite y qué no permite concluir.** El puntaje no es un factor ajeno que hubo que descontar: es el canal por el que opera la diferencia, un mediador y no un confusor. La dependencia se asocia con el resultado por dos caminos de signo opuesto —más puntaje, carreras más selectivas, menos alternativas; y, a igual puntaje, más alternativas— y en la comparación directa domina el primero. Dos lecturas tentadoras son incorrectas: que el colegio pagado "perjudica" (esa desventaja es enteramente el puntaje) y que la hipótesis quedó confirmada al ajustar (la hipótesis hablaba de proporciones observadas, que van al revés; la dirección que aparece al ajustar es otra afirmación, que no fue la planteada). El modelo es **descriptivo**: cuantifica asociaciones sobre la población completa, no efectos causales, y no fue evaluado fuera de muestra. Qué es esa ventaja residual a igual puntaje, el modelo no lo identifica; es consistente con que los estudiantes de colegios privados estimen mejor sus probabilidades de admisión (Larroucau et al., 2025), pero también podría responder a movilidad geográfica o a cómo se arma la lista.

**Robustez.** El resultado usa la comparación estricta `puntaje > corte`. Recalculado con `puntaje ≥ corte`, la diferencia entre los 22.218 habilitados es de **cero casos**. Y los dos modelos no se contradicen: la dependencia no muestra desventaja propia sobre *alcanzar* una alternativa a igual puntaje y decil —muestra ventaja—, y muestra un efecto pequeño y residual sobre *cuántas* alternativas existen.

## 8. Qué falta, qué no tiene sentido todavía y próximos pasos

**Un resultado que no tenía sentido, y el error propio que lo explicaba.** En la primera versión del análisis los 5.274 no habilitados se contaban como población válida, y el 86% de ellos figuraba erróneamente "con alternativa" pese a que su postulación se anula. Se detectó al contrastar el conteo propio con la minuta oficial del Mineduc, que reportaba unos 22 mil y no 27 mil. Corregirlo no cambió la narrativa: la hizo más nítida, porque esos falsos positivos se concentraban en los grupos públicos de puntaje bajo y ocultaban la ventaja condicional de particular pagado, que antes aparecía como ausencia de efecto (OR ajustado 1,033) y ahora como inversión (1,336). También se corrigió el denominador del promedio de puntaje del grupo sin alternativa, mal referido en la Evaluación 2.

**Limitaciones específicas de este análisis.**

- **Vacante no cubierta no es lo mismo que cupo disponible durante la selección.** El cálculo usa vacantes informadas menos matriculados, y la matrícula ocurre *después* de la asignación: un programa puede terminar con matrícula baja porque los seleccionados no usaron su cupo. Es la limitación más seria y afecta directamente al 70,99%, que debe leerse como una **cota superior de la oportunidad**, no como un conteo de errores de postulación.
- **El área de interés se define por igualdad exacta del nombre de carrera.** Carreras equivalentes con nombres distintos no se reconocen como sustitutas (subestima las alternativas) y la misma carrera en otra región se trata como sustituto perfecto (las sobreestima). Los dos sesgos existen y no se cancelan de forma conocida.
- **No se verificaron todos los requisitos de los programas**, solo el puntaje contra el corte: un programa que exige una prueba que el estudiante no rindió puede estar contado como alcanzable.
- **No se observan preferencias** —el proyecto no sabe si el estudiante *quería* la alternativa disponible, que pudo descartar por costo o distancia—, **la dimensión geográfica no se evaluó**, y el modelo describe asociaciones, no causalidad.

**Próximos pasos.** Refinar el área de interés mediante agrupación semántica de nombres de carrera; incorporar los requisitos específicos de cada programa al criterio de alcanzabilidad; introducir la dimensión geográfica; y caracterizar a los 5.274 que postularon sin estar habilitados, que constituyen un error procedimental de naturaleza distinta y no analizado aquí. La pregunta del proyecto se desplaza: ya no es quién tiene menos alternativas, sino por qué el sistema deja cupos sin llenar justo donde nadie los busca, y quién no alcanza a verlos.

**Relación con la literatura.** Larroucau et al. (2025) miden el error de postulación con encuestas y obtienen una **cota inferior**: cuántos omitieron un programa que ellos mismos declaran preferir. Este proyecto no observa preferencias, pero sí a toda la población, y obtiene una **cota superior**: cuánto espacio había. La cantidad real de decisiones evitables está entre ambas, y coincide con lo que Hoxby y Avery (2013) documentan para Estados Unidos: la brecha no está en el puntaje, sino en qué opciones se consideran.

**Reproducibilidad.** El código no se incluye en este informe: el análisis está en ocho notebooks ejecutados de principio a fin, que se adjuntan a la entrega (S0 a S5 de la Evaluación 2; S6, que construye la base única de 27.492 filas, y S7, que contrasta y modela sobre ella sin recalcularla). Las figuras 1 a 3 se generan al ejecutar S7.

**Declaración de uso de Inteligencia Artificial.** Se utilizó Claude (Anthropic) para la redacción y organización de este informe, para la revisión crítica de los notebooks S6 y S7 —incluida la detección del error de población al contrastar con la minuta oficial— y para explicar conceptos estadísticos. La pregunta de investigación, la hipótesis, la decisión de no redefinirla tras conocer los resultados y las decisiones metodológicas son del equipo. Todas las cifras provienen de los notebooks ejecutados y son reproducibles.

## Referencias

Gale, D., & Shapley, L. S. (1962). College admissions and the stability of marriage. *The American Mathematical Monthly, 69*(1), 9–15.

Hoxby, C. M., & Avery, C. (2013). The missing "one-offs": The hidden supply of high-achieving, low-income students. *Brookings Papers on Economic Activity, 44*(1), 1–65.

Larroucau, T., Ríos, I. A., Fabre, A., & Neilson, C. (2025). *College application mistakes and the design of information policies at scale* (NBER Working Paper No. 34164). National Bureau of Economic Research.

Ríos, I., Larroucau, T., Parra, G., & Cominetti, R. (2021). Improving the Chilean college admissions system. *Operations Research, 69*(4), 1186–1205.
