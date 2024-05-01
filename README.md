# Mashinani o'rganish va ilg'or tahlillar yordamida IoT sensori ma'lumotlarini tahlil qiling

Ushbu kod namunasi IBM Db2 Event Store bilan o'zaro ishlash uchun Jupyter noutbuklaridan foydalanishni namoyish etadi -- ma'lumotlar bazasi ob'ektlarini yaratishdan tortib, ilg'or tahlil va mashinani o'rganish modelini ishlab chiqish va joylashtirishgacha.

Ushbu kod namunasida ishlatiladigan namunaviy ma'lumotlar haqiqiy sanoat IoT sensorlari tomonidan to'plangan ma'lumotlarni simulyatsiya qiladi. IoT namunasi maʼlumotlariga sensor harorati, atrof-muhit harorati, quvvat sarfi va noyob sensor identifikatorlari va qurilma identifikatorlari bilan aniqlangan bir guruh sensorlar uchun vaqt tamgʻasi kiradi.
Oddiy IBM Streams oqimi CSV faylidan Event Store jadvaliga namuna ma'lumotlarini oqimlash uchun ishlatiladi.

Db2 Voqealar do'koni - bu Apache Spark va Apache Parket ma'lumotlar formatida qurilgan, katta tuzilgan ma'lumotlar hajmlari va real vaqtda tahlil qilish uchun mo'ljallangan xotiradagi ma'lumotlar bazasi. Yechim voqealarga asoslangan ma'lumotlarni qayta ishlash va tahlil qilish uchun optimallashtirilgan. U IoT yechimlari, toʻlovlar, logistika va veb-tijorat kabi voqealarga asoslangan rivojlanayotgan ilovalarni qoʻllab-quvvatlashi mumkin. U moslashuvchan, kengaytiriladigan va vaqt o'tishi bilan o'zgaruvchan biznes ehtiyojlaringizga tezda moslasha oladi. Db2 Event Store bepul ishlab chiquvchi nashrida va hozir yuklab olishingiz mumkin bo'lgan korporativ nashrda mavjud. Korxona nashri oldindan ishlab chiqarish va sinovdan o‘tkazish uchun bepul, qo‘shimcha ma’lumot olish uchun [rasmiy mahsulot veb-sahifasiga](https://www.ibm.com/products/db2-event-store) tashrif buyuring.

> Eslatma: Db2 Event Store IBM Watson Studio bilan yaratilgan

Ushbu kod namunasini to'ldirgandan so'ng, qanday qilishni tushunasiz:

* Python va Jupyter noutbukidan foydalanib Db2 Event Store bilan o'zaro aloqada bo'ling.
* Db2 Event Store'ga ma'lumotlarni yuborish uchun IBM Streams dasturidan foydalaning.
* Matplotlib diagrammalaridan foydalangan holda ma'lumotlarni ingl.
* Mashinani o'rganish modelini yarating va sinab ko'ring.
* Watson Machine Learning bilan modelni o'rnating va foydalaning.

![architecture](doc/source/images/architecture.png)

## Flow

1. Db2 Event Store ma'lumotlar bazasi va jadvalini yarating.
2. IoT ma'lumotlar to'plamini Db2 Event Store'ga yuboring.
3. Spark SQL yordamida jadvalga so‘rov o‘tkazing.
4. Matplotlib diagrammalari yordamida ma’lumotlarni tahlil qiling.
5. Mashinani o'rganish modelini yaratish va joylashtirish.

## Db2 EventStore-ni Cloud Pak for Data (CPD) da ishga tushirish

Agar sizda Cloud Pak for Data namunasi oʻrnatilgan boʻlsa, Db2 EventStore’ni CPD’da ishga tushirish uchun [Cloud Pak Readme](README_CLOUDPAK.md) ga amal qiling.
Aks holda, komponentlarni mahalliy sifatida joylashtirish uchun quyidagi amallarni bajaring.

## Qadamlar

1. [Clone the repo](#1-clone-the-repo)
2. [Install the prerequisites](#2-install-the-prerequisites)
3. [Create an IBM Db2 Event Store database and table](#3-create-an-ibm-db2-event-store-database-and-table)
4. [Add the sample IoT data](#4-add-the-sample-iot-data)
5. [Query the table](#5-query-the-table)
6. [Analyze the data](#6-analyze-the-data)
7. [Create and deploy a machine learning model](#7-create-and-deploy-a-machine-learning-model)

### 1. Repozitariyni yuklab olish

Clone the `iot-data-analytics` repo locally. In a terminal, run:

```bash
git clone https://github.com/shohzod-xayrullayev/iot-data-analytics.git
```

### 2. Oldindan shartlarni o'rnating

#### Voqealar do'konini o'rnating

> Eslatma: Ushbu kod namunasi EventStore-DeveloperEdition 1.1.4 bilan ishlab chiqilgan

1. IBM® Db2® Event Store Developer Edition-ni Mac, Linux yoki Windows-da ko'rsatmalarga rioya qilib o'rnating [here.](https://www.ibm.com/products/db2-event-store)

#### Install IBM Streams

> Eslatma: Maʼlumotlarni yuklash uchun taqdim etilgan Jupyter paketlaridan foydalanmoqchi boʻlsangiz, bu ixtiyoriy.

1. IBM Streams 4.3.1 uchun Quick Start Edition(QSE) Docker tasvirini [bu yerdagi](https://www.ibm.com/support/knowledgecenter/SSCRJU_4.3.0/com.ibm.streams) ko‘rsatmalariga rioya qilib o‘rnating. qse.doc/doc/qse-docker.html).

1. Event Store Developer Edition 1.1.4 [bu yerdan](https://github.com/IBMStreams/streamsx.eventstore/releases/tag/v1.2.0-Developer-v1.1.4) uchun asboblar to'plamini yuklab oling.

1. Asboblar to'plamining mazmunini chiqarib oling. Masalan, siz Quick Start Edition-ni o'rnatayotganda mahalliy katalogga moslashtirilgan hostdir-da "toolkits" katalogini yaratishingiz va fayllarni ushbu katalogga chiqarishingiz mumkin.

   ```bash
   mkdir $HOME/hostdir/toolkits
   tar -zxvf  streamsx.eventstore_1.2.0-RELEASE.tgz -C $HOME/hostdir/toolkits/
   ```

### 3. IBM Db2 Event Store ma'lumotlar bazasi va jadvalini yarating

Db2 Event Store ma'lumotlar bazasi va jadvali taqdim etilgan Jupyter noutbuklaridan biri bilan yaratilishi mumkin. Mavjud ma'lumotlar bazasi yoki jadvalingizni tashlab qo'yishingiz kerak bo'lsa, paket sharhlariga qarang.

Noutbuk yaratish va ishga tushirish uchun Db2 Event Store UI dan quyidagi tarzda foydalaning:

1. Yuqori chap burchakdagi `☰` ochiladigan menyudan `Mening noutbuklarim`-ni tanlang.
1. “Noutbuklar qo‘shish” tugmasini bosing.
1. "Fayldan" yorlig'ini tanlang.
1. Ism bering.
1. “Faylni tanlash” tugmasini bosing va klonlangan repodagi “noutbuklar” katalogiga o‘ting. **`Event_Store_Table_Creation.ipynb`** nomli Jupyter notebook faylini oching.
1. Pastga aylantiring va “Noutbuk yaratish” tugmasini bosing.
1. Birinchi kod katagida `HOST` doimiysini tahrirlang. Bu yerda siz uy egasining IP manzilini kiritishingiz kerak bo'ladi.
1. “Hujayra ▷ Hammasini ishga tushirish” menyusidan foydalanib paketni ishga tushiring yoki “Play” tugmasi bilan yacheykalarni alohida ishga tushiring.

### 4. IoT maʼlumotlarining namunasini qoʻshing

#### Ma'lumotlarni yaratish

Ushbu ombor 1 million yozuvni o'z ichiga olgan CSV formatida yaratilgan namunaviy IoT ma'lumotlar to'plamini o'z ichiga oladi. Namunaviy CSV maʼlumotlar toʻplamini “data/sample_IOT_table.csv” sahifasida topish mumkin.

Shu bilan bir qatorda, foydalanuvchi tomonidan belgilangan sonli yozuvlarni oʻz ichiga olgan CSV maʼlumotlar toʻplami taqdim etilgan Python skripti yordamida “data/generator.py” orqali yaratilishi mumkin. Skriptni ishga tushirish uchun pandalar va NumPy o'rnatilgan Python muhiti talab qilinadi.

```bash
cd db2-event-store-iot-analytics/data
python ./generator.py -c <Record Count>
```

#### Stream the data into Event Store

Agar siz IBM Streams dasturini o'rnatgan bo'lsangiz, Voqealar do'koniga namuna ma'lumotlarini yuborish uchun oqimlar oqimidan foydalaning. Aks holda, ma'lumotlar tasmasi paket yorliq sifatida taqdim etilgan.

IBM Streams yoki Jupyter noutbuki uchun maʼlumotlar tasmasi koʻrsatmalarini kengaytirish uchun bosing. Birini tanla:

<details><summary>IBM Streams oqimidan foydalaning</summary>
<p>

1.vnc://streamsqse.localdomain:5905 manzilida IBM Streams QSE-ga ulanish uchun VNC-dan foydalaning.

   > **Tip:** On macOS, you can use Finder's menu: `Go ▷ Connect to Server...` and connect to `vnc://streamsqse.localdomain:5905`, and then in your session use `Applications ▷ System Tools ▷ Settings ▷ Devices ▷ Displays` to set the display `Resolution` to `1280 x 1024 (5:4)`.

   ![connect_to_server.png](doc/source/images/connect_to_server.png)

1.Streams Studio-ni ishga tushiring

   ![streams_studio.png](doc/source/images/streams_studio.png)

1. Yangi loyiha yarating

    - [ ] Ish joyini tanlang

    - [ ] Agar "Oqimlar domeniga ulanishni qo'shish" so'ralsa, "streamsqse.localdomain" ni tanlash uchun "Domenni topish..." tugmasidan foydalaning.

    - [ ] Yangi loyiha yaratish uchun yuqori chapdagi ochiladigan menyudan foydalaning. "Sehrgarni tanlash" so'ralganda, "IBM Streams Studio ▷ SPL Project" dan foydalaning, "Keyingi" tugmasini bosing, nom bering va "Finish" tugmasini bosing.

1. Voqealar do'koni asboblar to'plamini almashtiring

    - [ ] `Project Explorer` yorlig'idan foydalanib, yangi loyihangizni kengaytiring, `Bog'liqlar` ustiga sichqonchaning o'ng tugmasini bosing va `Bog'liqlarni tahrirlash`-ni tanlang. `com.ibm.streamsx.eventstore` 2.0.3 versiyasini olib tashlang. Bu bizning 1.1.4 Developer Edition Event Store bilan ishlamaydigan yangiroq versiya.

    - [ ] “Oqimlar Explorer” yorlig‘idan foydalanib, “Asboblar to‘plami joylashuvi”ni ko‘rsatish uchun “IBM Streams Installations” va “IBM Streams” ni kengaytiring. Sichqonchaning o'ng tugmachasini bosing va "Asboblar to'plamining joylashuvini qo'shish ..." -ni tanlang. `Directory...` tugmasidan foydalaning va `/home/streamsadmin/hostdir/toolkits` katalogini qo'shing (bu erda siz streamsx.eventstore_1.2.0-RELEASE.tgz asboblar to'plamini chiqargansiz).

    - [ ] `Project Explorer` yorlig'iga qayting, `Bog'liqlar` ustiga sichqonchaning o'ng tugmasini bosing va `Bog'liqlarni tahrirlash`, `Qo'shish` va `Browse`-ni tanlang. “com.ibm.streamsx.eventstore 1.2.0” ni tanlang va “OK” tugmasini bosing. Endi biz 1.1.4 Developer Edition bilan ishlaydigan yuklab olingan versiyadan foydalanmoqdamiz va yangi versiyaga e'tibor bermaymiz.

     ![add_toolkit_location.png](doc/source/images/add_toolkit_location.png)

1. Fayl manbangizni yarating

    - [ ] Klonlangan repodan `data/sample_IOT_table.csv` faylini xost-direktoringizga nusxalang.

    - [ ] "Project Explorer" yorlig'ida loyihangizni o'ng tugmasini bosing va "Yangi ▷ Main Composite" ni tanlang va standart "Main.spl" ni yarating.

      ![main_composite.png](doc/source/images/main_composite.png)

    - [ ] `Main.spl` uchun SPL Grafik muharriridan foydalanib, `Asboblar to`plami ▷ spl ▷ spl.adapter ▷ FileSource ▷ FileSource`ni `Asosiy` maydoniga sudrab olib tashlang.

      ![file_source.png](doc/source/images/file_source.png)

    - [ ] FileSource oynasini ikki marta bosing, chap yon panelda `Param` ni tanlang va qiymatni `"/users/streamsadmin/hostdir/sample_IOT_table.csv"` (ma'lumotlar faylini joylashtirgan joy) qilib o'rnating. "Format" parametrini qo'shing va qiymatni "csv" ga o'rnating.

      ![file_source_params.png](doc/source/images/file_source_params.png)

    - [ ] Chap yon panelda “Chiqish portlari”ni tanlang. `<extends>` qatorini olib tashlang. "Atribut qo'shish..." tugmasini bosing. CSV fayli va Voqealar doʻkoni jadvali mazmuniga mos keladigan chiqish oqimi sxemasini yaratish uchun atributlar va turlarni qoʻshing. Bu erda atribut nomlari muhim emas, lekin turlari muhim. Quyidagi misolga amal qiling.

![output_stream_schema.png](doc/source/images/output_stream_schema.png)

    - [ ] FileSource xususiyatlarini yoping va saqlang.

1. Event Store sink yarating

    - [ ] `Main.spl` uchun SPL Grafik muharriridan foydalanib, `Asboblar to`plami ▷ com.ibm.streamx.eventstore ▷ com.ibm.streamx.eventstore ▷ EventStoreSink`ni `Asosiy` maydoniga sudrab o`tkazing (bo`ling) 1.2.0 versiyasini oling).

    - [ ] Sichqonchadan foydalaning va FileSource chiqish yorligʻidan EventStoreSink kiritish yorligʻiga torting.

      ![connected.png](doc/source/images/connected.png)

    - [ ] EventStoreSink oynasini ikki marta bosing, chap yon panelda `Param` ni tanlang va quyidagi koʻrsatilganidek, ulanish satri, maʼlumotlar bazasi nomi va jadval nomini oʻrnatish uchun qiymatlarni tahrirlang (lekin oʻz Voqealar doʻkonining IP manzilini almashtiring). Yoping va saqlang.

      ![event_store_params.png](doc/source/images/event_store_params.png)

1. Ishga tushirish

- [ ] Barcha o'zgarishlarni saqlang va qurilishni tugatsin.

    - [ ] Ilovangizni (`Asosiy`) sichqonchaning oʻng tugmasi bilan bosing va `Ishga tushirish ▷ Active Build Config-ni mustaqil ravishda ishga tushiring`.

      ![launch_standalone.png](doc/source/images/launch_standalone.png)

</p>
</details>

<details><summary>Ma'lumotlarni yuklash uchun Jupyter paektidan foydalaning</summary>
<p>

CSV kiritish faylini maʼlumotlar aktivi sifatida qoʻshish uchun Db2 Event Store UI dan foydalaning.

1. Yuqori chap burchakdagi `☰` ochiladigan menyudan `Mening noutbuklarim`-ni tanlang.

    ![mening_notebooks_ga_go_kun](doc/source/images/go_to_notebooks.png)

1. Pastga aylantiring va “maʼlumotlar aktivlarini qoʻshish” tugmasini bosing.

    ![mening_notebooksga_qo'shish](doc/source/images/add_to_notebooks.png)

1. "Browse" tugmasini bosing va klonlangan repodagi "ma'lumotlar" katalogiga o'ting. Sample_IOT_table.csv faylini oching.

    ![data_assets](doc/source/images/data_assets.png)

Noutbukni qo'shish va ishga tushirish uchun yuqoridagi kabi amallarni bajaring. Bu safar **`Event_Store_Data_Feed.ipynb`** nomli faylni tanlang.

Paket siz loyiha aktivi sifatida qo'shgan CSV faylidan bir million yozuv bilan jadvalni yuklaydi.

</p>
</details>

### 5. Jadvalga so'rov o'tkazing

Noutbukni qo'shish va ishga tushirish uchun xuddi shu jarayonni bajaring. Bu safar **`Event_Store_Querying.ipynb`** nomli faylni tanlang.

Ushbu paket IBM Db2 Event Store ma'lumotlar bazasida saqlangan ma'lumotlarni so'rash bo'yicha eng yaxshi amaliyotlarni namoyish etadi. Davom etishdan oldin jadvalni muvaffaqiyatli yaratganingizni va yuklaganingizni tekshiring.

### 6. Ma'lumotlarni tahlil qiling

Keyinchalik, ma'lumotlar tahlili paketini ishga tushiring. **`Event_Store_Data_Analytics.ipynb`** faylidan foydalaning.

Ushbu paket turli xil ma'lumotlarni tahlil qilish vazifalarini bajarish uchun IBM Db2 Event Store bir nechta mashhur ilmiy vositalar bilan qanday birlashtirilishi mumkinligini ko'rsatadi. paketni aylanib chiqayotib, siz ma'lumotlarni filtrlash uchun o'rganasiz va o'lchovlarning korrelyatsiyasi va kovariatsiyasiga misol keltirasiz. Ma'lumotlarni vizualizatsiya qilish uchun siz ba'zi diagrammalardan ham foydalanasiz.

#### Harorat ma'lumotlari uchun ehtimollik chizmalari, quti chizmalari va gistogrammalar

![plots](doc/source/images/plots.png)

#### Scatter plots to show the relationship between measurements

![scatters](doc/source/images/scatters.png)

### 7. Mashinani o'rganish modelini yarating va o'rnating

Ushbu bo'lim mashinani o'rganish modelini yaratish va qo'llashni ko'rsatadi. Noutbuk IoT harorat sensori ma'lumotlarimizdan bashorat qilish modelini yaratish va sinab ko'rish uchun Spark MLlib-dan foydalanadi. Oxirida u modelni qanday joylashtirish va ishlatishni ko'rsatadi.

**`Event_Store_ML_Model_Deployment.ipynb`** faylidan foydalanib, paketni yuklang.

Agar siz Db2 Event Store-ning **Enterprise Edition**-dan foydalanayotgan bo'lsangiz, noutbuk Watson Studio Local bilan yaratilgan Db2 Event Store yordamida modelni o'rnatadi. Noutbukni avvalgidek ishga tushirishingiz mumkin.

Agar siz Db2 Event Store ning **Developer Edition** dan foydalanayotgan bo‘lsangiz, joylashtirishni yakunlash uchun sizga IBM Cloud Watson Machine Learning xizmati namunasi kerak bo‘ladi. Joylashtirish uchun siz quyidagi amallarni bajarishingiz kerak bo'ladi:

* Tizimga kiring va xizmatni yarating [bu yerda](https://cloud.ibm.com/catalog/services/machine-learning).
* [bu yerda](https://dataplatform.cloud.ibm.com/docs/content/wsj/analyze-data/ml-authentication.html) koʻrsatmalaridan foydalanib API kaliti yoki IAM tokenini yarating. Keyingi qadam uchun ushbu kalit/tokenni qo'lingizda saqlang.
* *Db2 Event Store Developer Edition va IBM Cloud-da Machine Learning bilan modelni metama’lumotlar bilan saqlang* sarlavhali bo‘limga o‘ting.
"wml_credentials" bo'limida apikey va url manzilingizni xuddi shunday o'rnating

```
wml_credentials = {
  "url": "https://us-south.ml.cloud.ibm.com",
  "apikey": "<apikey>"
}
```

![wml_creds](doc/source/images/wml_creds.png)


<!-- * `Xizmat ma`lumotlari` va keyin `Yangi hisob ma`lumotlari` va `Qo`shish` ni bosing.
* “Hisob maʼlumotlarini koʻrish” dan foydalaning va JSON hisob maʼlumotlaridan nusxa oling. -->
<!-- * Notebookda `wml_credentials` o'zgaruvchisini o'rnatish uchun JSON-dan foydalanasiz. -->
* Notebook Watson-machine-learning-clientni o'rnatadi. O'rnatishdan so'ng, odatda yadroni qayta ishga tushirishingiz va noutbukni yuqoridan qayta ishga tushirishingiz kerak.

Model qurilgan va o'rnatilgandan so'ng, siz osongina unga yangi o'lchov yuborishingiz va bashorat qilingan haroratni olishingiz mumkin (bir vaqtning o'zida yoki partiyalarda).

#### Yangi maʼlumotlar nuqtasi berilgan

``` piton
new_data = {"deviceID" : 2, "sensorID": 24, "ts": 1541430459386, "ambient_temp": 30, "power": 10}
```

#### Natija taxmin qilingan haroratni qaytaradi

``` piton
Natija: [48.98055760884435]
```
