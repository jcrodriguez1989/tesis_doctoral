# Análisis Funcional Integrador {#cap:ifa}

## Motivación

\par La complejidad y heterogeneidad de ciertas enfermedades, como el cancer, demuestran que el análisis de los genes Diferencialmente Expresados (DE) no resulta suficiente para descrifrar el fenómeno biológico subyacente [@reis2011gene]. Por el contrario, resulta solo el punto de partida de un proceso de exploración en el que se buscan patrones utilizando diversas fuentes de información [@goeman2007analyzing], un proceso conocido como Análisis Funcional (AF), el cuál fue descripto en el Capítulo [1](#cap:af). Existen principalmente dos enfoques para llevar a cabo el AF: el Análisis de Sobre-Representación (ASR), y la Puntuación Funcional de Clase (PFC) [@manoli2006group; @pavlidis2004using]. Una de las principales críticas al ASR es que requiere de una lista de genes candidatos definida por el usuario, generalmente estableciendo un umbral de corte de los genes DE [@goeman2007analyzing; @manoli2006group; @pavlidis2004using; @tian2005discovering; @khatri2012ten]. Es por ello que los métodos de PFC emergen como una alternativa que supera esa limitación utilizando no solo todos los genes presentes en el experimento, sino que también sus niveles de expresión, donde los genes son ponderados de acuerdo a alguna métrica relacionada con el fenotipo analizado [@subramanian2005gene].

\par Se han propuesto varios algoritmos tanto de ASR como de PFC [@khatri2012ten], cada uno con supuestos y parámetros de entrada diferentes, los cuales pueden conducir a resultados muy diferentes. Adicionalmente, algunas ontologías, como Gene Ontology [@ashburner2000gene], organizan sus conjuntos de genes en alguna estructura particular que permite considerar estrategias de penalización adicionales para el AF. Por lo tanto, la selección del algoritmo apropiado y sus parámetros no resulta una decisión trivial para el investigador, y es una problemática que no ha sido abordada analíticamente. Por otra parte, no está claro qué se obtiene con cada método desde un punto de vista de recuperación de la información, si los resultados son independientes del método y sus parámetros o si los métodos son complementarios o igualmente útiles.

\par Manoli et al. [@manoli2006group] realizó una comparación de ambos enfoques: ASR y PFC. Sin embargo, dicha comparación fue teniendo en cuenta solo 20 conjuntos de genes enriquecidos de los 227 que evaluó, y solo tres bases de datos con 160 sujetos en total. Por otra parte, Pavlidis et al. [@pavlidis2004using] también comparó ambos enfoques, en ese trabajo se utilizaron 41 muestras apareadas de cerebro, y consideró sólo 10 de los conjuntos de genes de los 965 que analizó. Sin embargo, tanto Manoli como Pavlidis obtuvieron resultados inesperados de PFC, ya que evaluaron sólo un algoritmo y con una única parametrización posible, la cual no es comúnmente recomendada por la literatura para el algoritmo utilizado. Es por ello que, para lograr diseñar un mejor enfoque del AF, resulta fundamental un análisis exhaustivo que tenga en cuenta una variedad de algoritmos, de parametrizaciones, una gran cantidad de conjuntos de genes, así como más bases de datos y sujetos.

## Validación de resultados

\par Uno de los principales inconvenientes en la comparación de métodos de AF es la falta de _gold standards_ o de un conjunto de datos de referencia. Como lo establece Khatri y colaboradores [@khatri2012ten]; para esta situación, el uso de conjuntos de datos biológicos reales es preferible a datos simulados, ya que estos últimos carecen de factores biológicos importantes [@khatri2012ten]. Para superar este problema, en el presente trabajo proponemos el uso de varios experimentos para evaluar los mismos (y muy contrastantes) fenotipos de cáncer, asumiendo que deberían exhibir perfiles funcionales similares a través de los experimentos. Nuestra hipótesis es que en un meta-análisis que contrasta dos fenotipos con diferencias conocidas, los patrones de enriquecimiento funcional deben encontrarse en consenso compartidos entre todos los conjuntos de datos, independientemente del método utilizado. Por otra parte, las diferencias entre cohortes podrían considerarse como particularidades biológicas que pueden explorarse más a fondo. Por ejemplo, aunque se encontró poca o ninguna superposición de genes entre varias firmas moleculares en diferentes cohortes de pacientes con el mismo fenotipo [@ein2006thousands], se han reportado funcionalidades comunes en términos de funciones biológicas [@reis2011gene]. Por lo tanto, los resultados del AF deberían ser similares, mostrando un alto consenso entre los conjuntos de datos a pesar de los genes expresados diferencialmente en cada caso.

Adicionalmente, para determinar si los resultados de enriquecimiento están realmente relacionados con la condición bajo estudio, se requerirá una validación manual de la literatura, lo cual suele ser una tarea tediosa para el investigador, por lo que a modo de validación automática, resulta de extrema utilidad la Base de Datos de Toxicogenómica Comparada (BDTC). Utilizando la BDTC [@davis2014comparative], es posible consultar para un término biológico dado, si el mismo está relacionado o no con una determinada enfermedad. La BDTC se utilizó para, programáticamente, consultar y verificar si los términos enriquecidos por cada método están relacionados o no con la enfermedad bajo estudio: "cáncer de mama".

## Datos de entrada

### Matrices de expresión

Para el presente trabajo, se analizaron seis bases de datos de cáncer de mama, seleccionadas debido a su fácil acceso ya que se encuentran presentes en el repositorio R **Bioconductor**. Estas bases de datos fueron medidas mediante la tecnología de microarreglos de ADN y son: Mainz, Nki, Transbig, Unt, Upp y Vdx; ver la Tabla \@ref(tab:databases). Para cada una de las bases de datos, se obtuvo el subtipo intrínseco de cáncer de mama de cada sujeto mediante el método PAM50 [@parker2009supervised] utilizando la librería de R `genefu` [@gendoo2015genefu] y siendo procesados como sugiere Sørlie et al. [@sorlie2010importance]. Se mantuvieron únicamente aquellos sujetos clasificados como tipo Basal o Luminal A, obteniendo un total de 741 sujetos, ya que es sabido que ambos subtipos son muy contrastantes. Al comparar sobrevida de sujetos de ambos subtipos, Luminal A suele presentar un mejor pronóstico que Basal [@parker2009supervised; @dai2015breast; @ein2005outcome]. Por ende, se espera identificar muchos genes que se comporten contrariamente, es decir, desregulados entre ambas condiciones, y que impacten en varios términos (enriquecidos) que se detecten en común a través de las bases de datos. Para cada una de las seis matrices de expresión, se mantuvieron solo aquellos genes que contaban con identificador de genes _Entrez id_ válido, y valores de expresión detectados en al menos el 50% de las muestras de cada subtipo.

### Conjuntos de genes

Como Conjuntos de Genes (CG) se utilizaron las tres categorías de GO. Para descargar los CG se utilizó la versión _v3.0.0_ del paquete R `org.Hs.eg.db` [@carlson2013org], obteniendo un total de 19.693 CG. 

## Algoritmos y parámetros comparados {#sec:algosComparados}

Para determinar si un CG se encontraba enriquecido por un método dado, se utilizaron los criterios por defecto o recomendados de cada método.

### Análisis de Sobre-Representación

Dentro de los algoritmos de ASR, una de las herramientas más utilizadas es la plataforma web DAVID [@huang2008systematic] -del inglés _Database for Annotation, Visualization and Integrated Discovery_-. La lista de genes DE de cada base de datos fue enviada a la plataforma Web DAVID (en adelante llamado **WD**) a través del paquete R `RDAVIDWebService` [@fresno2013rdavidwebservice]. Uno de los principales inconvenientes de la plataforma DAVID es que no permite evaluar CG provistos por el usuario, y al momento de realizar este análisis, la versión estable (_v6.7_) contaba con una base de conocimiento actualizada por última vez en el año 2010. Para superar este inconveniente, desarrollamos **RD**, una versión en lenguaje R del algoritmo de DAVID, que nos permite analizar cualquier CG de interés. Además, a diferencia de DAVID y RDAVIDWebService, RD no requiere de comunicación a través de internet. El tercer algoritmo de ASR evaluado es **GOstats** [@falcon2006using], que fue diseñado específicamente para realizar análisis de enriquecimiento de GO. GOstats penaliza el enriquecimiento de los CG (penalización _elim_) teniendo en cuenta la estructura de grafos de GO. Finalmente, también se evaluó el método **dEnricher** [@fang2014thednet]. Este último método posee una representación interna de GO, y ofrece otro método de penalización adicional al provisto por GOstats (penalización _lea_). Además dEnricher permite seleccionar el test estadístico a utilizar: _Hipergeométrico_, _Binomial_ ó _Fisher_.

\par Como se mencionó en la Sección \@ref(sec:af), los métodos de ASR requieren, además de los CG y la lista de genes candidatos, la lista de genes de referencia (BR; del inglés _Background Reference_). Utilizar diferentes listas de referencia pueden dar lugar a resultados diferentes del método de ASR [@rivals2006enrichment; @fresno2012multi]. Por ello, para cada opción de ASR propuesta, a excepción de dEnricher que no lo permite, se evaluaron listas de referencia propuestas por Fresno et al. [@fresno2012multi]. Las listas de referencia evaluadas fueron: el genoma completo (BRI) y aquellos genes detectados en el experimento (BRIII).

\par Para cada base de datos, como lista de genes candidatos, se utilizaron los genes DE del experimento. Para obtener los genes DE de cada experimento se utilizó la función `treat` [@mccarthy2009testing] del paquete R `limma` [@berkeley2004linear], esta función tiene en cuenta aquellos genes que presentan un valor absoluto de Fold-Change mayor a un valor de corte llamado _treatLfc_. Los genes DE fueron aquellos que obtuvieron un p-valor ajustado por FDR $\leq$ 0,01. Para obtener resultados comparables entre los experimentos, para cada base de datos, se seleccionó un valor _treatLfc_ que obtuviera una cantidad de genes DE de alrededor el 5% de la longitud de su BRIII.

\par En resumen, los métodos y parámetros analizados de ASR, junto con el identificador a utilizar (en negrita), se listan a continuación:

\begin{small}
\[
\begin{matrix}
DAVID\left\{\begin{matrix}
	   BRI\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{WD BRI}
	\\ BRIII\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{WD BRIII}
\end{matrix}\right.
\end{matrix}
\]
\[
\begin{matrix}
RD\;\;\;\;\;\;\; \left\{\begin{matrix}
	   BRI\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{RD BRI}
	\\ BRIII\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{RD BRIII}
\end{matrix}\right.
\end{matrix}
\]
\[
\begin{matrix}
GOstats\left\{\begin{matrix}
	   BRI\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{GOstats BRI}
	\\ BRIII\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{GOstats BRIII}
\end{matrix}\right.
\end{matrix}
\]
\[
\begin{matrix}
dEnricher \left\{\begin{matrix}
	   \begin{matrix}Hipergeo-\\ métrico\end{matrix} \left\{\begin{matrix}
		   sin\;penalizar\;\;\;\;\;\;\;\;\;\textbf{dE\_H\_none}
		\\ lea\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_H\_lea}
		\\ elim\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_H\_elim}
	\end{matrix}\right.
	\\ Binomial\;\;\;\; \left\{\begin{matrix}
		   sin\;penalizar\;\;\;\;\;\;\;\;\;\;\textbf{dE\_b\_none}
		\\ lea\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_b\_lea}
		\\ elim\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_b\_elim}
	\end{matrix}\right.
	\\ Fisher\;\;\;\;\;\;\;\;\left\{\begin{matrix}
		   sin\;penalizar\;\;\;\;\;\;\;\;\;\textbf{dE\_F\_none}
		\\ lea\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_F\_lea}
		\\ elim\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{dE\_F\_elim}
	\end{matrix}\right.
\end{matrix}\right.
\end{matrix}
\]
\end{small}

### Puntuación Funcional de Clase

El método _Gene Set Enrichment Analysis_ propuesto por Subramanian et al. [@subramanian2005gene] fue de los primeros algoritmos de PFC. El método Subramanian (SM; del inglés de _Subramanian Method_) se incluyó entre los analizados, se utilizó su implementación en lenguaje Java (_v2-2.2.1.1_), ya que la implementación R se encuentra deprecada. El SM puede ser alimentado con un vector de genes pre-rankeados (**SMpr**) obtenidos a través de alguna métrica adecuada, ó con la matriz de expresión de genes. Para esta última alternativa, la significancia del estadístico _Enrichment Score_ (ES) se estima a través de una estrategia de permutaciones sobre las etiquetas de los genes (**SMgp**) o de las condiciones a las que pertenece cada muestra (**SMpp**), generando rankeos permutados de los genes. Para determinar a cada conjunto de genes un p-valor asociado, el SM calcula los ESs aplicando un estadístico de _Kolmogorov-Smirnov_ ponderado [@subramanian2005gene]. Esta ponderación está determinada por un peso $w$ que, por defecto, se establece en 1, pero que también puede sustituirse por 0 ó 2. Sin embargo, el efecto del peso $w$ no queda claro ya que los autores sugieren diferentes parametrizaciones de acuerdo con el tipo de dato de entrada (matriz de expresión ó genes pre-rankeados). En el presente trabajo se evaluaron las combinaciones de las diferentes estrategias de permutación con las diferentes alternativas de pesos _w_. Para el caso de SMpr, ya que debe contarse con un vector de rankeo de los genes, se aplicó la función `eBayes` de la librería R `limma`, obteniendo así para cada gen, el estadístico _t_ y p-valor asociados. Se analizaron tres métodos de rankeo: $t$, $1-p$-$valor$, y $-log(p$-$valor)$. Los métodos de PFC suelen filtrar conjuntos de genes previo a realizar el análisis. Este filtrado suele darse dependiendo de la cantidad de genes que componen cada CG. Al evaluar el SM se utilizaron los parámetros de filtrado de CG por defecto, que analiza CG que incluyen entre 15 y 500 genes.

\par Otra alternativa de PFC evaluada fue la librería R `mGSZ`, la cual se basa en la función de _Z-score_ para el cálculo del ES, y en estimación asintótica del p-valor de cada CG, mediante permutaciones de muestras e (implícitamente) de genes [@mishra2014gene]. El método mGSZ como entrada requiere la matriz de expresión, y rankea los genes mediante el estadístico _t_ moderado [@berkeley2004linear], obtenido utilizando la función `eBayes` de la librería R `limma`. El método mGSZ limita el tamaño de los CG, por defecto, usando un mínimo de 5 genes (sin filtrar por máximo). Adicionalmente, se analizó filtrando CG entre 15 y 500, como en el SM, para obtener una comparación más robusta.

\par En resumen, los métodos y parámetros analizados de PFC, junto con el identificador a utilizar (en negrita), se listan a continuación:

\begin{small}
\[
\begin{matrix}
\begin{matrix}Subra-\\ manian\end{matrix} \left\{\begin{matrix}
	\begin{matrix}Permutación\\ de\; genes\end{matrix} \left\{\begin{matrix}
		   w=0\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMgp0}
		\\ w=1\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMgp1}
		\\ w=2\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMgp2}
	\end{matrix}\right.
	\\ \begin{matrix}Permutación\\ de\; condición\end{matrix} \left\{\begin{matrix}
		   w=0\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpp0}
		\\ w=1\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpp1}
		\\ w=2\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpp2}
	\end{matrix}\right.
	\\ \begin{matrix}Pre-\\ rank\end{matrix} \left\{\begin{matrix}
		   \;\;\;\;\;\;\; t \;\;\;\;\;\;\; \left\{\begin{matrix}
		   w=0\;\;\;\;\;\;\;\;\;\;\textbf{SMpr tScore 0}
		\\ w=1\;\;\;\;\;\;\;\;\;\;\textbf{SMpr tScore 1}
		\\ w=2\;\;\;\;\;\;\;\;\;\;\textbf{SMpr tScore 2}
	\end{matrix}\right.
	\\ \begin{matrix}-log\\ p-valor\end{matrix} \left\{\begin{matrix}
		   w=0\;\;\;\;\;\;\;\;\;\textbf{SMpr -log(p) 0}
		\\ w=1\;\;\;\;\;\;\;\;\;\textbf{SMpr -log(p) 1}
		\\ w=2\;\;\;\;\;\;\;\;\;\textbf{SMpr -log(p) 2}
	\end{matrix}\right.
	\\ \begin{matrix}1-\\ p-valor\end{matrix} \left\{\begin{matrix}
		   w=0\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpr 1-p 0}
		\\ w=1\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpr 1-p 1}
		\\ w=2\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{SMpr 1-p 2}
	\end{matrix}\right.
	\end{matrix}\right.
\end{matrix}\right.
\end{matrix}
\]
\[
\begin{matrix}
mGSZ\;\; \left\{\begin{matrix}
	   CG\:filtrado [15,500)\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\textbf{mGSZ[15,500)}
	\\ CG\:filtrado [5,\infty)\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\mathbf{mGSZ[5,\infty)}
\end{matrix}\right.
\end{matrix}
\]
\end{small}

## Resultados

### Genes diferencialmente expresados por base de datos

\par El número de genes DE para cada base de datos analizada se presenta en la Tabla \@ref(tab:deGenes). Aunque para cada matriz de expresión se contrastaron los mismos subtipos de cáncer de mama, se observa muy poca superposición entre pares de experimentos. Por ejemplo, los datasets Unt y Nki sólo presentan 383 genes DE en común, siendo que Unt tiene 1.059 genes DE en total (sólo el 36% se superponen). En general, se encontró que sólo el 12% (195 genes) de los genes DE en común entre todas los bases de datos (la intersección) se superponen entre la unión (1.678 genes) de los genes DE de todos los datasets evaluados.

Table: Genes diferencialmente expresados entre bases de datos.\label{tab:deGenes}
Primera columna: Los Nombres de las bases de datos, con el umbral de valor absoluto de Fold-Change entre paréntesis. Diagonal principal: Número de genes Diferencialmente Expresados (DE) para cada base de datos (FDR $\leq$ 0,01) y porcentaje del total de genes en el experimento entre paréntesis. Triangular superior: Número de genes DE intersectados por cada par de bases de datos. Nótese que la intersección global de genes DE es sólo 195 de una unión total de 1.678 genes DE.

| Nombre       | Vdx     | Nki     | Transbig | Upp   | Unt       | Mainz   |
|--------------|:-------:|:-------:|:--------:|:-----:|:---------:|:-------:|
|Vdx (0,75)    |611 (4,7)|292      |465       |430    |425        |412      |
|Nki (0,2)     |         |568 (4,3)|310       |374    |383        |286      |
|Transbig (0,6)|         |         |628 (4,8) |448    |461        |433      |
|Upp (0,3)     |         |         |          |932 (5)|632        |428      |
|Unt (0,25)    |         |         |          |       |1.059 (5,7)|437      |
|Mainz (0,45)  |         |         |          |       |           |605 (4,6)|
|              |Intersección:| 195 |          |Unión: | 1.678     |         |

### Análisis de estabilidad de enriquecimiento

\par Para evaluar la estabilidad de detección de CG enriquecidos a través de las diversas bases de datos, para cada combinación de método/parámetros, se generó un boxplot con el número de CG enriquecidos. La integración de resultados de diversas bases de datos nos permite proporcionar validación inter-estudios tal y como se indica en el análisis de Edelman et al. [@edelman2006analysis].

\par Como se puede observar en los boxplot de la Figura \@ref(fig:ifa1), se aprecia una alta variabilidad tanto entre los métodos como para las diversas parametrizaciones de un mismo método. Teniendo en cuenta solo aquellos datasets que presentaron almenos un término enriquecido, se encontró una mediana de 284 términos enriquecidos. El SM parece ser muy sensible a diferentes parametrizaciones, así como a la forma en que se le proveen los datos, es decir, a través de la matriz de expresión de genes o mediante una lista pre-rankeada. Curiosamente, para SMpr cualquier valor del factor de ponderación $w$ devolvió casi cero términos enriquecidos. Además, para SMgp y SMpp, la selección de $w$ podría producir enriquecimientos muy diferentes, que van desde cero términos en SMpp0 para Nki o 59 términos en SMgp2 para Vdx hasta valores extremos como 1.019 en SMpp2 para Nki o 474 términos en SMgp0 para Vdx. En particular, el método SMpp presenta comportamientos muy diferentes dependiendo del valor $w$. Por ejemplo, no se encontró enriquecimiento para $w=0$, gran variabilidad resultó con $w=1$ con un Rango InterCuartil (RIC) de 257,57, y se obtuvieron resultados concordantes con $w=2$, es decir, pequeña dispersión sobre bases de datos (con un RIC de 83,25). Sin embargo, SMpp2 mostró un número extremo de enriquecimientos para una base de datos (1.019 para Nki) y un número muy bajo para otro (101 para Mainz), resultando en dos valores atípicos. Esto podría plantear un problema cuando se analiza un único dataset. Para el SMgp, el enriquecimiento es bastante estable en todos los datasets, con RICs de 105, 91,25 y 98,5 para SMgp0, SMgp1 y SMgp2, respectivamente, pero se obtuvo un número decreciente de enriquecimientos a medida que $w$ aumentaba de 0 a 2, lo que arroja una mediana de 611,5, 293 y 121, respectivamente. Para $w=1$ el número de términos enriquecidos es similar a la mediana general (284 términos enriquecidos) de los diferentes métodos, lo que sugiere que el SMgp1 se podría considerar una configuración más apropiada.

<div class="figure" style="text-align: center">
<img src="images/IFA_FIG1_es.PDF" alt="(ref:captionIfa1)" width="100%" />
<p class="caption">(\#fig:ifa1)(ref:captionIfa1)</p>
</div>
(ref:captionIfa1) Boxplot del número de términos enriquecidos por cada método para las diferentes bases de datos. Las siglas de los métodos se describen en la Sección \@ref(sec:algosComparados). Notar que las alternativas de SMpr enriquecen casi ningún conjunto de genes, mientras que la mediana de términos enriquecidos es de 284 para los métodos restantes (línea negra horizontal). El método SMpp1 obtuvo el número más variable de términos enriquecidos. A excepción de dEnricher con prueba de Fisher, todas las demás combinaciones de método/parámetro de dEnricher devolvieron valores extremos.

\par En el caso de mGSZ, los resultados mostraron una estabilidad adecuada entre los datasets (RICs de 26,5 para mGSZ$[15,500)$ y 73 para mGSZ$[5,\infty)$), produciendo un número muy similar de términos enriquecidos entre las bases de datos. Este método resultó sensible al tamaño de los CG analizados. Cuando el filtro de CG a analizar se estableció entre $[15,500)$, se logró un número bastante conservador de términos enriquecidos, es decir, número menor de CG enriquecidos. Estos resultados fueron más estables en comparación con el valor obtenido con los límites de filtro de CG de $[5,\infty)$. Vale la pena mencionar que los términos enriquecidos obtenidos con filtro $[15,500)$ fueron contenidos en su mayoría (93% en promedio) por el método con el filtro de CG por defecto. Estos términos enriquecidos adicionales generalmente contenían un menor número de genes, es decir, términos más específicos que pueden ser mucho más útiles para descifrar el fenómeno biológico bajo estudio. Basándonos en este concepto, en adelante, hemos seguido las recomendaciones del autor de mGSZ en cuanto al filtrado de conjuntos de genes de $[5,\infty)$.

\par A excepción de las diversas parametrizaciones de dEnricher, los resultados de las alternativas de ASR fueron bastante similares entre sí, produciendo enriquecimientos bastante estables en todos los datasets: RICs de 6,25 para RD BRI; 115,75 RD BRIII; 8,75 WD BRI; 58 WD BRIII; 21,75 GOstats BRI; y 45,5 GOstats BRIII. La configuración RD BRIII mostró una mayor variabilidad que su contraparte de WD BRIII, probablemente porque se analizó un mayor número de CG (76% más CG en promedio). Para cada alternativa de ASR, los enriquecimientos obtenidos con BRIII estaban en general contenidos en los obtenidos utilizando BRI (99% de los términos en promedio para RD y WD; y 86% para GOstats) en concordancia con la observación de Fresno et al. [@fresno2012multi].

\par En el caso de dEnricher, las pruebas Hipergeométrica y Binomial arrojaron un número extremo de términos enriquecidos en comparación con la mediana general de los demás métodos, obteniendo más de 1.168 términos enriquecidos, independientemente del algoritmo de penalización aplicado. Por otra parte, la prueba de Fisher arrojó resultados bastante estables, en comparación con los otros métodos de ASR, con valores medios de 210, 195,5 y 191,5 para dE_F_none, dE_F_lea y dE_F_elim, respectivamente; con RICs inferiores a 13,75. Los términos enriquecidos obtenidos con los algoritmos dE_F_lea y dE_F_elim fueron 100% contenidos en los obtenidos con dE_F_none; más aún, el 86% de estos términos enriquecidos adicionales encontrados por dE_F_none estaban relacionados con el cáncer de mama cuando fueron consultados en la BDTC. Para los siguientes análisis se descartaron las combinaciones de parámetros de dEnricher, excepto dE_F_none.

\par Sólo aquellos métodos y configuraciones que devolvieron un número concordante de términos enriquecidos entre datasets, y alrededor de la mediana general de los métodos (284 términos enriquecidos) fueron considerados para los siguientes análisis, es decir, SMgp1, SMpp2, mGSZ$[5,\infty)$, RD BRI, RD BRIII, WD BRI, WD BRIII, GOstats BRI, GOstats BRIII y dE_F_none.

### Análisis de profundidad en Gene Ontology

\par Gene Ontology está organizado en tres grafos acíclicos dirigidos (árboles: "funciones moleculares", "procesos biológicos" y "componentes celulares"), donde un nodo hijo representa un término más específico biológicamente que su padre. Para explorar la especificidad biológica sobre la estructura de GO, se calculó el número mínimo de ramas entre el nodo y la raíz (la profundidad) de cada término enriquecido. Luego, se calculó una tabla de frecuencias del número de términos enriquecidos por profundidad (agrupando los resultados para cada base de datos).

\par El porcentaje de términos enriquecidos agrupados por profundidad para cada método se muestra en la Figura \@ref(fig:ifa2). Todos los métodos tienden a explorar profundidades en su mayoría entre tres y seis. Los métodos mGSZ, dE_F_none, SMpp2 y GOstats BRIII enriquecieron el mayor número de términos específicos, es decir, nodos más profundos u hojas, con profundidades $>$ 6: 13%, 12,8%, 10,7% y 9,8% del enriquecimiento total, respectivamente. Además, WD y RD proporcionaron mayor enriquecimiento de términos generales, es decir, en profundidades $<$ 3, nodos más cercanos al nodo raíz de cada árbol de GO: 13,7% para RD BRI, 11,9% para WD BRI, 9,7% para WD BRIII y 8,8% para RD BRIII. Dentro de los métodos de ASR, proporcionalmente se enriquecen más términos cerca de la raíz cuando se usa BRI que BRIII.

\par Para cada método de ASR, excepto dEnricher, BRI enriqueció un número mayor de términos que BRIII. Sin embargo, como se mencionó anteriormente, tanto en WD, RD - como en GOstats - la mayoría de los términos enriquecidos por BRIII también fueron enriquecidos por BRI. Como se discutió en Fresno et al., usar BRIII, estadísticamente, tiene más sentido que usar BRI; por lo tanto, su uso se sugiere y se usa de aquí en adelante.

<div class="figure" style="text-align: center">
<img src="images/IFA_FIG2_es.PDF" alt="(ref:captionIfa2)" width="100%" />
<p class="caption">(\#fig:ifa2)(ref:captionIfa2)</p>
</div>
(ref:captionIfa2) Profundidad de enriquecimiento de Gene Ontology (GO) para cada método. Los colores más oscuros representan términos más profundos de la jerarquía del árbol GO. Note que todos los métodos tienden a enriquecer las profundidades en su mayoría entre tres y seis. Los métodos WD y RD enriquecieron los términos más superficiales de la estructura de árbol de GO. Por otro lado, mGSZ, dE_F_none, SMpp2 y GOstats BRIII enriquecieron los términos más profundos.

### Análisis de consenso {#sec:consensusSection}

\par Para evaluar la hipótesis de que los fenotipos comparados deberían presentar perfiles de enriquecimiento similares en todas las bases de datos, se construyó una matriz de enriquecimiento $E=e_{mdt}$. En esta matriz, cada fila contiene un término GO y cada columna contiene una combinación de método/parámetros por base de datos. Donde cada celda $e_{mdt}$ de la matriz se definió como sigue:

\[
e_{mdt}=\left\{\begin{matrix}
1 & si\:m\:\mathbf{enriqueció}\:t\:en\:la\:base\:de\:datos\:d \\
0 & si\:m\:\mathbf{no\:enriqueció}\:t\:en\:la\:base\:de\:datos\:d \\
NA & si\:m\:\mathbf{no\:analizó}\:t\:en\:la\:base\:de\:datos\:d
\end{matrix}\right.
\]
donde $m$ = {$1,\ldots,M$} combinación de método/parámetros; $t$ = {$1,\ldots,T$} un conjunto de genes; $d$ = {$1,\ldots,D$} una base de datos.

\par Aquellos términos que no fueron enriquecidos en ningún conjunto de datos fueron eliminados. Usando la librería R `vegan`, se aplicó un agrupamiento jerárquico a las filas y columnas de $E$ a través de la distancia Jaccard y enlace promedio para agrupar automáticamente perfiles de enriquecimiento similares. En la Figura \@ref(fig:ifa3) se observa el _heatmap_ resultante de la matriz $E$. El dendrograma superior muestra que los conjuntos de datos analizados tienden a agruparse según el método aplicado, excepto para SMpp2, GOstats y RD, que presentan un conjunto de datos disperso. También puede observarse una diferenciación entre los resultados obtenidos por ASR y PFC, es decir, los métodos de PFC forman un cluster separado, mientras que los métodos de ASR se dividen en dos sub-clusters principales: el de RD/WD y el de GOstats/dE_F_none. Como era esperado, puede verse un subconjunto de términos enriquecidos por casi todos los métodos a través de los conjuntos de datos (filas etiquetadas como **A**). El 64% de términos enriquecidos en al menos el 80% de los conjuntos de datos para cada método también fueron enriquecidos por mGSZ en la misma proporción. Esto sugiere que, hasta cierto punto, todos los métodos tienden a proporcionar la misma información. Sin embargo, cada método también proporciona términos exclusivamente enriquecidos (filas etiquetadas como **E**). La comparación de los métodos de PFC y ASR muestra que RD y WD tienden a enriquecer algunos términos que no se enriquecen por ningún otro método; lo mismo sucede con dE_F_none y con mGSZ, lo que sugiere que PFC y ASR se complementan entre sí. Los métodos dE_F_none, GOstats y RD, así como mGSZ, analizan más términos que los demás métodos (un número menor de $e_{mdt}=NA$). Además, el 63% de los términos enriquecidos en al menos el 80% de los conjuntos de datos por los SM también fueron enriquecidos por mGSZ en la misma proporción, mientras que el 47% de los términos enriquecidos por GOstats, RD y WD fueron enriquecidos por dE_F_none, lo que sugiere que mGSZ y dE_F_none pueden ser utilizados como métodos de referencia para PFC y ASR, respectivamente.

\par Dado que todos los experimentos comparan los mismos fenotipos de cáncer de mama, se esperaba encontrar resultados concordantes entre los conjuntos de datos para cada combinación de método/parámetros. Así, el número de términos enriquecidos en casi todos los conjuntos de datos (Frecuencia de Consenso en Enriquecimiento; $FCE$), así como los términos que no se enriquecieron en casi todos los conjuntos de datos (Frecuencia de Consenso en No Enriquecimiento; $FCNE$), se utilizaron como indicador de la estabilidad del método. Basado en este supuesto, definimos las métricas de comparación listadas en la Tabla \@ref(tab:comparisonMetrics). La concordancia entre los conjuntos de datos para cada método mostró que mGSZ supera con un 45% de términos enriquecidos concordantes ($FCE$), seguido de dE_F_none, RD y WD con un 39%, SMgp1 con un 36%, SMpp2 con un 30%, y GOstats con un 29%. La concordancia entre los términos no enriquecidos ($FCNE$) obtuvo 91% para dE_F_none, RD y GOstats, 89% para WD, 85% para mGSZ, 83% para SMgp1, y 82% para SMpp2. Por lo tanto, todos los métodos parecen tener un alto consenso para los términos no enriquecidos y un bajo consenso para los enriquecidos. Ambos conceptos son importantes a la hora de enfrentarse al AF, ya que no debe perderse ningún término biológicamente significativo, ni debe haber términos enriquecidos incorrectamente. De aquí se desprende que el análisis de PFC utilizando mGSZ demostró ser el método más consensuado, mientras que dE_F_none y RD para la contraparte de ASR.

<div class="figure" style="text-align: center">
<img src="images/IFA_FIG3.PDF" alt="(ref:captionIfa3)" width="90%" />
<p class="caption">(\#fig:ifa3)(ref:captionIfa3)</p>
</div>
(ref:captionIfa3) Heatmap de enriquecimiento. En columnas, cada combinación de método/parámetros por base de datos; en filas, esos conjuntos de genes (términos) enriquecidos en al menos una base de datos. Notar que los métodos de PFC y ASR se agrupan por separado en el dendrograma. Celdas rojas indican enriquecimiento, naranjas indican que no hay enriquecimiento, y blancas muestran términos que no fueron analizados. Se observan subconjuntos de términos que resultaron enriquecidos por casi todos los métodos analizados (**A**), y subconjuntos de términos enriquecidos exclusivamente por una sola combinación de método/parámetros (para todas las bases de datos; **E**). El color de la etiqueta de cada columna representa el algoritmo utilizado, y la letra representa la letra inicial del conjunto de datos: V Vdx; N Nki; T Transbig; U Upp; u Unt; M Mainz.

Table: Métricas de comparación.\label{tab:comparisonMetrics}
$d$ = {$1,\ldots,D$} base de datos seleccionada; $t$ = {$1,\ldots T$} conjunto de genes seleccionado; $m$: combinación de método/parámetros utilizada; $t_e=0.8$ y $t_n=0.2$ umbrales para el enriquecimiento y el no enriquecimiento, respectivamente; $I$ función indicadora.

| Nombre       | Definición     |
|--------------|:--------------:|
|Frecuencia de enriquecimiento inter-método|$F_{mt}=\frac{1}{D}\sum_{d=1}^{D}e_{mdt}$|
|Consenso en Enriquecimiento               |$CE_m=\sum_{t=1}^{T}I(F_{mt} > t_e)$|
|Consenso en No Enriquecimiento            |$CNE_m=\sum_{t=1}^{T}I(F_{mt} < t_n)$|
|No Consenso                               |$NC_m=\sum_{t=1}^{T}I(t_n \leq F_{mt} \leq t_e)$|
|Frecuencia de $CE$                        |$FCE_m=\frac{CE_m}{CE_m+NC_m}$|
|Frecuencia de $CNE$                       |$FCNE_m=\frac{CNE_m}{CNE_m+NC_m}$|

\par Dado que RD fue capaz de analizar más CG que WD, que puede ser utilizado con CG provistos por el usuario, y no depende de una conexión a Internet, se prefirió sobre WD. Por lo tanto, este último fue excluido de los análisis siguientes.

### Enriquecimiento exclusivo y relevancia de términos

\par Para determinar si los métodos pueden considerarse complementarios o no desde la perspectiva de la información obtenida, se analizaron los términos exclusivamente enriquecidos (TEE) para cada método, es decir, los términos enriquecidos por el método $m$ en el 80% de los datasets, pero en menos del 20% de los demás $m' \neq m$, es decir:

$TEE_m=\{t|F_{mt}>t_e \wedge \forall m' \neq m : F_{m't} < t_n \}$
donde $t$ es cualquier conjunto de genes; $m$ alguna combinación de método/parámetros; $t_e=0.8$ y $t_n=0.2$ umbrales para el enriquecimiento y el no enriquecimiento, respectivamente.

\par Para evaluar la asociación de los términos exclusivamente enriquecidos con el fenotipo bajo estudio, se evaluó la relación de cada término perteneciente a cada $TEE_m$ utilizando la base de datos dla BDTC [@davis2014comparative]. Utilizando la BDTC, para cada término se pudo determinar su condición patológica en relación con el concepto de "cáncer de mama".

\par Considerando los términos enriquecidos exclusivamente por cada método ($TEE_m$), encontramos que mGSZ y dE_F_none fueron los que obtuvieron más $TEE_m$ para PFC y ASR, respectivamente (ver Tabla \@ref(tab:eet)). Adicionalmente, mGSZ proporcionó más términos relacionados con el cáncer de mama de acuerdo con la BDTC, así como términos mucho más específicos (profundidad $>$ 6). En el caso del ASR, dE_F_none obtuvo más $TEE_m$, también relacionados con el cáncer de mama.

\begin{table}[!htbp]
\centering
\caption{Número de términos enriquecidos exclusivos. El número de términos enriquecidos relacionados con el cáncer de mama según la Base de Datos de Toxigenómica Comparativa se presenta entre paréntesis. Notar que mGSZ y dE\_F\_none enriquecen el mayor número de términos para PFC y ASR, respectivamente.}
\begin{tabular}{lccccc}
& \multicolumn{5}{c}{Profundidad en los arboles de GO} \\ \cline{2-6}
Método & 0-2 & 3-4 & 5-6 & 7-10 & Total \\
\hline
SMpp2 & & & 2 (0) & 1 (1) & 3 (1) \\
SMgp1 & 2 (1) & 15 (9) & 7 (4) & & 24 (14) \\
mGSZ & & 26 (13) & 27 (12) & 8 (3) & 61 (28) \\
dE\_F\_none & 1 (1) & 4 (3) & 5 (3) & 3 (1) & 13 (8) \\
RD & 4 (0) & 2 (1) & & & 6 (1) \\
GOstats & & 1 (0) & & & 1 (0) \\
\hline
\end{tabular}
\label{tab:eet}
\end{table}

### Análisis Funcional Integrador {#sec:ifa}

\par Como resultado de este estudio exhaustivo [@rodriguezciarp; @rodrigueziscb; @rodriguez2016improving; @rodriguezsabi], en la Figura \@ref(fig:ifa4) se presenta el pipeline del análisis funcional integrador (IFA; del inglés _Integrative Functional Analysis_). El IFA proporciona como una herramienta de software, cuyo código R se encuentra en el repositorio [https://github.com/jcrodriguez1989/IFA](https://github.com/jcrodriguez1989/IFA). Para utilizar la función principal del IFA, el usuario debe proporcionar la matriz de expresión, y la especificación de los fenotipos de cada sujeto. En caso que no se proporcionen los CG, IFA utilizará los CG de GO más actualizados provistos por la librería R `org.Hs.eg.db`. La lista de genes DE y su rankeo son calculados por la herramienta IFA mediante un modelo lineal utilizando la librería de R `limma`, con los cuales se llevan a cabo los análisis mGSZ y de dE_F_none. De este modo, IFA proporciona un enfoque sencillo, unificado y global para el AF.

<div class="figure" style="text-align: center">
<img src="images/IFA_FIG4_es.pdf" alt="(ref:captionIfa4)" width="100%" />
<p class="caption">(\#fig:ifa4)(ref:captionIfa4)</p>
</div>
(ref:captionIfa4) Flujo de trabajo del Análisis Funcional Integrador (IFA). El usuario proporciona la matriz de expresión y las correspondientes etiquetas de fenotipo como entrada. El IFA utiliza librerías auxiliares de R para obtener los genes expresados diferencialmente, rankearlos y realizar análisis de PFC y ASR. Finalmente, los resultados de enriquecimiento obtenidos por el IFA integran tanto los resultados del ASR como los de la PFC.

### IFA sobre TCGA

\par Para demostrar la utilidad del pipeline del IFA, se obtuvo el dataset proveniente de microarreglos de ADN de cáncer de mama del reconocido proyecto TCGA. En este dataset, 86 sujetos resultaron clasificados como tipo Basal y 198 como Luminal A. Se utilizó un valor $treatLfc=1$ de manera de obtener alrededor del 5% de los genes DE. Los resultados del IFA para este conjunto de datos se utilizaron como caso de prueba y, por lo tanto, se evaluaron y compararon con los conjuntos de datos analizados previamente (Tabla \@ref(tab:deGenes)). Como en la Sección \@ref(sec:consensusSection), la matriz de consenso de resultados del IFA se presentó mediante un _heatmap_ que se puede observar en la Figura \@ref(fig:ifa5), donde 812 términos resultaron enriquecidos para el conjunto de datos de TCGA. El treinta y tres por ciento de los términos (270) se encontraron enriquecidos también en todos los demás conjuntos de datos (marca **A** en la Figura \@ref(fig:ifa5)), el 43% (352 términos) se encontraron enriquecidos en más del 80% de los otros conjuntos de datos, y el 15% (123 términos) se encontraron enriquecidos exclusivamente en TCGA (marca **E$^*$** en la Figura \@ref(fig:ifa5)). En esta matriz de consenso, 445 términos resultaron enriquecidos en al menos el 80% de todos los conjuntos de datos, de los cuales 232 (52%) estaban relacionados con el cáncer de mama según la BDTC. En particular, la proporción media de términos exclusivamente enriquecidos presentes en la BDTC en cada conjunto de datos fue del 42%, con Nki primero con 170 términos exclusivos (84 en la BDTC), y con Mainz último con 58 términos (30 en la BDTC).

\par Los términos relacionados con la hormona y el receptor de estrógeno, la transición G1/S del ciclo celular mitótico, la replicación del ADN, la organización del huso mitótico, el desenrollado dual del ADN, la actividad de la histona cinasa, la actividad de la helicasa de hibridación, entre otros, se encontraron enriquecidos, en común, en todos los conjuntos de datos (marca **A** en la Figura \@ref(fig:ifa5)), lo que respalda las diferencias de proliferación entre los subtipos de cáncer de mama de tipo Basal y Luminal A. Además, términos como la actividad de la proteína de señalización del receptor tirosina cinasa, la diferenciación de células madre y otros relacionados con la diferenciación celular, sólo se encontraron en los conjuntos de datos de TCGA (marca **E$^*$** en la Figura \@ref(fig:ifa5)).

<div class="figure" style="text-align: center">
<img src="images/IFA_FIG5.PDF" alt="(ref:captionIfa5)" width="100%" />
<p class="caption">(\#fig:ifa5)(ref:captionIfa5)</p>
</div>
(ref:captionIfa5) Heatmap de enriquecimiento del Análisis Funcional Integrador en cáncer de mama (incluyendo TCGA). Los datasets se ubican en columnas; y los conjuntos de genes (términos), enriquecidos en al menos un dataset, se presentan en filas. Celdas rojas indican enriquecimiento, y las anaranjadas indican que no hay enriquecimiento. Se observan subconjuntos concordantes de términos enriquecidos entre cada conjunto de datos (**A**) y subconjuntos de términos enriquecidos exclusivamente en un solo conjunto de datos (**E**).

### IFA sobre datasets de cáncer de próstata

\par Con el fin de verificar que los resultados del IFA no dependen de la patología analizada, sino que puede extenderse a otros escenarios, el IFA se probó sobre cuatro datasets de cáncer de próstata disponibles en el repositorio R **Bioconductor**. En total, se obtuvieron 519 sujetos de las bases de datos: 74 sujetos con cáncer de próstata benigno y 125 con tumor para Camcap; 29 benignos y 150 tumor para Taylor; 6 vs. 13 para Varambally; y 28 vs. 94 para Grasso. Para obtener alrededor del 5% de los genes DE, se utilizaron valores de $treatLfc$ de 0,2, 0,15, 0,2, y 0,45 para Camcap, Taylor, Varambally y Grasso, respectivamente. Para el análisis de Varambally, los p-valores de los genes no fueron ajustados, ya que se no se obtuvieron genes DE bajo ningún valor de $treatLfc$ con un valor de corte de FDR fijado en 0,01.

\par La matriz de consenso resultante se muestra en la Figura \@ref(fig:ifa5), en la que 163 términos fueron enriquecidos en al menos el 80% de los conjuntos de datos, de los cuales 99 (61%) están relacionados con el cáncer de próstata según la BDTC. En particular, la proporción media de términos exclusivamente enriquecidos presentes en la BDTC en cada conjunto de datos fue del 44%, con Taylor primero con 448 términos exclusivos (194 en la BDTC) y Varambally último con 212 términos (98 en la BDTC).

\newpage
<div class="figure" style="text-align: center">
<img src="images/IFA_FIG6.PDF" alt="(ref:captionIfa6)" width="100%" />
<p class="caption">(\#fig:ifa6)(ref:captionIfa6)</p>
</div>
(ref:captionIfa6) Heatmap de enriquecimiento del Análisis Funcional Integrador en cáncer de próstata. Los datasets se ubican en columnas; y los conjuntos de genes (términos), enriquecidos en al menos un dataset, se presentan en filas. Celdas rojas indican enriquecimiento, y las anaranjadas indican que no hay enriquecimiento. Se observan subconjuntos concordantes de términos enriquecidos entre cada conjunto de datos (**A**) y subconjuntos de términos enriquecidos exclusivamente en un solo conjunto de datos (**E**).

## Conclusiones

### Comparación de métodos

\par En este capítulo se pudo mostrar que los resultados del AF pueden varían dependiendo del método y los parámetros utilizados. Esto podría influir negativamente en la interpretación biológica si no se aborda adecuadamente. Por ejemplo, el método de PFC Subramanian mostró una alta sensibilidad a diferentes configuraciones de parámetros y datos de entrada. Los métodos SMpp0 y SMpr, utilizando el valor de corte estadístico recomendado, casi no devolvieron términos enriquecidos para los conjuntos de datos analizados. Estos resultados fueron bastante inesperados porque la naturaleza de los subtipos de cáncer de mama considerados tienen mecanismos biológicos subyacentes muy contrastantes y se han reportado resultados de supervivencia altamente opuestos [@parker2009supervised]. Sin embargo, cuando el usuario sólo tiene una lista ordenada de genes, no hay otra alternativa que la versión pre-rankeada (SMpr) para realizar PFC. En este caso, sugerimos utilizar el rankeo por $1-p$-$valor$ con $w=1$, y verificar con diversos valores de corte que devuelvan términos enriquecidos. Cuando se alimentó el SM con la matriz de expresión y sus etiquetas de fenotipo, se encontró que tanto las permutaciones de fenotipo como de genes eran muy sensibles al valor de ponderación $w$ elegido, lo que producía diferentes cantidades de términos enriquecidos, así como diferentes niveles de variabilidad de enriquecimiento entre los conjuntos de datos. La permutación del fenotipo con $w=2$ (SMpp2) parece proporcionar resultados estables, pero la aparición de conjuntos de datos con un número extremo de términos enriquecidos desalienta su uso. Además, cuando $w=1$ los resultados fueron muy inestables (RIC alto). La estrategia de permutación de genes con $w=1$ y $w=0$ (SMgp1 y SMgp0) proporcionó resultados muy estables, pero estas parametrizaciones enriquecieron casi el doble de términos que todos los demás métodos analizados. Cuando se usó $w=2$ (SMgp2), se obtuvo un número bajo de términos enriquecidos. En desacuerdo con los autores del SM, recomendamos SMgp sobre SMpp. Sin embargo, estamos de acuerdo con su recomendación sobre el valor de ponderación $w=1$, es decir, recomendamos SMgp1 sobre cualquier otra configuración de SM.

\par El método mGSZ, a comparación del SM, fue más estable entre los diversos conjuntos de datos, produciendo un alto consenso entre datasets (alto $FCE$) y proporcionando el mayor número de términos informativos y exclusivamente enriquecidos ($TEE$). Se observó que, cuando no se utilizaba límite superior de tamaño de CG (mGSZ$[5,\infty)$), se enriquecieron términos específicos e informativos adicionales a con el límite [15, 500). Por lo tanto, fomentamos el uso de mGSZ$[5,\infty)$. Otra ventaja de mGSZ sobre el SM es que el primero tiene una implementación R actualizada, mientras que el segundo requiere de un entorno Java.

\par En cuanto a las metodologías de ASR, aunque el uso de BRI produce resultados estables y contiene los términos enriquecidos utilizando BRIII, este último es más apropiado desde un punto de vista estadístico [@fresno2012multi]. Además, se demostró que BRIII a diferencia de BRI no presenta un número extremo de términos enriquecidos, pero tiene un rango más variable de términos enriquecidos sobre los conjuntos de datos utilizados. La versión R implementada de DAVID (RD) fue desarrollada para funcionar lo más similar posible a la plataforma web de DAVID (WD), con la ventaja de utilizar una base de datos de anotaciones GO actualizada. Aún más, cualquier CG deseado de interés puede ser analizado. Además, RD no requiere una conexión a Internet ni una cuenta registrada en el sitio web de DAVID. En el caso de GOstats, se demostró que es demasiado variable cuando se prueba el enriquecimiento a través de bases de datos (bajo $FCE$). Esto podría volverse problemático cuando se analiza un solo conjunto de datos, es decir, daría una visión biológica muy limitada del experimento que se está analizando. Para las diversas parametrizaciones de dEnricher, se obtuvieron valores extremos de enriquecimiento cuando se utilizó test _Hipergeométrico_ o _Binomial_; sin embargo, cuando se utilizó la prueba de _Fisher_, se obtuvieron resultados concordantes. Además, al no aplicar ningún algoritmo de penalización (dE_F_none) se obtuvieron términos adicionales relacionados con la enfermedad bajo estudio. El dE_F_none resultó ser el método más estable entre los conjuntos de datos para las alternativas de ASR (el más alto $FCE$) y superó a sus competidores en cuanto al número de términos informativos enriquecidos exclusivamente ($TEE_m$).

\par En resumen, concluimos que si los parámetros se establecen correctamente, el AF obtiene información biológica consensuada y significativa a pesar del bajo nivel de genes DE en común entre diversas bases de datos. Se demostró que tanto el ASR como la PFC proporcionan resultados complementarios que pueden integrarse para obtener una visión biológica más amplia. Además, su integración nos permite abarcar una mayor profundidad de la estructura GO, una característica deseable a la hora de contrastar condiciones experimentales [@fresno2012multi]. En consecuencia, proponemos utilizar el pipeline del IFA, que realiza análisis simultáneos de ASR y PFC a través de dE_F_none y mGSZ, que resultaron ser los métodos más efectivos respectivamente. Ambos enfoques se basan en el mismo modelo lineal a través de la conocida librería R `limma` [@berkeley2004linear]. Por lo tanto, proporciona un marco completo y unificado que utiliza únicamente la matriz de expresión, el diseño experimental y (si no se dispone de CG) la base de datos GO de `org.Hs.eg.db` [@carlson2013org]. Aunque este trabajo se basó en los conjuntos de genes GO, cualquier otra base de conocimiento de conjuntos de genes podría ser aplicada para el IFA.

### Aplicación del IFA

\par La aplicación del IFA para estudiar los términos regulados diferencialmente entre los subtipos de cáncer de mama Luminal A y Basal resultó en la detección de varios términos altamente relacionados con el cáncer de mama. Por ejemplo, IFA permitió detectar términos relacionados con las vías de señalización de los receptores de hormonas y estrógenos, los cuales están fuertemente relacionados con los subtipos de cáncer de mama analizados, ya que los sujetos Luminal A son dependientes de estrógeno, mientras que los sujetos Basales no lo son [@dai2015breast; @goldhirsch2013personalizing; @bastien2012pam50]. A partir del IFA, los resultados concordantes revelaron términos asociados con el proceso de desenrollado del ADN, un evento asociado con el inicio de la síntesis de ADN y relacionado con la facilitación de la actividad de las helicasas. Se ha reportado que la helicasa BACH1/FANCJ está mutada en el cáncer de mama de inicio temprano, especialmente relacionado con el gen hereditario del cáncer de mama _BRCA1_ [@cantor2001bach1]. La mayoría de los cánceres de mama relacionados con el gen _BRCA1_ son triplemente negativos y Basales [@atchley2008clinical], por lo tanto son esperadas las diferencias en los genes que regulan el proceso de desenrollado del ADN entre Luminal A y Basal. Además, el IFA reveló un grupo de términos que sólo estaban regulados de forma diferencial en el conjunto de datos de TCGA. De ellos, la actividad de la histona metiltransferasa en H3-K9 fue uno de los términos más profundos encontrados en la estructura de árbol de GO. La metilación de la histona H3-K9 se ha correlacionado con la formación de heterocromatina y la represión transcripcional, que puede regular la expresión del receptor de estrógeno [@sharma2005release]. Además, el término de actividad de la tirosina cinasa está involucrada en la terapia de elusión en cánceres de mama triplemente negativos - Basales - [@scaltriti2016molecular]. La evaluación de la capacidad de generalización del IFA en los conjuntos de datos sobre el cáncer de próstata también arrojó resultados concordantes e informativos. Además, como era de esperar y visto en el caso del cáncer de mama, se enriquecieron términos presentes en consenso entre los conjuntos de datos, así como términos específicos enriquecidos para cada uno de ellos.

\par Estos hallazgos apoyan la utilidad de nuestra propuesta desde la perspectiva de la minería de datos biológicos. Además, el marco de análisis propuesto, IFA, supera las limitaciones de las bases de conocimiento que presentan los métodos, minimiza los parámetros a definir por el usuario, facilita el AF, permite la comparación de diferentes cohortes de pacientes, como se muestra con los resultados de TCGA y cáncer de próstata, y se proporciona gratuitamente como código abierto R en [GitHub](http://github.com/jcrodriguez1989/IFA).
