# 22. fejezet: Hálózatépítés Route 53

## Mi az a DNS?
A DNS (Domain Name System) emberbarát hosztneveket (pl. `www.google.com`) IP-címekké fordít. Ez az internet gerince, hierarchikus elnevezési struktúrát használva.
-   **Terminológia:**
    -   **Domain regisztrátor:** Ahol domain neveket regisztrál (pl. Route 53, GoDaddy).
    -   **DNS rekordok:** Meghatározzák a forgalom útvonalát (pl. A, AAAA, CNAME, NS).
    -   **Zónafájl:** Tartalmazza egy domain összes DNS rekordját.
    -   **Névszerverek:** Feloldják a DNS lekérdezéseket.
    -   **TLD (Top-Level Domain):** `.com`, `.org`, stb.
    -   **Második szintű domain:** `example.com`.
    -   **Aldomain:** `www.example.com`, `api.example.com`.
    -   **FQDN (Fully Qualified Domain Name):** Teljes domain név (pl. `api.www.example.com`).
-   **Hogyan működik a DNS:** Egy kliens lekérdezi a helyi DNS szerverét, amely rekurzívan lekérdezi a gyökér, TLD és második szintű domain névszervereket, amíg meg nem találja azt a hiteles névszervert (pl. Route 53), amelyik a rekordot tartalmazza. A válasz gyorsítótárazásra kerül, és visszaküldésre kerül a kliensnek.

## Route 53 áttekintés
A Route 53 egy magas rendelkezésre állású, skálázható, teljesen menedzselt és hiteles DNS szolgáltatás. Domain regisztrátor is.
-   **Hosted Zones (Hosztolt zónák):** DNS rekordok tárolói.
    -   **Nyilvános hosztolt zóna:** Nyilvános domain nevekre, válaszol a lekérdezésekre az internetről.
    -   **Privát hosztolt zóna:** Privát domain nevekre, csak a megadott VPC-kből válaszol a lekérdezésekre.
-   **Költség:** 0,50 dollár/hó hosztolt zónánként. A domain regisztrációs költségek változóak (pl. 12 dollár/év).
-   **DNS rekordtípusok:**
    -   **A rekord:** Hosztnevet IPv4 címre képez le.
    -   **AAAA rekord:** Hosztnevet IPv6 címre képez le.
    -   **CNAME rekord:** Hosztnevet egy másik hosztnévre képez le. Nem használható a zóna csúcsára (gyökér domain, pl. `example.com`).
    -   **NS rekord:** Meghatározza egy hosztolt zóna névszervereit.
-   **TTL (Time To Live):** Az az időtartam, ameddig egy rekordot a DNS feloldók gyorsítótáraznak. Magas TTL = kevesebb Route 53 forgalom, de lassabb rekordváltozások. Alacsony TTL = több Route 53 forgalom, gyorsabb rekordváltozások. Kötelező a legtöbb rekordhoz, kivéve az Alias rekordokat.

## CNAME vs. Alias
-   **CNAME rekord:** Egy hosztnevet *bármely* más hosztnévre mutat. Csak nem gyökér domainekre működik (pl. `app.mydomain.com`).
-   **Alias rekord:** Route 53 specifikus. Egy hosztnevet egy *specifikus AWS erőforrásra* mutat (ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 weboldalak, VPC Interface Endpoints, Global Accelerator, egyéb Route 53 rekordok). Működik gyökér és nem gyökér domainekre is. Ingyenes, automatikusan örökli az egészségügyi ellenőrzéseket a cél erőforrásból. Nem állítható be TTL.

## Útválasztási házirendek
Meghatározzák, hogyan válaszol a Route 53 a DNS lekérdezésekre.
-   **Egyszerű útválasztás:** Egyetlen erőforráshoz irányítja a forgalmat. Egyetlen rekordhoz több értéket (IP-t) is visszaadhat, a kliensek véletlenszerűen választanak egyet. Nem társítható egészségügyi ellenőrzésekkel.
-   **Súlyozott útválasztás:** A forgalmat több erőforráshoz irányítja a hozzárendelt súlyok (százalékok) alapján. Hasznos terheléselosztáshoz vagy A/B teszteléshez. Társítható egészségügyi ellenőrzésekkel.
-   **Átállási útválasztás:** A forgalmat egy elsődleges erőforráshoz irányítja, ha az egészséges, különben egy másodlagos erőforráshoz. Egészségügyi ellenőrzéseket igényel.
-   **Késleltetés alapú útválasztás:** A forgalmat a felhasználó számára legalacsonyabb késleltetésű erőforráshoz irányítja.
-   **Geolokációs útválasztás:** A forgalmat a felhasználó földrajzi elhelyezkedése (kontinens, ország, USA állam) alapján irányítja. Hasznos tartalom lokalizációhoz, terjesztés korlátozásához. Alapértelmezett rekordot igényel.
-   **Geoproximity útválasztás:** A forgalmat a felhasználók és az erőforrások földrajzi elhelyezkedése alapján irányítja. Egy "bias" értéket használ a forgalom nagyobb vagy kisebb mértékű eltolására egy erőforrás felé. Route 53 Traffic Flow-t igényel.
-   **Többértékű válasz útválasztás:** Egyetlen DNS lekérdezésre akár 8 egészséges rekordot is visszaad. A kliensek véletlenszerűen választanak egyet. Társítható egészségügyi ellenőrzésekkel. Nem helyettesíti az ELB-t.

## Egészségügyi ellenőrzések
Figyelik az erőforrások állapotát.
-   **Végpont egészségügyi ellenőrzés:** Nyilvános végpontokat (alkalmazás, szerver, AWS erőforrás) figyel HTTP, HTTPS vagy TCP protokollon keresztül. Több globális helyről érkező egészségügyi ellenőrzők küldenek kéréseket.
-   **Számított egészségügyi ellenőrzés:** Több gyermek egészségügyi ellenőrzés eredményeit kombinálja (akár 256) ÉS/VAGY/NEM logikával.
-   **CloudWatch riasztás egészségügyi ellenőrzés:** CloudWatch riasztást figyel. Hasznos privát erőforrások (pl. EC2 példány privát alhálózatban) ellenőrzésére a riasztás állapotának az egészségügyi ellenőrzéshez való kapcsolásával.
-   **Hálózati konfiguráció:** Az egészségügyi ellenőrzők IP-tartományait engedélyezni kell a biztonsági csoportokban/tűzfalakon.

## S3 weboldal Route 53-mal
Statikus weboldal hosztolása S3-on egyéni domainnel:
1.  Hozzon létre egy S3 tárolót a domainnel pontosan megegyező névvel (pl. `blog.example.com`).
2.  Engedélyezze a statikus weboldal hosztolást az S3 tárolón, és tegye nyilvánossá az objektumokat.
3.  Hozzon létre egy Alias rekordot a Route 53-ban, amely az S3 weboldal végpontjára mutat.
    -   Csak HTTP forgalomra működik, HTTPS-re nem (HTTPS-hez CloudFront szükséges).

## Route 53 Resolvers (Hibrid DNS)
Összeköti az AWS DNS-t a helyszíni DNS-sel.
-   **Bejövő végpont:** Lehetővé teszi a helyszíni DNS feloldóknak, hogy feloldják az AWS erőforrások (pl. privát hosztolt zónák) domain neveit.
-   **Kimenő végpont:** Lehetővé teszi a Route 53 Resolver számára, hogy a DNS lekérdezéseket a helyszíni DNS feloldóknak továbbítsa.

## Harmadik féltől származó domainek Route 53-mal
Használhatja a Route 53-at DNS szolgáltatásként egy harmadik féltől származó regisztrátorral (pl. GoDaddy) regisztrált domainhez.
1.  Hozzon létre egy nyilvános hosztolt zónát a Route 53-ban a domainjéhez.
2.  Frissítse az NS (Name Server) rekordokat a harmadik féltől származó regisztrátor weboldalán, hogy a Route 53 névszervereire mutassanak.