# `ContourGenerator` API

## Тойм

`ContourGenerator` нь DEM растераас тогтмол босоо алхмаар (`interval_m`) хаялбарын шугамуудыг "marching squares" аргаар татаж гаргадаг класс юм. Энэ нь `POST /terrain/contours` (W05 API)-ийн цаадах хөдөлгүүр. Үүсгэхээс өмнө DEM-ийг шалгадаг тул хэт том эсвэл өгөгдөл дутуу DEM дээр тодорхой алдаа үүсгэнэ.

## Жишээ

```python
from geodesk import ContourGenerator
from geodesk.terrain import ElevationGrid

zaisan_dem = ElevationGrid.from_rows(
    "layer_dem_44c1",
    [
        [1350.0, 1351.0, 1352.0],
        [1351.0, 1352.0, 1353.0],
        [1352.0, 1353.0, 1354.0],
    ],
    cell_size_m=30.0,
    origin=(650000.0, 5310000.0),
)

generator = ContourGenerator(interval_m=1.0)
contour_layer = generator.generate(zaisan_dem)
print(len(contour_layer.lines))   # 7
print(generator.levels(zaisan_dem))  # [1351.0, 1352.0, 1353.0]
```

## Параметрүүд

`ContourGenerator(interval_m, *, max_size_mb=200.0, max_nodata_ratio=0.2)`

| Параметр | Төрөл | Өгөгдмөл | Тайлбар |
|---|---|---|---|
| `interval_m` | float | заавал | Хаялбарын хоорондын босоо зай (м). Төгсгөлөг, ≥ `MIN_INTERVAL_M` (0.5) |
| `max_size_mb` | float | 200.0 | Хүлээн авах DEM-ийн дээд хэмжээ (МБ), keyword-only |
| `max_nodata_ratio` | float | 0.2 | NoData нүдний дээд хувь (0–1), keyword-only |

`generate(grid, *, output_layer_id="layer_contour_9a21")`: `grid` нь `ElevationGrid`, `output_layer_id` нь буцах давхаргын ID.
`levels(grid)`: `grid` нь шалгах `ElevationGrid`.

## Буцаах утга

- `generate(...) -> ContourLayer`: оролтын `grid`-ийн CRS-д. Давхарга дахь `ContourLine` бүр нэг нүдний сегмент (2 цэг), холбогдсон polyline биш. Эмээлийн (4 ирмэг огтлолцсон) нүд 2 сегмент өгнө.
- `levels(grid) -> list[float]`: DEM-ийн өндрийн хүрээнд (хатуу дотор) орох хаялбарын өндрүүд, хамгийн багаас нь. Тэгш газар (нэг интервалаас бага зөрүүтэй) бол хоосон жагсаалт.

## Алдаанууд

| Алдаа | `error_code` | Хэзээ үүсэх |
|---|---|---|
| `InvalidIntervalError` (`TerrainError`, `ValueError`) | `INVALID_INTERVAL` | `ContourGenerator(...)` үүсгэх үед `interval_m` < 0.5, хязгааргүй эсвэл NaN |
| `DEMTooLargeError` (`TerrainError`) | `PAYLOAD_TOO_LARGE` | `generate()`: `grid.size_mb` > `max_size_mb` |
| `NoDataCoverageError` (`TerrainError`) | `INVALID_DEM` | `generate()`: NoData-ийн хувь > `max_nodata_ratio`. `levels()`: зөвхөн **бүх** нүд NoData үед |

## Хязгаарлалт

- Хамгийн бага алхам 0.5 м (FR-09-ийн ±0.5 м нарийвчлал).
- `size_mb` нь нэг нүдийг 4 байт (float32 GeoTIFF) гэж тооцсон **тооцоолсон** хэмжээ, бодит санах ой биш.
- NoData-д шүргэсэн нүдийг алгасдаг тул шугам өгөгдлийн цоорхой дээр тасална.
- Эмээлийн нүдэнд нүдний төвийн дундаж биш, тогтмол хослол ашиглана (TODO #1).
- `to_geojson()` координатыг grid-ийн CRS-д үлдээнэ, WGS84 руу хөрвүүлэхгүй (TODO #3).
- Гүйцэтгэл: docstring-д "лавлах машин дээр 50 MB DEM 8 секундээс бага" гэж заасан боловч хэмжээгүй, баталгаагүй.

---
*AI-assisted draft: Gemini, 2026-11-23. Prompt ба шалгалт: `w11-ai-verification-log.md` L6. Кодтой тулгасан, зассан: С.Сумъяабазар (шалгалтын хүснэгт болон хэмжилтэд Claude тусалсан).*
