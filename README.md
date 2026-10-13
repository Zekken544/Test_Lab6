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