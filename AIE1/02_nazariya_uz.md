# Support Ticket Router uchun nazariya

Bu hujjat loyihani bajarish uchun kerak bo'ladigan barcha tushunchalarni, loyihaning mantiqiga mos tartibda tushuntiradi. Har bir bo'limda: **nima**, **nima uchun kerak**, **misol**. Oxirida pseudocode va murakkablik (complexity) tahlili bor.

## Mundarija

1. [Loyihaning g'oyasi: nima uchun baseline kerak](#1-loyihaning-goyasi-nima-uchun-baseline-kerak)
2. [Rule-based tizim nima](#2-rule-based-tizim-nima)
3. [Python asoslari: ro'yxat, lug'at, shart, sikl](#3-python-asoslari-royxat-lugat-shart-sikl)
4. [Satrlar bilan ishlash va normalizatsiya](#4-satrlar-bilan-ishlash-va-normalizatsiya)
5. [JSON formati va `json` moduli](#5-json-formati-va-json-moduli)
6. [CSV formati va `csv` moduli](#6-csv-formati-va-csv-moduli)
7. [Yagona ichki ko'rinish va validatsiya](#7-yagona-ichki-korinish-va-validatsiya)
8. [Funksiyalar va mas'uliyatni ajratish](#8-funksiyalar-va-masuliyatni-ajratish)
9. [Xatolar bilan ishlash: exception, try/except](#9-xatolar-bilan-ishlash-exception-tryexcept)
10. [CLI va `argparse`](#10-cli-va-argparse)
11. [Exit code va `traceback`](#11-exit-code-va-traceback)
12. [Fayl yo'llari va `pathlib`](#12-fayl-yollari-va-pathlib)
13. [Git ish jarayoni (workflow)](#13-git-ish-jarayoni-workflow)
14. [Qo'lda qabul tekshiruvi (manual acceptance)](#14-qolda-qabul-tekshiruvi-manual-acceptance)
15. [Pseudocode](#15-pseudocode)
16. [Murakkablik tahlili](#16-murakkablik-tahlili)
17. [Keng tarqalgan xatolar](#17-keng-tarqalgan-xatolar)
18. [P2P'ga tayyorgarlik savollari](#18-p2pga-tayyorgarlik-savollari)

---

## 1. Loyihaning g'oyasi: nima uchun baseline kerak

**Baseline** — masalaning eng oddiy, lekin ishlaydigan yechimi. Keyingi barcha murakkab yechimlar (masalan, LLM) shu yechim bilan **taqqoslanadi**.

Nima uchun bu muhim? Tasavvur qiling, LLM asosidagi tizim 90% to'g'ri javob beradi. Bu yaxshimi? Agar oddiy qoidalar ham 88% bersa, LLM'ning narxi, sekinligi va noaniqligi o'zini oqlamaydi. Agar qoidalar 40% bersa, LLM juda foydali. Baseline'siz bu savolga javob yo'q.

| Savol | Baseline'siz | Baseline bilan |
|---|---|---|
| LLM yaxshimi? | Bilmaymiz | "Qoidalardan +X% yaxshi" |
| Xato qayerda? | Noma'lum | Aniq so'z va qoida ko'rsatadi |
| Narxi arziydimi? | Bilmaymiz | Farqni pul bilan o'lchaymiz |

Baseline'ning asosiy xususiyati — **tushuntirib bo'lish (explainability)**: har bir qaror "bu so'z topildi, bu qoida ishladi" deb izohlanadi.

## 2. Rule-based tizim nima

Rule-based (qoidalarga asoslangan) tizim — qarorlar oldindan yozilgan **if-then** qoidalari bilan qabul qilinadigan tizim. Bu loyihada ikki turdagi qoida bor:

1. **Kategoriya qoidasi:** matnda ma'lum kalit so'z (keyword) bor bo'lsa → shu kategoriya.
2. **Ustuvorlik qoidasi:** matnda shoshilinchlik belgisi bor bo'lsa → `high`; bo'lmasa kategoriyaga qarab `medium` yoki `low`.

### Tartib muhim: first-match-wins

Bitta matnda bir nechta kategoriya belgisi bo'lishi mumkin:

> "To'lov o'tdi, lekin parolim ishlamayapti"

Bu yerda ham billing, ham access belgilari bor. Qaysi biri tanlanadi? Loyiha qoidasi: **ma'lumotnomada birinchi yozilgan kategoriya**, ya'ni ro'yxatni yuqoridan pastga yuramiz va birinchi mos kelganida to'xtaymiz.

```
for kategoriya in [kategoriya_1, kategoriya_2, kategoriya_3]:
    if kategoriyaning biror kalit so'zi matnda bor:
        return kategoriya        # to'xtaymiz
return "other"                   # hech biri mos kelmadi
```

Muhim: tartibni **ma'lumotnoma (`routing-rules.md`) dagi tartibda** saqlash kerak. Tartib o'zgarsa, natija ham o'zgaradi.

### Nima uchun id bo'yicha tayyor javob saqlash mumkin emas

Vazifada aytilgan: dastur har bir yozuv uchun yo'lni **hisoblashi** kerak. `{"r-001": "access"}` kabi lug'at — bu "ko'chirish", marshrutizator emas. P2P'da tekshiruvchi yangi yozuv qo'shadi va sizning dasturingiz uni qayta ishlay olishi kerak.

## 3. Python asoslari: ro'yxat, lug'at, shart, sikl

### Ro'yxat (list) — tartiblangan to'plam

```python
tickets = ["a", "b", "c"]
tickets.append("d")        # oxiriga qo'shish
len(tickets)               # 4
for t in tickets:          # aylanish
    print(t)
```

### Lug'at (dict) — kalit → qiymat

```python
record = {"id": "r-001", "text": "Не могу войти"}
record["id"]               # "r-001"
record.get("priority")     # None  (kalit yo'q bo'lsa xato bermaydi)
record.get("priority", "") # ""    (standart qiymat)
"text" in record           # True
```

`record["priority"]` — kalit yo'q bo'lsa `KeyError` beradi. `record.get(...)` — bermaydi. Foydalanuvchi ma'lumotlari bilan ishlaganda `.get()` xavfsizroq.

### Lug'atlar ro'yxati — loyihaning asosiy tuzilmasi

```python
records = [
    {"id": "r-001", "text": "..."},
    {"id": "r-002", "text": "..."},
]
```

### Shartlar va `any`

```python
keywords = ["войти", "пароль"]
text = "не могу войти"

# Uzun usul
found = False
for kw in keywords:
    if kw in text:
        found = True
        break

# Qisqa usul
found = any(kw in text for kw in keywords)
```

`any(...)` — kamida bittasi `True` bo'lsa `True` qaytaradi va birinchi moslikda to'xtaydi (**short-circuit**).

### Tuple ro'yxati tartibni saqlaydi

```python
CATEGORY_RULES = [
    ("access",  ["войти", "пароль", "нет доступа"]),
    ("billing", ["оплата", "счет", "тариф", "спис"]),
]
for name, words in CATEGORY_RULES:
    ...
```

Eslatma: Python 3.7+ da oddiy `dict` ham qo'yilish tartibini saqlaydi, lekin tartib muhim bo'lgan joyda ro'yxat (list of tuples) niyatni aniqroq ko'rsatadi.

## 4. Satrlar bilan ishlash va normalizatsiya

**Normalizatsiya** — matnni yagona ko'rinishga keltirish, shunda "Счёт", "счет", "СЧЁТ" bir xil deb hisoblanadi.

```python
def normalize(text):
    return text.lower().replace("ё", "е")
```

| Kirish | Chiqish |
|---|---|
| `"Не могу ВОЙТИ"` | `"не могу войти"` |
| `"Закрыли СЧЁТ"` | `"закрыли счет"` |

Muhim qoidalar:

1. **Ikkala tomonni** normalizatsiya qiling: matnni ham, kalit so'zlarni ham. Aks holda kalit so'zda `ё` qolsa, hech qachon mos kelmaydi.
2. Satrlar **o'zgarmas (immutable)**: `text.lower()` yangi satr qaytaradi, `text` o'zi o'zgarmaydi. Natijani o'zgaruvchiga saqlash kerak.
3. `.strip()` — chekkalardagi bo'sh joy, `\n`, `\t` ni olib tashlaydi (o'rtadagisini emas).
4. `in` operatori **qism satr** qidiradi:

```python
"парол" in "забыл пароль"   # True
"парол" in "паролей нет"    # True
```

Bu qulaylik ham, xavf ham: `"счет"` so'zi `"обсчетом"` ichida ham topiladi. Baseline'ning bu cheklovi ataylab qabul qilingan — keyingi loyihalarda aynan shu kamchiliklar tahlil qilinadi.

### Satrlarni ma'lumotnomadagidek ishlating (ildizga qisqartirmang)

Rus tilida so'zlar o'zgaradi: *пароль, пароля, паролем*. `парол` ildizini olsak, hammasi topilardi, lekin bu loyihada **ma'lumotnoma satrlari aynan qanday yozilgan bo'lsa, shunday** ishlatiladi (`пароль`, `нет доступа`, `спис`, ...). Ma'lumotnomada "qo'shimcha qoidalar o'ylab topma" deyilgan: o'zboshimchalik bilan ildizga qisqartirish yangi qoida qo'shish bo'ladi va natija `expected-results.json` dan farq qilishi mumkin. Baseline'ning bu cheklovini (masalan, `пароля` matnida `пароль` topilmaydi) bilish va P2P'da aytib berish foydali.

## 5. JSON formati va `json` moduli

**JSON** — ma'lumotlarni matn ko'rinishida saqlash formati. Python bilan mos keladi:

| JSON | Python |
|---|---|
| `{ }` obyekt | `dict` |
| `[ ]` massiv | `list` |
| `"matn"` | `str` |
| `42`, `3.14` | `int`, `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

```python
import json

# O'qish
with open("requests.json", encoding="utf-8") as f:
    data = json.load(f)       # fayl obyektidan

# Yozish
with open("result.json", "w", encoding="utf-8") as f:
    json.dump(results, f, ensure_ascii=False, indent=2)
```

Parametrlar:
- `encoding="utf-8"` — kirill va o'zbek harflari uchun shart;
- `ensure_ascii=False` — kirillcha matn `\u0432...` ko'rinishida emas, o'qiladigan holda yoziladi;
- `indent=2` — chiroyli, odam o'qiy oladigan format.

### `with` bloki nima uchun

`with open(...) as f:` blok tugaganda faylni **avtomatik yopadi**, xato bo'lsa ham. Qo'lda `f.close()` yozish esa xavfli (xato bo'lsa chaqirilmay qolishi mumkin).

### Buzilgan JSON

```python
json.loads("[{'id': 1}")     # json.JSONDecodeError
```

`JSONDecodeError` ob'ektida `.lineno`, `.colno`, `.msg` bor — foydalanuvchiga "N-qatorda xato" deb aytish mumkin.

### Ildiz massiv tekshiruvi

`json.load` har qanday JSON'ni o'qiydi: ro'yxat ham, lug'at ham, son ham. Dastur esa **ro'yxat** kutadi. Shuning uchun o'qigandan keyin `isinstance(data, list)` ni tekshirish kerak.

## 6. CSV formati va `csv` moduli

**CSV** — jadval ma'lumotlari: har qator bitta yozuv, qiymatlar vergul bilan ajratilgan, birinchi qator — sarlavhalar.

```csv
id,text
r-001,Не могу войти в личный кабинет
r-002,"Платеж прошел, но деньги не появились"
```

Ikkinchi qatorda matn ichida vergul bor, shuning uchun u **qo'shtirnoq** ichida. Shu sababli `line.split(",")` **ishlamaydi** — `csv` modulidan foydalaning.

```python
import csv

with open("requests.csv", encoding="utf-8-sig", newline="") as f:
    reader = csv.DictReader(f)
    rows = list(reader)
# rows[0] -> {"id": "r-001", "text": "Не могу войти в личный кабинет"}
```

Nuqtalar:
- `csv.DictReader` birinchi qatorni sarlavha deb oladi va har qatordan **lug'at** yasaydi — bu aynan bizga kerak bo'lgan shakl.
- `newline=""` — CSV moduli qator oxirlarini o'zi to'g'ri boshqarishi uchun tavsiya etiladi (matn ichida yangi qator bo'lsa ham).
- `utf-8-sig` — Excel saqlagan fayllar boshida BOM belgisi bo'ladi. Oddiy `utf-8` bilan o'qisangiz, birinchi sarlavha `"\ufeffid"` bo'lib qoladi va `id` topilmay qoladi. `utf-8-sig` BOM'ni avtomatik olib tashlaydi, BOM bo'lmasa ham ishlaydi.
- `DictReader` barcha qiymatlarni **satr** sifatida o'qiydi (`"42"`, `42` emas). Qatorda ustunlar yetishmasa, qiymat `None` bo'ladi — shuning uchun keyingi validatsiya muhim.

### JSON va CSV farqi (bu loyiha uchun)

| | JSON | CSV |
|---|---|---|
| Tuzilma | ichma-ich bo'lishi mumkin | tekis jadval |
| Turlar | son, satr, null... | hammasi satr |
| `id` | `42` (son) bo'lishi mumkin | har doim `"42"` |
| Maydon yo'q bo'lsa | kalit umuman bo'lmaydi | ustun bo'sh yoki `None` |

Shu farqlar sababli ikkalasini **bitta tozalash funksiyasidan** o'tkazamiz (7-bo'lim).

## 7. Yagona ichki ko'rinish va validatsiya

G'oya: ikkita formatni bir marta **yagona shaklga** keltiramiz, keyin qolgan kod format haqida o'ylamaydi.

```
requests.json ─┐
               ├─► [{"id": "...", "text": "..."}, ...] ─► route ─► result
requests.csv  ─┘
```

Bu **adapter** (moslashtiruvchi) naqshi deyiladi: har format uchun alohida "o'qigich", lekin chiqishi bir xil.

### Tozalash qoidalari

1. `id` va `text` ni `str(...)` ga aylantiring (`42` → `"42"`).
2. `.strip()` bilan chekkalarini tozalang.
3. Tozalangandan **keyin** bo'shligini tekshiring.

```python
{"id": 42, "text": " Не работает отчет "}   →   {"id": "42", "text": "Не работает отчет"}
{"id": "7", "text": "   "}                  →   XATO (text bo'sh)
{"id": None}                                 →   XATO (id va text yo'q)
```

### `None` tuzog'i

```python
str(None)    # "None"  ← bo'sh emas! Xato o'tib ketadi
```

Shuning uchun avval `None` ni tekshiring:

```python
value = raw.get(field)
value = "" if value is None else str(value).strip()
if not value:
    raise ...
```

"Tozalangandan keyin tekshirish" muhim: `"   "` tozalashdan oldin bo'sh emas, keyin bo'sh.

## 8. Funksiyalar va mas'uliyatni ajratish

Loyiha talabi: dasturni kichik funksiyalarga bo'lish, har biri **bitta ish** qilsin (**Single Responsibility**).

| Funksiya | Kirish | Chiqish | Mas'uliyat |
|---|---|---|---|
| `read_json(path)` | yo'l | xom yozuvlar | faqat JSON o'qiydi |
| `read_csv(path)` | yo'l | xom yozuvlar | faqat CSV o'qiydi |
| `clean_record(raw)` | xom yozuv | `{"id","text"}` | tozalaydi, tekshiradi |
| `load_records(path)` | yo'l | tozalangan ro'yxat | formatni tanlaydi |
| `normalize(text)` | satr | satr | kichik harf, ё→е |
| `detect_category(text)` | matn | kategoriya | faqat kategoriya |
| `detect_priority(text, cat)` | matn, kategoriya | priority | faqat ustuvorlik |
| `route_record(rec)` | `{"id","text"}` | `{"id","category","priority"}` | yuqoridagilarni birlashtiradi |
| `main()` | — | exit code | CLI, xatolarni ushlash |

Afzalliklari:
- har birini **alohida tekshirish** mumkin (`detect_category("не могу войти")`);
- o'zgartirish oson: yangi format (masalan, TSV) — faqat yangi `read_*` funksiyasi;
- P2P'da "bu qismni tushuntiring" deganda javob aniq.

### Sof funksiya (pure function)

`normalize`, `detect_category`, `detect_priority` — **sof funksiyalar**: faqat kirishga qarab natija beradi, fayl yoki global holatga tegmaydi. Ularni tekshirish eng oson.

## 9. Xatolar bilan ishlash: exception, try/except

**Exception** — dastur kutilmagan holatga duch kelganda ko'tariladigan signal. Ushlanmasa, dastur to'xtaydi va `traceback` chiqadi.

```python
try:
    with open(path, encoding="utf-8") as f:
        data = json.load(f)
except FileNotFoundError:
    print("Fayl topilmadi")
except json.JSONDecodeError as e:
    print(f"JSON buzilgan: {e.lineno}-qator")
```

### Ikki xil xato

| Tur | Misol | Kim aybdor | Nima qilamiz |
|---|---|---|---|
| **Kutilgan** | fayl yo'q, JSON buzilgan, maydon bo'sh | foydalanuvchi/ma'lumot | tushunarli xabar + nolga teng bo'lmagan kod |
| **Kutilmagan** | kodimizdagi `NameError`, `TypeError` | dasturchi | traceback'ni ko'rsatish foydali |

Loyiha faqat **kutilgan** xatolarni ushlashni so'raydi. Shuning uchun `except Exception:` (hammasini yutib yuborish) — **yomon amaliyot**: haqiqiy xatolarni yashiradi.

### Maxsus exception sinfi

```python
class RouterError(Exception):
    """Foydalanuvchiga ko'rsatiladigan kutilgan xato."""
```

Past darajadagi funksiyalar `raise RouterError("tushunarli matn")` qiladi, eng yuqorida (`main`) bitta `except RouterError` hammasini ushlaydi. Bu xabar matnini bir joyda boshqarishga imkon beradi.

### `raise ... from`

```python
except FileNotFoundError as e:
    raise RouterError("kirish fayli topilmadi") from e
```

`from e` asl sababni saqlaydi (debug uchun), lekin foydalanuvchi faqat bizning xabarni ko'radi.

### Yaxshi xato xabari

| Yomon | Yaxshi |
|---|---|
| `Error` | `Xato: kirish fayli topilmadi: data/x.json` |
| `KeyError: 'text'` | `Xato: 3-yozuvda 'text' maydoni yo'q yoki bo'sh` |
| `JSONDecodeError: Expecting ...` | `Xato: JSON buzilgan (5-qator, 12-ustun)` |

Yaxshi xabar: **nima bo'ldi + qayerda + (iloji bo'lsa) nima qilish kerak**.

## 10. CLI va `argparse`

**CLI (Command Line Interface)** — dasturni terminaldan argumentlar bilan boshqarish.

```python
import argparse

parser = argparse.ArgumentParser(
    description="Murojaatlarni kategoriya va ustuvorlik bo'yicha marshrutlaydi."
)
parser.add_argument("--input",  required=True, help="Kirish fayli (.json yoki .csv)")
parser.add_argument("--output", required=True, help="Natija JSON fayli")
args = parser.parse_args()

print(args.input, args.output)
```

`argparse` avtomatik qiladi:
- `--help` / `-h` — yordam matni, exit code 0;
- majburiy argument yo'q bo'lsa — xabar va exit code **2**;
- noma'lum argument bo'lsa — xato.

Nima uchun `sys.argv.split()` emas? Bo'sh joyli yo'llar (`"my file.json"`), qo'shtirnoqlar, `--input=x` shakli — hammasini `argparse` to'g'ri hal qiladi.

### Dasturni modul sifatida ham, skript sifatida ham ishlatish

```python
def main(argv=None):
    ...
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

`if __name__ == "__main__":` — fayl to'g'ridan-to'g'ri ishga tushirilganda ishlaydi, boshqa fayldan `import` qilinganda **ishlamaydi**. Bu funksiyalarni import qilib sinab ko'rishga imkon beradi.

## 11. Exit code va `traceback`

Har bir jarayon tugaganda operatsion tizimga **tugash kodi** qaytaradi.

| Kod | Ma'nosi |
|---|---|
| `0` | muvaffaqiyat |
| `1` | umumiy xato (biz tanlaymiz) |
| `2` | `argparse` xatosi (noto'g'ri argumentlar) |

Terminalda ko'rish:

```bash
python3 src/main.py --input yo_q.json --output /tmp/r.json
echo $?        # 1
```

⚠️ `$?` — **faqat oxirgi buyruq** kodi. Orada boshqa buyruq (hatto `echo`) bajarilsa, qiymat o'zgaradi. Shuning uchun 5-topshiriqda har bir buyruqdan **darhol keyin** yozib qo'yish talab qilingan.

Python'da:

```python
sys.exit(0)      # muvaffaqiyat
sys.exit(1)      # xato
```

Xato xabarini **stderr** ga chiqarish to'g'ri odat:

```python
print("Xato: ...", file=sys.stderr)
```

Sabab: natija (stdout) va xatolar (stderr) aralashmasligi, `> fayl` qilinganda xato xabari faylga tushmasligi uchun.

**Traceback** — ushlanmagan exception'da Python chiqaradigan chaqiruvlar zanjiri. Dasturchi uchun foydali, oddiy foydalanuvchi uchun qo'rqinchli va foydasiz. Maqsad: kutilgan xatolarda traceback **ko'rinmasin**.

## 12. Fayl yo'llari va `pathlib`

```python
from pathlib import Path

p = Path("datasets/requests.json")
p.suffix            # ".json"
p.suffix.lower()    # ".JSON" bo'lsa ham ".json"
p.exists()          # True/False
p.parent            # Path("datasets")
p.parent.mkdir(parents=True, exist_ok=True)   # papka yaratish
```

Kengaytma bo'yicha tanlash:

```python
suffix = Path(path).suffix.lower()
if suffix == ".json":
    ...
elif suffix == ".csv":
    ...
else:
    raise RouterError(f"qo'llab-quvvatlanmaydigan format: {suffix or 'kengaytmasiz'}")
```

Nisbiy yo'llar (`datasets/...`) **joriy ishchi papkaga** nisbatan hisoblanadi — shuning uchun buyruqlarni repozitoriy ildizidan ishga tushiring.

## 13. Git ish jarayoni (workflow)

Loyiha talabi: GitLab'dan klonlash → `develop` branch → `src/` papkasida ishlash → `develop` ni push qilish.

```bash
git clone <repozitoriy-url>
cd <repozitoriy>
git checkout -b develop        # yangi branch yaratib, unga o'tish
# ... ishlash ...
git status                     # nima o'zgardi
git add src/                   # faqat src/ ni staging'ga
git commit -m "Add JSON and CSV readers"
git push origin develop        # birinchi push: origin ga develop'ni yuborish
```

Asosiy tushunchalar:

| Atama | Ma'nosi |
|---|---|
| repository | loyiha + butun tarixi |
| branch | mustaqil ishlash chizig'i |
| working tree | diskdagi fayllaringiz |
| staging (index) | keyingi commit'ga kiradigan o'zgarishlar |
| commit | tarixdagi saqlash nuqtasi |
| remote / origin | GitLab'dagi server nusxasi |

Yaxshi odatlar:
- **kichik, mazmunli commit'lar** (har topshiriq — alohida commit);
- `git status` ni tez-tez ishlating, `git add .` o'rniga nimani qo'shayotganingizni biling;
- commit xabari: *nima qilindi* (`Add CLI with argparse`), `fix`, `update` kabi bo'sh so'zlar emas;
- direktoriyada **ortiqcha fayl bo'lmasligi** kerak: `__pycache__/`, vaqtinchalik fayllar, `.DS_Store` ni commit qilmang (kerak bo'lsa `.gitignore`ga qo'shing, lekin loyiha "boshqa fayllar bo'lmasin" deyapti, shuning uchun avval `git status` bilan tekshiring).

## 14. Qo'lda qabul tekshiruvi (manual acceptance)

Avtomatik testlar yo'q, shuning uchun muhandis stsenariylarni **qo'lda** ishga tushiradi va natijani jadvalga yozadi.

Asosiy tamoyil: **kutilgan natija ishga tushirishdan OLDIN yoziladi**. Aks holda siz natijaga qarab "kutilgan"ni moslab yozib qo'yasiz va tekshiruv ma'nosini yo'qotadi.

Ikki xil stsenariy:

- **Happy path** — hamma narsa to'g'ri (JSON, CSV).
- **Negative path** — foydalanuvchi xato qilgan (yo'q fayl, buzilgan JSON, maydon yo'q).

Holatlar:
- `PASS` — kutilgan = haqiqiy;
- `FAIL` — farq bor. Farqni yashirmang: "kutilgan" ustunini o'zgartirmang, farqni tasvirlang va kodni tuzating.

Buzilgan JSON tayyorlash (asl faylni buzmasdan):

```bash
head -c 120 datasets/requests.json > /tmp/broken.json
```

Bu faylning faqat dastlabki 120 baytini oladi, shuning uchun JSON yarim qoladi.

## 15. Pseudocode

```
FUNCTION normalize(text):
    RETURN replace(lowercase(text), "ё", "е")

FUNCTION detect_category(text):
    t ← normalize(text)
    FOR EACH (name, keywords) IN CATEGORY_RULES (ma'lumotnoma tartibida):
        FOR EACH kw IN keywords:
            IF normalize(kw) IS SUBSTRING OF t:
                RETURN name
    RETURN "other"

FUNCTION detect_priority(text, category):
    t ← normalize(text)
    IF any high-marker IS SUBSTRING OF t:
        RETURN "high"
    IF category IN {"access", "bug"}:
        RETURN "medium"
    RETURN "low"

FUNCTION route_record(rec):
    cat ← detect_category(rec.text)
    pri ← detect_priority(rec.text, cat)
    RETURN {id: rec.id, category: cat, priority: pri}

FUNCTION clean_record(raw, index):
    FOR field IN ("id", "text"):
        v ← raw[field] OR ""        # None → ""
        v ← strip(str(v))
        IF v IS EMPTY:
            RAISE RouterError("index-yozuvda field yo'q yoki bo'sh")
        out[field] ← v
    RETURN out

FUNCTION load_records(path):
    ext ← lowercase(extension(path))
    IF ext == ".json": raw ← read_json(path)
    ELSE IF ext == ".csv": raw ← read_csv(path)
    ELSE: RAISE RouterError("format qo'llab-quvvatlanmaydi")
    RETURN [clean_record(r, i) FOR (i, r) IN enumerate(raw)]

PROCEDURE main():
    args ← parse_command_line()          # --input, --output
    TRY:
        records ← load_records(args.input)
        results ← [route_record(r) FOR r IN records]
        write_json(args.output, results)
    CATCH RouterError AS e:
        print_to_stderr("Xato: " + e.message)
        RETURN 1
    RETURN 0
```

### Ma'lumot oqimi

```
--input fayl
    │ read_json / read_csv
    ▼
xom yozuvlar  (turli turlar, bo'sh joylar, None...)
    │ clean_record
    ▼
[{"id": str, "text": str}]
    │ route_record (har biri uchun)
    │    ├─ normalize
    │    ├─ detect_category  (first-match)
    │    └─ detect_priority
    ▼
[{"id", "category", "priority"}]
    │ json.dump
    ▼
--output fayl
```

## 16. Murakkablik tahlili

Belgilar: **n** — yozuvlar soni, **L** — bitta matn uzunligi, **K** — barcha kategoriyalardagi kalit so'zlar jami soni (doimiy, qoidalar bilan belgilangan), **M** — high-belgilar soni.

| Amal | Vaqt | Izoh |
|---|---|---|
| `normalize` | O(L) | matnni bir marta aylanadi |
| `detect_category` | O(K · L) | har kalit so'z uchun `in` qidiruvi (eng yomon holatda) |
| `detect_priority` | O(M · L) | |
| `route_record` | O(L) | K va M doimiy bo'lgani uchun |
| Butun dastur | **O(n · L)** | n ta yozuv, har biri O(L) |

Xotira: barcha yozuvlar ro'yxatda saqlanadi → **O(n · L)**.

**Nima uchun K doimiy deb olinadi?** Kalit so'zlar ro'yxati kirish hajmiga bog'liq emas, u kodda qat'iy yozilgan. Shuning uchun asimptotik hisobda doimiy. (Amalda K = 50 bo'lsa, bu "50 marta sekinroq" degani, lekin n ning o'sishiga ta'siri yo'q.)

**Optimallashtirish imkoniyati (majburiy emas):** agar kalit so'zlar minglab bo'lsa, `regex` yoki Aho–Corasick algoritmi bilan matnni bir o'tishda tekshirish mumkin. Bu loyiha uchun ortiqcha, lekin "baseline'ni qanday tezlashtirish mumkin" degan savolga javob sifatida bilish foydali.

**Amaliy chegara:** butun faylni xotiraga o'qiymiz. Millionlab yozuvlar uchun qatorma-qator (streaming) ishlash kerak bo'lardi. Qo'llab-quvvatlash jamoasi uchun kunlik yuzlab/minglab murojaatlarda bu muammo emas.

## 17. Keng tarqalgan xatolar

| Xato | Nima bo'ladi | Yechim |
|---|---|---|
| Kalit so'zni ildizga qisqartirish (`парол`) | ma'lumotnomadan farq qiladi | satrni aynan ko'chiring |
| `ё` ni faqat matnda almashtirib, kalit so'zda qoldirish | `счёт` kalit so'zi hech qachon topilmaydi | kalit so'zni ham `normalize` qiling |
| Kategoriya tartibini o'zgartirish | ba'zi yozuvlar boshqa kategoriya oladi | `routing-rules.md` tartibiga rioya qiling |
| `str(None)` ni tekshirmaslik | `"None"` matni o'tib ketadi | avval `None`ni tekshiring |
| Tozalashdan oldin bo'shlikni tekshirish | `"  "` o'tib ketadi | avval `strip`, keyin tekshiruv |
| CSV'ni `split(",")` bilan o'qish | matn ichidagi vergul buziladi | `csv.DictReader` |
| `encoding` ko'rsatmaslik | Windows'da kirill buziladi | har doim `encoding="utf-8"` |
| `ensure_ascii=False` unutish | natijada `\u043d...` | `json.dump(..., ensure_ascii=False)` |
| `except Exception:` | haqiqiy xatolar yashirinadi | aniq exception turlarini ushlang |
| Xabarni `print` bilan stdout'ga chiqarish | natija bilan aralashadi | `file=sys.stderr` |
| `echo $?` dan oldin boshqa buyruq | kod yo'qoladi | darhol tekshiring |
| Yangi yozuv uchun kodni o'zgartirish kerak bo'lib qolishi | P2P'da muammo | `id` bo'yicha tayyor javob emas, qoidalar |
| Tekshiruv `print`ini o'chirmaslik | 2-topshiriq talabi buziladi | topshirishdan oldin qidirib toping |
| `datasets/` ni nusxalab `src/` ga qo'yish | "ortiqcha fayl" | faqat topshiriqda aytilgan fayllar |

## 18. P2P'ga tayyorgarlik savollari

O'zingizga ovoz chiqarib javob bering:

1. Baseline nima va nima uchun u LLM'dan oldin kerak?
2. Nima uchun JSON va CSV'ni yagona ko'rinishga keltirdik?
3. `id: 42` (son) va `" Не работает "` yozuvi tozalangandan keyin qanday ko'rinadi?
4. Matnda ham billing, ham access so'zi bor. Qaysi kategoriya tanlanadi va nima uchun?
5. `счёт` va `счет` nima uchun bir xil ishlaydi?
6. `priority` qanday aniqlanadi? Qaysi tartibda?
7. Fayl yo'q bo'lsa nima bo'ladi? Exit code nima?
8. Traceback nima uchun foydalanuvchiga ko'rsatilmaydi?
9. `if __name__ == "__main__":` nima qiladi?
10. Tekshiruvchi yangi yozuv qo'shsa, siz nima kutasiz va kodni o'zgartirasizmi?
11. Kutilgan natija jadvalga nima uchun ishga tushirishdan **oldin** yoziladi?
12. Dasturingiz murakkabligi qancha? Nimaga bog'liq?

**P2P'da ko'rsatadigan yo'l (namuna):**

> `--input` → `load_records` kengaytmaga qaraydi → `read_json` → `clean_record` (id, text tozalanadi) → `route_record`: `normalize` → `detect_category` (birinchi mos kategoriya) → `detect_priority` (high → access/bug → low) → `json.dump` → `--output`.
