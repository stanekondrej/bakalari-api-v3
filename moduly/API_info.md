# API info

## Přehled všech dostupných API verzí

`GET /api`

### Odpověď


``` json
[
    {
        "ApiVersion": "3.12.0",
        "ApplicationVersion": "1.32.625.2",
        "BaseUrl": "api/3"
    }
]
```

## Informace o API verze 3

`GET /api/3`

### Odpověď

``` json
{
    "ApiVersion": "3.12.0",
    "ApplicationVersion": "1.32.625.2",
    "BaseUrl": "api/3"
}
```

## Chyby

### POST požadavek (špatná metoda)

`405 Method Not Allowed`

```json
{
    "Message": "The requested resource does not support http method 'POST'."
}```







