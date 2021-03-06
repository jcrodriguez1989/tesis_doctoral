# Desafíos que surgieron durante el trabajo de tesis {#cap:desafios}

## Eficiencia computacional del algoritmo `mGSZ`

\par Como se vio en el Capítulo [3](#cap:ifa), el método `mGSZ` resultó ser la mejor alternativa para llevar a cabo el análisis funcional del tipo _Puntuación Funcional de Clase_. La función principal de `mGSZ`, con sus parámetros por defecto -que sugerimos como resultado del Capítulo [3](#cap:ifa)-, realiza un máximo de 200 permutaciones y filtra solo aquellos conjuntos de genes que presentan menos de cinco genes. En este sentido, llevar a cabo el análisis funcional de `mGSZ` de un experimento ordinario, por ejemplo, Basal vs. Luminal A de datos de microarreglos del TCGA sobre Gene Ontology, en una computadora estándar, puede tomar alrededor de 2,46 horas, como mostramos a continuación.

\par Esta demora del algoritmo, puede resultar despreciable si se desea analizar un único experimento. Sin embargo, como objetivo global del presente trabajo de tesis, se propuso una herramienta de exploración, desde un punto de vista funcional, de grandes cantidades de experimentos. Si extrapolamos linealmente la demora del análisis funcional de un experimento a 118 experimentos (el número de experimentos analizados en el Capítulo [4](#cap:migsa)), estaríamos hablando de una espera de 290,28 horas, es decir, 12,09 días. Dada esta situación, resultó fundamental afrontar este desafío, mediante la optimización del algoritmo original de `mGSZ`, de manera de disminuir el tiempo de ejecución.

### Optimización del algoritmo `mGSZ`

\par Para disminuir el tiempo de ejecución de `mGSZ`, en primer instancia se debió analizar el código del paquete `mGSZ` de manera de encontrar porciones de código críticas. Mediante los paquetes `profvis` y `microbenchmark` se detectaron tanto aquellas porciones de código de más lenta ejecución, como aquellas que más frecuentemente se ejecutaban. Como resultado de este análisis, se obtuvieron unos pocos fragmentos de código que fueron **objetivos críticos** a optimizar.

\par La optimización de estos objetivos críticos se llevo a cabo aplicando diversas estrategias de optimización generales [@wolfe1996high; @cooper2011engineering] como también algunas específicas del lenguaje R [@wickham2014advanced; @gillespie2016efficient; @burns2011r]. Esta versión optimizada de la función `mGSZ` se implementó en el paquete `MIGSA`, y puede utilizarse, independientemente de `MIGSA`, llamando a la función `MIGSAmGSZ`.

\par Con el fin de evaluar las mejoras en tiempo de ejecución de `MIGSAmGSZ` con respecto a `mGSZ`, se analizaron ambas funciones dándoles como entrada los mismos input: la matriz de expresión de microarreglos de ADN de cáncer de mama del TCGA (16.207 genes $\times$ 237 sujetos), donde se contrastaron los subtipos Basal vs. Luminal A, analizando los conjuntos de genes del Gene Ontology y KEGG (20.425 conjuntos de genes). El análisis se llevó a cabo utilizando un procesador Intel(R) Xeon(R) E5-2620 v3 @ 2.40GHz (24 núcleos), y 128GB de memoria RAM.

\par Dicha comparación se puede replicar mediante el siguiente código:


```r
library("BiocParallel")
library("mGSZ")
library("MIGSA")
library("MIGSAdata")

# Cargamos la matriz de expresión del TCGA
data(tcgaMAdata)
subtipos <- tcgaMAdata$subtypes
expresion <- tcgaMAdata$geneExpr

dim(expresion) # #genes y #sujetos
```

```
[1] 16207   237
```

```r
table(subtipos) # #sujetos de cada subtipo
```

```
subtipos
Basal  LumA 
   95   142 
```


```r
# Cargamos los conjuntos de genes de KEGG y Gene Ontology
# utilizando funciones de MIGSA
conj_genes <- list(
  KEGG = downloadEnrichrGeneSets("KEGG_2015")[[1]],
  BP = loadGo("BP"),
  CC = loadGo("CC"),
  MF = loadGo("MF")
)
conj_genes <- do.call(c, lapply(conj_genes, MIGSA:::asList))
```


```r
# Ejecutamos mGSZ original y medimos su tiempo de ejecución
set.seed(8818)
mGSZ_tiempo <- system.time({
  mGSZ_res <- mGSZ(expresion, conj_genes, subtipos)$mGSZ
})
```


```r
# Ejecutamos MIGSAmGSZ y medimos su tiempo de ejecución
register(SerialParam()) # linea que aclararemos a continuación
set.seed(8818)
MIGSAmGSZ_tiempo <- system.time({
  MIGSAmGSZ_res <- MIGSAmGSZ(expresion, conj_genes, subtipos)
})
```




```r
# Vemos el tiempo total ("elapsed") y calculamos Speedup
mGSZ_tiempo <- mGSZ_tiempo[["elapsed"]]
MIGSAmGSZ_tiempo <- MIGSAmGSZ_tiempo[["elapsed"]]
c(
  mGSZ = mGSZ_tiempo / 60 / 60,            # en horas
  MIGSAmGSZ = MIGSAmGSZ_tiempo / 60 / 60,  # en horas
  Speedup = mGSZ_tiempo / MIGSAmGSZ_tiempo
)
```

```
     mGSZ MIGSAmGSZ   Speedup 
 2.461000  1.553534  1.584130 
```

Como se observa, la versión optimizada, `MIGSAmGSZ`, presenta un _speedup_ de 1,6X sobre el algoritmo original -de demorar 2,46 horas, la nueva versión pasó a demorar 1,55 horas-.

\par Por otra parte, la implementación original de `mGSZ` se ejecuta, exclusivamente, de modo secuencial. En la actualidad, siendo tan fácil el acceso a computadores multi-procesador, el algoritmo `mGSZ` deja de lado una alternativa muy favorable en cuanto a la eficiencia en el tiempo de ejecución.

\par Como segunda fase de la reimplementación de `mGSZ`, detectamos cuáles porciones de código de `MIGSAmGSZ` podrían ejecutarse en paralelo y otorgarían un _speedup_ considerable. Utilizando el paquete `BiocParallel`, se paralelizaron aquellas rutinas que consideramos pertinentes, y se brindó una sencilla interfaz de paralelismo al usuario. El análisis de la ganancia obtenida, gracias al paralelismo de `MIGSAmGSZ`, se puede replicar mediante el siguiente código:


```r
# Evaluamos MIGSAmGSZ con 1, 2, 4, 8, 10, 12 y 14 núcleos
nucleos <- c(1, 2, 4, 8, 10, 12, 14)
resultados <- lapply(nucleos, function(act_nucl) {
  # Mediante las siguientes 4 líneas de código se le indica
  # la cantidad de núcleos en las que debe ejecutarse MIGSAmGSZ.
  # El parámetro 'workers' indica la cantidad de núcleos a utilizar.
  register(MulticoreParam(
    workers = act_nucl, threshold = "DEBUG",
    progressbar = TRUE
  ))

  set.seed(8818)
  MIGSAmGSZ_tiempo <- system.time({
    MIGSAmGSZ_res <- MIGSAmGSZ(expresion, conj_genes, subtipos)
  })

  return(list(tiempo = MIGSAmGSZ_tiempo, res = MIGSAmGSZ_res))
})
```


```r
# Para cada ejecución conservamos el tiempo total ("elapsed") 
# y calculamos Speedup y Eficiencia sobre mGSZ
tiempos_segs <- sapply(resultados, function(act_res) {
  act_res$tiempo[["elapsed"]]
})
names(tiempos_segs) <- nucleos
metricas <- rbind(
  "Demora (mins)" = tiempos_segs / 60, # en minutos
  Speedup = mGSZ_tiempo / tiempos_segs,
  Eficiencia = (mGSZ_tiempo / tiempos_segs) / nucleos
)
round(metricas, 2)
```

```
                  1     2     4     8    10    12    14
Demora (mins) 93.21 46.50 24.98 15.63 13.67 14.79 28.43
Speedup        1.58  3.18  5.91  9.45 10.81  9.98  5.19
Eficiencia     1.58  1.59  1.48  1.18  1.08  0.83  0.37
```

\par Como puede observarse en la tabla superior, sin importar el número de núcleos en los que se haya corrido `MIGSAmGSZ`, su rendimiento fue superior al de `mGSZ`. Ejecutándose en un núcleo muestra un _speedup_ de 1,6X, alcanzando un máximo de 10,8X con diez núcleos. Gracias a la optimización desarrollada, es posible obtener en 14 minutos los mismos resultados que se obtenían en 2,46 horas de ejecución con el `mGSZ` original. Extrapolando a 118 experimentos, pasaríamos de una demora de 12,09 días, a tan solo 27,53 horas.

\par Como se mencionó previamente, la función optimizada y paralelizada `MIGSAmGSZ` se encuentra incluída en el paquete `MIGSA`, y se utiliza siguiendo los mismos parámtros que la versión original `mGSZ`.
