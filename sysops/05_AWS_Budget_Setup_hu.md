
# 5. Fejezet: EC2 Kezelése Nagy Méretekben (Systems Manager - SSM)

Ez a fejezet átfogó képet ad az AWS Systems Manager (SSM) szolgáltatáscsomagról, amelyet az EC2 példányok és a helyi (on-premises) szerverek flottájának nagy méretekben történő kezelésére és automatizálására terveztek.

## 5.1: Systems Manager Áttekintés
- **AWS Systems Manager (SSM):** Eszközök gyűjteménye, amely operatív betekintést nyújt és lehetővé teszi az AWS erőforrásokon való beavatkozást. Segít a frissítések (patching), a konfigurációkezelés és az automatizálás terén EC2 és helyi rendszerek esetében (ezért hibrid szolgáltatás).
- **Alapvető Követelmény:** Az **SSM Agent**-nek telepítve kell lennie és futnia kell minden kezelni kívánt gépen. Az Amazon Linux 2 és néhány Ubuntu AMI előre telepítve tartalmazza. A példánynak szüksége van egy megfelelő **IAM Szerepkörre** (`AmazonSSMManagedInstanceCore` házirend) is, hogy kommunikálhasson az SSM szolgáltatással.
- **Főbb Jellemzők:** A fejezet több kulcsfontosságú funkcióra összpontosít, beleértve a Resource Groups, Run Command, Automation, Parameter Store, Patch Manager és Session Manager szolgáltatásokat.

## 5.2: EC2 Példányok Indítása SSM Agent-tel
- Ahhoz, hogy a példányokat regisztráljuk az SSM-ben, telepíteni kell rájuk az SSM Agentet, és csatolni kell hozzájuk egy IAM szerepkört az `AmazonSSMManagedInstanceCore` házirenddel.
- Miután egy példány megfelelően van konfigurálva és fut, automatikusan megjelenik „menedzselt csomópontként” (managed node) az **SSM Fleet Managerben**.
- **Biztonsági Előny:** A példányoknak nincs szükségük nyitott bejövő portokra (mint az SSH 22-es portja) ahhoz, hogy az SSM kezelni tudja őket. Az ügynök kimenő kapcsolatot kezdeményez az SSM szolgáltatás felé.

## 5.3: AWS Címkék és SSM Erőforráscsoportok
- **Címkék (Tags):** Kulcs-érték párok, amelyeket az AWS erőforrások címkézésére és szervezésére használnak. Elengedhetetlenek az automatizáláshoz, a biztonsághoz és a költségelosztáshoz.
- **Erőforráscsoportok (Resource Groups):** Lehetőséget adnak az AWS erőforrások közös címkék alapján történő csoportosítására. Ez lehetővé teszi, hogy egy erőforrás-gyűjteményt egyetlen egységként kezeljünk és hajtsunk végre rajtuk műveleteket.
- **Felhasználási Terület:** Létrehozhat erőforráscsoportokat különböző környezetekhez (pl. `dev`, `prod`) vagy csapatokhoz (pl. `finance`, `operations`). Ezeket a csoportokat az SSM műveletek megcélozhatják, lehetővé téve például egy parancs futtatását csak a `dev` példányokon.

## 5.4: SSM Dokumentumok és Run Command
- **SSM Dokumentumok:** JSON vagy YAML fájlok, amelyek meghatározzák az SSM által végrehajtandó műveleteket. Az AWS számos előre konfigurált nyilvános dokumentumot biztosít, de létrehozhat sajátokat is.
- **SSM Run Command:** Lehetővé teszi egy parancs vagy egy SSM Dokumentum végrehajtását egy vagy több menedzselt példányon.
  - **Nincs Szükség SSH-ra:** A parancsot az SSM Agent hajtja végre, nem egy nyitott SSH porton keresztül.
  - **Célpontok:** Megcélozhat példányokat azonosító, címkék vagy erőforráscsoportok alapján.
  - **Sebességszabályozás (Rate Control):** Szabályozhatja a végrehajtás párhuzamosságát (pl. egyszerre csak egy példányon fusson), és beállíthat egy hibaküszöböt a parancs leállítására, ha túl sok hiba történik.
  - **Kimenet:** A parancs kimenete megtekinthető a konzolon, vagy naplózható S3-ba és CloudWatch Logs-ba.

## 5.5: SSM Automatizálás (Automation)
- **SSM Automation:** Az AWS erőforrásokon (nem csak EC2 példányokon) végzett általános karbantartási és telepítési feladatok egyszerűsítésére és automatizálására szolgál.
- **Runbook-ok:** Az automatizáláshoz használt SSM Dokumentumokat gyakran Runbook-oknak nevezik. Lépések sorozatát hajthatják végre, például egy példány újraindítását, AMI létrehozását vagy EBS pillanatkép készítését.
- **Automation vs. Run Command:** A Run Command szkripteket futtat a példányon *belül*, míg az Automation műveleteket vezényel az AWS erőforrásokon *kívülről* (pl. AWS API-k hívása a példány leállítására/elindítására).
- **Indítók (Triggers):** Az automatizálásokat el lehet indítani manuálisan, ütemezés szerint (EventBridge vagy Maintenance Windows segítségével), vagy egy AWS Config szabály javító intézkedéseként.

## 5.6: SSM Parameter Store
- **SSM Parameter Store:** Biztonságos, hierarchikus tároló a konfigurációs adatok és titkok számára.
- **Jellemzők:**
  - **Hierarchia:** A paramétereket útvonalakba lehet szervezni (pl. `/my-app/dev/db-password`), ami egyszerűsíti a kezelést és az IAM jogosultságokat.
  - **Paramétertípusok:** `String`, `StringList` és `SecureString`. A `SecureString` típusú paraméterek AWS KMS segítségével vannak titkosítva.
  - **Verziókövetés:** Nyomon követi a paraméterek időbeli változásait.
  - **Szintek:** Ingyenes **Standard** szint és fizetős **Advanced** szint, amely nagyobb paraméterméreteket és paraméter-házirendeket (pl. lejárati dátum beállítása egy jelszóhoz) tesz lehetővé.
- **Hozzáférés:** A paraméterek lekérdezhetők az AWS CLI-n vagy SDK-n keresztül. A `--with-decryption` kapcsoló szükséges a `SecureString` paraméterek tiszta szöveges értékének lekéréséhez.

## 5.7: SSM Inventory és State Manager
- **SSM Inventory:** Metaadatokat gyűjt a menedzselt példányokról, például a telepített alkalmazásokról, OS verziókról, hálózati konfigurációkról és egyebekről.
- **State Manager:** Egy automatizálási szolgáltatás, amely segít a menedzselt példányokat egy meghatározott állapotban tartani. Létrehoz egy **Asszociációt (Association)**, amely meghatározza a kívánt állapotot (egy SSM Dokumentum által definiálva) és a célpéldányokat. A State Manager ezután automatikusan alkalmazza ezt a konfigurációt egy Ön által meghatározott ütemezés szerint.
- **Felhasználási Terület:** Biztosítja, hogy egy adott szoftverkészlet mindig telepítve legyen, egy vírusirtó fusson, vagy a portok zárva legyenek.

## 5.8: SSM Patch Manager és Maintenance Windows
- **SSM Patch Manager:** Automatizálja a menedzselt példányok frissítésének (patching) folyamatát mind az operációs rendszerek, mind az alkalmazások esetében.
- **Patch Baselines (Frissítési Alapvonalak):** Meghatározzák, hogy mely frissítések legyenek jóváhagyva a telepítésre. Használhatók az AWS által menedzselt alapértelmezett alapvonalak, vagy létrehozhatók egyéniek is, amelyekben jóváhagyhat vagy elutasíthat bizonyos frissítéseket.
- **Patch Groups (Frissítési Csoportok):** Lehetőséget ad egy példánycsoport (egy adott címke alapján) egy adott frissítési alapvonalhoz való társítására. Például lehet egy `dev` csoport, amely azonnal megkapja a frissítéseket, és egy `prod` csoport, amely csak 7 napos késleltetéssel.
- **Maintenance Windows (Karbantartási Ablakok):** Meghatároz egy ismétlődő időszakot (pl. minden vasárnap hajnali 2 és 4 óra között), amely alatt zavaró műveleteket, például frissítéseket vagy szoftvertelepítéseket végezhet a példányain anélkül, hogy az befolyásolná az éles forgalmat.

## 5.9: SSM Session Manager
- **SSM Session Manager:** Biztonságos és naplózható példánykezelést biztosít anélkül, hogy bejövő portokat (mint az SSH) kellene megnyitni, bastion hosztokat kellene kezelni vagy SSH kulcsokkal kellene foglalkozni.
- **Működése:** Böngészőalapú shellt vagy parancssori kapcsolatot biztosít a példányokhoz egy biztonságos alagúton keresztül, az SSM szolgáltatáson át.
- **Előnyök:**
  - **Fokozott Biztonság:** A nyitott portok hiánya csökkenti a támadási felületet.
  - **Naplózhatóság:** Minden munkamenet-tevékenység naplózható a CloudWatch Logs-ba vagy az S3-ba a teljes körű auditálhatóság érdekében.
  - **IAM Szabályozás:** A hozzáférést teljes mértékben az IAM házirendek szabályozzák, lehetővé téve a részletes kontrollt afölött, hogy mely felhasználók mely példányokhoz férhetnek hozzá.
