# PRD · Citas Médicas (citas-medicas-pos)

> Documento de requisitos del producto (Product Requirements Document).
> Complétalo en el Sprint 0 (historia H0.3) y pide la aprobación del profesor en un Issue.
> Cada requisito funcional (RF) se convierte en un Issue con la plantilla "Historia de usuario".

| Campo | Valor |
| --- | --- |
| Alumno | Nayi Muñoz |
| Proyecto n.º | 19 |
| Sitio de referencia | Doctoralia |
| Compañero revisor | @usuario-de-tu-companero |
| URL desplegada | (se completa en el Sprint 5) |

## 1. Problema

En consultorios y clínicas pequeñas, la coordinación de citas suele hacerse por llamadas telefónicas o mensajes de WhatsApp sin una agenda centralizada. Esto provoca choques de horarios, tiempos muertos entre consultas y demoras para el paciente al buscar un espacio libre con un especialista. Además, cuando el volumen de bloques horarios crece, revisar manualmente la disponibilidad se vuelve lento y propenso a errores.

## 2. Usuarios

- **Usuario principal:** Pacientes que necesitan buscar un médico por especialidad, consultar rápidamente el próximo bloque horario libre y reservar o cancelar su cita desde el teléfono móvil (y ocasionalmente desde computadora).
- **Usuario secundario:** Médicos y recepcionistas que configuran los bloques de atención diaria, revisan la agenda consolidada y gestionan el estado de las citas (confirmada, atendida o cancelada) principalmente desde computadora o tablet.

## 3. Objetivo y métrica de éxito

Un paciente encuentra el próximo bloque disponible de un especialista y confirma su cita médica en menos de 20 segundos, sin producir reservas duplicadas en un mismo bloque.

## 4. Alcance del MVP

Entre 5 y 8 funcionalidades que SÍ entran este semestre:

1. Gestión del catálogo de especialidades médicas.
2. Directorio y perfiles de médicos filtrables por especialidad.
3. Registro y consulta básica de pacientes.
4. Configuración de agenda médica por bloques horarios fijos.
5. Búsqueda binaria de disponibilidad sobre bloques ordenados cronológicamente.
6. Reserva, consulta, reprogramación y cancelación de citas médicas.

### Funcionalidades → carpetas de `src/` (screaming architecture)

Cada funcionalidad del alcance será una carpeta en `src/`. Nómbralas en inglés y en plural.

| Funcionalidad | Carpeta |
| --- | --- |
| Especialidades médicas | `src/specialties` |
| Directorio de médicos | `src/doctors` |
| Gestión de pacientes | `src/patients` |
| Agenda por bloques horarios | `src/schedules` |
| Búsqueda binaria de disponibilidad | `src/availability` |
| Gestión de citas | `src/appointments` |

## 5. Fuera de alcance

Lo que NO vas a construir:

- Pagos en línea, pasarelas bancarias ni facturación fiscal.
- Videollamadas o consultas por telemedicina.
- Historial clínico electrónico, diagnósticos ni emisión de récipes médicos.
- Envío automatizado de recordatorios por SMS o WhatsApp.
- Reseñas públicas o sistema de calificación de médicos.

## 6. Requisitos funcionales

| ID | Requisito | Prioridad (alta, media, baja) |
| --- | --- | --- |
| RF-01 | Registrar, listar y consultar especialidades médicas. | alta |
| RF-02 | Registrar médicos asociados a una especialidad y permitir filtrarlos por especialidad o nombre. | alta |
| RF-03 | Registrar y consultar datos básicos de los pacientes (nombre, correo y teléfono). | alta |
| RF-04 | Generar y administrar bloques horarios ordenados por fecha y hora para cada médico. | alta |
| RF-05 | Localizar el primer bloque disponible a partir de una fecha/hora dada usando búsqueda binaria. | alta |
| RF-06 | Reservar una cita sobre un bloque libre garantizando que no existan solapamientos ni dobles reservas. | alta |
| RF-07 | Cancelar o reprogramar una cita existente liberando automáticamente el bloque horario anterior. | media |
| RF-08 | Consultar la agenda del día o de la semana filtrada por médico y estado de la cita. | media |

## 7. Requisitos no funcionales

- Responsive, móvil primero (375 px y 1280 px).
- Lighthouse ≥ 90 en Accesibilidad y Buenas prácticas.
- 0 errores en la consola del navegador.
- Funciona con 10.000 registros sin congelarse.
- Búsqueda de disponibilidad en tiempo logarítmico $O(\log n)$ sobre bloques ordenados.

## 8. Modelo de datos

Diagrama entidad-relación y tablas PostgreSQL con tipos, claves y restricciones.

```mermaid
erDiagram
    SPECIALTY ||--o{ DOCTOR : agrupa
    DOCTOR ||--o{ SCHEDULE_BLOCK : define
    PATIENT ||--o{ APPOINTMENT : reserva
    SCHEDULE_BLOCK ||--o| APPOINTMENT : ocupa

    SPECIALTY {
        int id PK
        text name
    }
    DOCTOR {
        int id PK
        int specialty_id FK
        text full_name
        text license_number
        text office_location
    }
    PATIENT {
        int id PK
        text full_name
        text email
        text phone
    }
    SCHEDULE_BLOCK {
        int id PK
        int doctor_id FK
        timestamptz starts_at
        timestamptz ends_at
        boolean is_available
    }
    APPOINTMENT {
        int id PK
        int patient_id FK
        int schedule_block_id FK
        text status
        text reason
        timestamptz created_at
    }

## 9. Reto técnico distintivo
Agenda por bloques + búsqueda binaria de disponibilidad: cada médico genera su agenda dividida en bloques discretos de duración fija (por ejemplo, 30 minutos) almacenados y consultados en orden cronológico ascendente por starts_at. Cuando un paciente busca disponibilidad a partir de una fecha y hora objetivo, el módulo src/availability aplica búsqueda binaria ($O(\log n)$) sobre los bloques ordenados del médico para saltar directamente al primer bloque válido igual o posterior a la hora solicitada y comprobar disponibilidad sin recorrer linealmente toda la agenda, manteniendo respuestas rápidas incluso con 10.000 bloques cargados. A nivel de base de datos, se reforzará la integridad con índices por (doctor_id, starts_at) y una restricción de unicidad sobre el bloque asignado para evitar reservas simultáneas.

## 10. Riesgos y preguntas abiertas

-Concurrencia en reservas: ¿Cómo manejar si dos pacientes intentan reservar el mismo bloque en el mismo segundo? (Se prevé usar una transacción en PostgreSQL y restricción UNIQUE sobre schedule_block_id para citas activas).

Duración de bloques: ¿Todos los médicos usarán bloques uniformes de 30 minutos en el MVP o se permitirá configurar la duración por médico?

Zona horaria: Garantizar que todas las fechas se guarden en timestamptz (UTC) y se muestren correctamente en la hora local del usuario.

Regla de cancelación: ¿Existe un límite mínimo de horas previas para permitir que un paciente cancele una cita?
