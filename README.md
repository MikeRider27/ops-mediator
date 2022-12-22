# Instalación Servidor HL7 FHIR OPS DDCC (Repositorio)

## 1. General

La siguiente guía le ayudará en el proceso de instalación y despliegue de un servidor HAPI FHIR. La creación
de esta guia ha sido realizada en un sistema Ubuntu 20.04. El servidor puede ser rapidamente desplegado 
con ayuda de tecnología Docker. Siga las instrucciones del punto 3 “Servidor HAPI FHIR mediante Docker-Compose”
de esta guía para tal propósito. Si se requieren configuraciones más avanzadas se puede compilar el proyecto 
mediante Maven. Para ello consulte el punto 4 de esta guía “Servidor HAPI FHIR mediante Maven”. Para mayor 
información de configuraciones adicionales consulte la documentación oficial del proyecto ([HAPI FHIR - The Open Source FHIR API for Java](https://hapifhir.io/))

---
## 2. Servidor HAPI FHIR mediante Docker-Compose
## 2.1 Pre-Requisitos para el servidor

Para ejecutar de manera correcta el servidor se requiere contar con las siguientes características:

* Docker 
* Docker-compose
* Procesador. Al menos dos nucleos de procesador.
* Al menos 4 GB de Memoría RAM. 

## 2.2 Instalación de Docker y Docker-compose

Las siguientes recetas le ayudaran en el proceso de instalación de docker y docker-compose respectivamente

### 2.2.1 Instalación de docker

* [Docker Engine installation overview](https://docs.docker.com/engine/install/) 
* [How To Install and Use Docker on Ubuntu 20.04  | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) 

### 2.2.2 Para instalar docker-compose:

* [Overview](https://docs.docker.com/compose/install/) 
* [How To Install and Use Docker Compose on Ubuntu 20.04  | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04) 

## 2.3 Configuración del servidor

1. Desde la consola de comandos crear el siguiente archivo .env para asignar variables de entorno.

```bash
POSTGRES_USER=admin
POSTGRES_PASSWORD=ddcc123
```

2. Mediante un editor de texto crear el archivo docker-compose.yml: y pegue el contenido a continuación. Guarde los cambios y salga del editor.


```bash
version: '3.3'
networks:
  ddcc-net:
    name: ddcc-net

volumes:
  hapi-postgres:

services:
  db:
    container_name: hapi-postgres
    image: postgres:11-alpine
    networks:
      - ddcc-net
    environment:
      POSTGRES_DB: 'hapi'
      POSTGRES_USER: ${POSTGRES_USER:-admin}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-admin}
    volumes:
      - 'hapi-postgres:/var/lib/postgresql'


  fhir:
    container_name: hapi-fhir
    image: hapiproject/hapi:v6.1.0
    networks:
      - ddcc-net
    restart: always
    environment:
      - spring.datasource.url=jdbc:postgresql://db:5432/hapi
      - spring.datasource.username=${POSTGRES_USER:-admin}
      - spring.datasource.password=${POSTGRES_PASSWORD:-admin}
      - spring.datasource.driverClassName=org.postgresql.Driver
      - spring.jpa.properties.hibernate.dialect=ca.uhn.fhir.jpa.model.dialect.HapiFhirPostgres94Dialect
      - hapi.fhir.allow_external_references=true
      - hapi.fhir.max_page_size=50
      - hapi.fhir.default_page_size=20
    ports:
      - 8081:8080
    depends_on:
      - db
```

## 2.4 Levantar el servicio mediante docker-compose

Con el servicio docker-compose arranque el servidor con la siguiente línea:

```bash
docker-compose up -d
```
La primera vez que ejecute el servicio docker realizará la descarga de las imagenes como se ve a continuación. Finalmente los contenedores son desplegados.

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/parzibyte/WaterPy/master/assets/ImagenV1.png)

---

## Configuring DDCC Mediator

This mediator is configured using environment variables.

The following variables can be set:

| Environment Variable | Default | Description |
| --- | --- | --- |
| FHIR_SERVER | http://fhir:8080/fhir/ | URL of data repository |
| MATCHBOX_SERVER | http://resource-generation-service:8080/fhir/ | URL of resource transformer |
| MEDIATOR_HOST | ddcc | HOST used in mediador|
| OPENHIM_URL | <https://localhost:8080> | The location of the the OpenHIM API |
| OPENHIM_USERNAME | root@openhim.org | Registered OpenHIM Username |
| OPENHIM_PASSWORD | openhim-password | Password of the registered OpenHIM user |
| TRUST_SELF_SIGNED | `false` | In development environments the OpenHIM uses self-signed certificates therefore it is insecure. To allow the mediator to communicate with the OpenHIM via HTTPS this variable can be set to `true` to ignore the security risk. **This should only be done in a development environment** |

---

## Configuring OpenHIM

To route requests from a client to destination systems, the OpenHIM needs to have `channels` configured to listen for specific requests and send them to specific endpoints.

This mediator is configured (within [mediatorConfig.json](mediatorConfig.json)) to create some default channels and endpoints. To create these channels navigate to the mediators page on the OpenHIM Console.

---

## Instructions for the development team

### Create a private key

* add the private key to the path /cert-data if you have one.
* generate one inside de folder cert-data/

```bash
cd cert-data/
openssl genrsa -out priv.pem 2048
```

### Run DDCC mediator with OpenHIM

#### For use with an external repository

* Review installation guide [Repository install](https://cens.atlassian.net/wiki/spaces/OD/pages/2011365377/Instalaci+n+Servidor+HL7+FHIR+OPS+DDCC+Repositorio)

* create an environment variable call FHIR_SERVER with the url of the repository and a network call ddcc_net

```bash
export FHIR_SERVER=http://fhir:8080/fhir/
docker network create -d bridge ddcc-net
docker build -t openhie/ddcc-transactions-openhim:latest -t openhie/ddcc-transactions-openhim:v1.0.20 -f Dockerfile.openhim .
docker-compose -f docker/docker-compose.openhim-external-repo.yml up -d
```
#### For use without an external repository 

```bash
docker build -t openhie/ddcc-transactions-openhim:latest -t openhie/ddcc-transactions-openhim:v1.0.20 -f Dockerfile.openhim .
docker-compose -f docker/docker-compose.openhim.yml up -d
```


* Change the password in the OpenHIM console http://localhost:9000 use the user:passdord for access (root@openhim.org:openhim-password)
* Use **ddcc.2022** as a new password
* Login again into the console and use the user:password (root@openhim.org:ddcc.2022)
* Create a client in http://localhost:9000/#!/clients (client ID = ddcc y client Name = ddcc)
    * Go to authentication tab and add a basic auth password using `ddcc`
    * Save changes
* Go to Mediators page in the sidebar and install(+ button) the mediator to install the new channel
* Test the new user using the credentials (user:password --> ddcc:ddcc)
    * Example:
```
GET /ddcc/shc_issuer/.well-known/jwks.json HTTP/1.1
Host: localhost:5001
Authorization: Basic ZGRjYzpkZGNj
```

### Update the code changes

* Run this to test changes

* Use `docker_file` **docker/docker-compose.openhim-external-repo.yml** or **docker/docker-compose.openhim.yml**

```bash
docker-compose -f `docker_file` stop ddcc
docker-compose -f `docker_file` rm ddcc -y
docker build -t openhie/ddcc-transactions-openhim:latest -t openhie/ddcc-transactions-openhim:v1.0.20 -f Dockerfile.openhim .
docker-compose -f `docker_file` up -d ddcc
docker-compose -f `docker_file` logs  --follow ddcc

```

* if you want to inspect the database of hapifhir(server: hapi-postgres)

```
docker run -itd --network=ddcc-net -p 9001:8080 adminer
```
* Go to localhost:9001 and access to the database with credentials.
