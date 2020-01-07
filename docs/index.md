---
author: 'Juan Cruz Rodriguez'
date: 'Septiembre 2019'
institution: 'UNIVERSIDAD NACIONAL DE CÓRDOBA'
division: 'Facultad de Matemática, Astronomía, Física y Computación'
advisor: 'Elmer Andrés Fernández'
department: 'Ciencias de la Computación'
degree: 'Doctor en Ciencias de la Computación'
title: 'Análisis e integración de información de datos biológicos mediante análisis funcional'
knit: "bookdown::render_book"
site: bookdown::bookdown_site
output: 
  thesisdown::thesis_pdf: default
  # thesisdown::thesis_gitbook: default
  # thesisdown::thesis_word: default
  # thesisdown::thesis_epub: default
abstract: |
  \par El análisis funcional refiere a un conjunto de técnicas que tienen como fin detectar aquellas funciones o procesos que se encuentran desregulados en un experimento biológico. Con el continuo avance en las tecnologías de obtención de expresión de muestras biológicas, la cantidad de bases de datos de libre disponibilidad aumenta constantemente. Las técnicas de análisis funcional se basan en el estudio de un único experimento, en la era del _Big Data_ resulta natural notar la necesidad de explotar esta gran cantidad de bases de datos para su integración, y así, generar nuevas fuentes de información.
  
  \par Esta tesis propone, como objetivo principal, brindar una metodología que permita integrar grandes cantidades de bases de datos de expresión biológica. Integrando información de diversas poblaciones, fenotipos, enfermedades, entre otros, se podrá detectar patrones que caractericen cada grupo. Como primer instancia de tesis, se realizó una comparación exhaustiva de diversas alternativas para llevar a cabo el análisis funcional. Con tantas alternativas existentes, que siguen diversos supuestos e ideas, esta evaluación nos llevó a la creación del pipeline de _Análisis Funcional Integrador_: `IFA` (del inglés _Integrative Functional Analysis_). El `IFA` realiza su análisis tomando alternativas que otorgaron los mejores resultados desde un punto de vista biológico y estadístico.
  
  \par Para cumplir con el objetivo principal de esta tesis, presentamos la herramienta `MIGSA` (del inglés _Massive and Integrative Gene Set Analysis_). Gracias a esta herramienta, es ahora posible llevar a cabo un análisis funcional masivo e integrador de grandes cantidades de bases de datos biológicas que provienen tanto de distintas poblaciones como de distintas fuentes biológicas (genes, proteínas, etc.). Además, `MIGSA` proveé diversas herramientas que permiten explorar y visualizar fácilmente los resultados, y de esta manera, validar y generar nuevas hipótesis de estudio. La utilidad de nuestra herramienta fue comprobada ya que permitió, para sub-grupos de cáncer de mama -con pronósticos bien distintivos-, detectar genes y procesos biológicos que los caracterizan. `MIGSA` representa una herramienta que permite detectar efectivamente aspectos biológicos que podrían ser blancos de drogas, y así contrarrestar la condición bajo estudio.
  
  **Clasificación (ACM CCS 2012):**
  
    * _Applied computing ~> Life and medical sciences ~> Computational biology_
    * _Applied computing ~> Life and medical sciences ~> Bioinformatics_
  
  **Palabras claves:** _Minería de datos - Ciencia de datos - Integración de información - Bioinformática_
  
acknowledgements: |
  \par Llega el tan ansiado día de defender mi tesis doctoral, luego de más de 5 años de investigación sin duda hay miles de agradecimientos que mencionar. Antes que nada, me siento bendecido y afortunado de haber nacido en la República Argentina. Le agradezco a mi país no solo por haberme brindado educación gratuita de excelencia, si no que además la beca con la cual pude preocuparme exclusivamente en mi investigación durante estos años de estudios doctorales.
  
  \par En segundo lugar, creo que el mayor agradecimiento se lo debo al Dr. Elmer A. Fernández, mi director, quien me inició, guió y formó en el universo de la investigación científica. Con todas las concordancias y discusiones que haya pasado con él, todo aprendizaje resulta positivo para la vida. Asimismo agradezco a mis compañeros de trabajo de la UCC, sus comentarios y sugerencias fueron un pilar fundamental para la calidad de este trabajo de tesis -sin contar el gran aporte de amistad y cerveceadas-.
  
  \par Obviamente, este trabajo no hubiera sido posible sin momentos de esparcimiento de quien escribe. Aquí es donde debo principalmente agradecer a toda mi familia: má, pá, sin saber mucho de ciencia o investigación, siempre me apoyaron en lo que me hiciera feliz. A mi abuelito, que cada vez que me preguntaba sobre la facu, me daba ánimos a seguir más y más. Por otra parte mis grupos de amigos, los del barrio, los de la facu, los de la cancha, grupos bien distintos, pero que siempre brindaron toneladas de cariño ~~y asados y escabio~~. Y si a esparcimiento me refiero, no puedo no incluir al Instituto Atlético Central Córdoba, fuente de gran parte de mis alegrías.
  
  \par Finalmente, imposible cerrar esta sección sin agradecer al Whisky, no, no soy alcohólico, me refiero a mi amigo de cuatro patas. El ser más cariñoso que conozco, el que mayor parte de tiempo estuvo presente durante mi doctorado -ahora mismo está acá mirándome con cara de ir a pasear-. Esas pausas de 15 minutos que me exigía, fueron fundamentales para despejar el cerebro y remontar con mejores ideas.
  
  \par Gracias, gracias a todos, les dedico este trabajo, y esta parte de mi vida.
dedication: |
  _Va Castro, va Castro, va Castro, sacó el centro pasado, Riggio, ..._
  
  _Aquella felicidad vuelve a atravesar mi cuerpo al defender esta tesis._
# preface: |
#   This is an example of a thesis setup to use the reed thesis document class
#   (for LaTeX) and the R bookdown package, in general.
bibliography: bib/thesis.bib
# Download your specific bibliography database file and refer to it in the line above.
csl: csl/apa.csl
lot: false
lof: false
---


