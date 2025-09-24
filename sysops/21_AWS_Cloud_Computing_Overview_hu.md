# 21. fejezet: Identitás

## IAM biztonsági eszközök
-   **IAM hitelesítő adatok jelentése:** Fiókszintű jelentés, amely felsorolja az összes felhasználót és különböző hitelesítő adataik állapotát (jelszó engedélyezve, MFA aktív, hozzáférési kulcsok generálva/utoljára használva/rotálva). Hasznos auditáláshoz és biztonsági kockázatok azonosításához.
-   **IAM Access Advisor:** Felhasználói szintű eszköz, amely megmutatja a felhasználónak adott szolgáltatási engedélyeket és azt, hogy mikor használták utoljára ezeket a szolgáltatásokat. Segít a legkisebb jogosultság elvének megvalósításában azáltal, hogy azonosítja a nem használt engedélyeket.

## IAM Access Analyzer
Azonosítja a külsőleg megosztott erőforrásokat (a meghatározott megbízhatósági zónán kívül, pl. az AWS fiókján vagy szervezetén kívül).
-   **Figyelt erőforrások:** S3 tárolók, IAM szerepkörök, KMS kulcsok, Lambda függvények/rétegek, SQS üzenetsorok, Secrets Manager titkok.
-   **Megbízhatósági zóna:** Az AWS fiókja(i) vagy szervezete.
-   **Találatok:** A megbízhatósági zónán kívüli entitásokkal megosztott erőforrások találatként kerülnek megjelölésre.
-   **Helyreállítás:** Linkeket biztosít az erőforrásokhoz a nem szándékos hozzáférés kijavításához. Az archivált találatok szándékosak lehetnek.

## Identitás-összevonás
Lehetővé teszi az AWS-en kívüli felhasználók számára, hogy ideiglenes szerepköröket vegyenek fel az AWS erőforrások eléréséhez anélkül, hogy egyedi IAM felhasználókat hoznának létre. A felhasználókezelés az AWS-en kívül történik.
-   **SAML összevonás:** Vállalatok számára SAML 2.0 kompatibilis identitásszolgáltatókkal (pl. Microsoft Active Directory). A felhasználók hitelesítik magukat az IDP-vel, kapnak egy SAML állítást, kicserélik azt az AWS STS-szel ideiglenes hitelesítő adatokra, majd hozzáférnek az AWS konzolhoz/CLI-hez.
-   **Egyéni identitás bróker:** Nem SAML 2.0 identitásszolgáltatókhoz. Egyéni alkalmazásfejlesztést igényel az identitás programozott cseréjéhez az AWS STS-szel ideiglenes hitelesítő adatokra. Bonyolultabb, mint az SAML.
-   **Cognito Identity Pools (Federated Identities):** Webes és mobil alkalmazásfelhasználók számára az AWS erőforrások eléréséhez.
    -   A felhasználók nyilvános szolgáltatókon (Google, Facebook, Amazon, Apple), Cognito User Pools-on, OpenID Connect-en, SAML-en vagy egyéni fejlesztő által hitelesített identitásokon keresztül jelentkeznek be.
    -   Identitás tokeneket cserél ideiglenes AWS hitelesítő adatokra az STS-en keresztül.
    -   Engedélyezheti a nem hitelesített (vendég) felhasználókat.
    -   Az IAM szerepkörökhöz csatolt IAM házirendek határozzák meg az engedélyeket. Használhat házirend-változókat a finomhangolt vezérléshez (pl. felhasználóspecifikus S3 előtagok, DynamoDB sorok).

## AWS Security Token Service (STS)
Korlátozott és ideiglenes hozzáférést biztosít az AWS erőforrásokhoz. A tokenek korlátozott ideig érvényesek (pl. legfeljebb 1 óra), és frissíteni kell őket.
-   **API hívások:**
    -   `AssumeRole`: IAM szerepkör felvétele ugyanazon fiókon belül vagy fiókok között.
    -   `AssumeRoleWithSAML`: SAML állítás cseréje ideiglenes hitelesítő adatokra.
    -   `AssumeRoleWithWebIdentity`: Identitás cseréje webes identitásszolgáltatóktól (pl. Facebook, Google) ideiglenes hitelesítő adatokra (most már a Cognito használata ajánlott).
    -   `GetSessionToken`: Ideiglenes hitelesítő adatok beszerzése MFA-val hitelesített felhasználók számára.
-   **Fiókok közötti hozzáférés:** Az `AssumeRole` gyakori felhasználási esete, ahol az egyik fiókban lévő felhasználók egy másik fiókban lévő szerepkört vesznek fel műveletek végrehajtásához.

## Cognito User Pools
Szerver nélküli felhasználói könyvtár webes és mobil alkalmazásokhoz. Hitelesítésre (identitás ellenőrzés) használják.
-   **Jellemzők:** Felhasználói regisztráció, bejelentkezés (felhasználónév/e-mail és jelszó), jelszó visszaállítás, e-mail/telefon ellenőrzés, MFA, közösségi bejelentkezések (Google, Facebook, Amazon), vállalati bejelentkezések (SAML, OpenID Connect).
-   **Kimenet:** Sikeres hitelesítés után JSON Web Token (JWT) ad vissza.
-   **Integráció:** Natívan integrálódik az API Gateway-vel és az Application Load Balancer-rel a hitelesítéshez.

## Cognito User Pools vs. Identity Pools
-   **User Pools (Hitelesítés):** Kezeli a felhasználói identitásokat, biztosítja a bejelentkezést/regisztrációt, JWT-ket ad ki. A hangsúly azon van, hogy *ki* a felhasználó.
-   **Identity Pools (Engedélyezés):** Identitás tokeneket (User Pools-ból vagy más identitásszolgáltatóktól) cserél ideiglenes AWS hitelesítő adatokra. A hangsúly azon van, hogy *mit* tehet a felhasználó az AWS-ben.
-   Használhatók együtt: A User Pool hitelesíti a felhasználót, majd az Identity Pool biztosítja az AWS hozzáférést a hitelesítés alapján.