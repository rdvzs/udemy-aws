
# 10. Fejezet: EC2 Tárolás és Adatkezelés (EBS és EFS)

Ez a fejezet mélyrehatóan foglalkozik az EC2 példányok elsődleges tárolási lehetőségeivel: az Elastic Block Store-ral (EBS) és az Elastic File System-mel (EFS). Bemutatja típusaikat, működési menedzsmentjüket és teljesítményjellemzőiket SysOps nézőpontból.

## 10.1: EBS (Elastic Block Store) Áttekintés
- **EBS:** Hálózathoz csatolt blokktároló szolgáltatás EC2 példányokhoz. Úgy viselkedik, mint egy hálózatra kötött USB-meghajtó.
- **Főbb Jellemzők:**
  - **Perzisztens:** Az EBS köteten lévő adatok a példány életétől függetlenül megmaradnak.
  - **AZ-hez Kötött:** Egy EBS kötet egy adott rendelkezésre állási zónában (AZ) jön létre, és csak az ugyanabban az AZ-ben lévő példányokhoz csatolható.
  - **Többnyire Egyetlen Csatolás:** Egy kötet egyszerre csak egy EC2 példányhoz csatolható. Kivételt képez az EBS Multi-Attach funkció az io1/io2 kötetek esetében.
- **Törlés Megszüntetéskor (Delete on Termination):** Alapértelmezetten a gyökér (root) EBS kötet törlődik a példány megszüntetésekor, de ez a viselkedés megváltoztatható. A további csatolt kötetek alapértelmezetten nem törlődnek.

## 10.2: EBS Kötettípusok
- **SSD-alapú (Solid State Drive):**
  - **Általános Célú (gp3, gp2):** Ár és teljesítmény egyensúlya. Ideális rendszerindító kötetekhez, virtuális asztalokhoz, fejlesztői/teszt környezetekhez. A `gp3` a legújabb generáció, lehetővé teszi az IOPS és az átviteli sebesség független beállítását. A `gp2`-nél az IOPS a kötet méretéhez van kötve.
  - **Provisioned IOPS (io2 Block Express, io1):** A legmagasabb teljesítményű SSD-k kritikus fontosságú, alacsony késleltetésű vagy nagy átviteli sebességű munkaterhelésekhez, mint például nagy adatbázisok. Előre meghatározott számú IOPS-t biztosít.
- **HDD-alapú (Hard Disk Drive):**
  - **Átviteli Sebességre Optimalizált (st1):** Alacsony költségű HDD gyakran hozzáférhető, nagy átviteli sebességet igénylő munkaterhelésekhez, mint a big data és a naplófeldolgozás.
  - **Hideg HDD (sc1):** A legalacsonyabb költségű HDD ritkábban hozzáférhető adatokhoz, ahol a költségmegtakarítás a legfontosabb.
- **Rendszerindító Kötetek:** Csak a `gp2`, `gp3`, `io1` és `io2` használható rendszerindító kötetként.

## 10.3: EBS Műveletek
- **Kötetek Átméretezése:** Növelheti a kötet méretét és módosíthatja az IOPS-t használat közben. A méret növelése után ki kell terjeszteni a fájlrendszert az operációs rendszeren, hogy használni tudja az új helyet. **A kötet méretét közvetlenül nem lehet csökkenteni.**
- **Pillanatképek (Snapshots):** Az EBS kötetek időponthoz kötött biztonsági mentései, amelyeket az S3 tárol. Ez az elsődleges mechanizmus az adatok biztonsági mentésére és a kötetek AZ-k vagy régiók közötti migrálására.
  - **Titkosítás:** Egy pillanatképet a másolási folyamat során titkosíthat. A titkosított pillanatképből létrehozott kötet szintén titkosított lesz.
  - **Gyors Pillanatkép Visszaállítás (Fast Snapshot Restore - FSR):** Egy funkció, amely teljesen „előmelegíti” a pillanatképből létrehozott kötetet, kiküszöbölve a késleltetést az egyes blokkok első elérésekor. Nagyon drága, percenkénti számlázással.
  - **Pillanatkép Archívum (Snapshot Archive):** Alacsony költségű tárolási szint ritkán szükséges pillanatképekhez. Az archívumból való visszaállítás 24-72 órát vehet igénybe.
  - **Kuka (Recycle Bin):** Megvédi a pillanatképeket a véletlen törléstől azáltal, hogy egy beállított ideig (1 naptól 1 évig) megőrzi őket.
- **Data Lifecycle Manager (DLM):** Automatizálja az EBS pillanatképek és EBS-alapú AMI-k létrehozását, megőrzését és törlését ütemezések és címkék alapján.

## 10.4: EC2 Példánytár (Instance Store)
- **Mi ez?** Nagy teljesítményű, ideiglenes blokkszintű tároló, amely fizikailag a hoszt számítógéphez van csatolva.
- **Teljesítmény:** Sokkal jobb I/O teljesítményt nyújt, mint az EBS, mivel nincs hálózati késleltetés.
- **Illékony (Ephemeral):** Az adatok **elvesznek**, ha a példányt leállítják vagy megszüntetik. Nem perzisztens.
- **Felhasználási Területek:** Ideális ideiglenes információk tárolására, amelyek gyakran változnak, mint például pufferek, gyorsítótárak, „piszkozat” adatok vagy egyéb ideiglenes tartalmak.

## 10.5: EFS (Elastic File System) Áttekintés
- **EFS:** Menedzselt, skálázható fájltároló EC2 példányokhoz és helyi szerverekhez. A Network File System (NFS) protokollt használja.
- **Főbb Jellemzők:**
  - **Megosztott Hozzáférés:** Egyszerre több száz vagy ezer példányra is csatlakoztatható, akár különböző AZ-kben is.
  - **Regionális Szolgáltatás:** A fájlrendszer egy régión belül több AZ-n átívelően létezik.
  - **Csak Linux:** Kizárólag Linux-alapú AMI-kkal kompatibilis.
  - **Rugalmas és Használat Alapú Fizetés:** Automatikusan skálázódik, ahogy fájlokat ad hozzá vagy távolít el, és a felhasznált tárhelyért fizet.

## 10.6: EFS vs. EBS
- **EBS:** Egy-az-egyhez kapcsolat egy példánnyal (többnyire), egyetlen AZ-hez kötve. A kapacitást előre kell kiépíteni.
- **EFS:** Több-az-egyhez kapcsolat a példányokkal, több AZ-n keresztül elérhető. A kapacitás rugalmas.

## 10.7: EFS Jellemzők és Műveletek
- **Teljesítmény Módok:**
  - **General Purpose (Alapértelmezett):** Késleltetés-érzékeny felhasználási esetekhez, mint a webkiszolgálás.
  - **Max I/O:** Nagy átviteli sebességű, erősen párhuzamosított munkaterhelésekhez, mint a big data analitika.
- **Átviteli Sebesség Módok:**
  - **Bursting (Régi):** Az átviteli sebesség a fájlrendszer méretével skálázódik.
  - **Provisioned:** Előre beállít egy adott átviteli sebességet, függetlenül a tárhely méretétől.
  - **Elastic (Ajánlott):** Az átviteli sebesség automatikusan skálázódik a munkaterhelés igényei alapján.
- **Tárolási Szintek és Életciklus Kezelés:** Az EFS automatikusan áthelyezheti a fájlokat, amelyeket egy meghatározott ideig (pl. 30 nap) nem értek el, egy alacsonyabb költségű Infrequent Access (EFS-IA) vagy Archive tárolási osztályba.
- **EFS Hozzáférési Pontok (Access Points):** Alkalmazás-specifikus belépési pontok egy EFS fájlrendszerbe, amelyek megkönnyítik az alkalmazások hozzáférésének kezelését. Kikényszeríthet egy adott felhasználót/csoportot, és korlátozhatja a hozzáférést a fájlrendszer egy adott könyvtárára.
- **Migráció/Titkosítás:** Egy titkosítatlan EFS fájlrendszer titkosításához vagy a teljesítmény módjának megváltoztatásához létre kell hozni egy új fájlrendszert a kívánt beállításokkal, és az adatokat át kell migrálni egy olyan szolgáltatással, mint az **AWS DataSync**.

## 10.8: EFS Monitorozás
- **CloudWatch Metrikák:** Az EFS számos kulcsfontosságú metrikát biztosít:
  - **`PercentIOLimit`:** General Purpose módban jelzi, mennyire van közel az I/O korláthoz. Ha tartósan magas, szükség lehet a Max I/O módra váltásra.
  - **`BurstCreditBalance`:** Bursting throughput módban a magasabb átviteli sebességhez rendelkezésre álló krediteket mutatja.
  - **`StorageBytes`:** A fájlrendszer méretét mutatja, tárolási osztályonként (Standard vs. IA) lebontva.
