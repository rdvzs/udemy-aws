
# 6. Fejezet: EC2 Magas Rendelkezésre Állás és Skálázhatóság

Ez a fejezet a skálázható, rugalmas és magas rendelkezésre állású alkalmazások AWS-en történő építésének alapvető koncepcióira és szolgáltatásaira összpontosít, elsősorban az Elastic Load Balancing (ELB) és az Auto Scaling Groups (ASG) köré épülve.

## 6.1: Magas Rendelkezésre Állás és Skálázhatóság Koncepciók
- **Skálázhatóság (Scalability):** Egy alkalmazás képessége a megnövekedett terhelés kezelésére.
  - **Vertikális Skálázhatóság:** Egyetlen példány erőforrásainak növelése (pl. `t2.micro`-ról `t2.large`-ra). Gyakori nem elosztott rendszereknél, mint az adatbázisok (RDS, ElastiCache). Hardveres korlátai vannak.
  - **Horizontális Skálázhatóság (Rugalmasság - Elasticity):** A példányok számának növelése (scale out/in). Ez a felhő-natív megközelítés, ideális webalkalmazásokhoz.
- **Magas Rendelkezésre Állás (High Availability - HA):** Biztosítja, hogy egy alkalmazás működőképes maradjon még egy adatközpont meghibásodása esetén is. Ezt legalább két rendelkezésre állási zónában (AZ) futó példányokkal érik el.

## 6.2: Elastic Load Balancing (ELB) Áttekintés
- **ELB:** Menedzselt AWS szolgáltatás, amely a bejövő alkalmazásforgalmat több háttér-célpont, például EC2 példányok között osztja el.
- **Előnyök:** Elosztja a terhelést, egyetlen hozzáférési pontot (DNS-nevet) biztosít, kezeli a hibákat azáltal, hogy elirányítja a forgalmat az egészségtelen példányoktól, és használható SSL/TLS lezárásra.
- **Load Balancer Típusok:**
  - **Application Load Balancer (ALB):** (Layer 7) HTTP/HTTPS forgalomhoz. Támogatja a fejlett, tartalom-alapú útválasztási szabályokat.
  - **Network Load Balancer (NLB):** (Layer 4) TCP/UDP/TLS forgalomhoz. Ultra-magas teljesítményt és statikus IP-címeket kínál AZ-onként.
  - **Gateway Load Balancer (GWLB):** (Layer 3) A forgalom harmadik féltől származó virtuális hálózati eszközökhöz, például tűzfalakhoz és behatolás-érzékelő rendszerekhez való irányítására szolgál.
  - **Classic Load Balancer (CLB):** Régi generáció, már elavult.

## 6.3: Application Load Balancer (ALB) Részletesen
- **Célcsoportok (Target Groups):** Az ALB-k a forgalmat célpontokhoz (EC2 példányok, Lambda függvények, IP-címek) irányítják, amelyeket célcsoportokba szerveznek.
- **Útválasztási Szabályok:** Az ALB-k különböző feltételek alapján irányíthatják a forgalmat:
  - **Útvonal (Path):** `example.com/users` vs. `example.com/posts`.
  - **Hosztnév (Hostname):** `one.example.com` vs. `two.example.com`.
  - **Lekérdezési Paraméterek és Fejlécek (Query Strings & Headers):** `?platform=mobile`.
- **Felhasználási Terület:** Ideális mikroszolgáltatásokhoz és konténer-alapú alkalmazásokhoz (ECS) a rugalmas útválasztási képességei miatt.
- **Kliens IP-címe:** A háttérpéldányok nem látják közvetlenül az eredeti kliens IP-címét; azt az `X-Forwarded-For` fejlécben kapják meg.

## 6.4: Network Load Balancer (NLB) Részletesen
- **Felhasználási Terület:** Extrém teljesítményigény (millió kérés/mp), alacsony késleltetés esetén, valamint ha statikus IP-címre van szükség.
- **Statikus IP:** Minden engedélyezett AZ-hez egy statikus IP-címet biztosít. Saját Elastic IP-ket is hozzárendelhet.
- **Célcsoportok:** Megcélozhat EC2 példányokat vagy privát IP-címeket (helyi szerverekhez).
- **Egészség-ellenőrzések:** Támogatja a TCP, HTTP és HTTPS egészség-ellenőrzéseket.

## 6.5: Gateway Load Balancer (GWLB) Részletesen
- **Funkció:** Átlátszó hálózati átjáróként működik, lehetővé téve harmadik féltől származó virtuális eszközök (tűzfalak, IDS/IPS) beillesztését a hálózati útvonalba a forgalom áramlásának megváltoztatása nélkül.
- **Működése:** Egy VPC-be és onnan ki irányuló összes forgalom a GWLB-n keresztül halad, amely aztán elosztja azt a virtuális eszközök flottáján ellenőrzés céljából. A GENEVE protokollt használja a 6081-es porton.

## 6.6: ELB Haladó Funkciók
- **Sticky Sessions (Munkamenet-affinitás):** Lehetővé teszi, hogy a load balancer egy felhasználó munkamenetét egy adott célpéldányhoz kösse. Ez biztosítja, hogy a felhasználótól érkező összes kérés ugyanahhoz a példányhoz kerüljön, ami hasznos a munkamenet-adatokat helyben tároló alkalmazásoknál. Engedélyezhető ALB, NLB és CLB esetén.
- **Cross-Zone Load Balancing (Zónák Közötti Terheléselosztás):**
  - **ALB:** Alapértelmezetten engedélyezve. A forgalom egyenletesen oszlik el az összes engedélyezett AZ összes regisztrált példánya között. Nincs díja az AZ-k közötti adatátvitelnek.
  - **NLB/GWLB:** Alapértelmezetten letiltva. Ha engedélyezi, díjat számítanak fel az AZ-k közötti adatátvitelért.
- **SSL/TLS Tanúsítványok:** A load balancerek képesek lezárni az SSL/TLS forgalmat. SSL tanúsítványokat (az AWS Certificate Manager - ACM segítségével kezelve) csatolhat a listenerekhez.
  - **Server Name Indication (SNI):** Lehetővé teszi több SSL-védett webhely hosztolását egyetlen load balancer mögött több SSL tanúsítvány betöltésével. A kliens a kezdeti SSL kézfogás során megadja a cél hosztnevet, lehetővé téve az ELB számára a megfelelő tanúsítvány bemutatását.
- **Connection Draining (Deregistration Delay - Kapcsolatok Leürítése):** Amikor egy példány regisztrációja törlődik vagy egészségtelenné válik, ez a funkció nyitva tartja a meglévő kapcsolatokat, hogy a folyamatban lévő kérések befejeződhessenek, miközben az új kéréseket más egészséges példányokhoz irányítja. Az időtúllépés konfigurálható (alapértelmezésben 300 másodperc).

## 6.7: Auto Scaling Groups (ASG) Áttekintés
- **ASG:** Automatikusan módosítja az EC2 példányok számát egy csoportban a keresletnek megfelelően.
- **Alapvető Funkciók:**
  - **Skálázás (Scale Out/In):** Példányokat ad hozzá (scale out) vagy távolít el (scale in) a terhelés alapján.
  - **Kapacitás Fenntartása:** Biztosítja, hogy mindig fusson egy minimális számú példány.
  - **Egészség-ellenőrzés:** Lecseréli azokat a példányokat, amelyek megbuknak az EC2 vagy ELB egészség-ellenőrzéseken.
- **Konfiguráció:** Szükség van egy **Launch Template**-re (vagy a régebbi Launch Configuration-re), amely meghatározza a példány paramétereit (AMI, példánytípus, biztonsági csoportok stb.).

## 6.8: ASG Skálázási Házirendek
- **Dinamikus Skálázás:**
  - **Target Tracking (Célkövetés):** A leggyakoribb és legegyszerűbb házirend. Beállít egy célértéket egy metrikára (pl. átlagos CPU kihasználtság 40%), és az ASG automatikusan módosítja a kapacitást a cél fenntartása érdekében.
  - **Simple/Step Scaling:** CloudWatch riasztási küszöbértékek alapján skáláz (pl. ha a CPU > 70%, adjon hozzá 2 példányt).
- **Ütemezett Skálázás (Scheduled Scaling):** Megváltoztatja az ASG méretét előre kiszámítható, ütemezett időpontokban (pl. minden pénteken 17:00-kor növelje a kapacitást).
- **Prediktív Skálázás (Predictive Scaling):** Gépi tanulást használ a jövőbeli terhelés előrejelzésére és a skálázási műveletek proaktív ütemezésére.
- **Skálázási Lehűlés (Scaling Cooldown):** Egy skálázási tevékenység utáni időszak (alapértelmezetten 300 másodperc), amely alatt az ASG nem kezdeményez újabb skálázási műveletet. Ez lehetővé teszi a metrikák stabilizálódását.

## 6.9: ASG SysOps Szemszögből
- **Életciklus Horgok (Lifecycle Hooks):** Lehetővé teszik egyéni műveletek végrehajtását, miközben a példányokat az ASG elindítja vagy megszünteti. Például szüneteltethet egy példányt, mielőtt szolgálatba állna, hogy futtasson egy beállító szkriptet, vagy szüneteltetheti a megszüntetés előtt, hogy naplókat mentsen ki.
- **Egészség-ellenőrzések:** Az ASG-k használhatnak `EC2` állapot-ellenőrzéseket (alapértelmezett) és/vagy `ELB` egészség-ellenőrzéseket a példányok állapotának meghatározására. Az egészségtelen példányokat megszünteti és lecseréli.
- **Egyéni Egészség-ellenőrzések:** A `SetInstanceHealth` API hívással manuálisan is jelentheti egy példány állapotát az ASG-nek.
