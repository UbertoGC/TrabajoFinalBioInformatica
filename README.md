# Réplica de vcMSA y comparación con un alineador propio

Este repositorio contiene un cuaderno de Google Colab que reproduce una ejecución de **vcMSA**, método propuesto por McWhite, Armour-Garb y Singh (2023) para el alineamiento múltiple de secuencias de proteínas mediante representaciones generadas por un modelo de lenguaje.

Además, el cuaderno implementa un alineador múltiple propio basado en alineamientos globales por pares, una matriz de distancias, un árbol guía UPGMA y alineamiento progresivo. Ambos métodos se evalúan bajo las mismas condiciones.

## Contenido del repositorio

- `VCMSA_VS_PROPIO_NEO.ipynb`: instalación, configuración, ejecución, validación y comparación de vcMSA con el alineador propio.

## Objetivo

Replicar el funcionamiento de vcMSA en un entorno actual de Google Colab y compararlo con una implementación propia de alineamiento múltiple de secuencias, considerando:

- precisión mediante el **Total Column Score (TCS)**;
- conservación de secuencias y aminoácidos;
- tiempo de ejecución;
- consumo de RAM y VRAM.

## Datos utilizados

Se utiliza la familia `csp` del benchmark **QuanTest2**:

- 20 secuencias de proteínas de entre 61 y 69 aminoácidos;
- 3 secuencias con alineamiento estructural de referencia;
- 17 secuencias adicionales que aportan contexto durante la construcción del alineamiento.

Los dos métodos alinean las 20 secuencias. El TCS se calcula únicamente sobre las tres secuencias que cuentan con referencia estructural.

## Métodos comparados

### vcMSA

El cuaderno emplea el código oficial de vcMSA con parches mínimos de compatibilidad para el entorno actual de Colab. La configuración principal incluye:

- `Rostlab/prot_t5_xl_uniref50` para generar embeddings;
- concatenación de las últimas 16 capas, con 16 384 componentes por aminoácido;
- 10 caracteres `X` de padding en cada extremo;
- corrección por lote con ComBat;
- similitud coseno y búsqueda de vecinos con FAISS;
- RBH, MNCM y MIS para establecer correspondencias entre aminoácidos;
- construcción y llenado de las columnas del alineamiento.

ProtT5 se ejecuta con GPU. La búsqueda de similitud utiliza `faiss-cpu` por compatibilidad con las versiones actuales de Colab; este cambio afecta al dispositivo de ejecución, pero no sustituye la lógica del algoritmo.

### Alineador propio

La implementación desarrollada por el grupo realiza:

1. alineamientos globales por pares mediante programación dinámica tipo Needleman–Wunsch;
2. cálculo de una matriz de distancias entre secuencias;
3. construcción de un árbol guía mediante promedio de distancias, siguiendo el criterio UPGMA;
4. fusión progresiva de secuencias y grupos;
5. propagación de gaps para obtener el alineamiento múltiple final.

La configuración predeterminada utiliza `match = 1`, `mismatch = -1` y `gap = -1`.

## Ejecución en Google Colab

1. Abra `VCMSA_VS_PROPIO_NEO.ipynb` en Google Colab.
2. Seleccione **Entorno de ejecución → Cambiar tipo de entorno de ejecución → GPU T4**.
3. Ejecute todas las celdas en orden.
4. Espere la descarga y carga inicial de ProtT5; esta etapa puede tardar varios minutos.
5. Al finalizar, Colab descargará un archivo ZIP con las métricas, alineamientos, tiempos, versiones, parches, hashes y demás evidencias de la ejecución.

Las dependencias se instalan desde el propio cuaderno. No es necesario configurar previamente un entorno local.

## Resultados de la ejecución incluida

| Método | TCS | Columnas correctas | Recuperación de columnas de referencia | Ancho del alineamiento | Residuos preservados |
|---|---:|---:|---:|---:|:---:|
| vcMSA | 100.00 % | 70 de 70 | 100.00 % | 72 | Sí |
| Alineador propio | 76.32 % | 58 de 76 | 82.86 % | 85 | Sí |

### Tiempo y recursos

| Etapa | Tiempo | RAM máxima | VRAM máxima |
|---|---:|---:|---:|
| ProtT5 y generación de embeddings | 677.102 s | 9393.473 MB | 4804 MB |
| Alineamiento vcMSA | 33.851 s | 2204.387 MB | 0 MB |
| Alineador propio | 4.424 s | No registrada | No registrada |

El tiempo de la primera etapa incluye la descarga y carga inicial de ProtT5. La generación reportada internamente por vcMSA, una vez cargado el modelo, tomó aproximadamente 12.46 segundos para las 20 secuencias.

## Validaciones y evidencias

El cuaderno comprueba automáticamente que:

- estén presentes las 20 secuencias de entrada;
- todas las filas del alineamiento tengan la misma longitud;
- ningún método pierda, agregue o sustituya aminoácidos;
- las métricas se calculen sobre los identificadores presentes en la referencia estructural.

El paquete final de evidencias incluye alineamientos, métricas, tiempos, información del entorno, parches aplicados y hashes SHA-256. Los embeddings no se incluyen por defecto debido a su tamaño, pero se registra su hash para verificar su integridad.

## Alcance y limitaciones

Esta es una réplica experimental de **una familia** de QuanTest2. El artículo original evalúa 147 familias y varios métodos adicionales; por ello, los resultados obtenidos para `csp` no demuestran que vcMSA sea superior en todos los casos.

Asimismo, el entorno actual de Google Colab y su GPU T4 difieren del servidor con GPU A100 empleado en el trabajo original, por lo que los tiempos no deben compararse directamente sin considerar esa diferencia.

## Referencias

- McWhite, C. D., Armour-Garb, A. y Singh, M. (2023). [vcMSA: leveraging protein language models for multiple sequence alignment](https://pmc.ncbi.nlm.nih.gov/articles/PMC10538487/).
- [Repositorio oficial de vcMSA](https://github.com/clairemcwhite/vcmsa).
- [Repositorio de benchmarking y QuanTest2](https://github.com/clairemcwhite/nf-benchmark-vcmsa).
