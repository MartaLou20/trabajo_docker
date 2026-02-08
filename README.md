# Trabajo Docker - To-Do List App

## Estructura del proyecto
Esta es la estructura de proyecto seguida:
    trabajo_docker/
    │
    ├── backend/
    │   ├── server.js
    │   ├── package.json
    │   └── Dockerfile
    │
    ├── frontend/
    │   ├── index.html
    │   ├── app.js
    │   ├── styles.css
    │   └── Dockerfile
    │
    ├── database/
    │   └── init.sql
    │
    ├── docker-compose.yml
    ├── .env
    ├── .env.example
    ├── .gitignore
    └── README.md


## Explicación de cada servicio (qué hace, puertos, variables de entorno)
### Base de datos
Utiliza la imagen oficial de postgres:15, almacena los datos en un volumen Docker para su persistencia. Facilita el almacenamiento y la consulta de las tareas. Se inicializa mediante un script de SQL.
Usa el puerto 5432

### Backend
Utiliza un contenedor node.js + Express. Expone una API REST para gestionar tareas(listar, crear, actualizar y eliminar). Se conecta a la base de datos.
Usa el puerto 4000

### Frontend
Utiliza un contenedor Nginx. Se trata de una aplicación web estática que consume la API REST del backend.
Usa el puerto 8080

### Variables de entorno
En el archivo .env no visible en git(una muestra es el .env.example) se almacenan las siguientes variables que se utilizan para configurar la conexión del backend con la base de datos:

POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=
DB_HOST=
DB_PORT=


## Cómo funcionan los volúmenes (gestionado vs bind mount)
### Base de datos
Para la base de datos se utiliza un volumen gestionado por Docker, declarado en el archivo docker-compose.yml:
```yaml
volumes:
  postgres-data:
```
Y montando en el servicio de base de datos:
```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```
Docker gestiona automáticamente la ubicación física del volumeny los datos persisten aunque el contenedor se elimine.

### Frontend
Para el frontend se utiliza un bind mount, que enlaza directamente una carpeta del sistema anfitrión con el contenedor:
```yaml
volumes:
  - ./frontend:/usr/share/nginx/html:ro
```
El contenido se sincroniza en tiempo real entre host y contenedor, es decir, cualquier cambio se refleja inmediatamente.El sufijo :ro indica que el montaje es solo lectura dentro del contenedor.


## Cómo funciona la red personalizada
El proyecto utiliza una red Docker personalizada para permitir la comunicación interna entre los contenedores del backend y la base de datos.
Se define la red
```yaml
networks:
  todo-network:
    driver: bridge
```
Y se asigna en los servicios que necesitan comunicarse:
```yaml
services:
  backend:
    networks:
      - todo-network

  database:
    networks:
      - todo-network
```


## Tabla de puertos expuestos
|  Servicio  | Puerto Host |  Acceso   |
|------------|-------------|-----------|
|  Database  |             |  Interno  |
|  Backend   |     4000    |  API REST |
|  Frontend  |     8080    | Navegador |


## Instrucciones de configuración del .env
El archivo .env contiene las variables de entorno necesarias para configurar el backend y la base de datos. Debe encontrarse en la raíz del proyecto al mismo nivel que el docker-compose.yml.

Como ejemplo puede tomar el .env.example que se ve asi:
```env
POSTGRES_USER=tu_usuario
POSTGRES_PASSWORD=tu_contraseña_segura
POSTGRES_DB=tu_nombre_base_datos

DB_HOST=tu_host
DB_PORT=5432#estandar

BACKEND_PORT=tu_puerto
```
**Importante** 
No deben existir espacios alrededor del signo =.

### Explicación de variables:
POSTGRES_USER ---> Usuario de la base de datos
POSTGRES_PASSWORD ---> Contraseña del usuario
POSTGRES_DB ---> Nombre de la base de datos
DB_HOST ---> Host de la base de datos (nombre del servicio Docker)
DB_PORT ---> Puerto interno de PostgreSQL