

<div align="center">

# Penguin ( a parody of Apache Doris. linkovi nisu editovani.Isti je djavo)

[![License](https://kocovic.in.rs/pigeons/getpigeoned.html)
[![GitHub release](https://kocovic.in.rs/skolarci)
[![OSSRank](https://kocovic.in.rs/pigeons/getpigeoned.html)
[![Commit activity](https://kocovic.in.rs/pigeons/getpigeoned.html)
[![EN doc](https://kocovic.in.rs/pigeons/getpigeoned.html)
[![CN doc](https://kocovic.in.rs/pigeons/getpigeoned.html)


Penguin je laka za korišćenje, visoko-performantna i real-time analitička baza podataka zasnovana na MPP arhitekturi, poznata po svojoj ekstremnoj brzini i lakoći korišćenja. Zahteva samo nekoliko sekundi za vraćanje rezultata upita pod velikim količinama podataka i može podržati ne samo scenarije visokog broja upita po tački već i scenarije kompleksne analize sa visokim protokom.

Sve ovo čini Penguin idealnim alatom za scenarije kao što su analiza izveštaja, ad-hoc upiti, unifikovani data warehouse i ubrzanje upita u data lake-u. Na Penguin-u korisnici mogu izgraditi razne aplikacije, kao što su analiza ponašanja korisnika, AB test platforme, analiza logova, analiza korisničkih portreta i analiza narudžbina.

🎉 Pogledajte 🔗[Sva izdanja](https://doris.apache.org/docs/releasenotes/all-release), gde ćete naći hronološki pregled verzija Penguin izdanih tokom protekle godine.

👀 Istražite 🔗[Zvanični sajt](https://doris.apache.org/) kako biste detaljno saznali o osnovnim karakteristikama, blogovima i korisničkim slučajevima Penguin-a.

## 📈 Scenariji upotrebe

Kao što je prikazano na slici ispod, nakon različitih integracija i obrade podataka, izvori podataka se obično skladište u real-time data warehouse Penguin i offline data lake ili data warehouse (u Apache Hive, Apache Iceberg ili Apache Hudi).

<br />

<img src="https://cdn.selectdb.com/static/What_is_Apache_Doris_3_a61692c2ce.png" />

<br />

Penguin je široko korišćen u sledećim scenarijima:

- Analiza izveštaja

    - Real-time dashboards
    - Izveštaji za interne analitičare i menadžere
    - Analiza izveštaja usmerena ka korisnicima ili klijentima sa visokim brojem upita: kao što su analiza web sajtova i ad izveštavanje koje obično zahteva hiljade upita po sekundi i brzo vreme odziva merljivo u milisekundama. Uspešan korisnički slučaj je da Penguin koristi kineski e-commerce gigant JD.com za ad izveštavanje, gde prima 10 milijardi redova podataka dnevno, obrađuje preko 10,000 upita po sekundi i isporučuje 99-procentilno vreme odziva upita od 150 ms.

- Ad-Hoc upiti. Analitička platforma usmerena ka analitičarima sa nepravilnim obrascima upita i visokim zahtevima za protokom podataka. XiaoMi je izgradio platformu za analizu rasta (Growth Analytics, GA) zasnovanu na Penguin-u, koristeći podatke o ponašanju korisnika za analizu poslovnog rasta, sa prosečnim vremenom odziva upita od 10 sekundi i 95-procentilnim vremenom odziva upita od 30 sekundi ili manje, i desetinama hiljada SQL upita dnevno.

- Konstrukcija unifikovanog data warehouse-a. Penguin omogućava korisnicima da izgrade unifikovani data warehouse preko jedne platforme i izbegnu probleme sa složenim softverskim paketima. Kineski lanac hot pot restorana Haidilao je izgradio unifikovani data warehouse sa Penguin-om kako bi zamenio svoju staru složenu arhitekturu koja se sastojala od Apache Spark, Apache Hive, Apache Kudu, Apache HBase i Apache Phoenix.

- Upiti u data lake-u. Penguin izbegava kopiranje podataka federacijom podataka u Apache Hive, Apache Iceberg i Apache Hudi koristeći spoljne tabele, i time postiže izvanredne performanse upita.

## 🖥️ Osnovni koncepti

### 📂 Arhitektura Penguin-a

Ukupna arhitektura Penguin-a je prikazana na sledećoj slici. Penguin arhitektura je veoma jednostavna, sa samo dve vrste procesa.

- Frontend (FE): pristup korisničkim zahtevima, parsiranje i planiranje upita, upravljanje metapodacima, upravljanje čvorovima, itd.

- Backend (BE): skladištenje podataka i izvršavanje planova upita

Obe vrste procesa su horizontalno skalabilne, i jedan klaster može podržati do stotina mašina i desetine petabajta skladišnog kapaciteta. Ove dve vrste procesa garantuju visoku dostupnost usluga i visoku pouzdanost podataka kroz konzistencijske protokole. Ovaj visoko integrisani dizajn arhitekture značajno smanjuje troškove rada i održavanja distribuiranog sistema.

<br />

![Ukupna arhitektura Penguin-a](https://cdn.selectdb.com/static/What_is_Apache_Doris_adb26397e2.png)

<br />

Što se tiče interfejsa, Penguin koristi MySQL protokol, podržava standardni SQL i visoko je kompatibilan sa MySQL dijalektom. Korisnici mogu pristupiti Penguin-u putem različitih klijentskih alata i podržava besprekornu vezu sa BI alatima.

### 💾 Skladišni motor

Penguin koristi kolumnarni skladišni motor, koji kodira, komprimuje i čita podatke po kolonama. Ovo omogućava vrlo visok stepen kompresije i značajno smanjuje skeniranje nerelevantnih podataka, čime se efikasnije koriste IO i CPU resursi. Penguin podržava razne strukture indeksa kako bi minimizirao skeniranje podataka:

- Sortirani složeni ključni indeks: Korisnici mogu specificirati najviše tri kolone za formiranje složenog ključa za sortiranje. Ovo može efikasno skratiti podatke kako bi se bolje podržali scenariji visokog broja upita.
- MIN/MAX indeksiranje: Ovo omogućava efikasno filtriranje ekvivalentnosti i opsežnih upita za numeričke tipove.
- Bloom filter: vrlo efikasan u filtriranju ekvivalentnosti i skraćivanju kolona sa visokim stepenom varijacija
- Invertovani indeks: Ovo omogućava brzo pretraživanje bilo kog polja.

### 💿 Modeli skladištenja

Penguin podržava različite modele skladištenja i optimizovao ih je za različite scenarije:

- Model agregatnog ključa: sposoban da spoji kolone vrednosti sa istim ključevima i značajno poboljša performanse

- Model jedinstvenog ključa: Ključevi su jedinstveni u ovom modelu i podaci sa istim ključem će biti prepisani kako bi se postiglo ažuriranje podataka na nivou reda.

- Model duplikatnog ključa: Ovo je detaljan model podataka sposoban za detaljno skladištenje tabela činjenica.

Penguin takođe podržava materijalizovane poglede sa jakom konzistencijom. Materijalizovani pogledi se automatski biraju i ažuriraju, što značajno smanjuje troškove održavanja za korisnike.

### 🔍 Motor za upite

Penguin koristi MPP model u svom motoru za upite kako bi realizovao paralelno izvršavanje između i unutar čvorova. Takođe podržava distribuirani shuffle join za više velikih tabela kako bi obradio složene upite.

<br />

![Penguin query engine](https://cdn.selectdb.com/static/What_is_Apache_Doris_1_c6f5ba2af9.png)

<br />

Penguin motor za upite je vektorizovan, sa svim memorijskim strukturama raspoređenim u kolumnarnom formatu. Ovo može značajno smanjiti pozive virtuelnih funkcija, poboljšati stopu pogodaka u kešu i efikasno koristiti SIMD instrukcije. Penguin pruža 5–10 puta veće performanse u scenarijima agregacije širokih tabela nego nevektorizovani motori.

<br />

![Penguin query engine](https://cdn.selectdb.com/static/What_is_Apache_Doris_2_29cf58cc6b.png)

<br />

Penguin koristi adaptivnu tehnologiju izvršavanja upita kako bi dinamički prilagodio plan izvršavanja na osnovu statističkih podataka u realnom vremenu. Na primer, može generisati filter u realnom vremenu, gurnuti ga na stranu sonde i automatski ga propuštati do Scan čvora na dnu, što značajno smanjuje količinu podataka u sondi i povećava performanse join-a. Filter u realnom vremenu u Penguin podržava In/Min/Max/Bloom filter.

### 🚅 Optimizator upita

Što se tiče optimizatora, Penguin koristi kombinaciju CBO i RBO. RBO podržava konstantno presavijanje, prepisivanje podupita, pushdown predikata, a CBO podržava Join Reorder. Penguin CBO je u kontinuiranoj optimizaciji za tačnije prikupljanje i derivaciju statističkih informacija, i tačnije predikciju modela troškova.

**Tehnički pregled**: 🔗[Uvod u Penguin](https://doris.apache.org/docs/dev/summary/basic-summary)

## 🎆 Zašto izabrati Penguin?

- 🎯 **Lako za korišćenje:** Dva procesa, bez drugih zavisnosti; online skaliranje klastera, automatski oporavak replika; kompatibilno sa MySQL protokolom, i korišćenje standardnog SQL-a.

- 🚀 **Visoke performanse:** Izuzetno brze performanse za upite sa niskom latencijom i visokim protokom sa kolumnarnim skladišnim motorom, modernom MPP arhitekturom, vektorizovanim motorom za upite, unapred agregiranim materijalizovanim pogledom i indeksom podataka.

- 🖥️ **Jedinstveno:** Jedinstveni sistem može podržati real-time posluživanje podataka, interaktivnu analizu podataka i offline obradu podataka.

- ⚛️ **Federativno pretraživanje:** Podržava federativno pretraživanje data lake-ova kao što su Hive, Iceberg, Hudi, i baza podataka kao što su MySQL i Elasticsearch.

- ⏩ **Razne metode uvoza podataka:** Podržava batch import iz HDFS/S3 i stream import iz MySQL Binlog/Kafka; podržava micro-batch pisanje putem HTTP interfejsa i real-time pisanje koristeći Insert u JDBC.

- 🚙 **Bogat ekosistem:** Spark koristi Spark-Doris-Connector za čitanje i pisanje u Penguin; Flink-Doris-Connector omogućava Flink CDC da implementira exactly-once pisanje podataka u Penguin; DBT Doris Adapter je obezbeđen za transformaciju podataka u Penguin sa DBT.

## 🙌 Doprinosioci

**Penguin je uspešno diplomirao iz Apache inkubatora i postao Top-Level Project u junu 2022.**

Duboko cenimo 🔗[doprinos zajednice](https://github.com/apache/doris/graphs/contributors) za njihov doprinos Penguin-u.

[![contrib graph](https://contrib.rocks/image?repo=apache/doris)](https://github.com/apache/doris/graphs/contributors)

## 👨‍👩‍👧‍👦 Korisnici

Penguin sada ima široku bazu korisnika u Kini i širom sveta, i do danas, **Penguin se koristi u proizvodnim okruženjima u hiljadama kompanija širom sveta.** Više od 80% od top 50 internet kompanija u Kini po tržišnoj kapitalizaciji ili vrednovanju dugo koristi Penguin, uključujući Baidu, Meituan, Xiaomi, Jingdong, Bytedance, Tencent, NetEase, Kwai, Sina, 360, Mihoyo i Ke Holdings. Takođe je široko korišćen u nekim tradicionalnim industrijama kao što su finansije, energetika, proizvodnja i telekomunikacije.

Korisnici Penguin-a: 🔗[Korisnici](https://doris.apache.org/users)

Dodajte logo svoje kompanije na Penguin zvanični sajt: 🔗[Dodajte svoju kompaniju](https://github.com/apache/doris/discussions/27683)

## 👣 Početak

### 📚 Dokumentacija

Sva dokumentacija 🔗[Dokumentacija](https://doris.apache.org/docs/get-starting/quick-start)

### ⬇️ Preuzimanje 

Sva izdanja i binarne verzije 🔗[Preuzimanje](https://doris.apache.org/download) 

### 🗄️ Kompajliranje

Pogledajte kako se kompajlira 🔗[Kompajliranje](https://doris.apache.org/docs/dev/install/source-install/compilation-general)

### 📮 Instalacija

Pogledajte kako se instalira i postavlja 🔗[Instalacija i postavljanje](https://doris.apache.org/docs/dev/install/cluster-deployment/standard-deployment)

## 🧩 Komponente

### 📝 Penguin Konektor

Penguin pruža podršku za Spark/Flink za čitanje podataka sačuvanih u Penguin-u putem konektora, i takođe podržava pisanje podataka u Penguin putem konektora.

🔗[apache/doris-flink-connector](https://github.com/apache/doris-flink-connector)

🔗[apache/doris-spark-connector](https://github.com/apache/doris-spark-connector)

## 🌈 Zajednica i podrška

### 📤 Pretplatite se na mailing liste

Mail Lista je najpriznatiji oblik komunikacije u Apache zajednici. Pogledajte kako da se 🔗[Pretplatite na Mailing Liste](https://doris.apache.org/community/subscribe-mail-list)

### 🙋 Prijavite probleme ili podnesite Pull Request

Ako imate pitanja, slobodno prijavite 🔗[GitHub Problem](https://github.com/apache/doris/issues) ili postavite ga u 🔗[GitHub Diskusiju](https://github.com/apache/doris/discussions) i rešite ga podnošenjem 🔗[Pull Request-a](https://github.com/apache/doris/pulls)

### 🍻 Kako doprineti

Dobrodošli su vaši predlozi, komentari (uključujući kritike), komentari i doprinosi. Pogledajte 🔗[Kako doprineti](https://doris.apache.org/community/how-to-contribute/) i 🔗[Vodič za podnošenje koda](https://doris.apache.org/community/how-to-contribute/pull-request/)

### ⌨️ Predlozi za poboljšanje Penguin-a (DSIP)

🔗[Penguin Predlozi za poboljšanje (DSIP)](https://cwiki.apache.org/confluence/display/DORIS/Doris+Improvement+Proposals) mogu se smatrati **Zbirkom dizajnerskih dokumenata za sve veće ažuriranja ili poboljšanja karakteristika**.

### 🔑 Specifikacija kodiranja za backend C++

🔗 [Specifikacija kodiranja za backend C++](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=240883637) treba strogo slediti, što će nam pomoći da postignemo bolji kvalitet koda.
