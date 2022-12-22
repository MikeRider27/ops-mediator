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

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/c52f9273-74fe-4032-acd6-cd2c025bf0e2.png?token=GHSAT0AAAAAAB4GIVW33UUXDNQ3D5YDQ4PEY5EWMWA)

### 2.4.1 Otras variantes sobre el servicio

Para detener y remover el servicio utilice:

```bash
docker-compose down
```

Iniciar los servicios de docker-compose sin crearlos:

```bash
docker-compose start
```

Para detener uno o todos los servicios:

```bash
docker-compose stop
```

## 2.5 Verificar funcionamiento correcto del servidor

Para verificar que el servidor esta en funcionamiento puede hacerlo desde la consola mediante el siguiente request:

```bash
curl -X GET http://localhost:8080/fhir/Patient
```

y obtener la siguiente respuesta:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/e7d2c681-7657-4326-9045-63d873faece5.png?token=GHSAT0AAAAAAB4GIVW3CHKHQFJBJFQY7LLMY5EWSRA)

Otra manera de hacerlo es escribir la direccion localhost:8080 en el navegador. Debería visualizar la siguiente imagen:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/0c894dfc-2007-4261-8d24-10b46b3db267.png?token=GHSAT0AAAAAAB4GIVW3OVAQA5TRNDNEAEMMY5EW37A)

---

## 3. Servidor HAPI FHIR mediante Maven

Esta instalación permite realizar otro tipo de ajustes This mediator is configured using environment variables.

## 3.1 Pre-requisitos

* Oracle Java JDK 17 o mayor
* Apache Maven última versión (>= 3.6) [Maven – Installing Apache Maven](https://maven.apache.org/install.html)
* Procesador. Al menos dos nucleos de procesador.
* Al menos 4 GB de Memoría RAM

## 3.2 Descarga del proyecto

1. Ir a la página principal del proyecto github hapi-fhir-jpaserver-starter [GitHub - hapifhir/hapi-fhir-jpaserver-starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter)
y entrar en releases

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/b3b2dcb3-b381-43ad-bd69-1eec83c0216a.png?token=GHSAT0AAAAAAB4GIVW3QSGAJA3PJUKRQQVOY5EXDLA)

2. Busque en la lista el release correspondiente a image/v6.1.0

3. Descargar “Source code”, seleccionar descarga en formato .zip o .tar.gz

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/9a957001-285c-4ea8-92ac-8eb1c398092c.png?token=GHSAT0AAAAAAB4GIVW3YFQXC5SFVR5ZV7ESY5EXFOQ)

4. Descomprimir proyecto y acceder a la carpeta por consola

```bash
wget https://github.com/hapifhir/hapi-fhir-jpaserver-starter/archive/refs/tags/image/v6.1.0.zip
unzip v6.1.0.zip
cd v6.1.0
```

## 3.3 Compilar el proyecto

En la consola ejecutar una de las siguientes instrucciones:

```bash
mvn clean install -DskipTests
```

La compilación exitosa del proyecto debiera arrojar un resultado como el que se muestra a continuación:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/5e484ba4-922b-4617-be31-200756690d56.png?token=GHSAT0AAAAAAB4GIVW2D6MQB544E5XPRQNKY5EXQLA)

## 3.4 Ejecutar el servidor

```bash
mvn jetty:run 
   "o" 
mvn -Djetty.port=8888 jetty:run
```
Al terminar la compilación debería ver los mensajes que se ven en la siguiente imagen:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/a7b47d89-d22b-4f5e-b451-88cf6f21e456.png?token=GHSAT0AAAAAAB4GIVW26CNZYTHQXAVNABXEY5EXUCQ)

## 3.5 Verificar funcionamiento correcto del servidor

Para verificar que el servidor esta en funcionamiento puede hacerlo desde la consola mediante el siguiente request:

```bash
curl -X GET http://localhost:8080/fhir/Patient
```

y obtener la siguiente respuesta:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/e7d2c681-7657-4326-9045-63d873faece5.png?token=GHSAT0AAAAAAB4GIVW3CHKHQFJBJFQY7LLMY5EWSRA)

Otra manera de hacerlo es escribir la direccion localhost:8080 en el navegador. Debería visualizar la siguiente imagen:

![Aquí la descripción de la imagen por si no carga](https://raw.githubusercontent.com/MikeRider27/ops-mediator/master/ddcc-transactions-mediator/assets/0c894dfc-2007-4261-8d24-10b46b3db267.png?token=GHSAT0AAAAAAB4GIVW3OVAQA5TRNDNEAEMMY5EW37A)
