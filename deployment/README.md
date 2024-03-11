# Deployment app

## Requisitos

* Credenciales AWS que permitan crear un repositorio en ECR, subir imagenes docker, crear un cluster EKS, crear un VPC.
* Terraform >= 1.6.0

## Ejecución

Primero se deben crear los servicios en AWS. Para esto se debe utilizar terraform.

```
$ terraform plan
```

```
$ terraform apply
```

Luego, una vez creado y teniendo un cluster de Kubernetes. con la herramienta `kubectl` se deberán aplicar los archivos YAML del directorio `kubernetes` en orden por como están numeros.

```
$ kubectl apply -f <archivo.yaml>
```

1. Creación de namespace
2. Creación del ConfigMap para la configuración de la aplicación mediante variables de entorno.
3. Creación de la variable de entorno secreto para la llave secreta de django.
4. Creación del servicio para exponer la app bajo el puerto 30800.
5. Creación del despliegue de la app con 2 replicas.
