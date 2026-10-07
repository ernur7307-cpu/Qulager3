# Qulager — Ет және сүт өнімдері дүкені

Түркістандағы Qulager дүкенінің сайты (HTML + CSS + JavaScript, серверсіз).

## Құрылымы

```
qulager/
├── index.html          ← басты бет
├── meat.html           ← Ет өнімдері
├── dairy.html          ← Сүт өнімдері
├── poultry.html        ← Аң-құс
├── sub.html            ← Суб набор
├── spices.html         ← Дәмдеуіштер
├── own.html            ← Біздің өнімдер
├── css/style.css       ← барлық беттің ортақ стилі
├── js/cart.js          ← себет (sessionStorage), барлық бетте қосылған
├── js/main.js          ← басты беттің скрипті (мәзір, FAQ, анимация)
├── js/reviews.js       ← пікірлер (оқу, жазу, өз пікірін өшіру)
├── js/firebase-config.js ← Firebase және Cloudinary баптаулары (пікірлер, фото/видео)
├── firestore.rules     ← Firebase қауіпсіздік ережелері
├── images/             ← логотип, санаттар, өнім суреттері
└── database/qulager.sql← SQL дерекқор (санаттар, өнімдер, тапсырыстар)
```

## Себет қалай жұмыс істейді
- «Себетке қосу» батырмасында `data-id`, `data-name`, `data-price` бар.
- `cart.js` себетті `sessionStorage` (`qulager_cart` кілті) ішіне сақтайды: беттен бетке өткенде **жоғалмайды**, ал сайтты (қойындыны) жауып қайта ашқанда себет **бос** болады.
- Бірдей өнімді қайта қоссаң, саны (`qty`) артады; себетте +/− және өшіру бар.
- «WhatsApp арқылы тапсырыс беру» себеттегі тізімді `+7 776 727 8100` нөміріне жібереді.

## Жергілікті ашу
`index.html` файлын екі рет шерту арқылы ашуға болады. Бірақ кейбір браузер (Firefox) `file://` режимінде
себетті беттер арасында бөліспейді, сондықтан VS Code-та **Live Server** кеңейтімімен ашқан дұрыс
(немесе төменде GitHub Pages сілтемесімен).

## Жаңа өнім қосу
Санат бетіндегі (мысалы `meat.html`) `<article class="menu-card">` блогын көшіріп, `data-id`, `data-name`, `data-price`
және `images/products/...` суретін өзгерт.

Нақты фото қою: суретті `images/products/` ішіне салып (мысалы `beef.jpg`), HTML-дегі `src="images/products/beef.svg"` жолын `beef.jpg` деп өзгерт.

## Дерекқор (SQL)
```bash
sqlite3 qulager.db < database/qulager.sql
```
Файлдың соңында емтиханға арналған мысал сұраулар (JOIN, GROUP BY, WHERE) бар.

## GitHub-қа салу
```bash
cd qulager
git init
git add .
git commit -m "Qulager дүкенінің сайты"
git branch -M main
git remote add origin https://github.com/<логин>/qulager.git
git push -u origin main
```
Содан кейін GitHub-та: **Settings → Pages → Branch: main / (root) → Save**.
Сайт `https://<логин>.github.io/qulager/` мекенжайында ашылады.

## Пікірлерді қосу (Firebase + Cloudinary)

Сайт GitHub Pages-та тұр, оның өз сервері жоқ. Пікірлер мен олардың фото/видеосы барлық келушіге көрінуі үшін
екі тегін қызмет қосылады (бір рет, ~10 минут). Қосылмағанша пікір формасы жабық тұрады (демо режим жоқ).

### 1) Firebase — пікір мәтіні (тегін, Spark жоспары)
1. https://console.firebase.google.com → **Add project** (Analytics керек емес).
2. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable**.
3. **Build → Firestore Database → Create database** (region: `eur3` немесе `europe-west`).
   Содан **Rules** қойындысына `firestore.rules` файлының мәтінін толық қойып, **Publish** бас.
4. **Project settings (⚙) → Your apps → Web (`</>`)** → қолданбаны тіркеп, шыққан `firebaseConfig`
   мәндерін `js/firebase-config.js` ішіндегі `QULAGER_FIREBASE` блогына көшір (`apiKey`, `authDomain`, `projectId`, `appId`).

> Firebase Storage-ты (файл сақтау) пайдаланбадық: жаңа жобаларда ол ақылы Blaze жоспарын (банк картасын) талап етеді.

### 2) Cloudinary — фото және видео (тегін жоспар)
1. https://cloudinary.com → тіркел (**Free**). Dashboard-тан **Cloud name** мәнін ал.
2. **Settings → Upload → Upload presets → Add upload preset**:
   **Signing mode: Unsigned**, **Folder:** `qulager-reviews` → Save. Preset атауын көшір.
   (Қаласаң preset ішінде **Allowed formats** көрсет: jpg, png, webp, heic, mp4, mov, webm.)
3. Екі мәнді `js/firebase-config.js` ішіндегі `QULAGER_CLOUDINARY` блогына жаз (`cloudName`, `uploadPreset`).
   Бос қалса — пікірге тек мәтін жазуға болады, фото/видео түймесі шықпайды.

### Ереже және шектеулер
- Пікірді кез келген адам оқиды және жазады; өшіруді **тек пікірді жазған адам** жасай алады
  (ол браузердегі жасырын `uid` арқылы анықталады, сервер ережесі `firestore.rules` оны тексереді).
  Басқа құрылғыдан немесе браузер деректерін тазалағаннан кейін өз пікірін өшіре алмайды.
- Бір пікірге бір фото немесе бір видео: фото ≤ 10 МБ, видео ≤ 40 МБ. Жіберу арасында 20 секунд күту бар.
- Пікір өшірілгенде Cloudinary-дағы файл қалып қояды (браузерден өшіру қауіпсіз емес). Оны өзің
  Cloudinary-дың **Media Library** бөлімінен тазалай аласың. Тегін жоспарда орын мен трафик шектеулі.
- Модерация: орынсыз пікір түссе, оны Firebase консолінде (**Firestore → reviews**) өшір.

## Себет және мекенжай
Себетте «Жеткізу мекенжайы» өрісі бар. Ол толтырылмайынша (кемінде 5 таңба) тапсырыс WhatsApp-қа кетпейді.
Мекенжай браузерде сақталады, келесі жолы қайта жазбаса да болады, және WhatsApp хабарламасына қосылады.

## Телефон нөмірлері мен тапсырыс
- Байланыс бөлімінде екі нөмір: `8 747 727 8100` және `8 776 727 8100` (басса — қоңырау, жанындағы батырма — WhatsApp).
- Себеттен тапсырыс `js/cart.js` ішіндегі `PHONE` нөміріне (қазір `77767278100`) жіберіледі; ауыстыруға болады.

## «Qulager» жүктеу экраны
Тек сайтқа (басты бетке) алғаш кіргенде бір рет шығады. Беттер арасында жүргенде, басты бетке қайтқанда
қайта шықпайды (браузер қойындысын жапқанша).
