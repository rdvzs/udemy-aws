
# 3. Fejezet: EC2 SysOps Szemszögből

Ez a fejezet mélyrehatóan, SysOps fókusszal foglalkozik az Amazon EC2-vel, lefedve a példánykezelést, hálózatkezelést, vásárlási opciókat, monitorozást és hibaelhárítást.

## Példánykezelés és Életciklus

- **Példánytípus Váltása:** Lehetőség van a példánytípus megváltoztatására (pl. `t2.micro`-ról `t2.small`-ra) vertikális skálázás céljából. Ehhez a példánynak EBS-alapúnak kell lennie. A folyamat: a példány **leállítása (Stop)**, a típus **módosítása**, majd a példány **elindítása (Start)**. Az EBS köteten lévő adatok megmaradnak, de a példány valószínűleg új fizikai hardverre kerül, ami új nyilvános IP-címet eredményez.
- **Leállítási Viselkedés (Shutdown Behavior):** Konfigurálható, hogy mi történjen, ha a leállítás az *operációs rendszeren belülről* történik.
  - **Stop (Alapértelmezett):** Az EC2 példány leáll.
  - **Terminate (Megszüntetés):** Az EC2 példány megszüntetésre kerül.
- **Megszüntetés Elleni Védelem (Termination Protection):** Egy beállítás, amely megakadályozza a véletlen megszüntetést az AWS Konzolról vagy CLI-ből. **Fontos:** Ez *nem* védi meg a példányt, ha a leállítást az OS-en belülről kezdeményezik egy "terminate" leállítási viselkedéssel konfigurált példányon.
- **EC2 Hibernálás (Hibernate):** A leállítás egy alternatívája. A memóriában lévő állapot (RAM) a gyökér EBS kötetre mentődik. Ez sokkal gyorsabb példányindítást tesz lehetővé, mivel az OS nem indul újra, és az alkalmazások onnan folytatódhatnak, ahol abbahagyták.
  - **Követelmények:** A gyökér EBS kötetnek titkosítottnak és elég nagynak kell lennie a RAM tartalmának tárolásához. A hibernálás legfeljebb 60 napig támogatott.

## Hálózatkezelés és Elhelyezés

- **Fejlett Hálózatkezelés (Enhanced Networking - ENA & EFA):**
  - **Elastic Network Adapter (ENA):** Nagyobb sávszélességet, több csomag/másodperc (PPS) értéket és alacsonyabb késleltetést biztosít az újabb generációs példányok számára (pl. T3 vs. T2). Akár 100 Gbps sebességet is támogathat.
  - **Elastic Fabric Adapter (EFA):** Az ENA egy továbbfejlesztett változata, kifejezetten a Nagy Teljesítményű Számítástechnikához (HPC) és szorosan csatolt alkalmazásokhoz (csak Linux). Megkerüli az OS-t az alacsonyabb késleltetés érdekében, ideális a csomópontok közötti kommunikációhoz (pl. MPI használatával).
- **Elhelyezési Csoportok (Placement Groups):** Az EC2 példányok fizikai elhelyezésének szabályozása a teljesítmény és a hibatűrés befolyásolására.
  - **Cluster:** A példányokat szorosan egymás mellé csoportosítja egyetlen racken, egy AZ-n belül. A legalacsonyabb késleltetést és a legmagasabb átviteli sebességet biztosítja. Használja HPC-hez és nagy hálózati teljesítményt igénylő alkalmazásokhoz. *Kockázat: Egyetlen hardverhiba az összes példányt érintheti.*
  - **Spread:** A példányokat különböző fizikai hardverekre (különböző rackekre) osztja szét. Minimalizálja az egyidejű meghibásodásokat. Használja kritikus, magas rendelkezésre állást igénylő alkalmazásokhoz. *Korlátozás: Max. 7 példány AZ-onként és elhelyezési csoportonként.*
  - **Partition:** A példányokat több partícióra (rackek logikai csoportjaira) osztja szét egy vagy több AZ-n belül. Egy partíció hibája nem érinti a többit. Több száz példányt tesz lehetővé. Használja nagy, elosztott és partíció-tudatos munkaterhelésekhez, mint a Hadoop, Cassandra és Kafka.

## Vásárlási Opciók és Költségoptimalizálás

- **On-Demand:** Használat alapú fizetés, legmagasabb költség, nincs elkötelezettség. Rövid távú, kiszámíthatatlan munkaterhelésekhez.
- **Reserved Instances (RI):** 1 vagy 3 éves elkötelezettség egy adott példánytípusra. Jelentős kedvezmény. Stabil, állandó munkaterhelésekhez, mint az adatbázisok.
- **Savings Plans:** 1 vagy 3 éves elkötelezettség egy bizonyos összegű kiadásra ($/óra). Rugalmasabb, mint az RI-k.
- **Spot Instances:** Akár 90%-os kedvezmény, de 2 perces figyelmeztetéssel megszüntethetők, ha az AWS-nek szüksége van a kapacitásra. Ideális hibatűrő, nem kritikus munkaterhelésekhez, mint a batch feladatok és adatelemzés.
  - **Spot Fleet:** Spot példányok (és opcionálisan On-Demand) gyűjteménye egy célkapacitás eléréséhez. Stratégiák, mint a `lowestPrice` vagy `diversified`, segítenek a költség vagy a rendelkezésre állás optimalizálásában.
  - **Megszüntetés:** A Spot példányok végleges megszüntetéséhez **először a Spot Request-et kell törölni**, majd utána a példányokat.
- **Dedicated Hosts:** Egy fizikai szerver dedikáltan az Ön használatára. Megfelelőségi vagy komplex licencelési modellek (BYOL) esetén.
- **Capacity Reservations:** On-Demand kapacitás lefoglalása egy adott AZ-ben bármilyen időtartamra. Biztosítja, hogy szükség esetén el tudja indítani a példányokat, de akkor is fizet érte, ha nem használja.

## Monitorozás és Hibaelhárítás

- **CloudWatch Metrikák EC2-höz:** Az AWS alapértelmezett metrikákat biztosít 5 percenként (vagy 1 percenként Részletes Monitorozással).
  - **Kulcsmetrikák:** CPU Kihasználtság, Hálózati Forgalom (Be/Ki), Állapot-ellenőrzések, Lemez I/O (csak Instance Store esetén).
  - **A RAM NEM alapértelmezett metrika.** Ezt egyéni metrikaként kell gyűjteni.
- **Unified CloudWatch Agent:** EC2-re vagy helyi szerverekre telepített ügynök, amely további rendszerszintű metrikákat (mint a RAM, lemezterület, egyedi folyamatok a `procstat` segítségével) és egyéni alkalmazásnaplókat gyűjt, és küldi a CloudWatch-ba.
- **Példány Állapot-ellenőrzések (Instance Status Checks):**
  - **System Status Check:** Az alapul szolgáló AWS hardvert és szoftvert monitorozza. Hiba esetén a példány **leállítása/elindítása** szükséges, hogy új hardverre kerüljön. Egy CloudWatch riasztás beállítható az automatikus **helyreállításra (recover)**.
  - **Instance Status Check:** A konkrét példány szoftver- és hálózati konfigurációját monitorozza. Hiba esetén **újraindítás (reboot)** vagy OS-szintű hibaelhárítás lehet szükséges.
- **Indítási Problémák Hibaelhárítása:**
  - **Instance Limit Exceeded:** Elérte a vCPU limitet a fiókjában egy régióban. Kérjen limitemelést vagy indítson másik régióban.
  - **Insufficient Instance Capacity:** Az AWS-nek nincs elegendő On-Demand kapacitása a választott AZ-ben. Várjon, próbáljon másik példánytípust vagy másik AZ-t.
  - **Instance Terminates Immediately:** Okozhatja az EBS kötet limit elérése, egy sérült EBS pillanatkép, vagy KMS kulcs jogosultsági probléma egy titkosított gyökérkötet esetén.
- **SSH Problémák Hibaelhárítása:**
  - **Connection Timed Out:** Hálózati probléma. Ellenőrizze a Security Group szabályokat (nyitva van-e a 22-es port?), a Network ACL-eket és az útválasztási táblákat. Győződjön meg róla, hogy a példánynak van nyilvános IP-címe és nincs túlterhelve (CPU 100%-on).
  - **Permission Denied / Unprotected Private Key:** Helytelen fájljogosultságok a `.pem` kulcsfájlon (`chmod 400` használata szükséges) vagy rossz felhasználónév használata az OS-hez (pl. `ec2-user` Amazon Linuxhoz, `ubuntu` Ubuntuhoz).

## IP és Egyéb Jellemzők

- **Elastic IPs (EIP):** Egy fix, nyilvános IPv4-cím, amelyet Ön birtokol. Hozzárendelheti egy EC2 példányhoz, hogy statikus nyilvános IP-címet biztosítson. Hasznos egy IP átirányítására egy új példányra hiba esetén. Az EIP-kért akkor kell fizetni, ha *nincsenek* egy futó példányhoz csatolva.
- **Burstable Instances (T-család):** Ezek a példányok egy alap CPU teljesítménnyel rendelkeznek, és CPU kreditek felhasználásával képesek egy magasabb teljesítményszintre „kiugrani” (burst). Ha a kreditek elfogynak, a teljesítmény az alapszintre korlátozódik. A `T2/T3 Unlimited` mód lehetővé teszi a tartósan magas teljesítményt, de ez további költségekkel járhat.
