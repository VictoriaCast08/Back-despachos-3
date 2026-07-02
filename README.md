# Backend Despachos - SpringBoot

API REST desarrollada en Java Spring Boot para la gestión de despachos de ITPCARGO™. Desplegada en AWS EKS como parte de la EP3 del curso Introducción a Herramientas DevOps (ISY1101).

## Tecnologías

- Java 17
- Spring Boot 3.4.4
- Spring Data JPA
- MySQL 8
- Docker (multi-stage build)
- GitHub Actions (CI/CD)
- Amazon ECR (registro de imágenes)
- Amazon EKS (orquestación Kubernetes)

## Requisitos previos

- Docker Desktop instalado
- Java 17 o superior
- Maven 3.9 o superior
- AWS CLI v2 configurado
- kubectl instalado

## Cómo ejecutar localmente

### Con Docker Compose

```bash
docker compose up -d
```

La API estará disponible en: http://localhost:8081

### Sin Docker

```bash
cd Springboot-API-REST-DESPACHO
mvn spring-boot:run
```

## Variables de entorno

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| SPRING_DATASOURCE_URL | URL de conexión a MySQL | jdbc:mysql://mysql-service:3306/despachos_db |
| SPRING_DATASOURCE_USERNAME | Usuario de la base de datos | appuser |
| SPRING_DATASOURCE_PASSWORD | Contraseña de la base de datos | Gestionada via Kubernetes Secret |
| SPRING_JPA_HIBERNATE_DDL_AUTO | Estrategia DDL de Hibernate | update |
| SERVER_PORT | Puerto del servidor | 8081 |

## Endpoints disponibles

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | /api/v1/despachos | Listar todos los despachos |
| GET | /api/v1/despachos/{id} | Obtener despacho por ID |

## Estructura del proyecto

```
back-Despachos_SpringBoot/
├── Springboot-API-REST-DESPACHO/
│   ├── Dockerfile                  # Multi-stage build (Maven builder + JRE Alpine)
│   └── src/                        # Código fuente Spring Boot
├── docker-compose.yml              # Stack local con MySQL para desarrollo
├── .dockerignore                   # Archivos excluidos del build
└── .github/
    └── workflows/
        └── cicd-backend-despachos.yml   # Pipeline CI/CD hacia EKS
```

## Pipeline CI/CD

El pipeline se activa automáticamente con cada push a la rama `deploy`:

1. Checkout del código fuente
2. Configuración de credenciales AWS
3. Login a Amazon ECR
4. Build de la imagen Docker con Maven
5. Push de la imagen a Amazon ECR
6. Configuración de kubectl con kubeconfig
7. Deploy en EKS mediante `kubectl rollout restart`

## Infraestructura AWS — EP3

| Componente | Detalle |
|------------|---------|
| Orquestador | Amazon EKS 1.31 |
| Clúster | cluster-innovatech |
| Namespace | innovatech |
| Tipo de Service | ClusterIP (acceso interno) |
| Réplicas | 2 pods (escalable hasta 4 con HPA) |
| Imagen | Amazon ECR — backend-despachos:latest |
| Registro | 865721471726.dkr.ecr.us-east-1.amazonaws.com |
| Región | us-east-1 |
| Base de datos | MySQL 8 en pod Kubernetes (mysql-service:3307) |

## Autoscaling (HPA)

El Horizontal Pod Autoscaler escala automáticamente los pods del backend despachos:

- Mínimo: 1 pod
- Máximo: 4 pods
- Umbral de escalado: 50% de uso de CPU

## Manifiestos Kubernetes

Los manifiestos del clúster se encuentran en la carpeta `k8s/` del proyecto principal:

- `namespace.yaml` — Namespace `innovatech`
- `secret.yaml` — Credenciales de base de datos
- `configmap.yaml` — Variables de entorno
- `backend-despachos-deployment.yaml` — Deployment + Service ClusterIP
- `hpa.yaml` — Horizontal Pod Autoscaler

## Comunicación entre servicios

El backend despachos se comunica con MySQL mediante el DNS interno de Kubernetes (`mysql-service`), sin necesidad de IPs estáticas. El frontend accede al backend despachos a través del Service ClusterIP en el puerto 8081.

## Principios DevOps aplicados

- **Contenedorización**: Dockerfile multi-stage con usuario no-root (`appuser`)
- **Orquestación**: Kubernetes en AWS EKS con autorecuperación de pods
- **CI/CD**: Pipeline completamente automatizado con GitHub Actions
- **Autoscaling**: HPA escala pods según demanda de CPU
- **Seguridad**: Credenciales manejadas via GitHub Secrets y Kubernetes Secrets
- **Alta disponibilidad**: 2 réplicas distribuidas en 2 zonas de disponibilidad (us-east-1a y us-east-1b)
