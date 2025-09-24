# 15. fejezet: CloudFront

## CloudFront áttekintés
A CloudFront egy tartalomkézbesítő hálózat (CDN), amely javítja az olvasási teljesítményt a weboldal tartalmának globálisan elosztott peremhálózati helyeken történő gyorsítótárazásával. Ez csökkenti a késleltetést a felhasználók számára világszerte, és DDoS védelmet biztosít.
-   **Peremhálózati helyek:** Több száz jelenléti pont világszerte.
-   **Működése:** A felhasználók a legközelebbi peremhálózati helyről kérik a tartalmat. Ha nincs gyorsítótárazva, a CloudFront lekéri az eredeti forrásból (pl. S3 tároló, EC2 példány), és gyorsítótárazza a jövőbeli kérésekhez.
-   **Eredeti források:**
    -   **Amazon S3 tárolók:** Fájlok terjesztésére és gyorsítótárazására. Origin Access Control (OAC) segítségével biztosított a privát hozzáférés.
    -   **VPC eredeti forrás:** Privát alhálózatokban tárolt alkalmazásokhoz (pl. ALB, NLB, EC2 példányok). Biztonságos, privát kapcsolatot biztosít.
    -   **Egyéni eredeti forrás:** Bármely nyilvános HTTP háttérrendszer, AWS-en belül vagy kívül.
-   **CloudFront vs. S3 Cross-Region Replication:**
    -   **CloudFront:** Globális peremhálózati hálózat (216+ PoP), statikus tartalmat gyorsítótáraz egy ideig (pl. egy napig), ideális a világszerte terjesztett statikus tartalomhoz.
    -   **S3 Cross-Region Replication:** Teljes tárolókat replikál specifikus régiókba, közel valós idejű frissítések, nincs gyorsítótárazás, ideális a dinamikus tartalomhoz, amely alacsony késleltetést igényel néhány régióban.

## CloudFront S3-mal gyakorlat
Bemutatja egy CloudFront disztribúció beállítását egy S3 tárolóhoz.
-   Fájlok (pl. `beach.jpg`, `coffee.jpg`, `index.html`) feltöltése egy privát S3 tárolóba.
-   CloudFront disztribúció létrehozása, az S3 tároló kiválasztása eredeti forrásként.
-   "Privát S3 tároló hozzáférés" (OAC használatával) engedélyezése, hogy a CloudFront hozzáférhessen a tárolóhoz anélkül, hogy az objektumok nyilvánossá válnának. A CloudFront automatikusan frissíti az S3 tárolóházirendet.
-   Telepítés után a tartalom a CloudFront domainnéven keresztül érhető el, gyorsítótárazással a peremhálózati helyeken.

## CloudFront ALB/EC2 mint eredeti forrás
-   **VPC eredeti források (ajánlott):** Közvetlenül összeköti a CloudFrontot a privát VPC alhálózatokban lévő alkalmazásokkal (ALB, NLB, EC2 példányok) anélkül, hogy azokat az internetre tenné. Ez a legbiztonságosabb módszer.
-   **Nyilvános hálózat (régebbi módszer):** Megkövetelte az ALB/EC2 példány nyilvános elérhetővé tételét és a biztonsági csoportok konfigurálását, hogy csak a CloudFront nyilvános IP-tartományaiból engedélyezze a forgalmat (amelyek dinamikusak és gyakori frissítést igényelnek). Kevésbé biztonságos és fáradságosabb.

## CloudFront földrajzi korlátozás
Lehetővé teszi a tartalomhoz való hozzáférés korlátozását a felhasználó országa alapján.
-   **Engedélyezési lista:** Engedélyezett országok listáját határozza meg.
-   **Tiltólista:** Tiltott országok listáját határozza meg.
-   Az országot egy harmadik féltől származó Geo-IP adatbázis segítségével határozzák meg.
-   **Felhasználási eset:** Szerzői jogi törvények, tartalomlicencelés. A disztribúció beállításai között konfigurálható.

## CloudFront jelentések, naplók és hibaelhárítás
-   **Hozzáférési naplók:** A CloudFront minden, a disztribúcióhoz intézett kérést naplóz egy megadott S3 naplózó tárolóba. Minden peremhálózati hely elküldi a naplóadatait. A naplózó tárolónak különböznie kell az eredeti tárolótól.
-   **Jelentések:** A CloudFront automatikusan generál különböző jelentéseket (gyorsítótár statisztikák, népszerű objektumok, leggyakoribb hivatkozók, használat, megtekintők) a hozzáférési napló adatok felhasználásával, még akkor is, ha az explicit hozzáférési naplózás az S3-ba nincs engedélyezve.
-   **Hibaelhárítás:** A CloudFront gyorsítótárazza a HTTP 4xx (ügyfélhibák, mint a 403 Forbidden, 404 Not Found) és 5xx (szerverhibák) állapotkódokat az eredeti forrásból. Ez hibás válaszok gyorsítótárazásához vezethet.

## CloudFront gyorsítótárazás mélyrehatóan
A CloudFront gyorsítótárazási viselkedése konfigurálható fejlécek, sütik és lekérdezési karakterlánc paraméterek alapján. A cél a gyorsítótár találatok maximalizálása és az eredeti forrásra irányuló kérések minimalizálása.
-   **Fejlécek:**
    -   **Összes továbbítása:** Nincs gyorsítótárazás, minden kérés az eredeti forráshoz megy (TTL=0).
    -   **Engedélyezési lista:** Gyorsítótárazás a megadott fejlécek értékei alapján.
    -   **Nincs (alapértelmezett):** A gyorsítótárazás nem a kérés fejlécei alapján történik, a legjobb gyorsítótárazási teljesítmény.
    -   **Eredeti egyéni fejlécek:** A CloudFront által az eredeti forrásnak küldött kérésekhez hozzáadott állandó fejlécek (nem gyorsítótárazásra).
    -   **Gyorsítótár viselkedési fejlécek:** Gyorsítótárazási logikához használt fejlécek.
-   **Gyorsítótárazási TTL (Time To Live):**
    -   Az eredeti forrásból származó `Cache-Control: max-age` vagy `Expires` fejlécek vezérlik.
    -   Testreszabható minimális, maximális és alapértelmezett TTL értékekkel a CloudFront beállításokban.
-   **Sütik:**
    -   **Ne dolgozza fel (alapértelmezett):** Nincs gyorsítótárazás sütik alapján, a sütik nem kerülnek továbbításra az eredeti forráshoz.
    -   **Engedélyezési lista:** Gyorsítótárazás a megadott sütik értékei alapján.
    -   **Összes továbbítása:** Legrosszabb gyorsítótárazási teljesítmény.
-   **Lekérdezési karakterlánc paraméterek:**
    -   **Ne dolgozza fel (alapértelmezett):** Nem kerülnek továbbításra az eredeti forráshoz, nincs gyorsítótárazás lekérdezési karakterláncok alapján.
    -   **Engedélyezési lista:** Gyorsítótárazás a megadott lekérdezési karakterláncok értékei alapján.
    -   **Összes továbbítása:** Legrosszabb gyorsítótárazási teljesítmény.
-   **Gyorsítótár találatok maximalizálása:**
    -   Minimalizálja a továbbított fejléceket, sütiket és lekérdezési karakterlánc paramétereket.
    -   Használja az eredeti forrásból származó `Cache-Control: max-age` fejlécet.
    -   Válassza szét a statikus és dinamikus tartalom disztribúciókat vagy viselkedéseket.

## CloudFront ALB ragadós munkamenetekkel
Ha a CloudFrontot ragadós munkamenetekkel rendelkező ALB-vel használja, kulcsfontosságú a CloudFront konfigurálása, hogy továbbítsa a munkamenet sütit (pl. `AWSALB`) az eredeti forráshoz. Ez biztosítja, hogy ugyanazon felhasználótól származó kérések következetesen ugyanahhoz a háttér EC2 példányhoz kerüljenek, fenntartva a munkamenet affinitást.