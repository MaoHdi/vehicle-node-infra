# co-hdi-vehicle-services-mediation-infra


---

## 📂 cloudformation/

Contiene los templates de infraestructura definidos en formato YAML para ser desplegados en AWS mediante CloudFormation.

- `infra.yml`:  
  Define los recursos principales del proyecto, como funciones Lambda, API Gateway, roles IAM, buckets S3, etc.  
  Este archivo puede ser parametrizado usando los archivos de entorno ubicados en `environments/`.

---

## 📂 environments/

Contiene archivos JSON con variables específicas para cada ambiente. Estos archivos permiten personalizar el despliegue según el entorno deseado.

- `dev.environment.json`:  
  Variables para el entorno de desarrollo.

- `nonprod.environment.json`:  
  Variables para entornos intermedios como QA, staging o pruebas.

- `prod.environment.json`:  
  Variables para el entorno de producción.

Cada archivo incluye parámetros como nombres de recursos, regiones, configuraciones específicas y valores sensibles que se inyectan en el template de CloudFormation.

---

## 🚀 Cómo desplegar

1. Selecciona el entorno deseado (`dev`, `nonprod`, `prod`).
2. Ejecuta el despliegue usando Jenkins.
3. Asegúrate de que los parámetros del archivo `.environment.json` coincidan con los requerimientos del template `infra.yml`.

---

## 🤝 Contribuciones

Si deseas contribuir, por favor revisa el archivo [CONTRIBUTING.md](./CONTRIBUTING) para seguir el flujo de trabajo GitFlow y las buenas prácticas del equipo.

---

