# Backend Ventas - Innovatech Chile

Microservicio REST desarrollado con Spring Boot 3.4.4 y Java 17 que gestiona las órdenes de compra/ventas de Innovatech Chile.

## Tecnologías
- Java 17
- Spring Boot 3.4.4
- MySQL 8.0
- Docker (multi-stage build)
- GitHub Actions (CI/CD)

## Estructura del proyecto
back-Ventas_SpringBoot/
├── Springboot-API-REST/
│   ├── src/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── pom.xml
└── .github/
└── workflows/
└── deploy.yml

## Variables de entorno requeridas

| Variable | Descripción | Ejemplo |
|---|---|---|
| `SPRING_DATASOURCE_URL` | URL de conexión a MySQL | `jdbc:mysql://mysql-ventas:3306/ventas_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuario de la BD | `ventas_user` |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña de la BD | `ventas_pass` |

## Endpoints disponibles

| Método | Endpoint | Descripción |
|---|---|---|
| GET | `/api/v1/ventas` | Lista todas las órdenes de venta |
| GET | `/api/v1/ventas/{id}` | Obtiene una venta por ID |
| PUT | `/api/v1/ventas/{id}` | Actualiza una venta (ej: marcar despacho generado) |

## Levantar con Docker Compose

```bash
# Crear archivo .env con las variables
cp .env.example .env

# Levantar servicios
docker-compose up -d

# Verificar
docker ps
```

## Dockerfile (multi-stage)

El Dockerfile usa multi-stage build para optimizar la imagen final:
- **Etapa 1 (builder):** Compila el proyecto con Maven y Java 17
- **Etapa 2 (producción):** Imagen mínima con JRE Alpine, usuario no root `appuser`

Esto reduce el tamaño de la imagen y aplica el principio de mínimo privilegio.

## Persistencia de datos

Se usa **named volume** (`mysql_ventas_data`) para persistir los datos de MySQL. A diferencia de bind mounts, los named volumes son gestionados por Docker, más portables y no dependen de la estructura de directorios del host, lo que los hace ideales para producción en EC2.

## Pipeline CI/CD

El pipeline se activa automáticamente con cada `push` a la rama `deploy`:

1. **Build:** Construye la imagen Docker desde el Dockerfile multi-stage
2. **Push:** Publica la imagen en Docker Hub (`daniel0netto/back-ventas:latest`)
3. **Deploy:** Conecta via SSH a la EC2 backend y actualiza el contenedor

### Secrets requeridos en GitHub

| Secret | Descripción |
|---|---|
| `DOCKER_USERNAME` | Usuario de Docker Hub |
| `DOCKER_TOKEN` | Token de acceso de Docker Hub |
| `EC2_HOST` | IP de la instancia EC2 |
| `EC2_SSH_KEY` | Clave privada SSH (.pem) |

## Despliegue en AWS EC2

La instancia EC2 backend está en una **subred privada** de la VPC Innovatech, sin acceso directo desde Internet. Solo el frontend puede comunicarse con ella a través de los Security Groups configurados.
