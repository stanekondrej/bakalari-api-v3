# Login

Slouží k získání a obnově tokenů pro přístup k API. Pro užívání ostatních
endpointů API potřebuje klient `access_token`, který získá na tomto endpointu.
Po přihlášení musí klient uvést `access_token` v hlavičce každého požadavku,
jako `Authorization: Bearer <access_token>`.

## Požadavek

```
POST /api/login
Content-Type: application/x-www-form-urlencoded
```

| Vlastnost | Hodnota |
|-----------|---------|
| Metoda    | `POST`  |

| Hlavička       | Hodnota                             |
|----------------|-------------------------------------|
| `Content-Type` | `application/x-www-form-urlencoded` |

## První přihlášení

Body: `client_id=ANDR&grant_type=password&username=USERNAME&password=PASSWORD`

| Parametr     | Hodnota               |
|--------------|-----------------------|
| `client_id`  | `ANDR`                |
| `grant_type` | `password`            |
| `username`   | `<uživatelské jméno>` |
| `password`   | `<heslo>`             |

## Odpověď

_Pozor: Počty znaků jsou pouze orientační, aby někdo nebyl zaskočen jejich
délkou, která se od starších verzí API rapidně změnila._

```jsonc
{
   "bak:ApiVersion": "3.13.0",
   "bak:AppVersion": "1.35.1029.1",
   "bak:UserId": "XXXXX",
   "access_token": "access_token",     // 2556 znaků
   "refresh_token": "refresh_token",   // 3459 znaků
   "id_token": "id_token - 872 znaků", // není vždy dostupné
   "token_type": "Bearer",
   "expires_in": 3599, // v sekundách
   "scope": "openid profile offline_access bakalari_api"
}
```

| Pole             | Typ                           |
|------------------|-------------------------------|
| `bak:ApiVersion` | `string`                      |
| `bak:AppVersion` | `string`                      |
| `bak:UserId`     | `string`                      |
| `access_token`   | `string`                      |
| `refresh_token`  | `string`                      |
| `id_token`       | `string?`                     |
| `token_type`     | `string`                      |
| `expires_in`     | `number` (v celých sekundách) |
| `scope`          | `string`                      |

### Starší verze API

```jsonc
{
  "access_token": "access_token",
  "token_type": "bearer",
  "expires_in": 599, // v sekundách
  "refresh_token": "refresh_token",
  "bak:ApiVersion": "3.8.0",
  "bak:AppVersion": "1.28.306.4",
  "bak:UserId": "XXXXX"
}
```

## Přihlášení pomocí refresh tokenu

Body: `client_id=ANDR&grant_type=refresh_token&refresh_token=REFRESHTOKEN`

| Parametr         | Hodnota           |
|------------------|-------------------|
| `client_id`      | `ANDR`            |
| `grant_type`     | `refresh_token`   |
| `refresh_token`  | `<refresh_token>` |

Vrací stejnou strukturu body jako při prvním přihlášení. Zdá se, že i refresh
token asi po měsíci bez obnovy vyprší a je nutné nové přihlášení pomocí
uživatelského jména a hesla.

## Význam tokenů

Tokeny jsou něco jako [JWT](https://cs.wikipedia.org/wiki/JSON_Web_Token).
Jednotlivé části jsou odděleny tečkami, ale s výjimkou hlavičky jsou šifrované
klíčem, který uchovává server ve svém úložišti, takže si je nepřečtete (viz.
políčko hlavičky `enc` - v příkladové odpovědi zde je tělo tokenu šifrováno
pomocí blokové šifry AES-256 a jako hash funkci používá server HS512). Výjimkou
je `id_token`, který šifrovaný není.

Jelikož JWT jsou zakódovány pomocí base64 pro snadný přenos, zde jsou pro
zajímavost příklady dekódovatelných částí tokenů:

### Hlavička `access_token`

```jsonc
{
  "alg":"RSA-OAEP",
  "enc":"A256CBC-HS512",
  "kid":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "typ":"at+jwt" // závěr se bude lišit podle konfigurace školního serveru, může
                 // tady být i políčko "cty", atd.
}
```

### Hlavička `refresh_token`

```jsonc
{
   "alg":"RSA-OAEP",
   "enc":"A256CBC-HS512",
   "kid":"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
   "typ":"oi_reft+jwt" // opět, mohou zde být i další políčka
}
```

### Dekódovaný `id_token`

```jsonc
// hlavička
{
  "alg": "RS256",
  "kid": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "typ": "JWT",
  "x5t": "XXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

```jsonc
// tělo
{
  "sub": "XXXXXX",
  "oi_au_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "jti": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "azp": "ANDR",
  "at_hash": "XXXXXXXXXXXXXXXXXXXXXX",
  "oi_tkn_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "aud": "ANDR",
  "exp": 1601454600, // platný 30 minut
  "iss": "https://bakalari.skola.cz/",
  "iat": 1601452800
}
```

## Chyby

Při chybě endpoint vždy odpovídá s `400 Bad Request`.

### Nesprávné přihlašovací údaje

```json
{
  "error": "invalid_grant",
  "error_description": "Špatný login nebo heslo"
}
```

### Neplatný `refresh_token`

```json
{
  "error": "invalid_grant",
  "error_description": "The specified token is invalid."
}
```

### Použitý `refresh_token`

_Aktuální verze API má v sobě chybu (pokud to tedy není funkce), že jeden
refresh token jde použít vícekrát pro získání rozdílných validních párů tokenů.
Tato chybová odpověď se začne objevovat až po čtvrtém requestu se stejným
tokenem. Tzn. jdou vygenerovat až 3 access tokeny a 3 refresh tokeny z jednoho
refresh tokenu. K této duplikaci tokenů by ale běžně nemělo docházet, a proto ji
prosím nepoužívejte (+není 100% zdokumentovaná) a **uchovávejte vždy pouze
poslední pár tokenů.**_

```json
{
  "error": "invalid_grant",
  "error_description": "The specified refresh token has already been redeemed."
}
```

### Chybějící `grant_type` v těle požadavku

```json
{
  "error": "invalid_request",
  "error_description": "The mandatory 'grant_type' parameter is missing."
}
```

### Chybějící `client_id` v těle požadavku

```json
{
  "error": "invalid_client",
  "error_description": "The mandatory 'client_id' parameter is missing."
}
```

### Odpověď starší API s chybějícím `client_id` či `grant_type`

```json
{
  "error": "invalid_client",
  "error_description": "Unknown Client or invalid grant type."
}
```

