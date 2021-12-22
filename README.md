# API Docs

A continuación se presenta la documentación de la API central de ThePackCo, se trata de una API REST que recibe y envía
información en formato JSON.

## Información general

Las _requests_ se deben hacer a la siguiente URL:

`https://central.thepackco.cl`

Este corresponde al entorno de producción. No existe un entorno de pruebas independiente, pero se entregará dos tokens
de autenticación diferentes: uno para hacer las pruebas y otro para usar de forma definitiva.

Para autenticarse es necesario enviar el token de autenticación entregado por ThePackCo en el _Authorization header_:

`Authorization: Token tu_token_de_autenticacion`

## Postman collection

Para los usuarios de Postman, es posible [descargar un _collection_](TPC-API.postman_collection.json) listo para usar.

## Ejemplo en Python

```python3
import requests

base_url = 'https://central.thepackco.cl'
token = 'd28a7428d66f27ea2cea266ece45c76efda1e5b5'
headers = {'Authorization': f'Token {token}'}

r = requests.get(f'{base_url}/api/shops/', headers=headers)
if r.status_code == 200:
    response = r.json()
```

## Listado de endpoints

- [Autenticación](#autenticación)
- [Obtener tiendas](#obtener-tiendas)
- [Obtener tienda y envíos](#obtener-tienda-y-envíos)
- [Crear envío](#crear-envío)
- [Obtener envío](#obtener-envío)

## Recursos adicionales

- [`Shop`](#shop)
- [`Order`](#order)
- [`OrderEvent`](#orderevent)

____

## Autenticación

Permite verificar el token de autenticación según el _status code_ devuelto.

#### Endpoint

`GET /api/auth/`

#### Parámetros

No recibe.

#### Status codes

- `204`: el token de autenticación es válido.
- `401`: el token de autenticación no es válido.

#### Respuestas

Las respuestas no incluyen información adicional.

## Obtener tiendas

Devuelve las tiendas a las que tiene acceso el usuario.

#### Endpoint

`GET /api/shops/`

#### Parámetros

No recibe.

#### Status codes

- `200`: retorna las tiendas a las que tiene acceso el usuario.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para consultar este recurso.

#### Respuestas

En el caso de una respuesta exitosa (`200`) se recibe una lista de [tiendas](#shop).

<details>
<summary>Ejemplo de usuario con acceso a una única tienda</summary>

```json
[
  {
    "id": 12,
    "name": "Zapatería Chilena"
  }
]
```

</details>

## Obtener tienda y envíos

Devuelve la información de una tienda y, opcionalmente, los envíos asociados a ella.

#### Endpoint

`GET /api/shops/:shop_id/`

#### Parámetros

Parámetros en URL (query string):

|Nombre|Tipo|Default|Descripción|
|------|----|-------|-----------|
|`include_orders`|`true\|false`|`false`|Indica si la respuesta debe incluir los envíos asociados a la tienda. En caso de incluir los envíos, estos contienen solo los atributos más relevantes (ver respuesta de ejemplo para saber qué atributos)|

#### Status codes

- `200`: retorna la tienda solicitida.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para consultar este recurso.
- `404`: no existe una tienda con el `:shop_id` entregado, o es una tienda a la que el usuario no tiene acceso.

#### Respuestas

En el caso de una respuesta exitosa (`200`) devuelve la [tienda](#shop).

<details>
<summary>Ejemplo tienda sin envíos (/api/shops/12/)</summary>

```json
{
  "id": 12,
  "name": "Zapatería Chilena"
}
```

</details>

<details>
<summary>Ejemplo tienda con envíos (/api/shops/12/?include_orders=true)</summary>

```json
{
  "id": 12,
  "name": "Zapatería Chilena",
  "orders": [
    {
      "id": 125,
      "public_id": "6KBULJQK7K",
      "reference_code": "TESTING9898",
      "shop": 12,
      "status": "received",
      "status_name": "Retirado",
      "courier": "tpc",
      "courier_name": "ThePackCo",
      "address_region": "RM",
      "address_region_name": "Metropolitana de Santiago",
      "address_city": "337",
      "address_city_name": "San Bernardo",
      "product_description": "",
      "customer_name": "",
      "customer_phone": "",
      "customer_email": ""
    },
    {
      "id": 81,
      "public_id": "VI74VA3VT4",
      "reference_code": "22222",
      "shop": 12,
      "status": "created",
      "status_name": "Creado",
      "courier": "tpc",
      "courier_name": "ThePackCo",
      "address_region": "RM",
      "address_region_name": "Metropolitana de Santiago",
      "address_city": "343",
      "address_city_name": "Santiago",
      "product_description": "",
      "customer_name": "",
      "customer_phone": "",
      "customer_email": ""
    },
    {
      "id": 82,
      "public_id": "8D5LA8FR4H",
      "reference_code": "5465465TXP",
      "shop": 12,
      "status": "delivered",
      "status_name": "Entregado",
      "courier": "tpc",
      "courier_name": "ThePackCo",
      "address_region": "RM",
      "address_region_name": "Metropolitana de Santiago",
      "address_city": "343",
      "address_city_name": "Santiago",
      "product_description": "",
      "customer_name": "",
      "customer_phone": "",
      "customer_email": ""
    }
  ]
}
```

</details>

## Editar tienda

Edita la información de una tienda. Más precisamente, la información de los webhooks y los correos.

#### Endpoint

`(PATCH/PUT) /api/shops/:shop_id/`

#### Parámetros

No recibe.

#### Status codes

- `200`: edita las configuraciones y retorna la tienda solicitida.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para consultar este recurso.
- `404`: no existe una tienda con el `:shop_id` entregado, o es una tienda a la que el usuario no tiene acceso.

#### Respuestas

En el caso de una respuesta exitosa (`200`) edita y devuelve la [tienda](#shop).

<details>
<summary>Ejemplo modificación con Postman</summary>

En este caso la petición se realiza para cambiar las configuraciones de correo.

```http request
PUT /api/shops/10/?='Authorization': Token 9371f6b8e9331e47b71f9646951b854680c288d9
```

Body

```json
{
    "name": "TyumenShop",
    "mail_preferences_data": {
        "send_at_created": false,
        "send_at_delayed": false,
        "send_at_received": true,
        "send_at_delegated": true,
        "send_at_delivered": true,
        "send_at_dispatched": true
    }
}
```

Respuesta

```json
{
    "id": 10,
    "name": "TyumenShop",
    "settings": {
        "webhooks": {
            "order_updated": {
                "enabled": true,
                "endpoint": "base_url/webhooks/order_update/"
            }
        }
    },
    "webhook_token": "1T15TTGV4T2vDNd6VMXZ548EZfIexrZ8",
    "mail_preferences_data": {
        "send_at_created": false,
        "send_at_delayed": false,
        "send_at_received": true,
        "send_at_delegated": true,
        "send_at_delivered": true,
        "send_at_dispatched": true
    }
}
```



</details>

## Obtener tarifas

Devuelve los precios de despacho (en valor bruto) para cada comuna.

#### Endpoint

`GET /api/shops/:shop_id/rates/`

#### Parámetros

No recibe.

#### Status codes

- `200`: retorna las tarifas asociadas a la tienda.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para consultar este recurso.
- `404`: no existe una tienda con el `:shop_id` entregado, o es una tienda a la que el usuario no tiene acceso.

#### Respuestas

En el caso de una respuesta exitosa (`200`) devuelve las tarifas.

<details>
<summary>Ejemplo respuesta exitosa</summary>

```json
{
  "Arica": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 6318,
    "price_formatted": "$5.265",
    "carrier": "CCH"
  },
  "Camarones": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 11144,
    "price_formatted": "$9.287",
    "carrier": "BLX"
  },
  "General Lagos": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 17145,
    "price_formatted": "$14.288",
    "carrier": "BLX"
  },
  "Putre": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 7581,
    "price_formatted": "$6.318",
    "carrier": "CCH"
  },
  "Alto Hospicio": {
    "name": "normal",
    "code": "normal",
    "transit_days": 8,
    "price": 6553,
    "price_formatted": "$5.461",
    "carrier": "URB"
  },
  "Camiña": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 7,
    "price": 93010,
    "price_formatted": "$77.509",
    "carrier": "SKN"
  },
  "Colchane": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 7,
    "price": 93010,
    "price_formatted": "$77.509",
    "carrier": "SKN"
  },
  "Huara": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 7581,
    "price_formatted": "$6.318",
    "carrier": "CCH"
  },
  "Iquique": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 6318,
    "price_formatted": "$5.265",
    "carrier": "CCH"
  },
  "Pica": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 7581,
    "price_formatted": "$6.318",
    "carrier": "CCH"
  },
  "Pozo Almonte": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 7581,
    "price_formatted": "$6.318",
    "carrier": "CCH"
  },
  "Antofagasta": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5956,
    "price_formatted": "$4.964",
    "carrier": "CCH"
  },
  "Calama": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5956,
    "price_formatted": "$4.964",
    "carrier": "CCH"
  },
  "María Elena": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 6978,
    "price_formatted": "$5.815",
    "carrier": "SKN"
  },
  "Mejillones": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 7148,
    "price_formatted": "$5.957",
    "carrier": "CCH"
  },
  "Ollagüe": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 120000,
    "price_formatted": "$100.000",
    "carrier": "BLX"
  },
  "San Pedro De Atacama": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 6978,
    "price_formatted": "$5.815",
    "carrier": "SKN"
  },
  "Sierra Gorda": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 10752,
    "price_formatted": "$8.960",
    "carrier": "BLX"
  },
  "Taltal": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 4887,
    "price_formatted": "$4.073",
    "carrier": "CHX"
  },
  "Tocopilla": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 6978,
    "price_formatted": "$5.815",
    "carrier": "SKN"
  },
  "Alto Del Carmen": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Caldera": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Chañaral": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Copiapó": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4582,
    "price_formatted": "$3.819",
    "carrier": "CCH"
  },
  "Diego De Almagro": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 4492,
    "price_formatted": "$3.744",
    "carrier": "CHX"
  },
  "Freirina": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Huasco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Tierra Amarilla": {
    "name": "normal",
    "code": "normal",
    "transit_days": 8,
    "price": 5194,
    "price_formatted": "$4.329",
    "carrier": "URB"
  },
  "Vallenar": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 5330,
    "price_formatted": "$4.442",
    "carrier": "SKN"
  },
  "Andacollo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Canela": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Combarbalá": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5398,
    "price_formatted": "$4.499",
    "carrier": "CHX"
  },
  "Coquimbo": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Illapel": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5193,
    "price_formatted": "$4.328",
    "carrier": "BLX"
  },
  "La Higuera": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "La Serena": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Los Vilos": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4582,
    "price_formatted": "$3.819",
    "carrier": "CCH"
  },
  "Monte Patria": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4582,
    "price_formatted": "$3.819",
    "carrier": "CCH"
  },
  "Ovalle": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4818,
    "price_formatted": "$4.015",
    "carrier": "SKN"
  },
  "Paihuano": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Punitaqui": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Río Hurtado": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Salamanca": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5499,
    "price_formatted": "$4.583",
    "carrier": "CCH"
  },
  "Vicuña": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5389,
    "price_formatted": "$4.491",
    "carrier": "BLX"
  },
  "Algarrobo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Cabildo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 4131,
    "price_formatted": "$3.443",
    "carrier": "CHX"
  },
  "Calera": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Calle Larga": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Cartagena": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Casablanca": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Catemu": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Concón": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "El Quisco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "El Tabo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 1,
    "price": 5176,
    "price_formatted": "$4.314",
    "carrier": "CHX"
  },
  "Hijuelas": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Isla De Pascua": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Juan Fernández": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "La Cruz": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "La Ligua": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Limache": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Llay-Llay": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 4860,
    "price_formatted": "$4.050",
    "carrier": "CHX"
  },
  "Los Andes": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Nogales": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Olmué": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Panquehue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Papudo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Petorca": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Puchuncaví": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Putaendo": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Quillota": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Quilpué": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Quintero": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Rinconada": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "San Antonio": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "San Esteban": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "San Felipe": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Santa María": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3355,
    "price_formatted": "$2.796",
    "carrier": "BLX"
  },
  "Santo Domingo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Valparaíso": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Villa Alemana": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Viña Del Mar": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Zapallar": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Chépica": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Chimbarongo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Codegua": {
    "name": "normal",
    "code": "normal",
    "transit_days": 7,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Coínco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Coltauco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Doñihue": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Graneros": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "La Estrella": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Las Cabras": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Litueche": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Lolol": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Machalí": {
    "name": "normal",
    "code": "normal",
    "transit_days": 7,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Malloa": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Marchigüe": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 1,
    "price": 5337,
    "price_formatted": "$4.448",
    "carrier": "CHX"
  },
  "Mostazal": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Nancagua": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Navidad": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Olivar": {
    "name": "normal",
    "code": "normal",
    "transit_days": 7,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Palmilla": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Paredones": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Peralillo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Peumo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Pichidegua": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Pichilemu": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Placilla": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Pumanque": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Quinta De Tilcoco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Rancagua": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "Rengo": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Requínoa": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 3884,
    "price_formatted": "$3.237",
    "carrier": "URB"
  },
  "San Fernando": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "San Vicente": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4910,
    "price_formatted": "$4.092",
    "carrier": "CCH"
  },
  "Santa Cruz": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4062,
    "price_formatted": "$3.385",
    "carrier": "SKN"
  },
  "Cauquenes": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Chanco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Colbún": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 5172,
    "price_formatted": "$4.310",
    "carrier": "CHX"
  },
  "Constitución": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Curepto": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Curicó": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Empedrado": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 8839,
    "price_formatted": "$7.366",
    "carrier": "BLX"
  },
  "Hualañé": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Licantén": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Linares": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4438,
    "price_formatted": "$3.699",
    "carrier": "CCH"
  },
  "Longaví": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Maule": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Molina": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Parral": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Pelarco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Pelluhue": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Pencahue": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Rauco": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Retiro": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Río Claro": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Romeral": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Sagrada Familia": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "San Clemente": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "San Javier": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "San Rafael": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Talca": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Teno": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4089,
    "price_formatted": "$3.408",
    "carrier": "URB"
  },
  "Vichuquén": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5326,
    "price_formatted": "$4.439",
    "carrier": "CCH"
  },
  "Villa Alegre": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Yerbas Buenas": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Bulnes": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Chillán": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Chillán Viejo": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Cobquecura": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Coelemu": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Coihueco": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "El Carmen": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Ninhue": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Ñiquén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Pemuco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Pinto": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Portezuelo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Quillón": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Quirihue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Ránquil": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "San Carlos": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "San Fabián": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "San Ignacio": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "San Nicolás": {
    "name": "normal",
    "code": "normal",
    "transit_days": 8,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Treguaco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Yungay": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Alto Biobío": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Antuco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Arauco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Cabrero": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4390,
    "price_formatted": "$3.659",
    "carrier": "SKN"
  },
  "Cañete": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Chiguayante": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Concepción": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4525,
    "price_formatted": "$3.771",
    "carrier": "CCH"
  },
  "Contulmo": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Coronel": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Curanilahue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Florida": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Hualpén": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Hualqui": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Laja": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Lebu": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Los Álamos": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Los Ángeles": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4525,
    "price_formatted": "$3.771",
    "carrier": "CCH"
  },
  "Lota": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Mulchén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Nacimiento": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Negrete": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Penco": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Quilaco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Quilleco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "San Pedro De La Paz": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "San Rosendo": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Santa Bárbara": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Santa Juana": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5430,
    "price_formatted": "$4.525",
    "carrier": "CCH"
  },
  "Talcahuano": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4525,
    "price_formatted": "$3.771",
    "carrier": "CCH"
  },
  "Tirúa": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Tomé": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4563,
    "price_formatted": "$3.803",
    "carrier": "URB"
  },
  "Tucapel": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Yumbel": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Angol": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Carahue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Cholchol": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4720,
    "price_formatted": "$3.934",
    "carrier": "URB"
  },
  "Collipulli": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 4574,
    "price_formatted": "$3.812",
    "carrier": "SKN"
  },
  "Cunco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Curacautín": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Curarrehue": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Ercilla": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Freire": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4720,
    "price_formatted": "$3.934",
    "carrier": "URB"
  },
  "Galvarino": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Gorbea": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Lautaro": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Loncoche": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Lonquimay": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Los Sauces": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Lumaco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Melipeuco": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Nueva Imperial": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Padre Las Casas": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4720,
    "price_formatted": "$3.934",
    "carrier": "URB"
  },
  "Perquenco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 7,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Pitrufquén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Pucón": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Purén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 5490,
    "price_formatted": "$4.575",
    "carrier": "SKN"
  },
  "Renaico": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5304,
    "price_formatted": "$4.420",
    "carrier": "BLX"
  },
  "Saavedra": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Temuco": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 4720,
    "price_formatted": "$3.934",
    "carrier": "URB"
  },
  "Teodoro Schmidt": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Toltén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Traiguén": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Victoria": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Vilcún": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Villarrica": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Corral": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Futrono": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5580,
    "price_formatted": "$4.650",
    "carrier": "CHX"
  },
  "La Unión": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Lago Ranco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5659,
    "price_formatted": "$4.716",
    "carrier": "SKN"
  },
  "Lanco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 5049,
    "price_formatted": "$4.208",
    "carrier": "SKN"
  },
  "Los Lagos": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Máfil": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Mariquina": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Paillaco": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Panguipulli": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5580,
    "price_formatted": "$4.650",
    "carrier": "CHX"
  },
  "Río Bueno": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Valdivia": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4860,
    "price_formatted": "$4.050",
    "carrier": "BLX"
  },
  "Ancud": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Calbuco": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Castro": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4988,
    "price_formatted": "$4.157",
    "carrier": "CCH"
  },
  "Chaitén": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Chonchi": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Cochamó": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Curaco De Vélez": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Dalcahue": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Fresia": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5835,
    "price_formatted": "$4.863",
    "carrier": "BLX"
  },
  "Frutillar": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Futaleufú": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Hualaihué": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 5042,
    "price_formatted": "$4.202",
    "carrier": "CHX"
  },
  "Llanquihue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5835,
    "price_formatted": "$4.863",
    "carrier": "BLX"
  },
  "Los Muermos": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 5463,
    "price_formatted": "$4.553",
    "carrier": "URB"
  },
  "Maullín": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Osorno": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 1,
    "price": 4860,
    "price_formatted": "$4.050",
    "carrier": "BLX"
  },
  "Palena": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Puerto Montt": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4988,
    "price_formatted": "$4.157",
    "carrier": "CCH"
  },
  "Puerto Octay": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Puerto Varas": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 4988,
    "price_formatted": "$4.157",
    "carrier": "CCH"
  },
  "Puqueldón": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Purranque": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Puyehue": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "Queilén": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 5985,
    "price_formatted": "$4.988",
    "carrier": "CCH"
  },
  "Quellón": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Quemchi": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Quinchao": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 5793,
    "price_formatted": "$4.828",
    "carrier": "CHX"
  },
  "Río Negro": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "San Juan De La Costa": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 5557,
    "price_formatted": "$4.631",
    "carrier": "BLX"
  },
  "San Pablo": {
    "name": "normal",
    "code": "normal",
    "transit_days": 6,
    "price": 5194,
    "price_formatted": "$4.329",
    "carrier": "URB"
  },
  "Aysén": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 8818,
    "price_formatted": "$7.349",
    "carrier": "CCH"
  },
  "Chile Chico": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 9321,
    "price_formatted": "$7.768",
    "carrier": "CHX"
  },
  "Cisnes": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 8761,
    "price_formatted": "$7.301",
    "carrier": "CHX"
  },
  "Cochrane": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 9414,
    "price_formatted": "$7.845",
    "carrier": "CHX"
  },
  "Coyhaique": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 10,
    "price": 6537,
    "price_formatted": "$5.448",
    "carrier": "SKN"
  },
  "Guaitecas": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Lago Verde": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "O'Higgins": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Río Ibáñez": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Tortel": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Antártica": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Cabo De Hornos": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Laguna Blanca": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 17028,
    "price_formatted": "$14.190",
    "carrier": "BLX"
  },
  "Natales": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 15,
    "price": 8745,
    "price_formatted": "$7.288",
    "carrier": "SKN"
  },
  "Porvenir": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 3,
    "price": 8848,
    "price_formatted": "$7.374",
    "carrier": "CHX"
  },
  "Primavera": {
    "name": "DEX (terrestre)",
    "code": "normal",
    "transit_days": 4,
    "price": 10582,
    "price_formatted": "$8.819",
    "carrier": "CCH"
  },
  "Punta Arenas": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 12,
    "price": 6734,
    "price_formatted": "$5.612",
    "carrier": "SKN"
  },
  "Río Verde": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 17028,
    "price_formatted": "$14.190",
    "carrier": "BLX"
  },
  "San Gregorio": {
    "name": "normal",
    "code": "normal",
    "transit_days": 10,
    "price": 9710,
    "price_formatted": "$8.092",
    "carrier": "URB"
  },
  "Timaukel": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 19,
    "price": 86020,
    "price_formatted": "$71.684",
    "carrier": "SKN"
  },
  "Torres Del Paine": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 4,
    "price": 17028,
    "price_formatted": "$14.190",
    "carrier": "BLX"
  },
  "Alhué": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 3622,
    "price_formatted": "$3.019",
    "carrier": "BLX"
  },
  "Buin": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Calera De Tango": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Cerrillos": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Cerro Navia": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Colina": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Conchalí": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Curacaví": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3304,
    "price_formatted": "$2.754",
    "carrier": "BLX"
  },
  "El Bosque": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "El Monte": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Estación Central": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Huechuraba": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Independencia": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Isla De Maipo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "La Cisterna": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "La Florida": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "La Granja": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "La Pintana": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "La Reina": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Lampa": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Las Condes": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Lo Barnechea": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Lo Espejo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Lo Prado": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Macul": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Maipú": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "María Pinto": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3304,
    "price_formatted": "$2.754",
    "carrier": "BLX"
  },
  "Melipilla": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3304,
    "price_formatted": "$2.754",
    "carrier": "BLX"
  },
  "Ñuñoa": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Padre Hurtado": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Paine": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3304,
    "price_formatted": "$2.754",
    "carrier": "BLX"
  },
  "Pedro Aguirre Cerda": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Peñaflor": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Peñalolén": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Pirque": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Providencia": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Pudahuel": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Puente Alto": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Quilicura": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Quinta Normal": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Recoleta": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Renca": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "San Bernardo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "San Joaquín": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "San José De Maipo": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "San Miguel": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "San Pedro": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 3,
    "price": 3622,
    "price_formatted": "$3.019",
    "carrier": "BLX"
  },
  "San Ramón": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Santiago": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Talagante": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  },
  "Til Til": {
    "name": "Normal",
    "code": "normal",
    "transit_days": 2,
    "price": 3304,
    "price_formatted": "$2.754",
    "carrier": "BLX"
  },
  "Vitacura": {
    "name": "Día hábil siguiente",
    "code": "nextday",
    "transit_days": 2,
    "price": 3500,
    "carrier": "TPC"
  }
}
```

</details>

## Crear envío

Crea un envío en la tienda asociada.

#### Endpoint

`POST /api/shops/:shop_id/orders/`

#### Parámetros

Recibe la información del envío en el _request body_ (formato JSON). En el recurso [order](#order) se detallan los
atributos que se pueden entregar al momento de crear el envío.

<details>
<summary>Ejemplo solo con atributos obligatorios</summary>

```json
{
  "reference_code": "2310TIENDA",
  "address_region": "RM",
  "address_city": "Huechuraba",
  "address_street": "Av. Recoleta 999"
}
```

</details>

<details>
<summary>Ejemplo con atributos obligatorios y algunos opcionales</summary>

```json
{
  "reference_code": "2310TIENDA",
  "address_region": "RM",
  "address_city": "Huechuraba",
  "address_street": "Av. Recoleta 999",
  "customer_name": "Juan Nieves",
  "customer_email": "juan@thepackco.cl",
  "custom_phone": "594444333",
  "product_description": "Zapatos de cuero negro T42"
}
```

</details>

**IMPORTANTE**: cuando obtenemos la información de un envío, el atributo `address_city` contiene el código de la comuna.
Cuando creamos un envío, el atributo `address_city` debe contener el nombre de la comuna.

#### Status codes

- `201`: se crea el envío y devuelve el envío creado.
- `400`: no se crea el envío y devuelve los errores asociados.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para realizar esta acción.
- `404`: no existe una tienda con el `:shop_id` entregado, o es una tienda a la que el usuario no tiene acceso.

#### Respuestas

Si la información proporcionada es correcta, retorna código de éxito (`201`) y devuelve el [envío](#order) creado:

<details>
<summary>Ejemplo respuesta exitosa</summary>

```json
{
  "id": 159,
  "events": [
    {
      "datetime": "2020-12-16T16:48:51.520265Z",
      "name": "created",
      "additional_data": {},
      "description": "Envío creado"
    }
  ],
  "address_city": "306",
  "courier": "tpc",
  "address_region_name": "Metropolitana de Santiago",
  "address_city_name": "Huechuraba",
  "courier_name": "ThePackCo",
  "status_name": "Creado",
  "public_id": "UNC5ZLIBE7",
  "status": "created",
  "reference_code": "2310TIENDA",
  "product_description": "Zapatos de cuero negro T42",
  "additional_data": "",
  "address_region": "RM",
  "address_street": "Av. Recoleta 999",
  "address_flat": "",
  "address_additional": "",
  "customer_email": "juan@thepackco.cl",
  "customer_name": "Juan Nieves",
  "customer_phone": "",
  "created_at": "2020-12-16T16:48:51.520265Z",
  "updated_at": "2020-12-16T16:48:51.540774Z",
  "shop": 12
}
```

</details>

Si la información proporcionada tiene errores, devuelve código de error (`400`) y un objeto de errores. En cada entrada
del objeto de errores, la llave corresponde al nombre del campo que tiene el error y el valor corresponde a una lista de
los errores asociados a ese campo.

<details>
<summary>Ejemplo en que faltan campos requeridos</summary>

```json
{
  "address_city": [
    "This field is required."
  ],
  "reference_code": [
    "This field is required."
  ]
}
```

</details>

<details>
<summary>Ejemplo en que no se entrega una comuna válida</summary>

```json
{
  "address_city": [
    "\"Chicureo\" is not a valid city"
  ]
}
```

</details>

<details>
<summary>Ejemplo en que no se entrega una región válida (deben usarse los códigos de región)</summary>

```json
{
  "address_region": [
    "\"Metropolitana\" is not a valid choice."
  ]
}
```

</details>

<details>
<summary>Ejemplo en que la comuna no está dentro de la región</summary>

```json
{
  "non_field_errors": [
    "Viña Del Mar do not belong to Metropolitana de Santiago region"
  ]
}
```

</details>

## Obtener envío

Devuelve la información de un envío asociado a una tienda.

#### Endpoint

`GET /api/shops/:shop_id/orders/:order_id/`

#### Parámetros

No recibe.

#### Status codes

- `200`: retorna el envío.
- `401`: el token de autenticación no es válido.
- `403`: el token de autenticación no tiene permisos para consultar este recurso.
- `404`: no existe una tienda con el `:shop_id` entregado, o es una tienda a la que el usuario no tiene acceso, o no
  existe el envío con el `:order_id` entregado.

#### Respuestas

En el caso de una respuesta positiva (`200`) retorna el envío. Ver recurso [Order](#order) para conocer todos los
atributos.

<details>
<summary>Ejemplo respuesta exitosa</summary>

```json
{
  "id": 119,
  "events": [
    {
      "datetime": "2020-11-13T03:44:51.220196Z",
      "name": "received",
      "additional_data": {
        "triggered_by": "leonardo"
      },
      "description": "Envío retirado por The Pack Co"
    },
    {
      "datetime": "2020-10-29T22:02:14.938584Z",
      "name": "created",
      "additional_data": {},
      "description": "Envío creado"
    }
  ],
  "address_city": "330",
  "courier": "tpc",
  "address_region_name": "Metropolitana de Santiago",
  "address_city_name": "Providencia",
  "courier_name": "ThePackCo",
  "status_name": "Retirado",
  "public_id": "YY4NYG9C51",
  "status": "received",
  "reference_code": "109231203",
  "product_description": "2x zapatos sport t41",
  "additional_data": "",
  "address_region": "RM",
  "address_street": "Los Leones 9000",
  "address_flat": "Depto 101",
  "address_additional": "",
  "customer_email": "diego@nieves.cl",
  "customer_name": "Diego Nieves",
  "customer_phone": "+56971234567",
  "created_at": "2020-10-29T22:02:14.938584Z",
  "updated_at": "2020-11-13T03:44:51.271263Z",
  "shop": 12
}
```

</details>

____

## Shop

Objeto que representa una tienda a la cual están asociados los envíos.

Las tiendas no pueden crearse ni modificarse desde la API.

| Atributo | Tipo      |Ejemplo| Descripción                                                                                       |
|----------|-----------|-------|---------------------------------------------------------------------------------------------------|
| `id`     | `integer` |12| ID asociado a la tienda. Es necesario para poder crear envíos                                     |
| `name`   | `string`  |Zapatería Chilena| Nombre de la tienda que se muestra en la plataforma de clientes                                   |
| `settings` | `json`  |{...}| Json que contiene la información de la configuración de la tienda en The Pack Co. Por ejemplo, Si un usuario desea recibir notificaciones cuando una orden allá cambiado de status, dicha información aparecerá en este campo.   |
| `webhook_token` | `string`  |1T15TTGV4T2vDNd6VMXZ548EZfIexrZ8| Token utilizado para autenticar a la tienda que desea recibir los cambios en los status de sus ordenes.   |
| `mail_preferences_data` | `json`  |{...}| Configuración de mensajería del usuario.   |

####  Webhooks

Los webhooks no son más que urls que permiten que los servicios externos sean notificados cuando ocurren ciertos
eventos.

Por ejemplo, el dueño de la tienda podrá configurar un webhook para ser notificado cuando ocurran cambios en el status
de sus órdenes.

Para que las notificaciones mediante webhook surtan efecto, es importante que la tienda remota tenga configurado en su
sistema aquellas settings que requieran de webhook.

<summary>Ejemplo de los settings de la tienda al tener webhook configurado</summary>

```json
{
  "webhooks": {
    "order_updated": {
      "enabled": true,
      "endpoint": "https://platform.thepackco.cl/webhooks/order_update/shopify/"
    }
  }
}
```

En el ejemplo anterior vemos como el dueño de la tienda tiene activado (enabled: True) el webhook de actualización de
órdenes (order_updated), para que The Pack Co use esté endpoint para notificarle cuando una orden haya sido actualizada.

Por otro lado, el usuario podrá configurar sus propios urls para sus propios sistemas externos de notificación, o bien
The Pack Co podrá proporcionar su propio endpoint para enviar los cambios a su tienda remota. En el ejemplo anterior la
tienda remota es shopify.



## Order

Objeto que representa un envío.

Los envíos pueden crearse, pero no es posible modificarlos.

Al crear un envío, deben entregarse obligatoriamente los atributos marcados con el símbolo :exclamation:, y
opcionalmente los con el símbolo :question:. Por otra parte, los atributos que tienen el símbolo :book: son atributos de
solo lectura y por lo tanto solo pueden ser leídos al obtener la información de un envío.

|Atributo|Tipo|Ejemplo|Descripción|
|--------|----|-------|-----------|
|`id`|:book: - `integer`|133|ID asociado al envío|
|`shop`|:book: - `integer`|12|ID asociado a la tienda|
|`public_id`|:book: - `string`|ND7AN34H9K|ID público de seguimiento del envío compuesto de 10 caracteres (letras mayúsculas y números)|
|`status`|:book: - `string`|created|Código del estado del envío. [Detalles](#estados)|
|`status_name`|:book: - `string`|Creado|Nombre descriptivo del estado del envío|
|`courier`|:book: - `string`|tpc|Código del courier a cargo del envío|
|`courier_name`|:book: - `string`|ThePackCo|Nombre descriptivo de courier a cargo del envío|
|`reference_code`|:exclamation: - `string`|2310TIENDA|Código de referencia del envío|
|`address_region`|:exclamation: - `string`|VS|Código de la región de destino. [Detalles](#regiones-y-comunas)|
|`address_region_name`|:book: - `string`|Valparaíso|Nombre descriptivo de la región de destino. [Detalles](#regiones-y-comunas)|
|`address_city`|:exclamation: - `string`|81|Código interno de la comuna de destino. [Detalles](#regiones-y-comunas)|
|`address_city_name`|:book: - `string`|Viña Del Mar|Nombre descriptivo de la comuna de destino. [Detalles](#regiones-y-comunas)|
|`address_street`|:exclamation: - `string`|Compañía de Jesús 1131|Dirección de destino, con calle y número|
|`address_flat`|:question: - `string`|Depto 301A|Número de departamento o casa (si fuera dentro de un condominio)|
|`address_additional`|:question: - `string`|Portón de madera frente a camino de tierra|Referencias adicionales para la dirección de destino en caso de que fueran necesarias|
|`customer_email`|:question: - `string`|juan@thepackco.cl|Correo del cliente que va a recibir el producto. Este correo se utiliza para envíar correos con información de seguimiento|
|`customer_name`|:question: - `string`|Juan Nieves|Nombre del cliente que va a recibir el producto|
|`customer_phone`|:question: - `string`|56944443333|Teléfono del cliente que va a recibir el producto. Se utiliza para contactar al cliente si hay una dificultar para entregar el paquete|
|`product_description`|:question: - `string`|Botines negros talla XS|Descripción del producto|
|`packages`|:question: - `integer`|4|Cantidad de bultos del producto|
|`additional_data`|:question: - `string`|Compra realizada a través de instagram usuario juan.nieves|Información adicional al envío que pudiera querer guardar la tienda|
|`events`|:book: - `OrderEvent`|[{...}, {...}, ...]|Lista de eventos asociados al envío|
|`created_at`|:book: - `string`|2020-11-27T22:42:32.586057Z|Hora de creación del envío, en formato ISO8601 (`Z` = GMT)|
|`updated_at`|:book: - `string`|2020-11-27T22:42:32.586057Z|Hora de última actualización del envío, en formato ISO8601 (`Z` = GMT)|

#### Estados

|Código estado|Descripción|
|-------------|-----------|
|`created`|El envío fue creado|
|`received`|El envío fue retirado por ThePackCo desde las bodegas de la tienda (no aplica en tiendas que usan las bodegas de ThePackCo)|
|`processed`|El envío ingresó al sistema de la empresa de courier que se encargará del despacho (puede ser ThePackCo mismo u otro courier en el caso de envíos a regiones)|
|`delivered`|El envío fue entregado al cliente final|
|`delegated`|El envío fue entregado a otro courier (distinto de ThePackCo) y ellos se están haciendo cargo del envío|
|`deleted`|El envío fue eliminado por la tienda|
|`canceled`|El envío fue cancelado por ThePackCo|

#### Regiones y comunas

En ThePackCo usamos el código [ISO3166-2](https://es.wikipedia.org/wiki/ISO_3166-2:CL) sin el prefijo `CL-` para
referirnos a cada region:

|Código región|Nombre región|
|-------------|-------------|
|AP|Arica y Parinacota|
|TA|Tarapacá|
|AN|Antofagasta|
|AT|Atacama|
|CO|Coquimbo|
|VS|Valparaíso|
|RM|Metropolitana de Santiago|
|LI|Libertador Gral. Bernardo O’Higgins|
|ML|Maule|
|NB|Ñuble|
|BI|Biobío|
|AR|Araucanía|
|LR|Los Ríos|
|LL|Los Lagos|
|AI|Aysén del Gral. Carlos Ibáñez del Campo|
|MA|Magallanes y de la Antártica Chilena|

Cada región se divide en comunas, dando a un total de 346 en todo el país.

En ThePackCo forzamos que los envíos tengan una región y comuna válidos. Al crear un envío, se debe entregar el código
de la región y el nombre de la comuna (los códigos de comuna solo se utilizan de manera interna). Es necesario verificar
que la comuna pertenezca a la región indicada, y además, tanto el código de la región como el nombre de la comuna deben
estar escritos de la manera correcta.

Para conocer los nombres de las comunas y la región a la que pertencen, puede
consultar [este archivo](regions_and_cities.json) (formato JSON).

## OrderEvent

Objeto que representa un evento asociado a un envío.

Los OrderEvent no pueden crearse ni modificarse por medio de la API.

|Atributo|Tipo|Ejemplo|Descripción|
|--------|----|-------|-----------|
|`name`|`string`|created|Tipo de evento|
|`description`|`string`|Envío creado|Nombre descriptivo del evento|
|`datetime`|`string`|2020-11-27T22:42:32.586057Z|Hora en que ocurrió el evento, en formato ISO8601 (`Z` = GMT)|
|`additional_data`|`JSON`|`{"eta": "14:13"}`|Información adicional del evento. Los atributos presentes en este objeto van a depender del tipo de evento|
