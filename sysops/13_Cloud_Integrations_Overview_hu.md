# 13. fejezet: Amazon S3 biztonság

## S3 titkosítás
Az S3 négy módszert kínál az objektumok titkosítására:
-   **Szerveroldali titkosítás S3 által kezelt kulcsokkal (SSE-S3):** AES-256 titkosítást használ AWS által kezelt és birtokolt kulcsokkal. Alapértelmezetten engedélyezve van az új tárolók és objektumok számára. `x-amz-server-side-encryption: AES256` fejlécet igényel.
-   **Szerveroldali titkosítás KMS által kezelt kulcsokkal (SSE-KMS):** AWS Key Management Service (KMS) szolgáltatást használ a kulcskezeléshez, felhasználói vezérlést és a kulcshasználat CloudTrail naplózását biztosítva. `x-amz-server-side-encryption: aws:kms` fejlécet igényel. A KMS API hívási kvóták hatálya alá tartozik.
-   **Szerveroldali titkosítás ügyfél által biztosított kulcsokkal (SSE-C):** A titkosítási kulcsokat az AWS-en kívül kezelik, és az ügyfél biztosítja minden kéréssel. Az AWS nem tárolja a kulcsot. HTTPS-t és a kulcs HTTP fejlécekben történő átadását igényli.
-   **Ügyféloldali titkosítás:** Az adatokat az ügyfél titkosítja, mielőtt elküldi az S3-nak, és az ügyfél fejti vissza a lekérés után. Az ügyfél teljes mértékben kezeli a kulcsokat és a titkosítási ciklust.

## Titkosítás átvitel közben (SSL/TLS)
Az S3 tárolók HTTP (titkosítatlan) és HTTPS (titkosított) végpontokkal rendelkeznek. A biztonságos adatátvitelhez a HTTPS használata ajánlott. A tárolóházirendek kikényszeríthetik a HTTPS-t azáltal, hogy megtagadják azokat a kéréseket, ahol az `aws:SecureTransport` hamis.

## Alapértelmezett titkosítás vs. tárolóházirendek
Alapértelmezetten az új tárolók SSE-S3 titkosítással rendelkeznek. Ez megváltoztatható SSE-KMS-re. A tárolóházirendek kikényszeríthetik a specifikus titkosítási típusokat (pl. SSE-KMS vagy SSE-C) azáltal, hogy megtagadják a PUT műveleteket a megfelelő titkosítási fejlécek nélkül. A tárolóházirendek kiértékelése az alapértelmezett titkosítási beállítások előtt történik.

## Cross-Origin Resource Sharing (CORS)
A CORS egy webböngésző biztonsági mechanizmus, amely engedélyezi vagy megtagadja a kéréseket különböző forrásokhoz (séma, gazdagép, port) egy fő forrás látogatása közben. Az S3 esetében, ha egy ügyfél cross-origin kérést hajt végre (pl. egy S3-on tárolt weboldal megpróbál betölteni eszközöket egy másik S3 tárolóból vagy domainről), az eszközöket szolgáltató S3 tárolónak rendelkeznie kell egy CORS konfigurációval, amely engedélyezi a kérést küldő weboldal forrását. Ez egy JSON házirendben van konfigurálva az S3 tárolón.

## MFA törlés
A Multi-Factor Authentication (MFA) Delete egy biztonsági funkció, amely MFA kódot igényel a kritikus S3 műveletekhez:
-   Objektumverzió végleges törlése.
-   Verziózás felfüggesztése egy tárolón.
Az MFA Delete-t a tároló tulajdonosának (gyökérfiók) kell engedélyeznie az AWS CLI használatával (nem közvetlenül a konzol felhasználói felületéről). A tárolón engedélyezni kell a verziózást.

## S3 hozzáférési naplók
Auditálási célokra az S3 tárolóhoz érkező összes hozzáférési kérés (engedélyezett vagy megtagadott) naplózható fájlként egy másik S3 tárolóba. Ezek a naplók ezután elemezhetők olyan eszközökkel, mint az Amazon Athena. A naplózó tárolónak ugyanabban az AWS régióban kell lennie, mint a forrás tárolónak. **Fontos:** A naplózó tároló soha nem lehet azonos a forrás tárolóval, hogy elkerüljük a végtelen naplózási hurkot.

## S3 előre aláírt URL-ek
Az előre aláírt URL-ek ideiglenes hozzáférést biztosítanak privát S3 objektumokhoz korlátozott ideig (akár 12 óra a konzolon keresztül, 168 óra a CLI/SDK-n keresztül). Az URL-t elérő felhasználó örökli az URL-t generáló felhasználó engedélyeit. Ez hasznos privát fájlok megosztására külső felhasználókkal anélkül, hogy a tárolót vagy az objektumot nyilvánossá tennénk, vagy ideiglenes feltöltések engedélyezésére specifikus helyekre.

## S3 objektumzár
Az S3 Object Lock lehetővé teszi a Write Once Read Many (WORM) modellt az S3 tárolókban lévő objektumok számára (verziózás engedélyezése szükséges). Megakadályozza az objektumverziók felülírását vagy törlését egy meghatározott ideig.
-   **Megőrzési módok:**
    -   **Megfelelőségi mód:** Az objektumverziókat semmilyen felhasználó, beleértve a gyökérfelhasználót sem, nem írhatja felül vagy törölheti. A megőrzési beállítások nem módosíthatók vagy rövidíthetők.
    -   **Kormányzási mód:** A legtöbb felhasználó nem írhatja felül vagy törölheti az objektumverziókat, és nem módosíthatja a zárolási beállításokat, de a jogosult felhasználók (specifikus IAM engedélyekkel) megtehetik.
-   **Megőrzési időszak:** Az objektumok rögzített ideig védettek, amely meghosszabbítható.
-   **Jogi zárolás:** Korlátlan ideig védi az objektumot, függetlenül a megőrzési időszaktól. Az alkalmazásához vagy eltávolításához `s3:PutObjectLegalHold` IAM engedély szükséges.

## S3 hozzáférési pontok
Az S3 hozzáférési pontok egyszerűsítik az S3 tárolók biztonsági kezelését, különösen a nagy adathalmazok és a változatos hozzáférési minták esetén. Minden hozzáférési pontnak saját DNS neve van, és rendelkezhet egy specifikus hozzáférési pont házirenddel (hasonlóan a tárolóházirendhez), amely részletes engedélyeket biztosít (pl. olvasási/írási hozzáférés egy specifikus előtaghoz). Ez lehetővé teszi a biztonság skálázható kezelését azáltal, hogy a hozzáférés-vezérlést egy komplex tárolóházirendről több egyszerűbb hozzáférési pont házirendre delegálja.
-   **Hálózati forrás:** Konfigurálható internetes hozzáférésre vagy csak VPC-hozzáférésre (privát forgalomhoz egy VPC-n belül).
-   **VPC végpontok:** Privát hozzáféréshez VPC Endpoint Gateway-t használnak az S3 hozzáférési pontokhoz való csatlakozáshoz a nyilvános interneten keresztül történő forgalom nélkül. A VPC végpont házirendek tovább korlátozhatják a hozzáférést.

## S3 több régiós hozzáférési pontok
Globális végpontot biztosítanak, amely több S3 tárolót fog át különböző régiókban. Dinamikusan irányítják a kéréseket a legközelebbi S3 tárolóhoz a legalacsonyabb késleltetés érdekében. Kétirányú replikációt igényelnek a társított S3 tárolók között az adatok konzisztenciájának biztosításához. Átállási vezérlőket (aktív/passzív vagy aktív/aktív beállítások) kínálnak a magas rendelkezésre állás és a katasztrófa-helyreállítás érdekében.