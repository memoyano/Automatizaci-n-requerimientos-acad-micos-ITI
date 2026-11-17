# Gestión de Requerimientos Académicos — ITI

Solución low-code para centralizar el registro, seguimiento y auditoría de trámites académicos del **Instituto Tecnológico Internacional Universitario (ITI)**.

> Componente práctico del examen complexivo — Carrera de Desarrollo de Software  
> **Autores:** Anthony David Dávila Pintado · Kevin Alexander Jiménez Vélez  
> **Asesora:** MSc. María Esther Moyano Lucio

---

## ¿Qué resuelve?

Hoy muchos trámites se siguen por carpeta compartida, correo y mensajes informales. Eso dificulta saber:

- en qué estado está cada solicitud  
- qué área debe actuar  
- qué observaciones y evidencias ya existen  

Esta aplicación ofrece un **canal formal** con código correlativo (`REQ-####`), etapas por tipo de trámite, adjuntos, notificaciones y historial auditable.

---

## Cómo funciona (visión general)

```text
Estudiante                    Áreas académico-administrativas
    │                                      │
    ▼                                      ▼
Nueva Solicitud  ──►  Azure SQL + SharePoint  ──►  Bandeja Solicitudes
 (tipo, obs.,          (metadatos + archivo)       (filtrada por depto.)
  adjunto)
    │                                      │
    │         Power Automate               │
    └──── correo al responsable ───────────┘
                                           │
                     Avanzar / Regresar / Rechazar
                                           │
                                           ▼
                                    Historial (Auditoría)
```

1. El usuario inicia sesión con su **cuenta institucional** (Microsoft Entra ID).  
2. El estudiante crea una solicitud: tipo de trámite, observación y, si aplica, documento.  
3. El sistema genera el código `REQ-####`, asigna la **primera etapa** según el catálogo y notifica al área responsable.  
4. El personal de área revisa la bandeja, registra observación y puede **avanzar**, **regresar** o **rechazar**.  
5. Cada acción queda en el historial (quién, cuándo, etapa y comentario).  
6. El Dashboard muestra totales por estado y solicitudes recientes.

---

## Módulos de la aplicación

| Módulo | Función |
|---|---|
| **Inicio (Dashboard)** | KPIs (total, en proceso, finalizadas, rechazadas) y listado de solicitudes recientes |
| **Solicitudes** | Bandeja de trabajo: buscar, filtrar y gestionar el detalle (avanzar / regresar / rechazar) |
| **Nueva Solicitud** | Formulario de ingreso: tipo de trámite, observación y adjuntos |
| **Auditoría** | Historial de solicitudes y traza detallada de cada actuación |
| **Usuarios** | Alta/edición de usuarios con rol y departamento |

Manual paso a paso con capturas: [`docs/MANUAL_USUARIO.md`](docs/MANUAL_USUARIO.md)

---

## Arquitectura técnica

| Capa | Tecnología | Responsabilidad |
|---|---|---|
| Interfaz | Microsoft Power Apps (Canvas) | Pantallas operativas y lógica de negocio en app |
| Datos | Azure SQL Database | Metadatos: solicitudes, etapas, usuarios, historial |
| Documentos | SharePoint | Archivos adjuntos (no se guardan binarios en SQL) |
| Identidad | Microsoft Entra ID | Autenticación con correo institucional |
| Automatización | Power Automate | Carga documental y notificaciones por correo |

### Modelo de datos (8 tablas)

`Roles` · `Departamentos` · `Usuarios` · `TiposTramite` · `EtapasTramite` · `Solicitudes` · `Documentos` · `HistorialSolicitud`

### Estados de una solicitud

| Estado | Significado |
|---|---|
| `ENVIADA` | Registrada; pendiente de atención en la etapa actual |
| `EN PROCESO` | Ya fue atendida en al menos una etapa y continúa el flujo |
| `APROBADA` | Trámite finalizado favorablemente |
| `RECHAZADA` | Trámite denegado con observación |

### Roles y áreas

Roles de ejemplo: Estudiante, Secretaría, Coordinación, Sistemas, Colecturía, USSE, Rectorado, Administrador.

Departamentos de ejemplo: Secretaría, Coordinación de Carrera, Unidad de Sistemas, Colecturía, USSE, Rectorado.

La bandeja de solicitudes se filtra según el rol y el departamento del usuario autenticado.

---

## Flujo de Power Automate

| Flujo | Qué hace |
|---|---|
| **SubirDocumentoSolicitud** | Toma el adjunto temporal y lo deja en SharePoint; registra metadatos en `Documentos` |
| **NotificarResponsableTramite** | Envía correo Outlook al responsable del departamento de la etapa |

---

## Alcance del prototipo

Implementado:

- ciclo completo de registro → notificación → avance/rechazo → auditoría  
- generación de código `REQ-####` desde Power Apps  
- asignación dinámica de la primera etapa según `TiposTramite` / `EtapasTramite`  
- adjuntos con separación SQL / SharePoint  
- administración de usuarios (rol y departamento)

Líneas de continuidad previstas:

- biblioteca SharePoint institucional definitiva  
- reportes en Power BI  
- ampliación progresiva del catálogo (el diagnóstico institucional contempla 15 categorías y 179 tipos)

---

## Requisitos para usar la app

- Cuenta institucional del ITI (Microsoft 365 / Entra ID)  
- Acceso a la aplicación Canvas publicada en el entorno Power Platform del proyecto  
- Para personal de área: usuario activo en la tabla `Usuarios` con rol y departamento correctos  

> No se cargan manualmente miles de cuentas estudiantiles: al iniciar sesión, la identidad institucional se resuelve (o se da de alta como Estudiante cuando corresponde).

---

## Documentación del repositorio

| Archivo | Contenido |
|---|---|
| [`README.md`](README.md) | Visión general y funcionamiento (este documento) |
| [`docs/MANUAL_USUARIO.md`](docs/MANUAL_USUARIO.md) | Manual de usuario con capturas de pantalla |
| [`docs/images/`](docs/images/) | Imágenes referenciadas en el manual |

---

## Autores

| Estudiante |
|---|---|
| **Anthony David Dávila Pintado** |
| **Kevin Alexander Jiménez Vélez** |

Institución: Instituto Tecnológico Internacional Universitario — ITI  
Carrera: Desarrollo de Software  
Año: 2026
