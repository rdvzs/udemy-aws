# 23. fejezet: Hálózatépítés VPC

## Mi az a VPC?
A VPC (Virtual Private Cloud) az AWS Cloud logikailag elkülönített része, ahol az AWS erőforrásokat egy Ön által definiált virtuális hálózatban indíthatja el. Olyan, mintha saját adatközpontja lenne a felhőben.
-   **Kulcsfontosságú komponensek:**
    -   **Alhálózatok:** A VPC alosztályai, lehetnek nyilvánosak (Internet Gateway-vel) vagy privátak (Internet Gateway nélkül).
    -   **Útválasztási táblák:** Szabályozzák az alhálózatok útválasztását.
    -   **Internet Gateway (IGW):** Lehetővé teszi a nyilvános alhálózatok számára az internethez való csatlakozást.
    -   **NAT Gateway/példány:** Lehetővé teszi a privát alhálózatok számára az internethez való csatlakozást (pl. frissítésekhez) anélkül, hogy nyilvánosan elérhetőek lennének.
    -   **Biztonsági csoportok:** Virtuális tűzfalakként működnek a példányok számára.
    -   **Hálózati ACL-ek (NACL-ek):** Állapotmentes csomagszűrőként működnek az alhálózatok számára.

## VPC Flow Logs
Információkat rögzít a VPC hálózati interfészein keresztül áramló IP-forgalomról.
-   **Rögzített adatok:** Forrás/cél IP, port, protokoll, művelet (ELFOGAD/ELUTASÍT), bájtok, csomagok.
-   **Célok:** CloudWatch Logs, S3, Kinesis Data Firehose.
-   **Felhasználási esetek:** Hálózati kapcsolat hibaelhárítása, hálózati forgalom monitorozása, biztonsági fenyegetések azonosítása.

## VPC Peering
Két VPC-t privátan összeköt, lehetővé téve az egyik VPC-ben lévő erőforrások számára, hogy kommunikáljanak a másik VPC-ben lévő erőforrásokkal, mintha ugyanabban a hálózatban lennének.
-   **Korlátozások:** Nincs tranzitív peering (VPC A B-vel, B C-vel van peeringelve, A nem tud közvetlenül beszélni C-vel). Nincsenek átfedő CIDR blokkok.
-   **Felhasználási esetek:** Erőforrások megosztása VPC-k között, alkalmazások összekötése különböző VPC-kben.

## VPC végpontok
Lehetővé teszi az AWS szolgáltatások privát elérését a VPC-ből anélkül, hogy a nyilvános interneten keresztül történne a forgalom.
-   **Interface Endpoints (AWS PrivateLink által működtetve):** Privát IP-címet biztosít egy AWS szolgáltatáshoz (pl. S3, DynamoDB, EC2, Lambda) a VPC-n belül. A forgalom az AWS privát hálózatán keresztül áramlik.
-   **Gateway Endpoints:** Kifejezetten S3 és DynamoDB számára. Útválasztási tábla bejegyzést hoz létre a forgalom szolgáltatáshoz való irányításához. Nincs privát IP-cím.

## AWS Direct Connect
Dedikált privát hálózati kapcsolatot létesít a helyszíni adatközpont és az AWS között.
-   **Előnyök:** Csökkentett hálózati költségek, megnövelt sávszélesség, konzisztensebb hálózati élmény, mint az internet alapú kapcsolatok.
-   **Felhasználási esetek:** Hibrid felhő környezetek, nagy adatátvitelek, valós idejű alkalmazások.

## AWS Site-to-Site VPN
Titkosított kapcsolatot létesít a helyszíni hálózat és a VPC között a nyilvános interneten keresztül.
-   **Előnyök:** Biztonságos kommunikáció, költséghatékony kisebb munkaterhelésekhez vagy ideiglenes kapcsolatokhoz.
-   **Komponensek:** Ügyfél Gateway (helyszíni), Virtual Private Gateway (VPC).

## AWS Client VPN
Lehetővé teszi a kliensek (pl. távoli alkalmazottak) számára, hogy biztonságosan hozzáférjenek az AWS erőforrásokhoz és a helyszíni hálózatokhoz bármely helyről VPN kliens segítségével.
-   **Előnyök:** Biztonságos távoli hozzáférés, integrálódik az Active Directory-val vagy tanúsítvány alapú hitelesítéssel.

## AWS Transit Gateway
Több ezer VPC-t és helyszíni hálózatot köt össze egy központi hubon keresztül.
-   **Előnyök:** Egyszerűsíti a hálózati topológiát, csökkenti a működési többletköltséget, lehetővé teszi a tranzitív peeringet.
-   **Csatolmányok:** VPC-k, VPN-ek, Direct Connect Gateway-ek.
-   **Útválasztás:** Központosított útválasztási tábla az összes csatlakoztatott hálózathoz.

## AWS Global Accelerator
Javítja az alkalmazások rendelkezésre állását és teljesítményét a globális felhasználók számára azáltal, hogy a forgalmat az optimális végpontokhoz irányítja az AWS globális hálózatán keresztül.
-   **Hogyan működik:** Két statikus anycast IP-címet biztosít, amelyek fix belépési pontként szolgálnak az alkalmazáshoz. A forgalom a legközelebbi AWS peremhálózati helyre, majd az AWS globális hálózatán keresztül az optimális végponthoz kerül.
-   **Előnyök:** Javított teljesítmény (akár 60%-kal gyorsabb), megnövelt rendelkezésre állás (automatikus átállás), egyszerűsített globális forgalomkezelés.
-   **Végpontok:** ELB-k, EC2 példányok, Elastic IP-k.

## AWS PrivateLink
Lehetővé teszi a privát kapcsolatot a VPC-k és az AWS szolgáltatások, vagy a VPC-k és más AWS fiókok (szolgáltatók) által hosztolt szolgáltatások között.
-   **Előnyök:** Megszünteti a nyilvános internetre való kitettséget, egyszerűsíti a hálózati architektúrát, növeli a biztonságot.
-   **Komponensek:** VPC végpont (fogyasztó), végpont szolgáltatás (szolgáltató).

## Hálózati ACL-ek (NACL-ek) vs. Biztonsági csoportok
-   **NACL-ek:**
    -   Alhálózati szinten működnek.
    -   Állapotmentesek (a szabályok a bejövő és kimenő forgalomra egymástól függetlenül vonatkoznak).
    -   A szabályokat sorrendben dolgozzák fel (legalacsonyabb szám először).
    -   Alapértelmezett tiltás minden bejövő/kimenő forgalomra (kivéve, ha expliciten engedélyezett).
-   **Biztonsági csoportok:**
    -   Példány szinten működnek.
    -   Állapotfüggőek (a szabályok automatikusan vonatkoznak a visszatérő forgalomra).
    -   Minden szabályt kiértékelnek a döntés előtt.
    -   Alapértelmezett tiltás minden bejövő forgalomra, engedélyezés minden kimenő forgalomra.

## VPC Flow Logs gyakorlat
Bemutatja a VPC Flow Logs engedélyezését a hálózati forgalom információinak rögzítéséhez.
-   **Konfiguráció:** Válassza ki a VPC-t, alhálózatot vagy hálózati interfészt. Válassza ki a célt (S3 vagy CloudWatch Logs). Határozza meg a naplóformátumot.
-   **Elemzés:** A Flow Logs elemezhetők a CloudWatch Logs Insights-ban, vagy lekérdezhetők az S3-ban Athena segítségével.

## VPC Peering gyakorlat
Bemutatja két VPC összekötését VPC Peering segítségével.
-   **Folyamat:** Peering kapcsolat kérése az egyik VPC-ből a másikba. Fogadja el a kérést. Frissítse az útválasztási táblákat mindkét VPC-ben, hogy engedélyezze a forgalmat a peeringelt VPC CIDR blokkjához.
-   **Ellenőrzés:** Tesztelje a kapcsolatot a peeringelt VPC-kben lévő példányok között.

## VPC végpontok gyakorlat
Bemutatja egy VPC végpont létrehozását S3-hoz.
-   **Gateway végpont:** Hozzon létre egy Gateway végpontot S3-hoz. Frissítse az útválasztási táblákat, hogy az S3 forgalmat a végponton keresztül irányítsa.
-   **Interface végpont:** Hozzon létre egy Interface végpontot S3-hoz. Ez privát IP-címet biztosít az S3-hoz a VPC-n belül.

## AWS Direct Connect gyakorlat
Bemutatja egy Direct Connect kapcsolat beállításának folyamatát.
-   **Lépések:** Direct Connect kapcsolat kérése. Válassza ki a helyet, portsebességet. Az AWS kiépíti a kapcsolatot. Csatlakoztassa a routert a Direct Connect helyhez. Konfigurálja a Virtual Interface-eket (VIF-eket) a VPC-hez való csatlakozáshoz.

## AWS Site-to-Site VPN gyakorlat
Bemutatja egy Site-to-Site VPN kapcsolat beállítását.
-   **Komponensek:** Hozzon létre egy Customer Gateway-t (helyszíni router adatai). Hozzon létre egy Virtual Private Gateway-t (csatolva a VPC-hez). Hozzon létre egy Site-to-Site VPN kapcsolatot a Customer Gateway és a Virtual Private Gateway között.
-   **Konfiguráció:** Töltse le a konfigurációs fájlt a helyszíni routerhez.

## AWS Client VPN gyakorlat
Bemutatja egy Client VPN végpont beállítását.
-   **Komponensek:** Hozzon létre egy Client VPN végpontot. Konfigurálja a kliens hitelesítést (pl. tanúsítvány alapú). Társítsa a célhálózatokat (alhálózatokat). Konfigurálja az engedélyezési szabályokat.
-   **Kliens:** Töltse le a VPN kliens konfigurációs fájlt.

## AWS Transit Gateway gyakorlat
Bemutatja egy Transit Gateway beállítását.
-   **Komponensek:** Hozzon létre egy Transit Gateway-t. Hozzon létre Transit Gateway csatolmányokat (VPC, VPN, Direct Connect). Konfigurálja a Transit Gateway útválasztási táblákat.
-   **Előnyök:** Központosított útválasztás, egyszerűsített kapcsolat komplex hálózatokhoz.

## AWS Global Accelerator gyakorlat
Bemutatja egy Global Accelerator beállítását.
-   **Komponensek:** Hozzon létre egy Global Accelerator-t. Konfigurálja a figyelőket (portok, protokollok). Konfigurálja a végpontcsoportokat (régiók, végpontok, mint az ELB-k, EC2).
-   **Előnyök:** Statikus IP-címek, a forgalom az AWS globális hálózatán keresztül irányul a teljesítmény és a rendelkezésre állás érdekében.

## AWS PrivateLink gyakorlat
Bemutatja a PrivateLink beállítását egy szolgáltatáshoz.
-   **Szolgáltató:** Hozzon létre egy Endpoint Service-t (pl. egy NLB-hez).
-   **Fogyasztó:** Hozzon létre egy VPC végpontot (Interface Endpoint) a Endpoint Service-hez való csatlakozáshoz.