# TFM — Evaluación de tecnologías de IA explicable aplicables a sistemas de prevención de intrusiones

Máster en Ciberseguridad, Universidad de Alcalá. Autor: Agustín Casuso Cabarga.

## Estructura del repositorio

- `notebooks_replica/` — réplica de 5 arquitecturas de detección de la literatura sobre NSL-KDD, DS2OS y Mirai (15 notebooks) + réplicas sobre los datasets originales (OPC UA, IoT-23, Anexo A) + generación de `flows.csv` para Mirai.
- `notebooks_xai/` — comparación EBM vs Random Forest + SHAP, las 4 lentes de importancia, consistencia (Spearman), latencia, y el análisis de subconjuntos suficientes / atajo de identidad.
- `notebooks_aportaciones/` — las 3 aportaciones propias con código: clasificación jerárquica de dos niveles, simplificación de un EBM en reglas, EBM de interacciones expertas.
- `figuras/` — figuras usadas en la memoria.
- `entono_tfm.yml` — entorno conda con las versiones exactas usadas.
- Ficheros sueltos en la raíz (`latencia_*.json`, `xai_rankings_*.json`, `subsets_*.csv`, `una_variable_*.csv`) — resultados intermedios que alimentan tablas y figuras de la memoria; se regeneran solos si se borran.

## 1. Entorno

```bash
conda env create -f entono_tfm.yml
conda activate entorno_tfm
```

Todas las librerías van con versión fijada para que los resultados numéricos coincidan con los de la memoria. Si tienes GPU NVIDIA, quita la línea `cpuonly` del `.yml` antes de crear el entorno.

## 2. Datasets

Ninguno se incluye en el repositorio (pesan demasiado o tienen licencia de Kaggle). Los notebooks buscan cada fichero en varias rutas típicas por este orden: carpeta del propio notebook, `~/Downloads`, y la caché por defecto de `kagglehub` (`~/.cache/kagglehub/...`). Si no lo encuentran en ninguna, paran con un error explicando qué falta.

La forma más simple de tenerlos en la ruta correcta es descargarlos con `kagglehub` (deja el fichero cacheado exactamente donde los notebooks lo buscan):

```python
import kagglehub
kagglehub.dataset_download("hassan06/nslkdd")                 # NSL-KDD
kagglehub.dataset_download("libamariyam/ds2os-dataset")        # DS2OS
kagglehub.dataset_download("nguyenquanitmo/mirai-raw-dataset") # Mirai (capturas PCAP en bruto)
```

| Dataset | Origen | Notas |
|---|---|---|
| NSL-KDD | Kaggle `hassan06/nslkdd` | Se usa `KDDTrain+_20Percent.txt` |
| DS2OS | Kaggle `libamariyam/ds2os-dataset` | `DS2OS.csv` |
| Mirai | Kaggle `nguyenquanitmo/mirai-raw-dataset` | Capturas PCAP en bruto, **no** el CSV final (ver paso 3) |
| OPC UA (Anexo A) | R. Pinto, "M2M using OPC UA", IEEE Dataport, doi: 10.21227/ychv-6c68 | Solo para la réplica sobre el dataset original de Ghazi et al. |
| IoT-23 (Anexo A) | Stratosphere IPS / Zenodo, doi: 10.5281/zenodo.4743746 | Solo para la réplica sobre el dataset original de Nguyen Van et al.; se espera en `~/Downloads/iot_23_datasets_small/...` |

## 3. Generar `flows.csv` (Mirai)

Los notebooks de Mirai (excepto `D-PACK Mirai (Paper).ipynb`, que usa las capturas directamente) no leen el PCAP, leen `data/flows.csv`, un CSV de características de flujo agregadas. Para generarlo:

1. Instala [Wireshark](https://www.wireshark.org/) (se usa `editcap` para convertir `.pcapng` a `.pcap`).
2. Descarga el dataset de Kaggle `nguyenquanitmo/mirai-raw-dataset`.
3. Ejecuta `notebooks_replica/Generacion_Flujos_Mirai.ipynb` de principio a fin. Deja `data/flows.csv` listo (~85 MB) para el resto de notebooks. Solo hace falta ejecutarlo una vez.

## 4. Reproducibilidad

Todos los procesos aleatorios (splits, submuestreos, entrenamiento) usan semilla fija (42). Con el mismo entorno (`entono_tfm.yml`) y los mismos datasets, los notebooks deberían reproducir las cifras de la memoria. Aun así, entrenar los modelos de deep learning (D-PACK, Autoencoder) puede dar pequeñísimas variaciones entre CPU y GPU, ya que PyTorch no garantiza determinismo bit a bit entre distintos backends.
