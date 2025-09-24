
# 7. Fejezet: Elastic Beanstalk SysOps Szemszögből

Ez a fejezet az AWS Elastic Beanstalk szolgáltatást mutatja be, egy Platform-as-a-Service (PaaS) megoldást, amely leegyszerűsíti a webalkalmazások telepítését és kezelését. Automatizálja az alapul szolgáló infrastruktúrát, lehetővé téve a fejlesztők számára, hogy a kódjukra összpontosítsanak.

## 7.1: Elastic Beanstalk Áttekintés
- **Mi az az Elastic Beanstalk?** Egy fejlesztőközpontú szolgáltatás webalkalmazások és -szolgáltatások telepítésére és skálázására. Ez egy PaaS, amely automatizálja egy teljes alkalmazás-verem (stack) beállítását, beleértve az EC2 példányokat, Auto Scaling Csoportokat, Elastic Load Balancereket és opcionálisan RDS adatbázisokat.
- **Fejlesztői Fókusz:** A fejlesztő elsődleges felelőssége az alkalmazáskód biztosítása. A Beanstalk kezeli a kapacitás-ellátást, a terheléselosztást, a skálázást és az alkalmazás állapotának figyelését.
- **Menedzselt Szolgáltatás:** Bár a Beanstalk egy menedzselt szolgáltatás, továbbra is teljes körű ellenőrzést biztosít az általa létrehozott AWS erőforrások konfigurációja felett.
- **Költségek:** Maga a Beanstalk szolgáltatás ingyenes; csak az általa létesített AWS erőforrásokért (pl. EC2 példányok, ELB, RDS) kell fizetni.

### Kulcsfogalmak
- **Alkalmazás (Application):** A Beanstalk komponensek logikai gyűjteménye, beleértve a környezeteket, verziókat és konfigurációkat.
- **Alkalmazásverzió (Application Version):** Az alkalmazáskód egy meghatározott, címkézett iterációja.
- **Környezet (Environment):** Az alkalmazás egy futó verziója. Több környezet is létezhet, például `dev`, `test` és `prod`.
- **Környezeti Szintek (Environment Tiers):**
  - **Webkiszolgáló Környezet (Web Server Environment):** HTTP(S) kérések kezelésére. Ez a szabványos szint a webalkalmazásokhoz, és tartalmaz egy terheléselosztót és egy auto-scaling csoportot.
  - **Feldolgozó Környezet (Worker Environment):** Háttérfeladatok feldolgozására. Üzeneteket olvas egy Amazon SQS üzenetsorból és aszinkron módon dolgozza fel őket.

### Telepítési Módok
- **Egyetlen Példány (Single Instance):** Az alkalmazást egyetlen EC2 példányon telepíti egy Elastic IP-vel. Ideális fejlesztési és tesztelési környezetekhez. **Nem** magas rendelkezésre állású.
- **Magas Rendelkezésre Állás (Load Balancerrel):** Az alkalmazást több EC2 példányon telepíti egy Auto Scaling Csoportban, egy Application Load Balancer mögött. Ez a szabvány a termelési környezetek számára, skálázhatóságot és hibatűrést biztosítva.

## 7.2: Elastic Beanstalk Gyakorlat
- **Alkalmazás Létrehozása:** A folyamat egy új alkalmazás létrehozásával kezdődik a Beanstalk konzolon.
- **Környezet Létrehozása:**
  1.  **Környezeti Szint Választása:** Válasszon a „Web server environment” vagy a „Worker environment” közül.
  2.  **Alkalmazás Információk:** Nevezze el az alkalmazást és a környezetet (pl. `MyApplication-dev`).
  3.  **Platform:** Válassza ki az alkalmazás technológiai veremét (pl. Node.js, Python, Go, Docker).
  4.  **Alkalmazáskód:** Töltse fel a kódját, vagy használja a mellékelt mintaalkalmazást.
  5.  **Előbeállítások (Presets):** Válasszon egy konfigurációs előbeállítást, mint például a „Single instance” (ingyenes szinten elérhető) vagy a „High availability”.
  6.  **Szolgáltatási Hozzáférés (Service Access):** Konfigurálja az IAM szerepköröket. A Beanstalk-nak szüksége van egy **Szolgáltatási Szerepkörre (Service Role)** az erőforrások kezeléséhez, és egy **EC2 Példányprofilra (Instance Profile)** a létrehozott példányok jogosultságainak biztosításához.
- **A Színfalak Mögött:** Amikor létrehoz egy környezetet, az Elastic Beanstalk a háttérben az **AWS CloudFormation**-t használja az összes szükséges erőforrás (EC2 példányok, biztonsági csoportok, Auto Scaling csoportok stb.) kiépítéséhez. Az eseményeket és a generált sablont megtekintheti a CloudFormation konzolon.
- **Az Alkalmazás Elérése:** A környezet sikeres elindítása után a Beanstalk egy nyilvános URL-t biztosít (pl. `myapplication-dev.elasticbeanstalk.com`) az alkalmazás eléréséhez.
- **Kezelés:** A Beanstalk konzol egy központi irányítópultot biztosít az állapot figyeléséhez, a naplók megtekintéséhez, az alkalmazásverziók kezeléséhez és a környezet alapjául szolgáló erőforrások konfigurálásához.
