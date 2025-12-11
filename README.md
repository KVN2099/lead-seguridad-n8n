# Servidor n8n con Docker Compose

Este repositorio contiene una configuración completa de Docker Compose para ejecutar **n8n**, una herramienta de automatización de flujos de trabajo de código abierto, junto con una base de datos PostgreSQL y un proxy inverso Nginx con características de seguridad.

## ¿Qué es n8n?

n8n es una plataforma de automatización de flujos de trabajo que permite conectar diferentes servicios y aplicaciones para crear flujos automatizados. Es una alternativa de código abierto a herramientas como Zapier o Make.

## Arquitectura

Este proyecto incluye tres servicios principales:

### 1. **PostgreSQL** (Base de datos)
- Almacena todos los datos de n8n (flujos de trabajo, credenciales, ejecuciones, etc.)
- Versión: PostgreSQL 16 Alpine
- Puerto: 5432
- Incluye healthcheck para verificar el estado del servicio

### 2. **n8n** (Servidor de automatización)
- Servidor principal de n8n
- Configurado con autenticación básica
- Conectado a PostgreSQL para persistencia de datos
- Puerto: 5678
- Volúmenes para persistir datos y configuraciones

### 3. **Nginx** (Proxy inverso y firewall)
- Actúa como proxy inverso frente a n8n
- Implementa medidas de seguridad:
  - Headers de seguridad HTTP
  - Limitación de velocidad (rate limiting)
  - Bloqueo de endpoints sensibles
  - Soporte para WebSockets
- Puertos: 80 (HTTP) y 443 (HTTPS, configurable)
- Configuración preparada para SSL/TLS

## Características de Seguridad

- **Autenticación básica**: Protección con usuario y contraseña
- **Rate limiting**: Limita las solicitudes para prevenir abusos
- **Headers de seguridad**: Protección contra ataques comunes (XSS, clickjacking, etc.)
- **Bloqueo de endpoints sensibles**: Restricción de acceso a rutas administrativas
- **Health checks**: Monitoreo del estado de los servicios

## Requisitos Previos

- Docker
- Docker Compose
- (Opcional) Certificados SSL si deseas habilitar HTTPS

## Instalación y Uso

### 1. Clonar o descargar el repositorio

```bash
git clone <url-del-repositorio>
cd n8n-server
```

### 2. Configurar variables de entorno (opcional)

Crea un archivo `.env` en la raíz del proyecto para personalizar la configuración:

```env
# Base de datos
DB_USER=n8n
DB_PASSWORD=tu_contraseña_segura
DB_NAME=n8n

# n8n
N8N_HOST=0.0.0.0
WEBHOOK_URL=http://tu-dominio.com/
N8N_USER=admin
N8N_PASSWORD=tu_contraseña_segura
```

Si no creas el archivo `.env`, se usarán los valores por defecto (no recomendado para producción).

### 3. Iniciar los servicios

```bash
docker-compose up -d
```

Este comando iniciará los tres servicios en segundo plano.

### 4. Acceder a n8n

Una vez que los servicios estén en ejecución, puedes acceder a n8n a través de:

- **Directamente**: http://localhost:5678
- **A través del proxy Nginx**: http://localhost:80

### 5. Verificar el estado de los servicios

```bash
docker-compose ps
```

### 6. Ver los logs

```bash
# Todos los servicios
docker-compose logs -f

# Servicio específico
docker-compose logs -f n8n
docker-compose logs -f postgres
docker-compose logs -f firewall
```

## Configuración de HTTPS (Opcional)

Para habilitar HTTPS:

1. Coloca tus certificados SSL en el directorio `certs/`:
   - `cert.pem` (certificado)
   - `key.pem` (clave privada)

2. Descomenta y configura la sección HTTPS en `nginx.conf` (líneas 117-130)

3. Descomenta la redirección HTTP a HTTPS (líneas 43-48 en `nginx.conf`)

4. Reinicia los servicios:

```bash
docker-compose restart firewall
```

## Estructura del Proyecto

```
n8n-server/
├── docker-compose.yml    # Configuración de los servicios Docker
├── nginx.conf            # Configuración del proxy inverso Nginx
├── certs/                # Directorio para certificados SSL (opcional)
├── .env                  # Variables de entorno (crear manualmente)
├── .gitignore            # Archivos ignorados por Git
└── README.md             # Este archivo
```

## Volúmenes Docker

El proyecto crea dos volúmenes persistentes:

- `postgres_data`: Almacena los datos de PostgreSQL
- `n8n_data`: Almacena los datos y configuraciones de n8n

Estos volúmenes persisten incluso si eliminas los contenedores.

## Comandos Útiles

### Detener los servicios
```bash
docker-compose down
```

### Detener y eliminar volúmenes (⚠️ elimina todos los datos)
```bash
docker-compose down -v
```

### Reiniciar un servicio específico
```bash
docker-compose restart n8n
```

### Actualizar n8n a la última versión
```bash
docker-compose pull n8n
docker-compose up -d n8n
```

## Solución de Problemas

### El servicio no inicia
- Verifica que los puertos 80, 443 y 5678 no estén en uso
- Revisa los logs: `docker-compose logs`

### Error de conexión a la base de datos
- Asegúrate de que PostgreSQL esté completamente iniciado (usa healthcheck)
- Verifica las credenciales en las variables de entorno

### Problemas con Nginx
- Verifica que el archivo `nginx.conf` esté correctamente configurado
- Revisa los logs de Nginx: `docker-compose logs firewall`

## Notas de Seguridad

⚠️ **IMPORTANTE**: Este proyecto incluye valores por defecto inseguros para desarrollo. Para producción:

1. Cambia todas las contraseñas por defecto
2. Habilita HTTPS con certificados válidos
3. Configura un dominio real en lugar de `localhost`
4. Revisa y ajusta las reglas de rate limiting según tus necesidades
5. Considera usar un gestor de secretos para las credenciales

## Licencia

Este proyecto es de código abierto. n8n tiene su propia licencia (ver [n8n.io](https://n8n.io) para más detalles).

## Recursos Adicionales

- [Documentación oficial de n8n](https://docs.n8n.io/)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)
- [Documentación de Nginx](https://nginx.org/en/docs/)

