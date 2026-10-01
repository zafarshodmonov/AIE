# Support Ticket Router: to'liq yechim

Bu fayl loyihaning barcha topshiriqlari (1–5) uchun tayyor yechimni o'z ichiga oladi: `src/main.py` kodi, `src/README.md` va `src/manual_checks.md` shablonlari hamda Git buyruqlari.

> ⚠️ **Muhim eslatma (kalit so'zlar haqida).** Menda `datasets/reference/routing-rules.md` fayli yo'q edi, faqat README'dagi misollar bor edi (`r-001` → access/medium, `r-003` → billing/low). Shuning uchun kodning boshidagi `CATEGORY_RULES` va `HIGH_MARKERS` ro'yxatlaridagi so'zlar **taxminiy**. Kod va tuzilma tayyor, lekin siz ularni `routing-rules.md` bilan **so'zma-so'z solishtirib**, kategoriyalar tartibini va so'zlarni to'g'rilashingiz shart. Aks holda `expected-results.json` bilan hamma yozuvlar mos kelmasligi mumkin. Kodning qolgan qismi (o'qish, tozalash, xatolar, CLI) qoidalarga bog'liq emas.

## Mundarija

1. [Qadamlar xaritasi](#1-qadamlar-xaritasi)
2. [Git bilan boshlash](#2-git-bilan-boshlash)
3. [1-topshiriq: src/README.md](#3-1-topshiriq-srcreadmemd)
4. [`src/main.py` (2, 3, 4-topshiriqlar)](#4-srcmainpy-2-3-4-topshiriqlar)
5. [Kod qanday ishlaydi](#5-kod-qanday-ishlaydi)
6. [Ishga tushirish va tekshirish](#6-ishga-tushirish-va-tekshirish)
7. [5-topshiriq: src/manual_checks.md](#7-5-topshiriq-srcmanual_checksmd)
8. [Qoidalarni routing-rules.md bilan moslashtirish](#8-qoidalarni-routing-rulesmd-bilan-moslashtirish)
9. [Ixtiyoriy: kategoriyalar bo'yicha hisob](#9-ixtiyoriy-kategoriyalar-boyicha-hisob)
10. [Topshirishdan oldingi chek-list](#10-topshirishdan-oldingi-chek-list)

---

## 1. Qadamlar xaritasi

| Topshiriq | Nima yaratiladi | Qayerda |
|---|---|---|
| 1 | `src/README.md` (dastlabki versiya) | 3-bo'lim |
| 2 | `read_json`, `read_csv`, `clean_record`, `load_records` | 4-bo'lim |
| 3 | `normalize`, `detect_category`, `detect_priority`, `route_record` | 4-bo'lim |
| 4 | `main()`, `argparse`, xatolarni ushlash, `src/result.json` | 4, 6-bo'lim |
| 5 | `src/manual_checks.md` | 7-bo'lim |

Yakuniy tuzilma (boshqa fayl bo'lmasligi kerak):

```
<repozitoriy>/
├── datasets/              (tayyor keladi)
└── src/
    ├── main.py
    ├── README.md
    ├── result.json
    └── manual_checks.md
```

## 2. Git bilan boshlash

```bash
git clone <GitLab-repozitoriy-url>
cd <repozitoriy-nomi>
git checkout -b develop
python3 --version          # Python 3.x chiqishi kerak
ls datasets datasets/reference
```

Har topshiriqdan keyin commit qiling:

```bash
git add src/
git commit -m "Task 1: add src/README.md"
# ... va hokazo ...
git push origin develop
```

## 3. 1-topshiriq: src/README.md

Avval `datasets/` fayllarini ko'zdan kechiring (README'dagi 1–5 bandlar), keyin `src/README.md` ni quyidagi dastlabki ko'rinishda yarating. Ishga tushirish buyruqlarini 4-topshiriqdan keyin qo'shasiz (6-bo'limdagi yakuniy versiya).

````markdown
# Support Ticket Router

Murojaatlar marshrutizatori: JSON yoki CSV fayldan murojaatlarni o'qiydi,
har biriga aniq qoidalar bo'yicha kategoriya (category) va ustuvorlik (priority)
beradi va natijani JSON faylga saqlaydi.

## Talablar

- Python 3 (faqat standart kutubxona, tashqi paketlar kerak emas)

## Kirish formatlari

- **JSON** — ildizi massiv bo'lgan fayl; har bir yozuvda `id` va `text` bor.
- **CSV** — birinchi qatori sarlavha (`id,text`), keyingi qatorlar yozuvlar.

## Natija

JSON-massiv. Har bir element:

| Maydon | Ma'nosi |
|---|---|
| `id` | murojaat identifikatori (satr) |
| `category` | kategoriya (masalan, `access`, `billing`, `bug`, `other`) |
| `priority` | ustuvorlik: `high`, `medium` yoki `low` |
````

## 4. `src/main.py` (2, 3, 4-topshiriqlar)

```python
#!/usr/bin/env python3
"""Support ticket router.

JSON yoki CSV fayldan murojaatlarni o'qiydi, har biriga aniq qoidalar bo'yicha
category va priority beradi va natijani JSON faylga saqlaydi.
"""

import argparse
import csv
import json
import sys
from pathlib import Path

EXIT_OK = 0
EXIT_ERROR = 1

# ---------------------------------------------------------------------------
# Marshrutlash qoidalari.
# DIQQAT: tartib va so'zlar datasets/reference/routing-rules.md bilan
# BIR XIL bo'lishi kerak. Quyidagilar - namuna, ularni tekshirib to'g'rilang.
# Kalit so'zlar ildiz (stem) ko'rinishida yoziladi: "парол" -> пароль, пароля...
# ---------------------------------------------------------------------------
CATEGORY_RULES = [
    ("access", ["войти", "вход", "парол", "доступ", "логин", "авториз"]),
    ("billing", ["оплат", "платеж", "счет", "списал", "тариф", "деньги"]),
    ("bug", ["ошибк", "не работает", "не открыва", "сбой", "обновлени"]),
]
DEFAULT_CATEGORY = "other"

HIGH_MARKERS = ["срочно", "критич", "немедленно", "авария", "все упало"]
MEDIUM_CATEGORIES = {"access", "bug"}


class RouterError(Exception):
    """Foydalanuvchiga ko'rsatiladigan kutilgan xato."""


# ---------------------------------------------------------------------------
# 2-topshiriq: o'qish va yagona ko'rinishga keltirish
# ---------------------------------------------------------------------------
def read_json(path):
    """JSON fayldan ildiz massivni qaytaradi."""
    try:
        with open(path, encoding="utf-8") as f:
            data = json.load(f)
    except FileNotFoundError as e:
        raise RouterError(f"kirish fayli topilmadi: {path}") from e
    except json.JSONDecodeError as e:
        raise RouterError(
            f"JSON buzilgan ({e.lineno}-qator, {e.colno}-ustun): {e.msg}"
        ) from e
    except UnicodeDecodeError as e:
        raise RouterError(f"fayl UTF-8 kodlashida emas: {path}") from e
    except OSError as e:
        raise RouterError(f"faylni o'qib bo'lmadi: {path} ({e.strerror})") from e

    if not isinstance(data, list):
        raise RouterError("JSON ildizi massiv (list) bo'lishi kerak")
    return data


def read_csv(path):
    """CSV fayldan qatorlarni lug'at ko'rinishida qaytaradi."""
    try:
        # utf-8-sig: Excel qo'shgan BOM belgisini avtomatik olib tashlaydi
        with open(path, encoding="utf-8-sig", newline="") as f:
            return list(csv.DictReader(f))
    except FileNotFoundError as e:
        raise RouterError(f"kirish fayli topilmadi: {path}") from e
    except UnicodeDecodeError as e:
        raise RouterError(f"fayl UTF-8 kodlashida emas: {path}") from e
    except csv.Error as e:
        raise RouterError(f"CSV buzilgan: {e}") from e
    except OSError as e:
        raise RouterError(f"faylni o'qib bo'lmadi: {path} ({e.strerror})") from e


def clean_record(raw, index):
    """Xom yozuvni {"id": str, "text": str} ko'rinishiga keltiradi va tekshiradi."""
    if not isinstance(raw, dict):
        raise RouterError(f"{index}-yozuv obyekt (lug'at) emas")

    cleaned = {}
    for field in ("id", "text"):
        value = raw.get(field)
        value = "" if value is None else str(value).strip()  # str(None) == "None" tuzog'i
        if not value:
            raise RouterError(f"{index}-yozuvda '{field}' maydoni yo'q yoki bo'sh")
        cleaned[field] = value
    return cleaned


def load_records(path):
    """Kengaytmaga qarab o'qiydi va tozalangan yozuvlar ro'yxatini qaytaradi."""
    suffix = Path(path).suffix.lower()
    if suffix == ".json":
        raw_records = read_json(path)
    elif suffix == ".csv":
        raw_records = read_csv(path)
    else:
        raise RouterError(
            f"qo'llab-quvvatlanmaydigan format: {suffix or 'kengaytmasiz'} "
            "(faqat .json va .csv)"
        )
    # Foydalanuvchiga qulay bo'lishi uchun yozuvlar 1 dan raqamlanadi
    return [clean_record(raw, i) for i, raw in enumerate(raw_records, start=1)]


# ---------------------------------------------------------------------------
# 3-topshiriq: marshrutlash qoidalari
# ---------------------------------------------------------------------------
def normalize(text):
    """Kichik harf va ё -> е."""
    return text.lower().replace("ё", "е")


def detect_category(text):
    """Ma'lumotnoma tartibidagi birinchi mos kategoriyani qaytaradi."""
    normalized = normalize(text)
    for category, keywords in CATEGORY_RULES:
        if any(normalize(word) in normalized for word in keywords):
            return category
    return DEFAULT_CATEGORY


def detect_priority(text, category):
    """high belgilari -> high; access/bug -> medium; qolgani -> low."""
    normalized = normalize(text)
    if any(normalize(marker) in normalized for marker in HIGH_MARKERS):
        return "high"
    if category in MEDIUM_CATEGORIES:
        return "medium"
    return "low"


def route_record(record):
    """Bitta yozuvni marshrutlaydi."""
    category = detect_category(record["text"])
    priority = detect_priority(record["text"], category)
    return {"id": record["id"], "category": category, "priority": priority}


# ---------------------------------------------------------------------------
# 4-topshiriq: saqlash va CLI
# ---------------------------------------------------------------------------
def write_results(path, results):
    """Natijani JSON-massiv ko'rinishida saqlaydi."""
    try:
        Path(path).parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w", encoding="utf-8") as f:
            json.dump(results, f, ensure_ascii=False, indent=2)
            f.write("\n")
    except OSError as e:
        raise RouterError(f"natijani saqlab bo'lmadi: {path} ({e.strerror})") from e


def build_parser():
    parser = argparse.ArgumentParser(
        description=(
            "Murojaatlarni (JSON yoki CSV) o'qiydi, har biriga category va "
            "priority beradi va natijani JSON faylga saqlaydi."
        )
    )
    parser.add_argument(
        "--input",
        required=True,
        help="kirish fayli yo'li (.json yoki .csv); har yozuvda id va text bo'lishi kerak",
    )
    parser.add_argument(
        "--output",
        required=True,
        help="natija JSON fayli saqlanadigan yo'l",
    )
    return parser


def main(argv=None):
    args = build_parser().parse_args(argv)
    try:
        records = load_records(args.input)
        results = [route_record(record) for record in records]
        write_results(args.output, results)
    except RouterError as e:
        print(f"Xato: {e}", file=sys.stderr)
        return EXIT_ERROR
    return EXIT_OK


if __name__ == "__main__":
    sys.exit(main())
```

> Xato xabarlari o'zbekchada yozilgan. P2P tekshiruvchisi rus yoki ingliz tilini afzal ko'rsa, f-satrlardagi matnni almashtirish kifoya, mantiq o'zgarmaydi.

## 5. Kod qanday ishlaydi

**O'qish qatlami (2-topshiriq).** `load_records` fayl kengaytmasiga qaraydi (`.JSON` katta harfda bo'lsa ham `lower()` tufayli ishlaydi) va `read_json` yoki `read_csv` ni chaqiradi. Ikkalasi ham xom yozuvlarni qaytaradi, ularni bitta `clean_record` funksiyasi tozalaydi. Shu sababli `{"id": 42, "text": " Не работает отчет "}` yozuvi `{"id": "42", "text": "Не работает отчет"}` ga aylanadi.

**`None` tuzog'i.** `str(None)` — `"None"` satri, u bo'sh emas. Shuning uchun `value is None` alohida tekshiriladi, aks holda maydoni yo'q yozuv xatosiz o'tib ketardi.

**Marshrutlash (3-topshiriq).** `detect_category` `CATEGORY_RULES` ni yuqoridan pastga yuradi va birinchi mos kategoriyada to'xtaydi (`any` + `return`). Kalit so'zlar ham `normalize` qilinadi, shuning uchun ro'yxatda `счёт` yozsangiz ham ishlaydi. `detect_priority` tartibi: avval high-belgilar, keyin `access`/`bug` → `medium`, qolgani `low`.

**Xatolar (4-topshiriq).** Past darajadagi funksiyalar `RouterError` ko'taradi, `main()` esa uni ushlaydi, `stderr` ga qisqa xabar chiqaradi va `1` qaytaradi. Traceback chiqmaydi. Uchta majburiy xato qamrab olingan:

| Holat | Qayerda ushlanadi | Xabar |
|---|---|---|
| fayl yo'q | `read_json` / `read_csv` | `Xato: kirish fayli topilmadi: ...` |
| JSON buzilgan | `read_json` | `Xato: JSON buzilgan (N-qator, M-ustun): ...` |
| `id`/`text` yo'q yoki bo'sh | `clean_record` | `Xato: N-yozuvda 'text' maydoni yo'q yoki bo'sh` |

`argparse` ning o'z xatolari (masalan, `--output` berilmagan) exit code `2` bilan tugaydi, bu normal.

## 6. Ishga tushirish va tekshirish

Barcha buyruqlar repozitoriy ildizidan bajariladi.

```bash
# 1) Yordam: exit code 0
python3 src/main.py --help
echo $?

# 2) JSON
python3 src/main.py --input datasets/requests.json --output src/result.json
echo $?

# 3) CSV (natijani vaqtincha /tmp ga yozamiz, src/ ni ifloslantirmaslik uchun)
python3 src/main.py --input datasets/requests.csv --output /tmp/result_csv.json
echo $?
```

### Natijani expected-results.json bilan `id` bo'yicha solishtirish

Bu skriptni faylga saqlash shart emas (direktoriyada ortiqcha fayl bo'lmasin), to'g'ridan-to'g'ri terminalda ishga tushiring:

```bash
python3 - <<'EOF'
import json

def by_id(path):
    with open(path, encoding="utf-8") as f:
        return {r["id"]: r for r in json.load(f)}

actual = by_id("src/result.json")
expected = by_id("datasets/expected-results.json")

bad = []
for rid, exp in expected.items():
    act = actual.get(rid)
    if act is None or act["category"] != exp["category"] or act["priority"] != exp["priority"]:
        bad.append((rid, exp, act))

print("Kutilgan yozuvlar:", len(expected), "| Mos kelmaganlar:", len(bad))
for rid, exp, act in bad:
    print(rid, "kutilgan:", exp["category"], exp["priority"],
          "| haqiqiy:", (act["category"], act["priority"]) if act else None)
EOF
```

Mos kelmaganlar chiqsa, har birining yo'lini qo'lda bosib o'ting: matn → `normalize` → qaysi kalit so'z topildi (yoki topilmadi) → `routing-rules.md` dagi qoida. Odatda sabab: kalit so'z yetishmaydi yoki kategoriyalar tartibi noto'g'ri.

### Uchta xato stsenariysi

```bash
# a) fayl yo'q
python3 src/main.py --input /tmp/router-file-that-does-not-exist.json --output /tmp/router-result.json
echo $?

# b) buzilgan JSON (asl faylga tegmasdan, nusxa)
head -c 120 datasets/requests.json > /tmp/broken.json
python3 src/main.py --input /tmp/broken.json --output /tmp/router-result.json
echo $?

# c) text yo'q yozuv
python3 src/main.py --input datasets/missing-field.json --output /tmp/router-result.json
echo $?
```

Kutilgan: har birida bitta qatorli `Xato: ...` xabari, traceback yo'q, `echo $?` → `1`.

Eslatma: 5-topshiriqdagi "buzilgan `datasets/requests.json`" ni asl faylni o'zgartirmasdan, `/tmp` dagi nusxa bilan tekshirish xavfsizroq. Jurnalda buni aniq yozib qo'ying (masalan, "buzilgan nusxa: `/tmp/broken.json`, `datasets/requests.json` dan `head -c 120`").

### Yakuniy src/README.md (4-topshiriqdan keyin)

````markdown
# Support Ticket Router

Murojaatlar marshrutizatori: JSON yoki CSV fayldan murojaatlarni o'qiydi,
har biriga aniq qoidalar bo'yicha kategoriya (category) va ustuvorlik (priority)
beradi va natijani JSON faylga saqlaydi.

## Talablar

- Python 3 (faqat standart kutubxona)

```bash
python3 --version
```

## Kirish formatlari

- **JSON** — ildizi massiv; har bir yozuvda `id` va `text`.
- **CSV** — birinchi qator sarlavha (`id,text`).

## Natija

JSON-massiv; har bir element:

| Maydon | Ma'nosi |
|---|---|
| `id` | murojaat identifikatori |
| `category` | `access`, `billing`, `bug` yoki `other` |
| `priority` | `high`, `medium` yoki `low` |

## Ishga tushirish

Buyruqlar repozitoriy ildizidan bajariladi.

```bash
# Yordam
python3 src/main.py --help

# JSON'ni qayta ishlash va natijani saqlash
python3 src/main.py --input datasets/requests.json --output src/result.json

# CSV'ni qayta ishlash
python3 src/main.py --input datasets/requests.csv --output /tmp/result_csv.json
```

## Xatolar

Kutilgan xatolarda (fayl topilmadi, JSON buzilgan, `id`/`text` yo'q yoki bo'sh)
dastur qisqa xabar chiqaradi va noldan farqli kod (`1`) bilan tugaydi.
Muvaffaqiyatli ishga tushirishda kod `0`.
````

> README'ga faqat **amalda tekshirgan** buyruqlaringizni yozing. Yuqoridagi barcha buyruqlarni o'zingiz ishga tushirib ko'rganingizdan keyin qoldiring.

## 7. 5-topshiriq: src/manual_checks.md

Kutilgan natijalarni **ishga tushirishdan oldin** to'ldiring. Quyida "Kutilgan" ustuni tayyor; "Haqiqiy", "Kod" va "Holat" ustunlarini faktlar bilan **o'zingiz** to'ldirasiz (men sizning terminalingizda ishga tushira olmayman, shuning uchun to'qib yozmayman).

````markdown
# Manual checks

| **Stsenariy** | **Buyruq** | **Kutilgan natija** | **Haqiqiy natija** | **Tugash kodi** | **Holat** |
| --- | --- | --- | --- | --- | --- |
| To'g'ri JSON | `python3 src/main.py --input datasets/requests.json --output src/result.json` | Natija `datasets/expected-results.json` bilan `id` bo'yicha mos keladi | | | |
| To'g'ri CSV | `python3 src/main.py --input datasets/requests.csv --output /tmp/result_csv.json` | Natija `datasets/expected-results.json` bilan `id` bo'yicha mos keladi | | | |
| Mavjud bo'lmagan fayl | `python3 src/main.py --input /tmp/router-file-that-does-not-exist.json --output /tmp/router-result.json` | Tushunarli xabar (fayl topilmadi), traceback yo'q, kod ≠ 0 | | | |
| Buzilgan JSON | `python3 src/main.py --input /tmp/broken.json --output /tmp/router-result.json` (`/tmp/broken.json` = `head -c 120 datasets/requests.json`) | Tushunarli xabar (JSON buzilgan), traceback yo'q, kod ≠ 0 | | | |
| `text` yo'q yozuv | `python3 src/main.py --input datasets/missing-field.json --output /tmp/router-result.json` | Tushunarli xabar (`text` yo'q yoki bo'sh), traceback yo'q, kod ≠ 0 | | | |
````

To'ldirish tartibi:

1. Buyruqni ishga tushiring.
2. **Darhol** chiqishni va `echo $?` natijasini yozib oling (keyingi buyruq kodni ustidan yozmasin).
3. "Kutilgan" bilan solishtiring: mos bo'lsa `PASS`, aks holda `FAIL` va farqni tasvirlang.

"Haqiqiy natija" ustuniga yozish namunasi (xato holati uchun):

```
Chiqish: "Xato: kirish fayli topilmadi: /tmp/router-file-that-does-not-exist.json"; traceback yo'q
Kod: 1
Holat: PASS
```

Jadvaldagi yacheykada qator bo'lish uchun `<br>` ishlatishingiz mumkin. Kod yacheykasida bitta son (`0` yoki `1`) yetarli.

## 8. Qoidalarni routing-rules.md bilan moslashtirish

Bu eng muhim qadam. Tartib:

1. `datasets/reference/routing-rules.md` ni oching.
2. Kategoriyalarni **o'sha tartibda** `CATEGORY_RULES` ga ko'chiring (har kategoriya uchun kalit so'zlar).
3. High-belgilarni `HIGH_MARKERS` ga, `access`/`bug` qoidasini `MEDIUM_CATEGORIES` ga tekshirib yozing. Agar ma'lumotnomada boshqa kategoriyalar (masalan, `account`, `feature`) ham bo'lsa, ularni ham shu qoidalarga moslab qo'shing.
4. So'zlarni **ildiz** ko'rinishida kiriting, agar ma'lumotnoma to'liq so'zni bersa, to'liq so'zni qoldiring (ildizga qisqartirish kutilmagan moslik berishi mumkin).
5. `src/result.json` ni qayta yarating va 6-bo'limdagi solishtirish skriptini ishga tushiring. Mos kelmaganlar `0` bo'lguncha takrorlang.

Muhim: kalit so'zlarni `id` ga bog'lab yozmang ("r-001 → access" kabi). Dastur har qanday yangi yozuvni qayta ishlay olishi kerak (P2P'da aynan shu tekshiriladi).

## 9. Ixtiyoriy: kategoriyalar bo'yicha hisob

Majburiy emas va `src/result.json` formatini o'zgartirmasligi kerak. Eng xavfsiz yo'l: natijani `stdout`ga chiqarmasdan, alohida `--summary` bayrog'i bilan `stderr`ga yozish. `main()` ga qo'shimcha:

```python
from collections import Counter

# build_parser() ichida:
parser.add_argument(
    "--summary",
    action="store_true",
    help="kategoriyalar bo'yicha murojaatlar sonini ko'rsatadi",
)

# main() ichida, write_results(...) dan keyin:
if args.summary:
    counts = Counter(r["category"] for r in results)
    for category, count in sorted(counts.items()):
        print(f"{category}: {count}", file=sys.stderr)
```

`--summary` ixtiyoriy bayroq bo'lgani uchun asosiy buyruqlar o'zgarmaydi. Qo'shsangiz, `src/README.md` ga ham yozishni unutmang.

## 10. Topshirishdan oldingi chek-list

- [ ] `git branch` → `develop` da ekanligingiz tasdiqlandi
- [ ] `src/` da faqat: `main.py`, `README.md`, `result.json`, `manual_checks.md`
- [ ] `__pycache__/` va boshqa ortiqcha fayllar yo'q (`git status` bilan tekshiring)
- [ ] `main.py` da vaqtinchalik `print` qolmagan (`grep -n "print(" src/main.py` faqat `stderr` xabarlarini ko'rsatishi kerak)
- [ ] `python3 src/main.py --help` → kod 0
- [ ] `src/result.json` **joriy** kod versiyasi bilan qayta yaratilgan
- [ ] `expected-results.json` bilan solishtirish: mos kelmaganlar 0 (JSON va CSV uchun)
- [ ] Uchta xato stsenariysi: tushunarli xabar, traceback yo'q, kod ≠ 0
- [ ] `src/manual_checks.md`: 5 qator, hamma ustun **haqiqiy faktlar** bilan to'ldirilgan
- [ ] `src/README.md` dagi buyruqlar yangi terminalda ishlaydi
- [ ] `git push origin develop`
- [ ] P2P uchun bitta yozuvning yo'lini ovoz chiqarib tushuntira olasiz (nazariya faylining 18-bo'limi)
