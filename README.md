# co-hdi-vehicle-services-mediation-infra

Infraestructura de las lambdas que migran el servicio Java `VehicleServices`:
**`crearConsultaInspMils`** y **`crearPolizaIaxis`**. Se define con **Serverless
Framework**, siguiendo el patron del resto de los servicios del equipo.

El codigo de las lambdas vive en
[`co-hdi-vehicle-service-mediation-lambda`](../vehicle-nodejs).

## Estructura

```
serverless.yml              Servicio: functions, API, IAM, VPC, stackTags y resources
environments/dev.yml        Valores por ambiente: region, subredes, security group,
environments/nonprod.yml    nombre del secreto de iAxis y memoria de las lambdas
environments/prod.yml
Jenkinsfile                 CI/CD (shared-pipelines/serverless)
```

## Que despliega

| Recurso | Detalle |
|---|---|
| API Gateway REST | `POST` y `GET` en `/vehicle-services/inspecciones` y `/vehicle-services/polizas`, X-Ray activo y logs de ejecucion sin cuerpo de peticion |
| `crearConsultaInspMils` | Node 22 en VPC. `POST` atiende JSON y SOAP; `GET ?wsdl` devuelve el contrato |
| `crearPolizaIaxis` | Node 22 en VPC, mismas rutas para emision de polizas |
| Rol de ejecucion | Lectura solo del secreto de iAxis, `ec2:Describe*` para la VPC y X-Ray |
| `Custom::ResourceLookup` | Resolucion de VPC y subredes privadas |
| Security group | `vehicle-services-sg-azb-<stage>` con las etiquetas `hdi_*` |

## Como se empaqueta el codigo

Serverless despliega codigo e infraestructura en una sola operacion. Como el codigo
esta en otro repositorio, el pipeline lo clona en `app/` antes de desplegar y
`serverless.yml` empaqueta desde esa ruta (`custom.app_path`):

```
stage('Checkout App Code')        git co-hdi-vehicle-service-mediation-lambda -> app/
                                  npm install --omit=dev
stage('Continuos Integration')    shared-pipelines/serverless/JenkinsfileCI
stage('Continuos Deployment')     shared-pipelines/serverless/JenkinsfileCD
```

El parametro `AppRef` del job selecciona la rama del repositorio de codigo (por
defecto `master`). Los handlers quedan como `app/src/handlers/<funcion>.handler`;
todas las rutas internas del codigo se resuelven con `__dirname`, asi que el prefijo
es transparente.

Para desplegar localmente:

```bash
git clone https://github.com/hdiseguroscol/co-hdi-vehicle-service-mediation-lambda.git app
(cd app && npm install --omit=dev)
npx serverless deploy --stage dev
```

## Configuracion por ambiente

`environments/<stage>.yml` se entrega con los valores de red **vacios**. Antes del
primer despliegue hay que completar:

| Llave | Valor esperado |
|---|---|
| `vpc` | VPC del ambiente |
| `subnet_c`, `subnet_d` | Subredes privadas con salida hacia Oracle iAxis |
| `securityGroupID` | Security group que permite el puerto de Oracle |
| `iaxis_secret_name` | Secreto con `{"username","password","connectString"}` |
| `memory_size` | Memoria de las lambdas (512 en dev/nonprod, 1024 en prod) |

Requisitos adicionales:

- El secreto `co-hdi-vehicle-services-iaxis-secret-<stage>` debe existir en Secrets
  Manager. Es el unico origen de credenciales: no hay credenciales en este
  repositorio ni en variables de entorno.
- El bucket `co-s3-vehicle-service-mediation-deployment-<stage>` debe existir
  (Serverless no crea buckets de despliegue propios).
- La configuracion no sensible de la aplicacion (URLs de los servicios SOAP externos,
  timeouts, nivel de log, auditoria) vive en `src/env/<stage>.env` del repositorio de
  codigo.

## Historico

Este repositorio usaba CloudFormation (`cloudformation/infra.yml` y
`environments/*.environment.json`). Esos archivos se reemplazaron por la definicion
Serverless; el scaffold anterior nunca fue desplegable (todos sus valores eran
literales `"String"` / `"Number"`) y queda en el historico de git.

## Contribuciones

Revisar [CONTRIBUTING.md](./CONTRIBUTING.md) para el flujo GitFlow del equipo.
