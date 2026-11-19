# Улаанбаатар хотын хүн ам, өрх, гишүүнчлэлийн дашбоард

Улаанбаатар хотын 9 дүүрэг, 204 хорооны хүн ам, өрх, нягтрал, гишүүдийн тоо ба бүтцийг газрын зураг, график, хүснэгтээр харуулна. Build алхамгүй, цэвэр статик сайт тул Vercel дээр шууд байршина.

## Бүтэц

```
ub-dashboard/
├── vercel.json            # Vercel тохиргоо: public/ хавтсыг үйлчилнэ, аюулгүй байдлын толгой
├── public/                # Вэбд гарах бүх файл
│   ├── index.html
│   ├── favicon.svg
│   ├── css/tokens.css     # Өнгө, үсэг, зай (гэрэл/харанхуй горим)
│   ├── css/ub.css         # Дашбоардын загвар
│   ├── js/d3.min.js       # D3 v7 (ISC лиценз, d3.LICENSE.txt)
│   ├── js/dash-runtime.js # Өгөгдөл ачаалах, шүүлтүүр, дахин зурах жижиг орчин
│   ├── js/app.js          # Тооцоолол, газрын зураг, график, хүснэгт
│   └── data/
│       ├── khoroo.csv          # 204 хорооны хүн ам, өрх, талбай, гишүүдийн тоо
│       ├── members_scope.csv   # Хот, дүүргийн гишүүдийн задаргаа (хүйс, нас, хөдөлгөөн, анкет)
│       ├── members_monthly.csv # Сар бүрийн шинэ гишүүд
│       └── boundaries.json     # Хороо, дүүргийн хялбарчилсан хил (UTM 48N, метр)
└── tools/                 # Өгөгдөл бэлтгэх хэрэгсэл (вэбд гарахгүй)
    ├── build_data.py      # shapefile → boundaries.json, khoroo_base.csv
    ├── build_excel.py     # Нэгтгэлийн Excel → khoroo.csv, members_*.csv
    ├── khoroo_base.csv    # Хорооны код, талбай, хилийн файлын гэр хорооллын хүн ам
    ├── requirements.txt
    └── package.json
```

## Vercel дээр байршуулах

### А. Vercel CLI-аар (хамгийн хурдан)

```bash
npm i -g vercel
cd ub-dashboard
vercel          # эхний удаа: нэвтэрч, төслийн нэрээ сонгоно → preview холбоос гарна
vercel --prod   # үндсэн (production) хаяг руу гаргана
```

Асуулт гарвал анхдагч хариултыг сонгоно. Тохиргоо `vercel.json`-д байгаа тул build command, output directory-г өөрчлөх шаардлагагүй.

### Б. GitHub-аар (автомат шинэчлэлттэй)

1. `ub-dashboard` хавтсыг шинэ GitHub репо болгон push хийнэ.
2. vercel.com → **Add New… → Project** → репогоо **Import** хийнэ.
3. **Framework Preset**: `Other`. Бусад талбарыг хоосон орхино (`vercel.json` удирдана).
4. **Deploy**. Цаашид `main` салбар руу push хийх бүрт автоматаар шинэчлэгдэнэ.

> **Хандалтын тухай (заавал уншина уу):** Vercel-ийн production хаяг анхдагчаар хэн ч нээх боломжтой бөгөөд `public/data/` доторх файлуудыг шууд татаж болно. Энэ сайтад хороо бүрийн гишүүдийн тоо орсон тул нийтэд нээлттэй байршуулахаас өмнө Project → Settings → **Deployment Protection**-ийг асааж, зөвхөн эрх бүхий хүмүүст нээнэ үү (зарим сонголт төлбөртэй багцад л байдаг).

## Орон нутагт турших

`fetch` ашигладаг тул `index.html`-ийг шууд давхар дарж нээвэл ажиллахгүй. Жижиг сервер ажиллуулна:

```bash
cd ub-dashboard
python3 -m http.server 8000 -d public   # эсвэл: npx serve public
# http://localhost:8000
```

## Өгөгдлийг шинэчлэх

### Хүн ам, өрх, гишүүд (шинэ нэгтгэл ирэх бүрт)

```bash
cd ub-dashboard/tools
pip install -r requirements.txt
python build_excel.py --excel /зам/2026.11.19_Negtgel.xlsx
```

Excel-д хоёр хуудас байх ёстой: «Өрх, хүн амын тоо, хороогоор» (Дүүрэг, Хорооны нэр, Нийт өрхийн тоо, Хүн амын тоо) ба «Гишүүнчлэл» (2-р мөр толгой). Скрипт хороо бүрийг газрын зурагтай тулгаж, таарахгүй мөр байвал зогсоно. Дараа нь дахин deploy хийнэ.

**Нууцлал.** Скрипт регистр, нэр, утас, MEMBER_ID-г огт уншдаггүй. Гаралтад зөвхөн нэгтгэсэн тоо гарна: хороо түвшинд гишүүдийн тоо, хүйс, нас, хөдөлгөөн, анкетын задаргаа дүүрэг ба хотын түвшинд. 5-аас цөөн гишүүнтэй дүүргийн задаргааг хоосон үлдээнэ (`MIN_GROUP`). Excel файлыг өөрийг нь репо эсвэл `public/` руу хэзээ ч бүү хуул (`.gitignore` хаасан).

### Хил (shapefile шинэчлэгдвэл)

```bash
cd ub-dashboard/tools
npm install
python build_data.py \
  --khoroo /зам/204_Shine_hil_shinechilsen1.shp \
  --duureg /зам/Dvvreg_hil_last9.shp
python build_excel.py --excel /зам/2026.11.19_Negtgel.xlsx   # дараа нь заавал
```

`build_data.py` нь хоёр полигонтой хороог нэгтгэж, дүүргийн кодыг `BA_KO_C`-ээс авч, 9 дүүргийн хилийг хорооноос гаргаж, хилийг 15 м-ээр хялбарчилна.

## Шүүлтүүрийг холбоосоор хуваалцах

Сонгосон шүүлтүүр URL-д хадгалагдана, жишээ нь:

```
/?district=Баянгол&metric=gerpct
/?district=Хан-Уул&khoroo=HUD_21&metric=pop
/?level=district&metric=hhsize
```

`metric`: `dens`, `pop`, `hh`, `hhsize`, `members`, `mem_rate`, `gerpct` · `level`: `khoroo`, `district`.

## Кодын тухай

`js/app.js` нь Claude дээрх дашбоардын `dash` API-аар бичигдсэн бөгөөд `js/dash-runtime.js` тэр API-г (өгөгдөл ачаалах, `calc`, `draw`, шүүлтүүр) статик сайтад хэрэгжүүлдэг. Шинэ эх өгөгдөл нэмэх бол `dash-runtime.js` доторх `SOURCES`-д бүртгэнэ, шинэ шүүлтүүр нэмэх бол `DEFAULTS` (сонголт хязгаартай бол `ALLOWED`)-д нэмнэ.
