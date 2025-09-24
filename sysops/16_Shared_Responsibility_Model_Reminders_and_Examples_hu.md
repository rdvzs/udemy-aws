# 16. fejezet: Adatbázisok SysOps számára

## Amazon Relational Database Service (RDS)
Az RDS egy menedzselt relációs adatbázis-szolgáltatás, amely támogatja az SQL-t. Automatizálja a kiépítést, az operációs rendszer javítását, a folyamatos biztonsági mentéseket (Point-in-Time Restore - PITR), a monitorozást és a skálázást (vertikális és horizontális Read Replicák segítségével). Nem biztosít SSH hozzáférést a példányokhoz.
-   **Támogatott motorok:** PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM DB2, Aurora.
-   **Tároló automatikus skálázás:** Automatikusan növeli a tárhelyet, ha kevés a szabad hely (pl. <10% maradt, >5 percig tart, 6 óra telt el az utolsó módosítás óta). Hasznos a kiszámíthatatlan munkaterhelésekhez.
-   **Read Replicák:**
    -   Olvasások skálázása (akár 15 replika).
    -   Lehet ugyanazon AZ-n belül, AZ-k között vagy régiók között.
    -   Aszinkron replikáció (végül konzisztens).
    -   Önálló adatbázisokká léptethető elő.
    -   A replikációs forgalom ingyenes ugyanazon régión belül (AZ-k között), de régiók közötti forgalom esetén költséggel jár.
    -   Olvasás-intenzív munkaterhelések (pl. jelentéskészítés, analitika) tehermentesítésére használják az elsődleges adatbázisról.
-   **Multi-AZ telepítések:**
    -   Elsősorban katasztrófa-helyreállításra (DR) és magas rendelkezésre állásra (HA).
    -   Szinkron replikáció egy készenléti példányra egy másik AZ-ben.
    -   Egyetlen DNS név az alkalmazáshoz.
    -   Automatikus átállás készenléti állapotba az elsődleges adatbázis meghibásodása esetén (példány, tároló, hálózat, AZ leállás, OS javítás, manuális újraindítás átállással).
    -   Nem használják olvasások skálázására.
    -   Az egy-AZ-ról több-AZ-ra való konvertálás nulla állásidővel járó művelet.
-   **Paramétercsoportok:** Az adatbázismotor beállításainak testreszabása.
    -   **Dinamikus paraméterek:** Azonnal alkalmazódnak.
    -   **Statikus paraméterek:** Az adatbázis példány újraindítása után alkalmazódnak.
    -   Kikényszerítheti az SSL kapcsolatokat (pl. `rds.force_ssl=1` PostgreSQL/SQL Server esetén, `require_secure_transport=1` MySQL/MariaDB esetén).
-   **Biztonsági mentések és pillanatfelvételek:**
    -   **Automatikus biztonsági mentések:** Folyamatos, PITR engedélyezése, megőrzési idő 1-35 nap (nem tiltható le, 0-ra állítás letiltja). A visszaállítás *új* DB példányt hoz létre.
    -   **Manuális pillanatfelvételek:** Felhasználó által kezdeményezett, IO műveleteket igényel (rövid állásidőt okozhat), inkrementális az első teljes pillanatfelvétel után, nem jár le. A visszaállítás *új* DB példányt hoz létre.
    -   **Pillanatfelvételek megosztása:** A manuális pillanatfelvételek megoszthatók fiókok között. A titkosított pillanatfelvételekhez a KMS CMK megosztása szükséges.
-   **Események és naplók:**
    -   **Események:** Az RDS rögzíti az eseményeket (DB állapotváltozások, pillanatfelvételek, paramétercsoport változások). SNS témakörökbe vagy EventBridge-be küldhetők értesítések céljából.
    -   **Naplók:** Az adatbázis naplók (általános, audit, hiba, lassú lekérdezés) exportálhatók a CloudWatch Logs-ba hosszú távú megőrzés, elemzés és riasztás céljából (pl. metrikaszűrők és riasztások hibák esetén).
-   **CloudWatch metrikák:** Alapvető metrikákat biztosít a hipervizorból (pl. `DatabaseConnections`, `ReadIOPS`, `WriteLatency`, `FreeStorageSpace`).
-   **Továbbfejlesztett monitorozás:** Részletesebb metrikákat gyűjt egy ügynöktől a DB példányon (pl. folyamat/szál CPU/memória használat, lemez I/O). Konfigurálható granularitás (akár 1 másodperc).
-   **Performance Insights:** Vizualizálja az adatbázis terhelését, segít elemezni a teljesítményproblémákat a várakozások (CPU, IO, zárolások), SQL utasítások, gazdagépek és felhasználók szerinti szűréssel.

## Amazon Aurora
AWS saját fejlesztésű, felhőre optimalizált relációs adatbázis, amely kompatibilis a MySQL-lel és a PostgreSQL-lel. 5x MySQL teljesítményt, 3x PostgreSQL teljesítményt kínál.
-   **Tárolás:** Automatikusan növekszik 10 GB-ról 128 TB-ra. Hat másolatot tárol az adatokból 3 AZ-ben a magas rendelkezésre állás érdekében (4/6 másolat íráshoz, 3/6 olvasáshoz). Öngyógyító, automatikusan bővülő.
-   **Read Replicák:** Akár 15 read replika, 10 ms alatti replika késleltetés. Automatikusan skálázható.
-   **Átállás:** Azonnali átállás (sokkal gyorsabb, mint az RDS Multi-AZ).
-   **Végpontok:**
    -   **Író végpont:** Mindig az aktuális master példányra mutat (íráshoz).
    -   **Olvasó végpont:** Az olvasási terhelés elosztásához az egyik read replikához csatlakozik.
-   **Visszakövetés:** Az adatbázis visszatekerése bármely időpontra (akár 72 órára) új fürt létrehozása nélkül (helyben történő visszaállítás). Csak az Aurora MySQL-t támogatja.
-   **Adatbázis klónozás:** Új DB fürtöt hoz létre ugyanazzal a DB fürtkötettel, mint az eredeti (copy-on-write). Gyors és egyszerű tesztkörnyezetekhez.
-   **Biztonság:**
    -   **Titkosítás nyugalmi állapotban:** KMS-sel titkosítva (indításkor definiálva). A titkosítatlan masterek nem rendelkezhetnek titkosított replikákkal. Meglévő titkosítatlan DB titkosításához pillanatfelvétel készítése és titkosítottként történő visszaállítás szükséges.
    -   **Titkosítás átvitel közben:** Alapértelmezetten támogatott TLS gyökér tanúsítványok használatával.
    -   **Hitelesítés:** Felhasználónév/jelszó vagy IAM szerepkörök.
    -   **Hálózati hozzáférés:** Biztonsági csoportok vezérlik.
    -   **Nincs SSH hozzáférés** (kivéve az RDS Custom).
    -   **Audit naplók:** Engedélyezhetők és elküldhetők a CloudWatch Logs-ba.
-   **SysOps specifikumok:**
    -   **Prioritási szintek:** 0-15 prioritás hozzárendelése a Read Replicákhoz az átállási vezérléshez (a legalacsonyabb szintű kerül először elő).
    -   **Migráció:** RDS MySQL pillanatfelvétel migrálható Aurora MySQL-re.
    -   **CloudWatch metrikák:** `AuroraReplicaLag` (késleltetés az elsődleges és a replika között), `DatabaseConnections`, `InsertLatency`.

## Amazon ElastiCache
Menedzselt memóriabeli gyorsítótár szolgáltatás (Redis, Memcached). Csökkenti az adatbázis terhelését az olvasás-intenzív munkaterhelések esetén, és állapotmentessé teszi az alkalmazásokat a felhasználói munkamenetek tárolásával. Alkalmazáskód módosításokat igényel.
-   **Redis:**
    -   **Fürt mód letiltva:** Egy elsődleges csomópont, legfeljebb 5 read replika (HA és olvasási skálázás céljából). A replikáció aszinkron. Horizontális skálázás replikák hozzáadásával/eltávolításával. Vertikális skálázás csomóponttípus megváltoztatásával (új csomópontcsoportot hoz létre, adatok replikálódnak, DNS frissül).
    -   **Fürt mód engedélyezve:** Az adatok több shard között vannak particionálva (írási skálázás céljából). Minden shard rendelkezik egy elsődleges és legfeljebb 5 replikával. Támogatja az automatikus skálázást (shardok/replikák számának növelése/csökkentése metrikák, például CPU kihasználtság alapján).
    -   **Végpontok:**
        -   **Fürt mód letiltva:** Elsődleges végpont (írások), Olvasó végpont (olvasások, terheléselosztott), Csomópont végpontok (egyedi csomópontok).
        -   **Fürt mód engedélyezve:** Konfigurációs végpont (minden olvasási/írási művelethez, kompatibilis klienst igényel).
    -   **Funkciók:** Multi-AZ automatikus átállással, read replikák, adat tartósság (AOF perzisztencia), biztonsági mentés/visszaállítás, halmazok/rendezett halmazok.
    -   **Metrikák:** `Evictions` (lejárt elemek eltávolítva), `CPUUtilization`, `SwapUsage`, `CurrentConnections`, `DatabaseMemoryUsagePercentage`, `NetworkIn/Out`, `ReplicationBytes`, `ReplicationLag`.
-   **Memcached:**
    -   Több csomópont (akár 40) az adatok shardingjához.
    -   Nincs beépített HA vagy replikáció (adatvesztés csomóponthiba esetén).
    -   Biztonsági mentés/visszaállítás csak szerver nélküli verzióhoz.
    -   Többszálú architektúra.
    -   **Skálázás:**
        -   **Horizontális:** Csomópontok hozzáadása/eltávolítása (automatikus felderítés frissíti a klienseket).
        -   **Vertikális:** Új fürt létrehozása a kívánt csomóponttípussal, alkalmazás végpontok frissítése, régi fürt törlése (manuális folyamat, adatvesztés).
    -   **Metrikák:** Hasonló a Redishez: `Evictions`, `CPUUtilization`, `SwapUsage`, `CurrConnections`, `FreeableMemory`.

## Amazon DynamoDB
Teljesen menedzselt, magas rendelkezésre állású, skálázható NoSQL (kulcs-érték és dokumentum) adatbázis. Alacsony késleltetésű, nagy átviteli sebességű alkalmazásokhoz használják (mobil, játék, IoT).
-   **Séma:** Rugalmas, táblákat, elemeket és attribútumokat használ.
-   **Elsődleges kulcs:** Egyszerű (partíciókulcs) vagy összetett (partíciókulcs + rendezési kulcs).
-   **Kapacitás módok:**
    -   **Igény szerinti kapacitás:** Kérésenkénti fizetés, kiszámíthatatlan munkaterhelésekhez.
    -   **Kiépített kapacitás:** RCU/WCU kiépítése, kiszámítható munkaterhelésekhez, költséghatékonyabb. Támogatja az automatikus skálázást a kapacitás forgalom alapján történő beállításához.

## Amazon Redshift
Teljesen menedzselt, petabájt méretű, oszlopos adattárház szolgáltatás. Üzleti intelligencia, jelentéskészítés és analitika céljára használják.
-   **Csomóponttípusok:**
    -   **Sűrű számítás (DC):** Számítás-intenzív munkaterhelésekre optimalizálva (nagy teljesítményű analitika).
    -   **Sűrű tárolás (DS)::** Tárolás-intenzív munkaterhelésekre optimalizálva (nagy adathalmazok).
-   **Skálázás:** Csomópontok hozzáadásával/eltávolításával skálázható a Redshift fürt. Átméretezhető a csomóponttípus vagy a csomópontok számának megváltoztatásával.

## Amazon Neptune
Teljesen menedzselt, magas rendelkezésre állású, skálázható gráf adatbázis szolgáltatás. Adatok közötti kapcsolatokat igénylő alkalmazásokhoz használják (közösségi hálózatok, ajánlórendszerek, csalásfelderítés, tudásgráfok).
-   **Séma:** Csomópontok és élek tulajdonságokkal.
-   **Lekérdezési nyelvek:** Gremlin (gráf bejárás) vagy SPARQL (gráf lekérdezés).
-   **Skálázás:** Példányok hozzáadásával/eltávolításával skálázható. Átméretezhető a példánytípus vagy a példányok számának megváltoztatásával.