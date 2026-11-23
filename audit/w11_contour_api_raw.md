<!-- Gemini түүхий гаралт, засаагүй. Part 1: Зориулалт ба Жишээ (w11_contour_api_raw.md) -->
## `ContourGenerator` API

### Зориулалт (Purpose)

`ContourGenerator` нь тогтмол босоо алхмаар (vertical interval) DEM растер өгөгдлөөс хаялбарын шугамуудыг (contour lines) "marching squares" аргаар татаж гаргадаг класс юм. Энэ нь `geodesk` багцын нийтийн API-ийн нэг хэсэг бөгөөд `POST /terrain/contours` W05 API-ийн цаадах үндсэн хөдөлгүүр юм. Үүсгэхээс өмнө оролтын DEM-ийг шалгадаг тул хэт том эсвэл гэмтсэн растер дээр гэнэт унахын оронд тодорхой алдаа үүсгэдэг. Мөн `levels(grid)` аргаар DEM-ийн өндрийн хүрээнд багтах хаялбарын түвшнүүдийг буцаадаг.

### Хэрэглээний жишээ (Usage Example)

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
print(len(contour_layer.lines))


<!-- Part 2: prompt ба үлдсэн гаралт (w11_contour_api_raw.txt) -->
Role: You are a software documentation engineer.

Context: Below is the full source code of the Python package "geodesk"
(contour line generation module).

Task: Write the API section of a README for ContourGenerator: purpose,
a short usage example, parameters, return value, errors it raises, and limits.

Format: Markdown, in Mongolian. 

Constraint: Use ONLY facts visible in the code above. Do not invent default
values, limits, performance numbers or supported formats.  Every code example must run against this code. At the end, list the 5 claims you are least sure about.

Параметрүүд (Parameters)ContourGenerator.__init__interval_m: float — Хаялбарын шугамууд хоорондын босоо зай (метрээр). Төгсгөлөг тоо бөгөөд MIN_INTERVAL_M (0.5 м)-ээс багагүй байх ёстой.   max_size_mb: float = 200.0 (keyword-only) — Хүлээн авах хамгийн том DEM-ийн хэмжээ (мегабайтаар; анхдагч утга 200.0, docstring-д 200 гэж бичигдсэн).   max_nodata_ratio: float = 0.2 (keyword-only) — Хүлээн авах NoData нүдний хамгийн их хувь, 0-ээс 1 хооронд (анхдагч утга 0.2).   ContourGenerator.generategrid: ElevationGrid — Хаялбар татах DEM тор.   output_layer_id: str = "layer_contour_9a21" (keyword-only) — Буцаагдах давхаргад өгөх ID.   ContourGenerator.levelsgrid: ElevationGrid — Шалгах DEM тор.   Буцаах утга (Return Value)generate(grid, *, output_layer_id="layer_contour_9a21") -> ContourLayer — Торны огтлолцол бүрд нэг ContourLine байхаар үүсгэсэн ContourLayer объектыг оролтын grid-ийн CRS-ээр буцаана.   levels(grid) -> list[float] — DEM-ийн өндрийн хүрээнд багтах хаялбарын өндрүүдийг (метрээр, хамгийн багаас нь эхлэн) list[float] хэлбэрээр буцаана. Тэгш газар (нэг интервалаас бага өндрийн зөрүүтэй) бол хоосон жагсаалт буцаана.   Үүсгэх алдаанууд (Errors Raised)InvalidIntervalError (TerrainError, ValueError өвлөнө) — ContourGenerator(...) дуудах үед interval_m нь 0.5-аас бага, хязгааргүй (inf/-inf) эсвэл NaN бол үүснэ (error_code = "INVALID_INTERVAL").   DEMTooLargeError (TerrainError өвлөнө) — generate() дуудах үед grid.size_mb нь max_size_mb-аас хэтэрвэл үүснэ (error_code = "PAYLOAD_TOO_LARGE").   NoDataCoverageError (TerrainError өвлөнө) — generate() эсвэл levels() дуудах үед grid-ийн бүх нүд NoData бол (valid_range() дотор) эсвэл NoData-ийн эзлэх хувь max_nodata_ratio-аас хэтэрвэл үүснэ (error_code = "INVALID_DEM").   Хязгаарлалтууд (Limits)Хамгийн бага алхам (MIN_INTERVAL_M): 0.5 м (FR-09 нь ±0.5 м нарийвчлал баталгаажуулдаг тул үүнээс нарийн алхам зөвшөөрөхгүй).   DEM хэмжээний дээд хязгаар: Анхдагчаар 200.0 MB (max_size_mb), size_mb нь нэг нүдийг 4 байт (_BYTES_PER_CELL = 4, float32 GeoTIFF тооцоолол) гэж үзэж мегабайтаар тооцно.   NoData нүдний хувийн дээд хязгаар: Анхдагчаар 0.2 (20%, max_nodata_ratio). NoData нүд шүргэсэн торны нүднүүдийг алгасаж, шугамыг өгөгдлийн цоорхой дээр тасална.   Дөрвөн ирмэг огтлолцсон тохиолдолд (4 edge crossings) нүдний төвийн дунджаар биш, тогтмол хослолоор (north-west, south-east) сегмент үүсгэнэ (TODO(#1)).   ContourLayer.to_geojson() нь координатуудыг grid-ийн CRS хэвээр үлдээдэг бөгөөд WGS84 (EPSG:4326) руу хөрвүүлэхгүй (TODO(#3)).   Гүйцэтгэлийн дурдсан утга: Лавлах машин дээр 50 MB DEM нь 8 секундийн дотор дуусна гэж generate docstring-д заасан (FR-09 verification).   Хамгийн эргэлзээтэй 5 дүгнэлт (5 Claims Least Sure About)ContourGenerator.generate() нь бүх тохиолдолд ElevationGrid дахь бүх NoData нүдийг nodata_ratio()-оор шалгасны дараа _trace_level() дотор дахин нүд бүрээр _is_nodata()-аар шалгаж давхар ажилладаг ч энэ нь бодит ажиллагаанд ямар нөлөөтэйг код доторх нэмэлт тайлбаргүйгээр зөвхөн урсгалаар нь дүгнэв.   levels() доторх first_level = math.floor(lowest / self.interval_m + 1) * self.interval_m болон [round(..., 6) ...] томьёонууд хөвөгч таслалын (float) зарим ирмэгийн утга дээр яг хэрхэн ажиллахыг бүх боломжит тоон утгаар шалгаагүй[cite: 5].ContourGenerator нь geodesk.terrain дотор тодорхойлогдсон бөгөөд geodesk/__init__.py дээр импортлогдож __all__-д орсон тул from geodesk import ContourGenerator гэж дуудагддаг ч бусад дэд модулиудын гадаад хамаарал хэрхэн холбогддог нь бүрэн харагдахгүй[cite: 3, 5].max_size_mb нь grid.size_mb (row_count * column_count * 4 / 1_000_000) шинж чанартай харьцуулагддаг тул бодит санах ойн хэмжээ биш, зөвхөн торны нүдийн тоонд суурилсан тооцоолсон хэмжээ болохыг код дээрх харьцуулалтаас уншиж дүгнэв[cite: 5].generate() docstring дээр 50 MB DEM-ийг 8 секундийн дотор гүйцэтгэнэ гэсэн нь кодын дотоод хэмжилт бус, баримт бичгийн (FR-09 verification) заалтаас иш татсан мэдээлэл юм[cite: 5].