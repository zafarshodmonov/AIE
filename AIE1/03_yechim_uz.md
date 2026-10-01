# Support Ticket Router: to'liq yechim

Bu fayl loyihaning barcha topshiriqlari (1–5) uchun tayyor yechimni o'z ichiga oladi: `src/main.py` kodi, `src/README.md` va `src/manual_checks.md` shablonlari hamda Git buyruqlari.

> ✅ **Holat:** qoidalar endi sizning `routing-rules.md` faylingizdan olingan va kod haqiqiy `requests.json`, `requests.csv` va `expected-results.json` bilan sinab ko'rilgan: JSON bo'yicha 6/6, CSV bo'yicha 4/4 yozuv kutilganga mos keldi. Uchta xato stsenariysi ham tekshirildi (har birida tushunarli xabar, traceback yo'q, kod 1).
>
> ⚠️ `expected-results.json` tuzilishiga e'tibor bering: bu **ro'yxat emas, fayl nomi bo'yicha guruhlangan obyekt** (`"requests.json": [...]`, `"requests.csv": [...]`). Pastdagi solishtirish skripti shunga moslangan.

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
# Marshrutlash qoidalari: datasets/reference/routing-rules.md bo'yicha.
# Kategoriyalar shu tartibda tekshiriladi, birinchi moslik tanlanadi.
# Moslik = normalizatsiya qilingan matn ko'rsatilgan satrni o'z ichiga oladi.
# ---------------------------------------------------------------------------
CATEGORY_RULES = [
    ("access", ["войти", "пароль", "нет доступа"]),
    ("billing", ["оплата", "счет", "тариф", "спис"]),
    ("bug", ["ошибка", "не работает", "падает", "недоступен"]),
]
DEFAULT_CATEGORY = "other"

HIGH_MARKERS = ["срочно", "критично", "недоступен всем"]
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

`expected-results.json` fayl nomi bo'yicha guruhlangan, shuning uchun har bir kirish fayli uchun o'z bo'limini olamiz. Skriptni faylga saqlash shart emas (direktoriyada ortiqcha fayl bo'lmasin), terminalda ishga tushiring:

```bash
python3 - <<'EOF'
import json

def load(path):
    with open(path, encoding="utf-8") as f:
        return json.load(f)

expected_all = load("datasets/expected-results.json")
checks = [
    ("requests.json", "src/result.json"),
    ("requests.csv", "/tmp/result_csv.json"),
]

for name, result_path in checks:
    actual = {r["id"]: r for r in load(result_path)}
    bad = []
    for exp in expected_all[name]:
        act = actual.get(exp["id"])
        if (act is None or act["category"] != exp["category"]
                or act["priority"] != exp["priority"]):
            bad.append((exp["id"], exp, act))
    print(f"{name}: kutilgan {len(expected_all[name])}, mos kelmagan {len(bad)}")
    for rid, exp, act in bad:
        print("  ", rid, "kutilgan:", exp["category"], exp["priority"],
              "| haqiqiy:", (act["category"], act["priority"]) if act else None)
EOF
```

Kutilgan chiqish:

```
requests.json: kutilgan 6, mos kelmagan 0
requests.csv: kutilgan 4, mos kelmagan 0
```

Mos kelmaganlar chiqsa, har birining yo'lini qo'lda bosib o'ting: matn → `normalize` → qaysi satr topildi (yoki topilmadi) → `routing-rules.md` dagi qoida.

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

## 8. Qoidalar va namunaviy yozuvlar tahlili

Qoidalar `routing-rules.md` dan so'zma-so'z ko'chirilgan. Ikkita muhim nuqta:

1. **Satrlarni qisqartirmang.** Ma'lumotnoma "matn ko'rsatilgan satrni o'z ichiga oladi" deydi va "qo'shimcha qoidalar o'ylab topma" deydi. Masalan, `пароль` satri `пароля` ichida topilmaydi (oxirgi `ь` yo'q). `парол` ga qisqartirish o'zingizdan qoida qo'shish bo'lardi. Bu loyihaning baseline sifatidagi cheklovi, uni yashirmay qabul qilamiz.
2. **`спис`** — ataylab qisqa satr: `списалась`, `списание` unga mos keladi.

Barcha 10 yozuvning yo'li (`normalize` dan keyin):

| id | Matn (normalizatsiyadan keyin) | Topilgan satr | category | priority sababi |
|---|---|---|---|---|
| r-001 | не могу войти в личный кабинет после смены пароля | `войти` (`пароль` ham bor) | access | high yo'q → access → **medium** |
| r-002 | срочно: сервис не работает и недоступен всем сотрудникам | access/billing yo'q; `не работает` | bug | `срочно` → **high** |
| r-003 | дважды списалась оплата за тариф | `оплата` (`тариф`, `спис` ham bor) | billing | → **low** |
| r-004 | на странице отчета появляется ошибка 500 | `ошибка` | bug | → **medium** |
| r-005 | подскажите, как изменить язык интерфейса | hech biri | other | → **low** |
| r-006 | критично: не могу войти в рабочий кабинет | `войти` | access | `критично` → **high** |
| c-001 | не работает экспорт отчета | `не работает` | bug | → **medium** |
| c-002 | вопрос по счету за май | `счет` (`счёту` → `счету`) | billing | → **low** |
| c-003 | забыл пароль от личного кабинета | `пароль` | access | → **medium** |
| c-004 | спасибо за новый интерфейс | hech biri | other | → **low** |

Qiziq holatlar:
- **r-002:** `не работает` ham, `недоступен всем` ham bor. Kategoriya baribir `bug`, ustuvorlik esa `срочно` va `недоступен всем` tufayli `high`.
- **c-002:** `счёту` → `ё` ni `е` ga almashtirgandan keyin `счету` bo'ladi va `счет` satrini o'z ichiga oladi. Normalizatsiyasiz bu `other` bo'lib qolardi.
- **r-003:** billing ichida bir nechta satr mos keladi, lekin natija bitta (kategoriya nomi).

P2P uchun mashq: tekshiruvchi yangi yozuv qo'shadi. Masalan `"Сайт падает, срочно!"` → `bug` / `high`; `"Нет доступа к счету"` → `access` (access birinchi tekshiriladi) / `medium`. Avval o'zingiz taxmin qiling, keyin dasturni ishga tushiring.

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
