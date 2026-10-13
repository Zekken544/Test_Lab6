# Лаб 06 — Category-Partition ба PICT

**М. Билгүүн-Эрдэнэ — B222270046**

## Орчин

Windows, WSL2, Ubuntu 26.04.1 LTS.

PICT-ийг `~/tools/pict` хавтаст эх кодоос CMake, g++ ашиглан бүтээсэн. Эх код болон build хавтсыг лабын репод оруулаагүй.

[PICT бүтээсэн нотолгоо](results/pict-build.txt)

## A1 — Шилжүүлгийн оролтын ангилал

`POST /transfer` функцийн тестийг загварчлав. Бодит сервер өгөөгүй тул хариунууд нь ажиллуулсан үр дүн биш, хүлээлт юм.

Илгээгчийн валют USD, үлдэгдэл 1000 USD, өдрийн нийт лимит 1500 USD гэж авсан. Туршилтын ханш **1 EUR = 2 USD**. Үлдэгдэл, лимитийг хөрвүүлсний дараах дүнгээр шалгана.

| Сонголт | Утгууд | Төлөөлөх өгөгдөл |
|---|---|---|
| From | Valid, Frozen, Missing | Идэвхтэй, царцаагдсан, үүсгээгүй данс |
| To | Valid, Missing, SameAsFrom | Өөр байгаа данс, үүсгээгүй данс, илгээгчтэй ижил ID |
| Amount | Negative, Zero, Normal, OverBalance, OverLimit | Хөрвүүлсний дараах −1, 0, 100, 1200 USD, үлдсэн лимит + 100 USD |
| Currency | Same, Different | USD, EUR |
| Daily | Unused, PartUsed | Өмнө нь 0 эсвэл 1000 USD шилжүүлсэн; 1500 эсвэл 500 USD лимит үлдсэн |

Daily нь хүсэлтийн талбар биш боловч үр дүнд нөлөөлөх далд орчны сонголт юм.

Олон алдаа зэрэг байвал дараах эрэмбийг сонгов:

`ERROR_NO_FROM → ERROR_FROZEN → ERROR_NO_TO → ERROR_SAME_ACCOUNT → ERROR_BAD_AMOUNT → ERROR_INSUFFICIENT → ERROR_LIMIT`.

Эхлээд дансны хүчинтэй байдал, дараа нь мөнгөн дүнг шалгах нь ойлгомжтой гэж үзсэн. Энэ эрэмбийг тодорхойлолтод заагаагүй тул өөрийн таамаглал болгон хэрэглэв.

## A2 — Хязгаарлалт ба тооцоо

| Утга / нөхцөл | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| From: Valid | `[property: active]` | Хэвийн шилжүүлэгт идэвхтэй илгээгч хэрэгтэй. |
| From: Frozen, Missing | `[ERROR]` | Тус бүрийг бусад оролт зөв үед шалгана. |
| To: Valid | `[property: differentAccount]` | Байгаа бөгөөд илгээгчээс өөр данс байна. |
| To: Missing, SameAsFrom | `[ERROR]` | Хүчинтэй илгээгчтэйгээр алдааг тусгаарлана. |
| To: SameAsFrom | `[IF fromExists]` | Ижил дансны тестэд илгээгч данс байгаа байна. |
| Amount: Normal | `[property: positiveWithinLimits]` | 100 USD нь хоёр лимитийн төлөвт багтана. |
| Amount: Negative, Zero | `[ERROR]` | amount > 0 дүрмийг зөрчинө. |
| Amount: OverBalance | `[ERROR]`, `[IF Daily=Unused]` | 1200 USD нь зөвхөн үлдэгдлийг хэтрүүлнэ. |
| Amount: OverLimit | `[ERROR]`, `[IF Daily=PartUsed]` | 600 USD нь зөвхөн үлдсэн лимитийг хэтрүүлнэ. |
| Currency: Same, Different | `[property: noConversion / conversion]` | Өөр валютыг USD-д хөрвүүлж шалгана. |
| Daily: Unused, PartUsed | `[property: remaining1500 / remaining500]` | Үлдсэн лимитийг тодорхойлно. |
| Яг үлдэгдэлтэй тэнцүү 1000 USD | `[SINGLE]` | Лимит 1500 үед зөвшөөрөгдөх хилийг шалгана. |
| Яг үлдсэн лимиттэй тэнцүү 500 USD | `[SINGLE]` | Үлдэгдэл 1000 үед лимитийн хилийг шалгана. |

Нэг алдааг тусгаарлах багцад дараах дүрэм хэрэглэв:

- From буруу бол To=Valid, Amount=Normal, Currency=Same, Daily=Unused.
- To буруу бол From=Valid, Amount=Normal, Currency=Same, Daily=Unused.
- Amount буруу бол From=Valid, To=Valid, Currency=Same.
- Negative, Zero, OverBalance үед Daily=Unused; OverLimit үед Daily=PartUsed.
- Алдаагүй үед Currency, Daily-ийн бүх хослолыг үлдээнэ.

Эдгээр нь тест сонгох дүрэм болохоос бусад хослол бодит амьдралд боломжгүй гэсэн үг биш.

**Бүрэн үржвэр:** `3 × 3 × 5 × 2 × 2 = 180`.

**Хязгаарлалтын дараа:**

- Алдаагүй: `1 × 1 × 1 × 2 × 2 = 4`.
- From алдаа: 2.
- To алдаа: 2.
- Amount алдаа: 4.

Ингэснээр **180 → 12** хослол болно. Үндсэн үржвэрт ороогүй хоёр SINGLE хил дээрх тест болон нэг давхар алдааны тест нэмээд **15 спецификаци** гаргав. Энэ нь PICT-ийн гаралтын мөрийн тоо биш.

## A3 — Тестийн спецификаци

Unused үед 1500 USD, PartUsed үед 500 USD лимит үлдсэн байна. EUR-ийн 50 нэгж нь 100 USD-тэй тэнцэнэ.

| № | From | To | amount | currency | Daily | Хүлээгдэх хариу |
|---|---|---|---:|---|---|---|
| 1 | Valid | Valid | 100 | USD | Unused | 201 OK |
| 2 | Valid | Valid | 50 | EUR | Unused | 201 OK |
| 3 | Valid | Valid | 100 | USD | PartUsed | 201 OK |
| 4 | Valid | Valid | 50 | EUR | PartUsed | 201 OK |
| 5 | Frozen | Valid | 100 | USD | Unused | 200 ERROR_FROZEN |
| 6 | Missing | Valid | 100 | USD | Unused | 200 ERROR_NO_FROM |
| 7 | Valid | Missing | 100 | USD | Unused | 200 ERROR_NO_TO |
| 8 | Valid | SameAsFrom | 100 | USD | Unused | 200 ERROR_SAME_ACCOUNT |
| 9 | Valid | Valid | -1 | USD | Unused | 200 ERROR_BAD_AMOUNT |
| 10 | Valid | Valid | 0 | USD | Unused | 200 ERROR_BAD_AMOUNT |
| 11 | Valid | Valid | 1200 | USD | Unused | 200 ERROR_INSUFFICIENT |
| 12 | Valid | Valid | 600 | USD | PartUsed | 200 ERROR_LIMIT |
| 13 | Valid | Valid | 1000 | USD | Unused | 201 OK |
| 14 | Valid | Valid | 500 | USD | PartUsed | 201 OK |
| 15 | Frozen | Missing | 100 | USD | Unused | 200 ERROR_FROZEN |

Valid данснууд нь байгаа, хоорондоо өөр данс. Missing нь үүсгээгүй данс, SameAsFrom нь илгээгчтэй ижил ID байна.

13, 14-р мөр нь `[SINGLE]` тест: дүн хязгаартай яг тэнцүү тул зөвшөөрөгдөнө. 15-р мөр хоёр алдаатай боловч сонгосон эрэмбээр ERROR_FROZEN түрүүлж буцна.

## B1 — Браузерын тохиргоо

Бүх боломжит хослол: `3 × 3 × 2 × 2 × 2 × 2 = 144`.

```bash
pict browser.model > results/browser-2way.txt
pict browser.model /o:3 > results/browser-3way.txt
```

| Арга | Тестийн мөр |
|---|---:|
| Бүх хослол | 144 |
| PICT 2-way | 10 |
| PICT 3-way | 20 |

Толгой мөрийг тоолоогүй, `/r` ашиглаагүй.

PICT нь лекцийн 9 мөрөөс нэгээр олон буюу 10 мөр гаргасан. Greedy алгоритм нь бүх хосыг хамрах боловч хамгийн цөөн мөрийг заавал олдоггүй. 3-way нь гурван тохиргооны утгын хослолуудыг хамарч, 20 мөр гарсан нь бүрэн 144 хослолоос бага байна.

- [2-way гаралт](results/browser-2way.txt)
- [3-way гаралт](results/browser-3way.txt)

## B2 — Шилжүүлгийн PICT модель

```bash
pict transfer-nc.model > results/transfer-nc.txt
pict transfer.model > results/transfer.txt
pict transfer-neg.model > results/transfer-neg.txt
```

| Модель | Мөрийн тоо | Happy path мөр | Нэг мөр дэх алдаатай сонголтын тоо |
|---|---:|---:|---|
| transfer-nc | 16 | 0 | 1–3 |
| transfer | 10 | 2 | 0–1 |
| transfer-neg | 20 | 4 | 0–1 |

Толгой мөрийг тоолоогүй. Happy path гэдэг нь From=Valid, To=Valid, Amount=Normal мөр юм. Алдаатай сонголтыг From, To, Amount баганын буруу утгаар тоолсон; энэ нь серверийн буцаах алдааны тоо биш.

Хязгаарлалтгүй модель олон алдааг нэг мөрөнд оруулсан бөгөөд happy path гараагүй. Эхний алдаа дараагийн алдааг нуух тул ийм мөрөөр бүх алдааг тус тусад нь батлах боломжгүй. Жишээлбэл, OverBalance + PartUsed нь үлдэгдэл болон лимитийг хоёуланг хэтрүүлж болно.

A2-ын 12 нь зөвшөөрөгдөх бүрэн хослолын тоо, харин PICT-ийн 10 нь тэдгээрийн хосын хамралтад сонгосон мөрийн тоо юм. Иймээс эдгээр тоо заавал тэнцэхгүй.

`~` тэмдэглэгээ нэг мөрөнд хамгийн ихдээ нэг сөрөг сонголт оруулсан. Гэхдээ OverBalance үед Daily=Unused, OverLimit үед Daily=PartUsed гэсэн хязгаарлалтууд хэрэгтэй хэвээр: нэг сөрөг сонголт санхүүгийн хоёр дүрмийг зэрэг зөрчиж болохоос сэргийлнэ. Currency-г зөвхөн Same болгох зэрэг бусад тусгаарлах дүрмийг negative модельд сулруулж, алдаатай утгыг өөр валюттай мөн хослуулсан.

### Гаралтаас сонгосон гурван бодит тест кейс

Мөрийн дугаарт толгой мөрийг оруулаагүй. Эдгээр нь ажиллуулахад бэлэн тестийн загвар бөгөөд бодит API дээр ажиллуулаагүй.

| Кейс / эх мөр | Setup | POST /transfer body | Oracle |
|---|---|---|---|
| Happy path — transfer.txt, мөр 2 | A идэвхтэй, 1000 USD үлдэгдэлтэй; B байгаа өөр данс; өдрийн лимит 500 USD үлдсэн; 1 EUR=2 USD | `{"from":"A","to":"B","amount":50,"currency":"EUR"}` | 50 EUR=100 USD тул 201, `result="OK"` |
| Нэг алдаа — transfer-neg.txt, мөр 5 | A идэвхтэй, 1000 USD үлдэгдэлтэй; B байгаа; өдрийн лимит 1500 USD үлдсэн; 1 EUR=2 USD | `{"from":"A","to":"B","amount":600,"currency":"EUR"}` | 600 EUR=1200 USD нь үлдэгдлээс их боловч лимитэд багтана: 200, `result="ERROR_INSUFFICIENT"` |
| Олон алдаа — transfer-nc.txt, мөр 14 | A царцаагдсан, 1000 USD үлдэгдэлтэй; NO_TO байхгүй; өдрийн лимит 1500 USD үлдсэн | `{"from":"A","to":"NO_TO","amount":0,"currency":"USD"}` | Frozen, Missing, Zero гэсэн гурван буруу сонголтоос тогтоосон эрэмбээр 200, `result="ERROR_FROZEN"` |

### Нотолгоо

- [Хязгаарлалтгүй гаралт](results/transfer-nc.txt)
- [Хязгаарлалттай гаралт](results/transfer.txt)
- [Сөрөг утгатай гаралт](results/transfer-neg.txt)

## Дүгнэлт

Category-partition аргаар оролтыг утгатай ангилалд хувааж, хүлээгдэх алдааг тодорхойлов.
Хамгийн их бодол шаардсан зүйл нь үлдэгдэл болон өдрийн лимитийн алдааг тусгаарлах байсан.
Гараар бичсэн хүснэгт нь хил дээрх утга болон алдааны утга агуулгыг тодорхой харуулсан.
PICT нь моделийн зөвшөөрсөн хосуудыг автоматаар хамруулж өгсөн.
Гэхдээ greedy алгоритм хамгийн цөөн мөрийг заавал гаргахгүй.
Хязгаарлалтгүй гаралтад олон алдаа давхцаж, happy path ч гараагүй.
Сөрөг утгын тэмдэглэгээ алдааг тусгаарласан боловч санхүүгийн нөхцөлийн хязгаарлалт шаардлагатай хэвээр байв.
Pairwise хамралт нь бүх бүрэн хослол болон бүх согогийг шалгасан гэсэн үг биш.
Иймээс гараар сонгосон хил дээрх тестүүдийг PICT-ийн хосын хамралттай хамт ашиглах нь тохиромжтой гэж үзэв.