# Tienda de Alimentos para Perritos

Aplicación Web CRUD Fullstack desplegada en AWS usando Docker y pipeline CI/CD automatizado con Github Actions y Amazon ECR.

## Lenguajes y tecnologías
* Frontend: html, JavaScript
* Backend: JavaScript (Node, Express)
* DB: MySQL

En cada EC2, por User Data:
* Docker
* Docker-compose
* Git

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
   * Dockerfile
   * index.html
   * app.js
   * default.conf

- backend/ (BACKEND-EC2)
   * Dockerfile
   * server.js
   * package.json

- db/ (DB-EC2)
   * Dockerfile
   * init.sql

Config de pipelines en repositorio:
- .github/workflows/
    * cicd-tienda-frontend.yml
    * cicd-tienda-backend.yml
    * cicd-tienda-db.yml

## Flujo de los pipelines
* Checkout del código
* Config credenciales AWS
* Login en Amazon ECR
* Build imagen Docker
* Push a Amazon ECR con tag latest
* Deploy via AWS SSM en la EC2 correspondiente
* docker pull de la nueva imagen
* docker stop + docker rm del contenedor viejo
* docker run del contenedor nuevo

## API Endpoints
* GET /api/productos
* GET /api/productos/:id
* POST /api/productos
* PUT /api/productos/:id
* DELETE /api/productos/:id
* GET /api/health

## Link a repositorios Docker Hub
https://hub.docker.com/repositories/pear1s1ug

## Imágenes Docker Hub
- tienda-frontend
- tienda-backend
- tienda-db

## Repositorios ECR
- tienda-frontend
- tienda-backend
- tienda-db

## Triggers
- `/frontend` -> ejecuta cicd-tienda-frontend.yml
- `/backend` -> ejecuta cicd-tienda-backend.yml
- `/db` -> ejecuta cicd-tienda-db.yml

## Secrets Github Actions
* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* AWS_SESSION_TOKEN
* AWS_REGION
* ECR_REGISTRY
* ECR_REPO_URL_BACKEND
* ECR_REPO_URL_FRONTEND
* ECR_REPO_URL_DB
* EC2_BACKEND_INSTANCE_ID
* EC2_FRONTEND_INSTANCE_ID
* EC2_DB_INSTANCE_ID
* DB_HOST
* DB_USER
* DB_PASSWORD
* DB_NAME
* DB_PORT
     
## Verificar antes de correr
- default.conf del frontend debe a puntar a la IP privada actual de BACKEND-EC2.
- Actualizar secrets, ya que las credenciales AWS expiran cada pocas horas.

## Comandos útiles Linux AMI
- Ver contenedores corriendo
  * docker ps

- Ver todos los contenedores (incluye detenidos)
  * docker ps -a
    
- Ver imágenes existentes localmente
  * docker images
 
- Ver logs de un contenedor
  * docker logs tienda-backend
  * docker logs tienda-frontend
  * docker logs tienda-db

- Iniciar un contenedor existente
  * docker start tienda-<capa>

 - Detener un contenedor
   * docker stop tienda-<capa>

 - Eliminar un contenedor
   * docker rm tienda-<capa>
 
- Entrar a un contenedor interactivamente
  * docker exec -it tienda-<capa> sh
    
- Verificar productos existentes en la db (en su EC2)
  * docker exec -it tienda-db mysql -u root -p<DB_PASSWORD> -e "USE tienda_perritos; SELECT * FROM productos;"

- Reiniciar la config de docker en una instancia (ejecutar en este orden)
  * docker stop tienda-<capa>
  * docker rm tienda-<capa>
  * docker volume rm dbdata (sólo si es db)
  * docker run correspondiente (ver sección "Despliegue manual de las imágenes Docker")
 
- Health check del backend
  * curl http://localhost:3001/api/health

## Despliegue manual de las imágenes Docker (de ser necesario, primera vez, en este orden, y 
tras haberse conectado a las instancias por Session Manager)
- DB-EC2
  * docker run -d --name tienda-db -p 3306:3306 -v dbdata:/var/lib/mysql <ECR_REPO_URL_DB>:latest
- BACKEND-EC2
  * mucho texto
- FRONTEND-EC2
  * docker run -d --name tienda-frontend -p 80:80 <ECR_REPO_URL_FRONTEND>:latest

## Flujo de comunicación entre instancias

1. FRONT-EC2: Nginx sirve el index.html y app.js al browser.
2. Nginx hace proxy_pass de /api/* hacia BACKEND-EC2.
3. BACKEND-EC2: Express (Node) procesa las peticiones REST
4. Se conecta a DB-EC2
5. DB-EC2: MySQL almacena y devuelve los datos.
