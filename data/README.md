# Data data

Data is stored locally and excluded from Git. Scripts create `raw/` for initial downloads, `interim/` for normalized observations and `samples/` for research sampling.

## fire-aoi/

Competitive monitoring area from [DATA monitoring](https://disk.yandex.ru/d/-rpmevTflbXZQg):

- `fire_monitoring_aoi.geojson` - AOI "Lower Volga and Podonie" + zones UTM 37N/38N (in git))
- map copy: `frontend/public/data/fire_monitoring_aoi.geojson`
- locally (not in git): `.gpkg`, `_shapefile.zip`

Regions in UI/API: IDF subjects (Rostov, Volgograd, Astrakhan, Saratov, Kalmykia) + demo scenario of Siberia.

Update:

```bash
bash scripts/download_monitoring_data.sh          # AOI
bash scripts/download_monitoring_data.sh --with-tars
python3 scripts/ingest_monitoring_chips.py        # footprints train → карта
```

On the map: a layer of “dataset chips (train)” – footprints with real UTM binding.
Test chips without geo-reference are not placed on the map.

## yandex/

Here add `fire-test-renamed.tar`/`fire-train-renamed.tar` (not in git)):

```bash
bash scripts/download_monitoring_data.sh --with-tars
```

This is an AF/BS ML chip, not a live card layer. Only `fire-aoi` is transferred to the service map.

## Official train/test (locally, not in git))

The ML code and service are waiting for this layout (paths are configured by the `KROMA_AF_ROOT`, `KROMA_BS_ROOT` environment variables)):

```text
data/Мониторинг DATA/train/af/{meta.csv,viirs/,aux/,masks/}   # 420 AF-чипов
data/Мониторинг DATA/train/bs/{meta.csv,sentinel2_pre/,sentinel2_post/,sentinel1_pre/,sentinel1_post/,aux/,masks/}   # 224 BS-чипа
data/test/{meta.csv,sample_submission.csv,af/,bs/}            # 180 AF + 89 BS, без меток и без геопривязки
```

Test chips are not georeferenced or shown on a map; train chips have UTM binding in `meta.csv`.
