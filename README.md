# Hetzner API DynDNS

Un pequeño script para actualizar registros DNS dinámicamente utilizando Hetzner DNS-API.

**Hetzner DNS API Doc:**

https://dns.hetzner.com/api-docs/

# Preparations

## Instalar las dependencias

- [`curl`](https://curl.se/)
- [`jq`](https://stedolan.github.io/jq/): [install](https://stedolan.github.io/jq/download/)
- Ubuntu 20.x >

```
apt install curl
```

```
apt install jq
```

## Generar token de acceso

Primero, se debe crear un nuevo token de acceso en la [Consola DNS de Hetzner](https://dns.hetzner.com/). Esto debe copiarse inmediatamente, ya que por motivos de seguridad no será posible mostrar el token más adelante. Pero puedes generar tantos tokens como quieras.

## Descarga del script

- `cd /opt`
- Clona el repositorio `git clone https://pypi.org/project/hetzner-dns-tools/`
- cd `hetzner-dns-tools`
- Establece como ejecutable el script `chmod -x ./dyndns.sh`
- Edita las variables dentro del archivo `./dyndns.sh`, las instrucciones vienen a continuación.

> Sólo usaremos el archivo `dyndns.sh` el cual también puedes copiar y pegar en la maquina dónde vas a instalar e escript.

# Uso

Escribe el token de acceso en el script o configuralo como una variable de entorno del sistema operativo. Para almacenarlo en el script, reemplza `<your-hetzner-dns-api-token>` en la siguiente línea del script.

```
...
auth_api_token=${HETZNER_AUTH_API_TOKEN:-'<your-hetzner-dns-api-token>'}
...
```

Una vez que se haya asignado el token, se puede llamar al script con los parámetros apropiados. Esto permite crear varios registros DynDNS en diferentes zonas. Opcionalmente, se puede especificar el TTL y el tipo de registro. Es aconsejable mantener el TTL lo más bajo posible, para que los registros modificados se utilicen lo antes posible.


```
./dyndns.sh [ -z <Zone ID> | -Z <zone_name> ] [-r <Record ID>] -n <Record Name> [-t <TTL>] [-T <Record Type>]
```

Para mantener actualizados el registro DynDNS, se debe crear un **cronjob** que llame al script de forma periódica.

## Ejemplos:

Tienes varias posibilidades para llamar el script. En estos ejemplos, se lanza el escript cada 5 minutos y actualiza la entrada DNS si es necesario.

**Ejemplo 1**

En el primer ejemplo, solo se pasa el token de API como variable de entorno y la información restante como parámetros. Esto permite, por ejemplo, configurar varias entradas DynDNS en diferentes zonas cuando el script se llama varias veces con diferentes parámetros.

```
HETZNER_AUTH_API_TOKEN='<your-hetzner-dns-api-token>'

*/5 * * * * /opt/hetzner-dns-tools/dyndns.sh -Z example.com -n dyn
```

**Ejemplo 2**

También puedes pasar toda la información como variables de entorno para crear una entrada DynDNS.

```
HETZNER_AUTH_API_TOKEN='<your-hetzner-dns-api-token>'
HETZNER_ZONE_NAME='example.com'
HETZNER_RECORD_NAME='dyn'

*/5 * * * * /opt/hetzner-dns-tools/dyndns.sh
```

**Ejemplo 3**

Si todo está apuntado dentro del archivo `/opt/hetzner-dns-tools/dyndns.sh`

```
*/5 * * * * /opt/hetzner-dns-tools/dyndns.sh
```


# Variables de entorno del sistema operativo

Puedes utilizar las siguientes variables de entorno.

|NAME                   | Value                            | Description                                                     |
|:----------------------|----------------------------------|:----------------------------------------------------------------|
|HETZNER_AUTH_API_TOKEN | 925bf046408b55c313740eef2bc18b1e | Your Hetzner API access token                                   |
|HETZNER_ZONE_NAME      | example.com                      | The zone name                                                   |
|HETZNER_ZONE_ID        | DaGaoE6YzDTQHKxrtzfkTx           | The zone ID. Use either the zone name or the zone ID. Not both. |
|HETZNER_RECORD_NAME    | dyn                              | The record name. '@' to set the record for the zone itself.     |
|HETZNER_RECORD_TTL     | 120                              | The TTL of the record. Default(60)                              |
|HETZNER_RECORD_TYPE    | AAAA                             | The record type. Either A for IPv4 or AAAA for IPv6. Default(A) |

# Ayuda

Establece el argumento `-h` para mostrar la ayuda.

```
./dyndns.sh -h

exec: ./dyndns.sh -Z <Zone Name> -n <Record Name>

parameters:
  -z  - Zone ID
  -Z  - Zone Name
  -r  - Record ID
  -n  - Record name

optional parameters:
  -t  - TTL (Default: 60)
  -T  - Record type (Default: A)

help:
  -h  - Show Help 

example:
  .exec: ./dyndns.sh -Z example.com -n dyn -T AAAA
  .exec: ./dyndns.sh -z 98jFjsd8dh1GHasdf7a8hJG7 -r AHD82h347fGAF1 -n dyn

``` 
# Cosas adicionales

## Obtener todas las zonas

Si necesitas obtener todas las zonas y verificar la ID de la zona deseada ejecuta.

```
curl "https://dns.hetzner.com/api/v1/zones" -H \
'Auth-API-Token: ${apitoken}' | jq
```
## Get a record ID

If you want to get a record ID manually you may use the following curl command.

```
curl -s --location \
    --request GET 'https://dns.hetzner.com/api/v1/records?zone_id='${zone_id} \
    --header 'Auth-API-Token: '${apitoken} | \
    jq --raw-output '.records[] | select(.type == "'${record_type}'") | select(.name == "'{record_name}'") | .id'
```
## Agregar registro manualmente

```
curl -X "POST" "https://dns.hetzner.com/api/v1/records" \
     -H 'Content-Type: application/json' \
     -H 'Auth-API-Token: ${apitoken}' \
     -d $'{
  "value": "${yourpublicip}",
  "ttl": 60,
  "type": "A",
  "name": "dyn",
  "zone_id": "${zoneID}"
}'
```

## Bibliografía

https://pypi.org/project/hetzner-dns-tools/
https://pypi.org/project/hetzner-dns-tools/
https://stackoverflow.com/questions/352098/how-can-i-pretty-print-json-in-a-shell-script
https://github.com/hetzneronline