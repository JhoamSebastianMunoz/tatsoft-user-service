# TatSoft Gestión de Usuarios

Microservicio REST para la gestión de usuarios de la plataforma **TatSoft**. Expone operaciones CRUD, autenticación basada en JWT y restablecimiento de contraseña, pensado para integrarse con el microservicio de autenticación.

Construido con **Spring Boot 3.4**, **Spring Security**, **Spring Data JPA** y **MySQL**.

## Características

- CRUD completo de usuarios con validación de datos.
- Autenticación *stateless* mediante tokens **JWT** (HMAC256).
- Contraseñas almacenadas con hash **BCrypt**.
- Control de acceso por roles: `ADMINISTRADOR` y `COLABORADOR`.
- Endpoints públicos para consulta por cédula/correo y restablecimiento de contraseña, consumibles por otros microservicios.
- Documentación interactiva con **Swagger UI / OpenAPI**.
- Manejo centralizado de errores con respuestas JSON consistentes.

## Stack tecnológico

| Componente | Versión |
| --- | --- |
| Java | 17 |
| Spring Boot | 3.4.0 |
| Spring Security | 6.x |
| Spring Data JPA / Hibernate | 6.x |
| MySQL | 8.x |
| java-jwt (Auth0) | 4.4.0 |
| springdoc-openapi | 2.1.0 |
| Maven | Wrapper incluido |

## Requisitos previos

- JDK 17 o superior.
- MySQL 8 en ejecución.
- Maven (opcional, el wrapper `mvnw` está incluido).

## Configuración

1. Crea la base de datos ejecutando el script incluido:

   ```sql
   -- dbUsuarios.sql
   CREATE DATABASE microservicioUsuarios;
   ```

2. Ajusta las credenciales y la clave JWT en `src/main/resources/application.properties`:

   ```properties
   server.port=10101
   spring.datasource.url=jdbc:mysql://localhost:3306/microservicioUsuarios
   spring.datasource.username=root
   spring.datasource.password=tu_contraseña
   jwt.secret.key=una_clave_larga_y_segura_de_al_menos_256_bits
   ```

> [!WARNING]
> No publiques credenciales reales ni la clave `jwt.secret.key` en el repositorio. Usa variables de entorno o un gestor de secretos en producción.

## Ejecución

```bash
# Linux / macOS
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

El servicio queda disponible en `http://localhost:10101`.

Para empaquetar y ejecutar el JAR:

```bash
./mvnw clean package
java -jar target/tatSoftGestionUsuarios-0.0.1-SNAPSHOT.jar
```

## API

La documentación interactiva está disponible en:

- **Swagger UI:** `http://localhost:10101/swagger-ui/index.html`
- **OpenAPI JSON:** `http://localhost:10101/api-docs`

### Usuarios

| Método | Ruta | Descripción | Acceso |
| --- | --- | --- | --- |
| `POST` | `/api/usuarios` | Crear un usuario | `ADMINISTRADOR` |
| `GET` | `/api/usuarios` | Listar todos los usuarios | `ADMINISTRADOR` |
| `GET` | `/api/usuarios/{id}` | Obtener usuario por ID | `ADMINISTRADOR` |
| `PUT` | `/api/usuarios/{id}` | Actualizar un usuario | `ADMINISTRADOR` |
| `DELETE` | `/api/usuarios/{id}` | Eliminar un usuario | `ADMINISTRADOR` |
| `GET` | `/api/usuarios/cedula/{cedula}` | Buscar por cédula | Público |
| `GET` | `/api/usuarios/email/{email}` | Buscar por correo | Público |
| `GET` | `/api/usuarios/id_usuario/{id_usuario}` | Obtener usuario por ID | Público |
| `PUT` | `/api/usuarios/update-password` | Restablecer contraseña | Público |

### Ejemplo

Crear un usuario:

```bash
curl -X POST http://localhost:10101/api/usuarios \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "cedula": "1234567890",
    "nombreCompleto": "Ana Gómez",
    "celular": "3001234567",
    "correo": "ana.gomez@example.com",
    "contraseña": "Secreta123",
    "rol": "COLABORADOR"
  }'
```

## Autenticación

El microservicio **no emite** tokens; los valida. Los tokens JWT son generados por el microservicio de autenticación y deben incluir los claims `role` y `cedula`.

Envíalos en cada solicitud protegida mediante la cabecera:

```http
Authorization: Bearer <TOKEN>
```

El filtro `JwtAuthenticationFilter` valida la firma y expiración del token (1 hora) y otorga la autoridad `ROLE_<rol>`.

## Estructura del proyecto

```
src/main/java/com/tatSoftGestionUsuarios
├── config/       # Seguridad, filtro JWT, utilidades y manejo de excepciones
├── controller/   # Endpoints REST
├── dto/          # Objetos de entrada (request)
├── model/        # Entidades JPA
├── repository/   # Repositorios Spring Data
├── service/      # Lógica de negocio
└── utils/        # DTOs de respuesta
```

## Despliegue

El workflow de GitHub Actions (`.github/workflows/master_backendusermanagement.yml`) compila el proyecto con Maven y lo despliega en **Azure App Service** al hacer push sobre la rama `master`.

## Pruebas

```bash
./mvnw test
```
