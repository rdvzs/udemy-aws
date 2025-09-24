
# 8. Fejezet: CloudFormation SysOps Szemszögből

Ez a fejezet mélyrehatóan foglalkozik az AWS CloudFormation szolgáltatással SysOps nézőpontból, a sablonstruktúrára, a telepítési stratégiákra és az infrastruktúra mint kód (IaC) kezelésének haladó funkcióira összpontosítva.

## 8.1: CloudFormation Áttekintés
- **Infrastruktúra mint Kód (IaC):** A CloudFormation egy deklaratív módja a teljes AWS infrastruktúra modellezésének, kiépítésének és kezelésének kód (YAML vagy JSON) segítségével. Ez lehetővé teszi a verziókövetést, a szakmai felülvizsgálatot és az automatizálást.
- **Előnyök:**
  - **Automatizálás és Irányítás:** Kiküszöböli a manuális beállítást és biztosítja a következetességet.
  - **Termelékenység:** Gyorsan hozhat létre, frissíthet és törölhet teljes környezeteket (ún. Stack-eket).
  - **Költségkezelés:** Az erőforrások címkézve vannak, és a költségek a telepítés előtt megbecsülhetők.
- **Folyamat:** Ír egy sablont, feltölti az S3-ba, és a CloudFormation létrehoz egy **Stack**-et azáltal, hogy a megadott erőforrásokat a megfelelő sorrendben kiépíti.

## 8.2: CloudFormation Sablonstruktúra
- **YAML:** Az olvashatósága miatt preferált formátum. Kulcs-érték párokat, beágyazott objektumokhoz behúzást és listákhoz kötőjeleket használ.
- **Alapvető Szekciók:**
  - **`Resources` (Kötelező):** Az egyetlen kötelező szekció. Meghatározza a létrehozandó AWS erőforrásokat (pl. `AWS::EC2::Instance`, `AWS::S3::Bucket`).
  - **`Parameters`:** Dinamikus bemeneteket definiál a sablonhoz, lehetővé téve a testreszabást a telepítéskor (pl. példánytípus megadása).
  - **`Mappings`:** Fix kulcs-érték párok gyűjteménye. Hasznos értékek kiválasztásához egy kulcs, például az AWS Régió alapján (pl. a megfelelő AMI ID kiválasztása `us-east-1` vs. `eu-west-1` esetén).
  - **`Outputs`:** Olyan kimeneti értékeket deklarál, amelyeket megtekinthet vagy importálhat más stack-ekbe (pl. egy VPC ID exportálása).
  - **`Conditions`:** Szabályozza, hogy bizonyos erőforrások létrejönnek-e vagy tulajdonságok hozzárendelődnek-e egy feltétel alapján (pl. erőforrás létrehozása csak akkor, ha a környezet `prod`).

## 8.3: Belső Függvények (Sablon Segédfüggvények)
- **`!Ref`:** Egy paraméter értékére vagy egy erőforrás fizikai azonosítójára hivatkozik.
- **`!GetAtt`:** (Get Attribute) Egy attribútumot kér le egy erőforrásból (pl. `!GetAtt MyEC2Instance.PublicIp`).
- **`!FindInMap`:** Egy értéket kér le egy `Mappings` szekcióból.
- **`!ImportValue`:** Egy másik stack `Outputs` szekciójából exportált értéket importál.
- **`!Join`:** Értékek listáját fűzi össze egy elválasztóval.
- **`!Sub`:** Változókat helyettesít be egy sztringbe.
- **Feltételes Függvények:** `!And`, `!Equals`, `!If`, `!Not`, `!Or`.

## 8.4: Példányok Inicializálása (User Data és cfn-init)
- **`UserData`:** Egy EC2 példány erőforrás tulajdonsága, amellyel egy szkriptet adhatunk át, ami a példány első indításakor fut le. A szkriptet Base64 kódolással kell ellátni.
- **`cfn-init` és `AWS::CloudFormation::Init`:** A példányok konfigurálásának strukturáltabb és hatékonyabb módja.
  - Egy konfigurációt (telepítendő csomagok, létrehozandó fájlok, indítandó szolgáltatások) definiál az EC2 erőforrás `Metadata` szekciójában, az `AWS::CloudFormation::Init` alatt.
  - A `UserData` szkript ezután egyszerűen meghívja a `cfn-init` segédszkriptet, amely beolvassa a metaadatokat és alkalmazza a konfigurációt.
  - Ez olvashatóbb és karbantarthatóbb, mint egy hosszú bash szkript a `UserData`-ban.
- **`cfn-signal` és `WaitCondition`:** Egy mechanizmus, amellyel egy példány konfigurációjának sikerességét vagy sikertelenségét jelezhetjük vissza a CloudFormation-nek.
  - A `UserData` szkript futtatja a `cfn-init`-et, majd a `cfn-signal` segítségével elküldi a kilépési kódot (0 a siker) egy `WaitCondition` erőforrásnak.
  - Ha a `WaitCondition` nem kapja meg a szükséges számú sikeres jelet egy időkorláton belül, a stack létrehozása meghiúsul, ami visszagörgetést (rollback) indít el. Ez biztosítja, hogy egy hibásan induló példány ne eredményezzen „sikeres”, de valójában hibás stack-et.

## 8.5: Stack Kezelés és Házirendek
- **Visszagörgetések (Rollbacks):**
  - **Hiba Létrehozáskor:** Alapértelmezetten, ha a stack létrehozása meghiúsul, minden létrehozott erőforrás törlődik (visszagörgetés). Ezt letilthatja hibaelhárítás céljából.
  - **Hiba Frissítéskor:** A stack automatikusan visszagördül az utolsó ismert, működő állapotra.
- **`DependsOn` Attribútum:** Kifejezetten meghatározza a létrehozási függőséget. Például kényszerítheti, hogy egy S3 vödör csak egy EC2 példány létrehozása *után* jöjjön létre. A CloudFormation a legtöbb függőséget kikövetkezteti (a `!Ref` segítségével), de a `DependsOn` a explicit sorrendiségre szolgál.
- **`DeletionPolicy` (Törlési Házirend):** Szabályozza, mi történik egy erőforrással, amikor eltávolítják egy stack-ből.
  - **`Delete` (Alapértelmezett):** Az erőforrás törlődik.
  - **`Retain` (Megtartás):** Az erőforrás megmarad (árván marad) a stack törlése után. Hasznos kritikus erőforrások, például adatbázisok védelmére.
  - **`Snapshot` (Pillanatkép):** Az ezt támogató erőforrások (mint az EBS kötetek és RDS adatbázisok) esetében egy utolsó pillanatkép készül az erőforrás törlése előtt.
- **`TerminationProtection` (Megszüntetés Elleni Védelem):** Egy stack-szintű beállítás, amely megakadályozza a stack véletlen törlését.
- **Stack Házirendek (Stack Policies):** Egy JSON dokumentum, amely megvédi a stack erőforrásait a nem szándékos frissítésektől. Amikor egy stack házirendet alkalmaznak, alapértelmezetten minden erőforrás védetté válik, és explicit módon kell `Allow` (engedélyezni) a frissítéseket a kívánt erőforrásokon.

## 8.6: Haladó CloudFormation Témák
- **Egyéni Erőforrások (Custom Resources):** Lehetővé teszi egyéni kiépítési logika írását egy AWS Lambda függvényben olyan erőforrásokhoz, amelyeket a CloudFormation natívan nem támogat. Gyakori felhasználási eset egy olyan egyéni erőforrás létrehozása, amely kiürít egy S3 vödröt, mielőtt a CloudFormation törölné azt.
- **Dinamikus Hivatkozások (Dynamic References):** Lehetőséget ad értékek lekérésére az SSM Parameter Store-ból vagy az AWS Secrets Manager-ből a telepítés időpontjában. Ezzel elkerülhető az érzékeny információk vagy konfigurációs értékek keménykódolása a sablonokban. A szintaxis: `{{resolve:service:reference-key}}`.
- **StackSets:** Egy hatékony funkció, amellyel ugyanazt a CloudFormation stack-et lehet telepíteni több AWS-fiókban és régióban egyetlen művelettel. Ideális alapvető erőforrások, biztonsági szerepkörök vagy konfigurációk beállítására egy egész AWS Organization-ön keresztül.
