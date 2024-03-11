# Postulación Devsu - DevOps

Repositorio: https://github.com/0x00cl/postulacion-devsu

Este proyecto es parte de la prueba tecnica para la postulación al cargo de DevOps en Devsu

Se utilizó como base Python: [demo-devops-python](https://bitbucket.org/devsu/demo-devops-python/src/master/)

El repositorio está estructurado en dos partes, el directorio `app` el cual contiene la demo de la aplicación escrita en Python y luego el directorio `deployment` el cual contiene tanto los archivos YAML como terrafom para la creación de recursos en la nube y despliegue de la aplicación.

**Nota**: El directorio `doc` se utiliza solo para las imagenes e información adicional que pueda ser relevante para el `README.md`

## App

### Dockerización

Dentro de los primeros puntos que se solicitaron desarrollar fue la dockerización del proyecto.

Para esto se creó un `Dockerfile` el cual utiliza como imagen base la imagen "oficial" de python la cual está basada en alpine. Principalmente se decidió por alpine ya que es una base liviana y por lo tanto genera imagenes livianes en comparación a las imagenes basadas en debian.

Las variable de ambiente `PYTHONUNBUFFERED` es para que cuando se ejecute la aplicación con python el texto de salida (logs) aparezca inmediatamente tanto en Docker como Kubernetes. Y la variable `DJANGO_DEBUG` que se utiliza para el desarrollo de la app, pero para ambientes de producción no se necesitan. 

La aplicación utiliza gunicorn para la ejecución del servidor de la app. Debido a que es un proyecto que sigue recibiendo actualizaciones y es simple de utilizar, se puede instalar con `pip` y ejecutar.

### CI Workflow

El siguiente diagrama muestra los pasos y el proceso por el que pasa un vez que se realiza el __push__ de un commit.

![Diagrama CI Github actions](/doc/diagramaCI.png "Diagrama CI Github actions")

["Ultima" ejecución del workflow completo](https://github.com/0x00cl/postulacion-devsu/actions/runs/8225629321)

El desarrollador hace push a la rama `main` a github, esto gatilla en Github Actions al flujo o __workflow__ definido en el archivo `.github/workflows/django.yml`. El cual pasa por una primera etapa definida como **build** y luego **build-docker-image**.

![Resultado ultima ejecucion CI en Github Actions](/doc/resultadoUltimoCI.png "Resultado ultima ejecucion CI en Github Actions")

#### Build

["Ultima" ejecución del job `build`](https://github.com/0x00cl/postulacion-devsu/actions/runs/8225629321/job/22490901615)

1. Hace pull del codigo nuevo
2. Intala Python 3.11
3. Instala las dependencias del proyecto
4. Ejecuta las pruebas unitarias
5. Realiza analisis con Sonarcloud (SonarQube)

#### build-docker-image

["Ultima" ejecución del job `build-docker-image`](https://github.com/0x00cl/postulacion-devsu/actions/runs/8225629321/job/22490911176)

1. Se configuran credenciales AWS
2. Se realiza __login__ a AWS ECR publico
3. Se configura Dcoker buildx
4. Se construye la imagen de la app y se hace push a AWS ECR

### Sonarcloud

El ultimo paso dentro del workflow es el analisis con Sonarcloud (SonarQube), la [ultima ejecución se puede encontrar aquí](https://sonarcloud.io/project/overview?id=0x00cl_postulacion-devsu).

![Resultado Sonarcloud](/doc/Sonarcloud.png "Resultado Sonarcloud")

## Deployment

### Terraform

Para la creación de los servicios en la nube se utilizó terraform el cual tiene sus recursos definido en el archivo [`main.tf`](/deployment/main.tf)

Debido al alcance del proyecto desarrollado solo se definió la creación del repositorio de AWS ECR publico.

De haber hecho el despliegue en Kubernetes en la nube, se habria tenido que definir la creación o utilización de un **VPC**, la creación de un role IAM si es que no existe para el cluster de EKS y finalmente la creación del cluster de EKS en sí.

![Terraform Plan](/doc/terraformPlan.png "Terraform Plan")

### Kubernetes

Para el despliegue de kubernetes la definición de los objetos están en el directorio [`/deployment/kubernetes`](/deployment/kubernetes) enumerados en el orden en el cual deben ser creados.

Este expone el servicio en cada nodo del cluster en el puerto 30800 y permite configurar la app mediante el `ConfigMap`.

![Salida Kubernetes](/doc/kubernetesOutput.png "Salida Kubernetes")

### Base de datos

Si bien no fue algo que se realizó por falta de tiempo pero si se tiene en cuenta la base de datos. La aplicación utiliza una base de datos SQLite que se puede utilizar para el desarrollo pero en este caso para producción en el cual se tienen al menos 2 replicas, cada nodo tendria su propia base de datos las cuales no estarian sincronizadas entre sí, ademas de que un `pod` al morir o ser eliminado borraria lo que se escribió en la base de datos perdiendo la información.

Por lo que idealmente se deberia utilizar una base de datos como PostgreSQL ya sea en la nube o algun servidor externo del cual las "N" replicas puedan acceder a ella y tener la misma información y sin perderla en caso que un `pod` sea eliminado.