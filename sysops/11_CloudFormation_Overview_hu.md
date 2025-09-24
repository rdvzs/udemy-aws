
# 11. Fejezet: Bevezetés az Amazon S3-ba

Ez a fejezet bemutatja az Amazon S3-at (Simple Storage Service), az AWS egyik alapvető szolgáltatását. Lefedi az S3 alapkoncepcióit, beleértve a vödröket (bucket), objektumokat, biztonságot, verziókezelést, replikációt és a tárolási osztályokat.

## 11.1: S3 Áttekintés
- **S3 (Simple Storage Service):** Végtelenül skálázható objektumtároló szolgáltatás.
- **Felhasználási Területek:** Biztonsági mentés és tárolás, katasztrófa-helyreállítás, adatarchiválás, hibrid felhő tárolás, statikus weboldalak hosztolása, szoftverek terjesztése és adatóként (data lake) való funkcionálás analitikai célokra.
- **Vödrök (Buckets):** Az objektumokat vödrökben tárolják, amelyek legfelső szintű mappáknak tekinthetők. A vödörneveknek **globálisan egyedinek** kell lenniük az összes AWS-fiók és régió között.
- **Objektumok (Objects):** Az S3-ban tárolt fájlok. Egy objektumot egy **kulcs (key)** azonosít, ami a fájl teljes elérési útja (pl. `sajat-mappa/sajat-fajl.txt`). A kulcs egy prefixből (`sajat-mappa/`) és egy objektumnévből (`sajat-fajl.txt`) áll.
- **Objektum Mérete:** Az objektumok mérete legfeljebb 5 TB lehet. 5 GB-nál nagyobb objektumok esetén a több részből álló feltöltési (multi-part upload) API-t kell használni.

## 11.2: S3 Biztonság
- **Hozzáférés-szabályozás:** Az S3 biztonságát felhasználó-alapú és erőforrás-alapú házirendek kombinációjával kezelik.
  - **IAM Házirendek (Felhasználó-alapú):** Meghatározzák, hogy egy IAM felhasználó vagy szerepkör milyen műveleteket hajthat végre az S3 erőforrásokon.
  - **S3 Vödör Házirendek (Erőforrás-alapú):** Egy vödörhöz csatolt JSON dokumentumok, amelyek jogosultságokat adnak más entitásoknak (felhasználóknak, fiókoknak, szolgáltatásoknak). Ez az elsődleges módja a nyilvános hozzáférés vagy a fiókok közötti hozzáférés biztosításának.
  - **Hozzáférés-vezérlési Listák (ACL-ek):** Egy régebbi mechanizmus az egyes objektumokhoz való hozzáférés kezelésére. Ma már ajánlott az ACL-ek letiltása és a vödör házirendek használata az egyszerűbb hozzáférés-kezelés érdekében.
- **Nyilvános Hozzáférés Blokkolása (Block Public Access):** Fiók- és vödörszintű beállítások, amelyek központi vezérlést biztosítanak az adatok véletlen nyilvánosságra hozatalának megakadályozására. Ezek a beállítások felülírnak minden ellentmondó vödör házirendet vagy ACL-t.

## 11.3: S3 Statikus Weboldal Hosztolás
- **Funkcionalitás:** Az S3 konfigurálható statikus weboldalak (amelyek csak HTML, CSS, JavaScript és egyéb statikus fájlokat tartalmaznak) hosztolására.
- **Konfiguráció:** A statikus weboldal hosztolást a vödör tulajdonságaiban kell engedélyezni, és meg kell adni egy index dokumentumot (pl. `index.html`).
- **Követelmény:** Ahhoz, hogy a webhely elérhető legyen, a vödörben lévő objektumokat nyilvánossá kell tenni, ami általában egy S3 vödör házirenddel történik, amely engedélyezi az `s3:GetObject` műveletet mindenki (`*`) számára.

## 11.4: S3 Verziókezelés (Versioning)
- **Mi ez?** Egy vödörszintű funkció, amely megőrzi egy objektum összes verziójának előzményeit. Amikor felülír egy objektumot, egy új verzió jön létre a meglévő felülírása helyett.
- **Előnyök:**
  - **Védelem a véletlen törlés ellen:** Egy objektum törlése egy „törlési jelölőt” (delete marker) hoz létre, de nem távolítja el az alapul szolgáló verziókat, lehetővé téve a könnyű helyreállítást.
  - **Könnyű visszaállás:** Könnyen visszaállhat egy objektum egy korábbi verziójára.
- **Megjegyzés:** A verziókezelést engedélyezni kell az S3 Replikáció használatához.

## 11.5: S3 Replikáció
- **Funkcionalitás:** Aszinkron módon másolja az objektumokat egy forrás vödörből egy cél vödörbe.
- **Típusok:**
  - **Régiók Közötti Replikáció (Cross-Region Replication - CRR):** Objektumokat replikál egy *másik* AWS régióban lévő vödörbe. Megfelelőségi, alacsonyabb késleltetésű hozzáférési és katasztrófa-helyreállítási célokra használják.
  - **Azonos Régión Belüli Replikáció (Same-Region Replication - SRR):** Objektumokat replikál az *azonos* AWS régión belüli vödörbe. Naplók összesítésére vagy egy külön másolat létrehozására használják tesztelési célokra.
- **Fontos Megjegyzések:**
  - Alapértelmezetten csak az **új** objektumok replikálódnak a szabály létrehozása után. A meglévő objektumok replikálásához az **S3 Batch Replication** funkciót kell használni.
  - A törlési jelölők replikációja engedélyezhető vagy letiltható.
  - Nincs láncreplikáció (ha A replikál B-be, és B replikál C-be, az A-ból származó objektumok nem kerülnek át C-be).

## 11.6: S3 Tárolási Osztályok
- Az S3 számos tárolási osztályt kínál, amelyek különböző hozzáférési mintákhoz és költségkövetelményekhez vannak optimalizálva. Minden osztály ugyanazt a magas tartósságot (11 kilences) kínálja.
- **Gyakran Hozzáférhető Adatokhoz:**
  - **S3 Standard:** Alapértelmezett szint. Alacsony késleltetés és magas átviteli sebesség. Gyakran hozzáférhető adatokhoz tervezték.
- **Ritkán Hozzáférhető Adatokhoz:**
  - **S3 Standard-IA (Infrequent Access):** Alacsonyabb tárolási ár, de lekérési díjjal rendelkezik. Jó hosszú élettartamú, ritkábban hozzáférhető adatokhoz, mint a biztonsági mentések és katasztrófa-helyreállítási fájlok.
  - **S3 One Zone-IA:** Egyetlen AZ-ben tárolódik. Olcsóbb, mint a Standard-IA, de az adatok elvesznek, ha az AZ meghibásodik. Jó másodlagos biztonsági másolatok vagy újra létrehozható adatok tárolására.
- **Archiválási Adatokhoz:**
  - **S3 Glacier Instant Retrieval:** Hosszú távú archiváláshoz, milliszekundumos lekéréssel. Olcsóbb, mint a Standard-IA.
  - **S3 Glacier Flexible Retrieval:** Konfigurálható lekérési idők percek és órák között. Egyensúly a költség és a hozzáférési idő között.
  - **S3 Glacier Deep Archive:** A legalacsonyabb költségű tárolási osztály. A lekérés 12-48 órát vesz igénybe. Ritkán, vagy soha nem hozzáférhető adatokhoz.
- **Ismeretlen/Változó Hozzáférési Mintákhoz:**
  - **S3 Intelligent-Tiering:** Automatikusan mozgatja az objektumokat a hozzáférési szintek (Gyakori, Ritka és opcionális Archív szintek) között a használati minták alapján a költségek optimalizálása érdekében. Kis havi monitorozási és automatizálási díjjal jár.
