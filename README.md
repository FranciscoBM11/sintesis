# SISTESIS-UNI

Sistema Integral de Seguimiento y Gestión de Tesis Monográficas (UNI).

## Arquitectura

### C4 Nivel 1 - Contexto

```mermaid
---
config:
  c4:
    c4ShapeMargin: 100
---
C4Context
  title Diagrama de Contexto (Nivel 1 C4) - SISTESIS-UNI
  Person(estudiante, "Estudiante UNI", "Somete protocolos y propuestas de tesis")
  Person(docente, "Docente / Tutor", "Revisor de propuestas y evaluador de avances")
  Person(admin, "Administrador TIC", "Gestiona usuarios, roles y parámetros institucionales")
  System_Ext(siat, "SIAT UNI", "Sistema de Control Académico Central")
  System(sistesis, "SISTESIS-UNI", "Sistema de Gestión de Tesis Monográficas")
  System_Ext(notif, "Servicio de Notificaciones", "SMTP / Servidor de correo UNI")
  Rel(estudiante, sistesis, "Envía protocolos")
  Rel(docente, sistesis, "Revisa y aprueba")
  Rel(admin, sistesis, "Configura periodos")
  Rel(sistesis, siat, "Sincroniza matrícula")
  Rel(sistesis, notif, "Envía avisos")
  UpdateRelStyle(sistesis, siat, $offsetX="-35", $offsetY="-18")
  UpdateRelStyle(sistesis, notif, $offsetX="-5", $offsetY="-18")
  UpdateRelStyle(estudiante, sistesis, $offsetX="-90", $offsetY="12")
  UpdateRelStyle(docente, sistesis, $offsetX="35", $offsetY="0")
  UpdateRelStyle(admin, sistesis, $offsetX="90", $offsetY="12")
  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

### C4 Nivel 2 - Contenedores

```mermaid
---
config:
  c4:
    c4ShapeMargin: 100
---
C4Container
  title Diagrama de Contenedores (Nivel 2 C4) - SISTESIS-UNI
  System_Ext(siat, "SIAT UNI API", "Sistema de Control Académico")
  System_Ext(smtp, "Servidor SMTP UNI", "Notificaciones")
  Person(docente, "Docente / Tutor", "Revisor")
  Person(estudiante, "Estudiante UNI", "Usuario")
  Boundary(sistesis, " ", " ") {
    Container(mobile, "Aplicación Móvil", "Flutter", "Consultas rápidas y notificaciones")
    Container(api, "API RESTful backend", "ASP.NET Core 8 / C#", "Lógica de negocio y control de acceso")
    Container(spa, "Single Page Application", "Angular / TypeScript", "Interfaz web para usuarios")
    ContainerDb(db, "PostgreSQL", "PostgreSQL", "Almacenamiento relacional de metadatos")
    ContainerDb(blob, "Almacenamiento de objetos", "MinIO / S3", "Documentos PDF")
  }
  Rel(estudiante, spa, "Usa", "HTTPS / WSS")
  Rel(docente, spa, "Usa", "HTTPS / WSS")
  Rel(estudiante, mobile, "Usa", "HTTPS")
  Rel(spa, api, "Consume", "JSON / HTTPS")
  Rel(mobile, api, "Consume", "JSON / HTTPS")
  Rel(api, db, "Lee y escribe", "Npgsql")
  Rel(api, blob, "Guarda PDF", "S3 SDK")
  Rel(api, siat, "Consulta datos", "REST")
  Rel(api, smtp, "Envía correos", "SMTP")
  UpdateRelStyle(estudiante, spa, $offsetX="150", $offsetY="-25")
  UpdateRelStyle(estudiante, mobile, $offsetX="40", $offsetY="0")
  UpdateRelStyle(docente, spa, $offsetX="60", $offsetY="0")
  UpdateRelStyle(api, siat, $offsetX="-60", $offsetY="0")
  UpdateRelStyle(api, smtp, $offsetX="60", $offsetY="0")
  UpdateRelStyle(mobile, api, $offsetX="0", $offsetY="-30")
  UpdateRelStyle(spa, api, $offsetX="0", $offsetY="-30")
  UpdateRelStyle(api, db, $offsetX="-110", $offsetY="10")
  UpdateRelStyle(api, blob, $offsetX="70", $offsetY="10")
  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

### Secuencia

```mermaid
sequenceDiagram
  autonumber
  actor E as Estudiante UNI
  participant C as SPA (Angular)
  participant AU as API Gateway / Auth Controller
  participant T as Thesis Controller / Command Service
  participant S as Cloud / Blob Storage
  participant DB as PostgreSQL
  E->>C: Ingresa credenciales (UNI ID y contraseña)
  C->>AU: POST /api/v1/auth/login
  AU->>DB: Validar credenciales e identidad
  DB-->>AU: Usuario válido (Role: Student)
  AU-->>C: 200 OK: JWT Access Token + Refresh Token
  E->>C: Carga el protocolo de tesis (.pdf) y los metadatos
  C->>T: POST /api/v1/theses/upload (Authorization: Bearer JWT)
  activate T
  T->>T: Validar el JWT y los claims
  T->>S: Enviar el PDF como flujo de bytes
  S-->>T: URI del archivo almacenado
  T->>DB: Registrar metadatos de la tesis (Submitted)
  DB-->>T: Confirmación (ID de la tesis)
  T-->>C: 201 Created (JSON de la tesis)
  deactivate T
  C-->>E: Muestra la confirmación de entrega registrada
```

## API

El contrato está en `openapi.yaml` (OpenAPI 3.0.3).
