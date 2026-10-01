# Birinchi murojaatlar marshrutizatoringiz (Support Ticket Router)

Ushbu loyihada siz basseyndan keyin Python, terminal va Git bo'yicha ko'nikmalaringizni yangilaysiz: qo'llab-quvvatlash (support) jamoasi uchun murojaatlar marshrutizatorini yaratasiz. Bu — JSON va CSV fayllardan murojaatlarni o'qiydigan, ularning kategoriyasi (category) va ustuvorligini (priority) aniq qoidalar bo'yicha aniqlaydigan, natijani saqlaydigan va xatolar haqida tushunarli xabar beradigan dastur.

Bu boshlang'ich loyiha hali LLM yechimi haqida emas, balki shaffof **baseline** haqida: keyingi loyihalarda murakkabroq tizimlarni taqqoslaydigan boshlang'ich nuqta.

💡 [Bu yerni bosing](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624), **shu loyiha bo'yicha fikr-mulohazangizni bildirish uchun**. Bu anonim bo'lib, jamoamizga o'qitishni yaxshilashga yordam beradi. So'rovnomani loyihani bajarganingizdan so'ng darhol to'ldirishni tavsiya qilamiz.

## Mundarija

  - [Mundarija](#mundarija)
  - [«21-Maktab»da qanday o'qish kerak](#21-maktabda-qanday-oqish-kerak)
  - [Chapter I](#chapter-i)
  - [Chapter II](#chapter-ii)
    - [1-topshiriq. Ma'lumotlar bilan tanishuv](#1-topshiriq-malumotlar-bilan-tanishuv)
    - [2-topshiriq. JSON va CSV uchun yagona ko'rinish](#2-topshiriq-json-va-csv-uchun-yagona-korinish)
    - [3-topshiriq. Marshrutlash qoidalari](#3-topshiriq-marshrutlash-qoidalari)
    - [4-topshiriq. Terminaldan ishga tushirish](#4-topshiriq-terminaldan-ishga-tushirish)
    - [5-topshiriq. Topshirishdan oldin tekshiruv](#5-topshiriq-topshirishdan-oldin-tekshiruv)


## «21-Maktab»da qanday o'qish kerak

- Bu yerda sizni erkinlik juda ko'p bo'lgan noyob ta'lim tajribasi kutmoqda. Siz topshiriq olasiz va yechim yo'llarini mustaqil izlaysiz, buning uchun istalgan qulay usuldan foydalanasiz — internet resurslari yoki neyrotarmoqlar (masalan, GigaChat). Lekin axborot sifatiga e'tiborli bo'ling: tekshiring, o'ylang, tahlil qiling, solishtiring.
- Peer-to-Peer (P2P) o'zaro o'qitish — bu boshqa pirlar bilan bilim va tajriba almashinuvi bo'lib, unda har kim ham o'qituvchi, ham o'quvchi bo'ladi. Bunday yondashuv o'zaro bir-biridan o'rganish orqali materialni chuqurroq tushunishga imkon beradi.
- O'zingizni erkin his qiling va yordam so'rang — atrofingizda bu yo'lni ilk bor bosib o'tayotgan insonlar bor. Tajriba va g'oyalaringiz bilan boshqalar bilan bo'lishing. Hamjamiyatimiz yangiliklaridan xabardor bo'lish uchun RocketChat'ga qo'shiling.
- Agar birovning yechimini ko'chirib olsangiz, o'qishingiz hech qanday ma'noga ega bo'lmaydi. Agar boshqalarning yordamidan foydalansangiz — nima uchun, qanday va nimaga bunday ekanini oxirigacha tushunib oling. Xato qilishdan qo'rqmang.
- Topshiriq bajarib bo'lmaydigandek tuyuladimi? Tanaffus qiling, havo yutib keling, boshingizni "qayta yuklang" — bu ko'plarga yordam bergan. Ehtimol, shundan keyin yechim o'zi keladi.
- Faqat o'qish natijasi emas, jarayonning o'zi ham muhim. Masalani shunchaki yechish emas, uni QANDAY yechishni tushunish kerak.

Loyiha bilan qanday ishlash kerak:

- Bajarishdan oldin loyihani GitLab'dan bir xil nomli repozitoriyga klonlash (clone) kerak.
- Barcha fayllarni klonlangan repozitoriyning `src/` papkasida yaratish kerak.
- Loyihani klonlagandan so'ng `develop` branch'ini yaratib, ishlab chiqishni shu branch'da olib borish kerak. Shundan keyin GitLab'ga ham `develop` branch'ini push qilish kerak.
- Direktoriyangizda topshiriqlarda ko'rsatilganlardan boshqa fayllar bo'lmasligi kerak.

## Chapter I

Siz agentlar yig'adigan, HTTP-mijozlar yozadigan va vektor ma'lumotlar bazalari bilan ishlaydigan dasturning boshlanishida tayyorgarlik qadamini bajarish muhim. Usiz siz asosiy savolga javob bera olmaysiz: LLM haqiqatan ham masalani oddiy qoidalardan yaxshiroq yechadimi? Va uning murakkabligi sarflangan pul va vaqtga arziydimi?

Tasavvur qiling, qo'llab-quvvatlash jamoasi har kuni foydalanuvchilarning xabarlarini qayta ishlashi va ular nima so'rayotganini tushunishi kerak. Ba'zi murojaatlarni birinchi qarashdayoq tanib olish oson:

_«Shaxsiy kabinetga kira olmayapman»._

_«To'lov o'tdi, lekin pul hisobga tushmadi»._

_«Yangilanishdan keyin hisobot ochilmay qoldi»._

Xabarlar beshta bo'lsa, ularni qo'lda saralash qiyin emas. Yuzlab bo'lsa, qo'llab-quvvatlash jamoasi bir xil harakatga vaqt sarflaydi: mavzuni aniqlash, shoshilinchlikni tushunish va murojaatni kerakli mutaxassisga uzatish.

Bu yerda darhol til modelini (language model) ulash g'oyasi paydo bo'lishi mumkin. U matn bilan ishlay oladi va bunday vazifa uchun tabiiy nomzoddek ko'rinadi. Lekin tushunarli boshlang'ich nuqtasiz muhim savollarga javob berib bo'lmaydi: model haqiqatan ham murojaatlarni yaxshiroq taqsimlaydimi, uning xatolari qancha turadi va tizimni murakkablashtirish o'zini oqlaydimi.

Ushbu loyihada siz aniq qoidalarga asoslangan murojaatlar marshrutizatorini yaratasiz. Muhandislik ishida bunday birinchi ishchi versiyani ko'pincha **baseline** deb atashadi — murakkabroq tizimlar bilan keyingi taqqoslash uchun asos. U mutlaqo har qanday matnni tushunmaydi, ammo uning har bir qarorini topilgan so'z va aniq qoida bilan tushuntirish mumkin.

Support jamoasi uchun yaratadigan dasturingiz:

- JSON va CSV fayllardan murojaatlarni o'qiydi;
- har bir murojaatning kategoriyasi va ustuvorligini aniqlaydi;
- natijani tushunarli formatda saqlaydi;
- kirish ma'lumotlarini o'qiy olmasa, buni tushunarli tarzda xabar qiladi.

## Chapter II

Ushbu loyihada siz quyidagilarni eslab olasiz:

- JSON va CSV'dan ma'lumotlarni o'qish;
- ro'yxatlar (list), lug'atlar (dictionary), shartlar va sikllar bilan ishlash;
- dasturni turli vazifalarga ega kichik funksiyalarga bo'lish;
- kutilgan xatolarni tushunarli xabarlarga aylantirish;
- dasturni terminaldan ishga tushirish va boshqa odam uchun takrorlanadigan buyruqlarni qoldirish;
- faqat muvaffaqiyatli stsenariyni emas, noto'g'ri ma'lumotlardagi xatti-harakatni ham tekshirish.

Topshiriqlarni yechish uchun OOP, uchinchi tomon paketlari, HTTP, SQL, Docker yoki LLM haqida oldindan bilishingiz shart emas. Ular tarmoqning keyingi loyihalarida, kod va ma'lumotlar bilan asosiy ishlash yana odatiy holga kelganda paydo bo'ladi.

Murojaatlar marshrutizatorini yaratish loyihasida siz sintetik ma'lumotlar to'plami bilan ishlaysiz. Boshlang'ich to'plamda sintetik murojaatlar, kutilgan natijalar va marshrutlash qoidalari allaqachon bor. Dastur lokal ishlaydi — tokenlarsiz, pullik servislarsiz va tashqi API'larsiz.

Loyiha uchun Python 3 va standart kutubxona yetarli. Funksiyalar nomlari va dasturning ichki tuzilishini, agar topshiriqda boshqacha ko'rsatilmagan bo'lsa, o'zingiz tanlaysiz.

## 1-topshiriq. Ma'lumotlar bilan tanishuv

Ma'lumotlar asosida qaror qabul qiladigan har qanday tizim koddan emas, kirish va kutilgan natija bilan tanishuvdan boshlanadi. Aks holda toza ishlaydigan, lekin noto'g'ri masalani yechadigan dastur yozib qo'yish oson.

`datasets/` papkasida murojaatlar ikki formatda joylashgan. `datasets/reference/routing-rules.md` faylida qaysi so'zlar qaysi kategoriyaga tegishli ekani va ustuvorlik qanday aniqlanishi yozilgan.

**Maqsad:** marshrutizator qayta ishlaydigan kirish ma'lumotlarini va uning ishining kutilgan natijasini o'rganish.

Nima qilish kerak:

1.  `python3 --version` ni ishga tushiring. Javobda Python 3 versiyasi ko'rsatilgan bo'lishi kerak. Agar buyruq topilmasa, avval terminalda Python'ni ishga tushirishni sozlang, so'ng davom eting.
2.  `datasets/requests.json` ni oching. Faylda murojaatlar massivi borligiga va har bir yozuvda id va text borligiga ishonch hosil qiling.
3.  `datasets/requests.csv` ni oching. Sarlavhalar qatorida o'sha maydonlarni toping va bitta CSV qatorini bitta JSON yozuvi bilan solishtiring.
4.  `datasets/reference/routing-rules.md` ni o'qing. Har bir kategoriya uchun JSON yoki CSV'dan kamida bitta mos murojaatni toping va unga qaysi ustuvorlik qoidasi tegishli ekanini tekshiring.
5.  `datasets/expected-results.json` ni oching. Bir nechta id tanlang, asl murojaatlarni toping va matndan qoida orqali kutilgan category va priority'gacha bo'lgan yo'lni bosib o'ting.
6.  `src/README.md` yarating. Unda dasturning vazifasini, Python 3 ishlatish talabini, qo'llab-quvvatlanadigan JSON va CSV formatlarini hamda natijadagi id, category va priority maydonlarini tasvirlang. Ishga tushirish buyruqlarini keyinroq, dastur tayyor bo'lganda qo'shasiz.

Masalan, `r-001` yozuvida «войти» (kirish) va «пароля» (parol) so'zlari bor. Ma'lumotnoma bo'yicha birinchi moslik access kategoriyasini beradi. Matnda high belgilari yo'q, shuning uchun access kategoriyasi uchun medium ustuvorlik chiqadi. `expected-results.json` da `r-001` yozuvida aynan shu qiymatlar turishi kerak.

**Natija:**

Repozitoriyda `src/README.md` bor, u orqali dasturning vazifasi, muhitga talablar, kirish formatlari va natija maydonlarini tushunish mumkin.

## 2-topshiriq. JSON va CSV uchun yagona ko'rinish

JSON va CSV turlicha ko'rinadi, lekin bu loyihada ular bir xil obyektlarni — identifikator va matnga ega murojaatlarni — o'z ichiga oladi. Agar ularni darhol yagona ichki tuzilmaga keltirsangiz, marshrutlash qoidalari ma'lumot qaysi fayldan kelganini bilishi shart bo'lmay qoladi.

Ichki ko'rinish uchun lug'atlar ro'yxati mos keladi. Har bir lug'at bitta murojaatni tasvirlaydi va ikkita maydonni — id va text'ni — o'z ichiga oladi.

Bunday tuzilmaning shakli bitta qisqartirilgan yozuvda quyidagicha ko'rinadi:

```json
[
    {
        "id": "r-001",
        "text": "Не могу войти в личный кабинет"
    }
]
```

Bu o'qish natijasiga misol, funksiyaning tayyor realizatsiyasi emas.

**Maqsad:** dasturni JSON va CSV'ni o'qishga o'rgatish va ikkala formatdagi yozuvlarni yagona ko'rinishga keltirish.

`src/main.py` da nima qilish kerak:

1.  JSON o'qish funksiyasini yarating. U fayl yo'lini oladi va ildiz massivdan yozuvlarni qaytaradi.
2.  Alohida CSV o'qish funksiyasini yarating. Har bir qator uchun lug'at olish uchun sarlavhalar qatoridan foydalaning. Buning uchun `csv.DictReader` mos keladi, lekin standart kutubxonadan boshqa usulni ham tanlashingiz mumkin.
3.  O'qilgan har bir yozuvni `id` va `text` maydonlariga ega lug'atga keltiring. Qiymatlarni satrlarga (string) aylantiring va chekkalaridagi bo'sh joylarni olib tashlang.
4.  Tozalashdan keyin ikkala majburiy maydonni tekshiring. Agar `id` yoki `text` yo'q bo'lsa yoki bo'sh qolsa, xato bilan qayta ishlashni to'xtating. Foydalanuvchi uchun tushunarli xabarni 4-topshiriqda qo'shasiz.
5.  O'qishni tanlaydigan umumiy funksiya qo'shing. U yo'lni oladi, `.json` yoki `.csv` kengaytmasiga qaraydi va mos funksiyani chaqiradi.
6.  Har bir to'g'ri fayl o'qish natijasini vaqtincha chiqaring va birinchi yozuvga qarang. Ikkala holatda ham bu bo'sh bo'lmagan satrli `id` va `text` ga ega lug'atlar ro'yxati bo'lishi kerak. Tekshirgandan so'ng vaqtincha chiqarishni olib tashlang.

Masalan, tozalashdan keyin

`{"id": 42, "text": " Не работает отчет "}`

yozuvi

`{"id": "42", "text": "Не работает отчет"}`

ko'rinishiga kelishi kerak.

Ushbu topshiriq uchun standart kutubxonadagi json va csv modullari yetarli.

**O'zingizni tekshiring:** o'qish funksiyalari marshrutlashdan mustaqil ishlaydi va shakli bir xil tuzilmani qaytaradi. Kodda faqat tekshirish uchun qo'shilgan vaqtinchalik `print` qolmagan.

**Natija:**

`src/main.py` da JSON va CSV'ni o'qishning alohida funksiyalari hamda o'qish usulini tanlovchi umumiy funksiya bor. Ikkala to'g'ri fayl ham bo'sh bo'lmagan satrli `id` va `text` ga ega yozuvlar ro'yxatiga aylanadi.

## 3-topshiriq. Marshrutlash qoidalari

Endi dastur murojaatlarni fayl formatidan qat'i nazar o'qiy oladi, lekin hali qaror qabul qilmaydi. Keyingi qadam — `datasets/reference/routing-rules.md` dagi qoidalarni kodga o'tkazish.

Bu yerda qoidalar tartibi muhim. Bitta xabarda bir nechta kategoriya belgilari uchrashi mumkin, shuning uchun marshrutizator ma'lumotnomadagi **birinchi moslikni** tanlaydi. Rus matnining odatiy xususiyatini ham hisobga olish muhim: «счет» va «счёт» so'zlari bir xil natijaga olib kelishi kerak.

`r-003` yozuvini ko'rib chiqamiz:

>Дважды списалась оплата за тариф
>
>    ↓ normalizatsiya
>
>дважды списалась оплата за тариф
>
>    ↓ ma'lumotnoma bo'yicha birinchi moslik
>
>category = billing
>
>    ↓ high belgilari yo'q, category access ham, bug ham emas
>
>priority = low

Dasturingiz har bir yozuv uchun shu yo'lni takrorlashi kerak, aniq `id` uchun tayyor javobni saqlamasligi kerak.

**Maqsad:** alohida funksiyalar yordamida murojaatning kategoriyasi va ustuvorligini aniqlash.

Nima qilish kerak:

1.  Kalit so'zlarni qidirishdan oldin matnni normalizatsiya qiling: uni kichik harflarga o'tkazing va ё ni е ga almashtiring. Keyingi barcha tekshiruvlarni normalizatsiya qilingan matnda bajaring.
2.  `category` ni aniqlash funksiyasini yarating. Kategoriyalarni `datasets/reference/routing-rules.md` da yozilgan tartibda tekshiring va topilgan birinchisini qaytaring. Agar moslik bo'lmasa, `other` qaytaring.
3.  Alohida `priority` ni aniqlash funksiyasini yarating. Avval `high` belgilarini tekshiring, so'ng access va `bug` kategoriyalari uchun qoidalarni; qolgan barcha holatlarda `low` qaytaring.
4.  Bitta yozuvni marshrutlash funksiyasini yarating. Unga `id` va `text` li lug'atni bering; javobda `id`, `category` va `priority` li lug'at oling.
5.  Marshrutlash funksiyasiga barcha o'qilgan yozuvlarni birma-bir bering va natijalarni ro'yxatga yig'ing.
6.  Olingan `category` va `priority` ni `datasets/expected-results.json` bilan solishtiring. Yozuvlarni `id` bo'yicha moslang, chunki massivdagi obyektlar tartibi baholanmaydi. Agar qiymat farq qilsa, bu yozuvning qoidalar bo'yicha yo'lini yuqoridagi misoldagi kabi bosib o'ting.

**Natija:**

`src/main.py` da kategoriyani, ustuvorlikni aniqlash va bitta yozuvni marshrutlashning alohida funksiyalari bor. Ikkala to'g'ri to'plam uchun `category` va `priority` qiymatlari `id` bo'yicha kutilganlarga mos keladi.

## 4-topshiriq. Terminaldan ishga tushirish

Hozircha funksiyalarni faqat koddan chaqirish mumkin. Lekin boshqa odam yangi murojaatlar to'plamini qayta ishlamoqchi bo'lganida har safar `main.py` ni ochib, fayllar yo'llarini o'zgartirishi shart emas.

Buning uchun dasturga buyruq qatori interfeysi (CLI) kerak. Foydalanuvchi bitta buyruqni ishga tushiradi va unga manba ma'lumotlarga yo'l hamda natija uchun joyni uzatadi:

`python3 src/main.py --input src/data/requests.json --output src/result.json`

Bu yerda `--input` qaysi faylni o'qishni, `--output` esa yakuniy JSON'ni qayerga saqlashni ko'rsatadi. Yo'llar buyruqdan olinishi kerak, shuning uchun yangi fayl uchun `main.py` ni o'zgartirish shart bo'lmaydi.

Har bir buyruqning tugash kodi (exit code) bor. 0 kodi terminalga hammasi muvaffaqiyatli o'tganini bildiradi. Noldan farqli kod dastur xatoga duch kelganini va kutilgan natijani ola olmaganini bildiradi.

Agar exception qayta ishlanmay qolsa, Python chaqiruvlarning uzun zanjirini — `traceback` ni — chiqaradi. Dasturchiga u nuqsonni qidirishda yordam beradi, lekin shunchaki noto'g'ri yo'l yoki buzilgan fayl uzatgan foydalanuvchiga qisqa va tushunarli xabar kerak.

**Maqsad:** funksiyalarni kodni o'zgartirmasdan terminaldan ishga tushiriladigan dasturga yig'ish.

Nima qilish kerak:

1.  Majburiy `--input` va `--output` argumentlarini qo'shing. Standart `argparse` modulidan foydalanishingiz mumkin.
2.  Yordamni (help) shunday sozlangki, `python3 src/main.py --help` buyrug'i xatosiz tugasin va ikkala argumentning vazifasini tushuntirsin.
3.  Asosiy stsenariyni shunday tartibda yig'ing: `--input` dagi faylni o'qish → barcha yozuvlarni marshrutlash → ro'yxatni `--output` yo'liga saqlash.
4.  Muvaffaqiyatli ishga tushirishda to'g'ri JSON-massivni saqlang va dasturni 0 kod bilan tugating.
5.  Uchta xato kirishni tekshiring va qayta ishlang:
    - kirish faylining yo'li mavjud emas;
    - JSON sintaktik jihatdan buzilgan;
    - yozuvda `id` yoki `text` maydoni yo'q yoki bo'sh.
6.  Har bir xato kirish uchun qisqa tushunarli xabar chiqaring, qayta ishlanmagan traceback ko'rsatmang va dasturni noldan farqli kod bilan tugating.
7.  Dasturni `datasets/requests.json` bilan ishga tushiring va natijani `src/result.json` ga saqlang. Faylni oching va ichida `id`, `category` va `priority` maydonlariga ega JSON-massiv borligiga ishonch hosil qiling.
8.  `src/README.md` ga qayting va amalda tekshirgan buyruqlaringizni qo'shing: `--help` chaqiruvi, JSON'ni qayta ishlash, CSV'ni qayta ishlash va natijani saqlash.

Argumentlar qatorini `split` orqali qo'lda tahlil qilmang: bu ishni `argparse` allaqachon bajara oladi.

Xato holatidagi xatti-harakat quyidagicha ko'rinishi mumkin:

`$ python3 src/main.py --input /tmp/router-file-that-does-not-exist.json --output /tmp/router-result.json`

Ошибка: входной файл не найден *(Xato: kirish fayli topilmadi)*

```bash
$ echo $?

1
```

Xabar matnini va aniq noldan farqli kodni o'zingiz tanlang. Muhimi — foydalanuvchi sababni tushunishi va `traceback` paydo bo'lmasligi. `echo $?` buyrug'i `bash` va `zsh` da oldingi buyruqning tugash kodini ko'rsatadi.

**O'zingizni tekshiring:** `--help` va to'g'ri kirish 0 kod bilan tugaydi; uchta xato kirishning har biri noldan farqli kod va tushunarli xabar bilan tugaydi.

**Natija:**

Repozitoriyda quyidagilar bor:

1.  `src/main.py` — ishga tushirish buyrug'i, JSON saqlash va uchta kutilgan xatoni qayta ishlash bilan;
2.  `src/result.json` — `datasets/requests.json` dan yaratilgan;
3.  `src/README.md` — kodni o'zgartirmasdan takrorlash mumkin bo'lgan buyruqlar bilan.

## 5-topshiriq. Topshirishdan oldin tekshiruv

Bitta faylda ishlaydigan ishga tushirish dastur topshirishga tayyor ekanini hali anglatmaydi. Buning oldidan muhandis asosiy stsenariylarni takrorlaydi va foydalanuvchi fayl yoki ma'lumotlarda xato qilgan vaziyatlarni alohida tekshiradi.

Hozir siz qo'lda qabul tekshiruvini (manual acceptance) o'tkazasiz va natijalarni hamkasblaringiz xuddi shu buyruqlarni takrorlay oladigan qilib qoldirasiz.

**Maqsad:** muvaffaqiyatli va xatoli stsenariylarni tekshirish va dasturning haqiqiy xatti-harakatini yozib qo'yish.

Jadvalda kutilgan natija ishga tushirishdan oldin, haqiqiy natija esa undan keyin yoziladi. `PASS` holati ular mos kelganini bildiradi; `FAIL` — nomuvofiqlik topilganini.

To'g'ri JSON uchun qator ishga tushirishdan oldin quyidagicha ko'rinishi mumkin:

| **Stsenariy** | **Buyruq** | **Kutilgan natija** | **Haqiqiy natija** | **Tugash kodi** | **Holat** |
| --- | --- | --- | --- | --- | --- |
| To'g'ri JSON | `python3 src/main.py --input datasets/requests.json --output src/result.json` | Natija `datasets/expected-results.json` bilan `id` bo'yicha mos keladi | _Ishga tushirgandan keyin to'ldiring_ | _Ishga tushirgandan keyin to'ldiring_ | _Solishtirgandan keyin to'ldiring_ |

Bu faqat shakl namunasi. Tayyor jurnalda maslahatlar o'rniga sizning ishga tushirishingiz faktlari turishi kerak.

Nima qilish kerak:

1.  `src/manual_checks.md` yarating va unga xuddi shunday oltita ustunli jadval qo'shing.
2.  Tekshirish uchun beshta qator tayyorlang:
    - to'g'ri `datasets/requests.json`;
    - to'g'ri `datasets/requests.csv`;
    - mavjud bo'lmagan fayl yo'li;
    - buzilgan `datasets/requests.json`;
    - `datasets/missing-field.json` dagi `text`siz yozuv.
3.  Ishga tushirishdan oldin har bir qator uchun buyruq va kutilgan natijani to'ldiring. Ikkita to'g'ri fayl uchun `datasets/expected-results.json` bilan `id` bo'yicha moslikni kuting; uchta xato uchun — tushunarli xabar va noldan farqli kod.
4.  Har bir buyruqni bajaring. Undan keyin darhol haqiqiy chiqish va tugash kodini yozib qo'ying, shunda keyingi buyruq bu kodni ustidan yozib yubormaydi.
5.  Kutilganni fakt bilan solishtiring. Mos kelsa `PASS` qo'ying. Mos kelmasa `FAIL` qo'ying, nomuvofiqlikni tasvirlang va haqiqiy natijani xohlangan natija bilan almashtirmang.

**O'zingizni tekshiring:** `src/manual_checks.md` da barcha beshta majburiy stsenariy bor va har bir qatorda buyruq, kutilgan natija, fakt, tugash kodi va holat to'ldirilgan.

**Natija:**

Repozitoriyda `src/manual_checks.md` bor: beshta bajarilgan stsenariy, kutilgan va haqiqiy natijalar, tugash kodlari va holatlar bilan. `src/README.md` dagi buyruqlar, joriy `src/main.py` va `src/result.json` o'zaro kelishilgan.

________

Asosiy topshiriqlarni bajargandan so'ng, xohishingizga ko'ra qo'shimcha funksionallikni amalga oshirishingiz mumkin: kategoriyalar bo'yicha murojaatlar sonini chiqarish yoki kalit so'zlarni alohida lokal faylga chiqarish.

Bu amal `src/result.json` formatini o'zgartirmasligi kerak.

Bu qadam tekshiruv uchun majburiy emas.

**P2P-tekshiruvdan oldin:**

- `src/README.md` dagi buyruqlarni yangi terminalda takrorlang;
- `src/result.json` kodning joriy versiyasi tomonidan yaratilganiga ishonch hosil qiling;
- bitta yozuvning `--input` dan yakuniy JSON'gacha bo'lgan yo'lini ovoz chiqarib aytib berishga tayyorlaning.

Uchrashuvda tekshiruvchi to'g'ri kirish faylining nusxasiga bitta sintetik yozuv qo'shadi. Ishga tushirishdan oldin qanday `category` va `priority` olishni kutayotganingizni tushuntiring, so'ng yozuvni dasturni o'zgartirmasdan qayta ishlang.

💡 [Bu yerni bosing](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624), **shu loyiha bo'yicha fikr-mulohazangizni bildirish uchun**. Bu anonim bo'lib, jamoamizga o'qitishni yaxshilashga yordam beradi. So'rovnomani loyihani bajarganingizdan so'ng darhol to'ldirishni tavsiya qilamiz.
