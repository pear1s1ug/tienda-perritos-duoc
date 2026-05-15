# Tienda de Alimentos para Perritos

Aplicación Web CRUD Fullstack desplegada en AWS usando Docker y pipeline CI/CD automatizado con Github Actions y Amazon ECR.

## Infraestructura Cloud del proyecto

La nube está organizada en una VPC con una zona de disponibilidad, que se divide en tres subnets (una pública y dos privadas). 
Dentro de cada subnet, hay una instancia EC2 para levantar el backend, frontend y db de la aplicación. 

### Instancias EC2 y sus Security Groups
- FRONT-EC2 (Nginx server) :
  * 
- BACKEND-EC2 :
- DB-EC2 :
