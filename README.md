# Tienda de Alimentos para Perritos

Aplicación Web CRUD Fullstack desplegada en AWS usando Docker y pipeline CI/CD automatizado con Github Actions y Amazon ECR.

## Lenguajes y tecnologías
Frontend: html, JavaScript
Backend: JavaScript (Node, Express)
DB: MySQL

En cada EC2:
Docker
Docker-compose
Git

## Infraestructura Cloud del proyecto

La nube está organizada en una VPC con una zona de disponibilidad, que se divide en tres subnets (una pública y dos privadas). 
Dentro de cada subnet, hay una instancia EC2 para levantar el backend, frontend y db de la aplicación. 

### Instancias EC2 y sus Security Groups
- FRONT-EC2 (IP pública, subnet pública) :
  * HTTP: 8080 - 0.0.0.0/0
  * SSH: 22 - 0.0.0.0/0
  * TCP: 8080 - 0.0.0.0/0
  * ICMP: ALL - 0.0.0.0/0
- BACKEND-EC2 (IP privada, subnet privada 1) :
  * SSH: 22 - Origin SG-FRONT
  * ICMP: ALL - Origin SG-FRONT
  * TCP: 3001 - Origin SG-FRONT
- DB-EC2 (IP privada, subnet privada 2) :
  * ICMP: ALL - Origin SG-BACK
  * SSH: 22 - Origin SG-BACK
  * MYSQL: 3306 - Origin SG-BACK
 
## Estructura de carpetas de la aplicación (/home/ec2-user/ de cada instancia)
- frontend/ (FRONT-EC2)
   Dockerfile
   index.html
   app.js
   default.conf

- backend/ (BACKEND-EC2)
   Dockerfile
   server.js
   package.json

- db/ (DB-EC2)
   Dockerfile
   init.sql

Config de pipelines en repositorio:
- .github/workflows/
    cicd-tienda-frontend.yml
    cicd-tienda-backend.yml
    cicd-tienda-db.yml
     
## Verificar antes de correr
- default.conf del frontend debe a puntar a la IP privada actual de BACKEND-EC2.


## Flujo de comunicación entre instancias

1. FRONT-EC2: Nginx sirve el index.html y app.js al browser.
2. Nginx hace proxy_pass de /api/* hacia BACKEND-EC2.
3. BACKEND-EC2: Express (Node) procesa las peticiones REST
4. Se conecta a DB-EC2
5. DB-EC2: MySQL almacena y devuelve los datos.
