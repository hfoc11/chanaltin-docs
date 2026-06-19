# 1. Sistema Avanzado de Programación de Viajes (Trip Scheduling)

El Sistema Avanzado de Programación de Viajes para Chanaltín centraliza la planificación, validación de seguridad y control de costos. Su objetivo es reemplazar el uso de Excel y procesos manuales por un flujo integrado en Odoo v18.

## Gestión de Flota y Operaciones Avanzadas

El estándar de Odoo v18 proporciona los cimientos de flota, empleados, mantenimiento, ventas, compras, contabilidad y servicios de campo. Sin embargo, la operación de Chanaltín requiere una arquitectura extendida porque el proceso real combina programación de viajes, doble conductor, tracto, carreta, retornos con carga, control documental, checklists, incidencias y rentabilidad por placa.

### Módulos de Odoo Involucrados

- **Flota (`fleet`):** Gestión de activos como tractos, carretas y montacargas. Aporta `fleet.vehicle`, placa, odómetro, contratos, servicios y logs de asignación.
- **Empleados (`hr`):** Legajos de conductores, datos privados de empleado, puesto, calendario laboral y estructura organizativa mediante `hr.employee`.
- **Mantenimiento (`maintenance`):** Órdenes de trabajo, mantenimiento preventivo/correctivo, equipos, estados, prioridades y bloqueos operativos mediante `maintenance.request`.
- **Servicios de Campo (`industry_fsm`):** Ejecución móvil de tareas, firmas, reportes y checklists sobre `project.task`.
- **Ventas (`sale`):** Facturación de fletes, servicios de alquiler y vínculo comercial con el cliente.
- **Compras (`purchase`):** Órdenes de servicio o compra para transportistas terceros.
- **Contabilidad (`account_accountant`):** Asientos, líneas contables, conciliación y distribución analítica para medir rentabilidad por placa, ruta o centro de costo.

### Enfoque de Arquitectura

La recomendación es extender modelos estándar cuando ya existe un objeto de negocio sólido, y crear modelos nuevos solo donde Odoo no cubre el dominio logístico de Chanaltín.

- Extender `fleet.vehicle` para datos técnicos de tractos, carretas, montacargas, documentos y horómetros.
- Extender `hr.employee` para datos de conductor, licencias, SCTR, descanso y estado operativo.
- Crear `chanaltin.trip` como objeto central de viaje.
- Crear `chanaltin.route` para parametrizar origen, destino, distancia, zona y valor referencial MTC.
- Crear `chanaltin.checklist` o integrar con `project.task`/FSM cuando el checklist deba ejecutarse en móvil.
- Crear `chanaltin.incident` para papeletas, SUTRAN, SAT, incidencias disciplinarias y responsabilidad económica.

## Gestión de Insumos para la Programación

El proceso inicia cuando se recibe la necesidad de servicio y requiere consolidar los siguientes datos maestros:

- **Asignación multivariable:** Unidad (tracto), carreta, conductor titular y, en rutas largas, conductor asistente.
- **Definición de rutas:** Clasificación entre rutas cortas y rutas largas para aplicar reglas de descanso y viáticos.
- **Tercerización:** Posibilidad de asignar viajes a proveedores terceros, generando una Orden de Servicio vinculada para controlar el costo.

## Motor de Validaciones Críticas

Validaciones automáticas antes de confirmar una programación:

- **Control de descansos:** Si un conductor realiza una ruta larga, no debe programarse inmediatamente otra ruta larga.
- **Disponibilidad documental:** Bloqueo si la unidad o el conductor tienen documentos vencidos: SOAT, revisión técnica, brevete, SCTR, entre otros.
- **Estado de mantenimiento:** Sincronización con el módulo de taller para impedir despachos si hay mantenimiento crítico pendiente.

## Ejecución y Seguimiento de Viaje

Durante y después del viaje se registran y gestionan:

- **Documentación de salida:** Generación o vinculación de Guía de Remisión Transportista electrónica.
- **Carga operativa:** Registro de capacidad vs. carga real.
- **Integración de viáticos:** Conexión con el módulo de Gastos para liquidar montos por placa y ruta.

## Cierre Operativo y Data Sucia

Al retorno, el planificador cierra el viaje y alimenta el repositorio operacional para KPIs:

- **KPI Entrega Perfecta:** Incidencias, faltantes, daños y demoras.
- **Tiempos de atención:** Horas de llegada, inicio de descarga y salida en centros de distribución.
- **Gestión de retornos:** Indicador de retorno con o sin carga que impacta la rentabilidad por placa.

## Integración con Planilla y Contabilidad

Automatizaciones administrativas relacionadas con la programación:

- **Cálculo de incentivos:** Pagos variables a conductores basados en viajes y retornos.
- **Valor referencial MTC:** Cálculo automatizado para detracciones del 4%.

## Propuesta de Nuevos Modelos y Campos

Odoo nativo no tiene un objeto de viaje que gestione simultáneamente dos conductores, una unidad tracto, una carreta, retorno con carga, data operativa y validaciones de seguridad. Por ello, el modelo `chanaltin.trip` debe ser el corazón operativo del flujo.

### Diagrama de Clases

```mermaid
classDiagram
    class Trip {
        +char name
        +datetime date_start_planned
        +datetime date_end_planned
        +datetime date_start_real
        +datetime date_end_real
        +selection trip_type
        +selection state
        +float planned_load
        +float actual_load
        +boolean perfect_delivery
        +boolean has_return_load
        +boolean driver_rest_check
        +boolean document_validity_check
        +datetime arrival_cd_time
        +datetime discharge_start_time
        +datetime discharge_end_time
    }

    class Route {
        +char name
        +float distance_km
        +float ref_value_mtc
        +selection zone
        +selection route_type
    }

    class FleetVehicle {
        +char license_plate
        +char internal_code
        +boolean is_trailer
        +char mtc_weight_format
        +float current_engine_hours
        +float odometer
    }

    class Employee {
        +char name
        +selection license_type
        +date license_expiry
        +date sctr_expiry
        +selection last_trip_type
        +datetime last_long_trip_end
    }

    class Checklist {
        +selection type
        +json observations
        +image evidence_photo
        +boolean maintenance_required
        +char signed_by
    }

    class Incident {
        +date date_incident
        +char ticket_number
        +float amount_fine
        +selection severity
        +selection incident_source
        +boolean is_billable_to_client
    }

    Trip --> "1" Route : asignada
    Trip --> "1" FleetVehicle : unidad_tracto
    Trip --> "0..1" FleetVehicle : carreta
    Trip --> "1" Employee : conductor_titular
    Trip --> "0..1" Employee : conductor_asistente
    Trip --> "0..*" Checklist : inspecciones
    Trip --> "0..*" Incident : incidencias
    Employee --> "0..*" Incident : historial_disciplinario
    FleetVehicle --> "0..*" Incident : historial_unidad
```

### Flujo de Validación de Viaje

```mermaid
flowchart TD
    A[Solicitud de viaje] --> B[Crear chanaltin.trip]
    B --> C[Seleccionar ruta]
    C --> D[Asignar tracto, carreta y conductor]
    D --> E{Validar documentos}
    E -- Vencidos --> E1[Bloquear programación]
    E -- Vigentes --> F{Validar descanso}
    F -- No cumple --> F1[Bloquear conductor]
    F -- Cumple --> G{Validar mantenimiento}
    G -- Critico pendiente --> G1[Bloquear unidad]
    G -- Disponible --> H{Viaje tercerizado?}
    H -- Si --> I[Crear orden de compra / servicio]
    H -- No --> J[Confirmar viaje propio]
    I --> K[Programado]
    J --> K[Programado]
    K --> L[Checklist de salida]
    L --> M[En ruta]
    M --> N[Registro de tiempos CD y carga real]
    N --> O[Checklist de retorno]
    O --> P[Cierre y KPIs]
```

### Integración Funcional con Odoo

```mermaid
flowchart LR
    Trip[chanaltin.trip] --> Fleet[fleet.vehicle]
    Trip --> Driver[hr.employee]
    Trip --> Route[chanaltin.route]
    Trip --> Maintenance[maintenance.request]
    Trip --> FSM[project.task / industry_fsm]
    Trip --> Sale[sale.order]
    Trip --> Purchase[purchase.order]
    Trip --> Accounting[account.move.line]

    Fleet --> Odometer[fleet.vehicle.odometer]
    Fleet --> Contracts[fleet.vehicle.log.contract]
    Fleet --> Services[fleet.vehicle.log.services]
    FSM --> Checklist[chanaltin.checklist]
    Accounting --> Analytic[analytic_distribution por placa]
```

## Detalle de Campos Críticos

### A. `chanaltin.trip` (Viaje) - Nuevo

Este modelo concentra la programación, seguimiento y cierre del viaje.

Campos principales:

- `name`: folio de viaje.
- `date_start_planned` / `date_end_planned`: fechas planificadas.
- `date_start_real` / `date_end_real`: fechas reales.
- `trip_type`: corta o larga.
- `state`: borrador, programado, en ruta, retorno, cerrado o cancelado.
- `route_id`: ruta asignada.
- `vehicle_id`: unidad tracto.
- `trailer_id`: carreta.
- `driver_id`: conductor titular.
- `assistant_driver_id`: conductor asistente.
- `planned_load`: capacidad planificada.
- `actual_load`: carga real.
- `perfect_delivery`: indicador de entrega perfecta.
- `has_return_load`: retorno con carga.

Campos de validación:

- `driver_rest_check`: valida si el conductor cumplió descanso tras una ruta larga.
- `document_validity_check`: bloquea salida si SOAT, revisión técnica, brevete o SCTR están vencidos.
- `maintenance_block_check`: bloquea si la unidad tiene mantenimiento crítico abierto.
- `third_party_check`: valida proveedor, unidad y conductor externo cuando el viaje es tercerizado.

Campos de data operativa:

- `arrival_cd_time`: llegada al centro de distribución.
- `discharge_start_time`: inicio de descarga.
- `discharge_end_time`: fin de descarga.
- `return_start_time`: inicio del retorno.
- `return_end_time`: fin del retorno.
- `dead_time_hours`: tiempo muerto calculado.

### B. `chanaltin.route` (Ruta) - Nuevo

Campos principales:

- `name`: origen - destino.
- `origin_id` / `destination_id`: puntos operativos.
- `distance_km`: distancia estimada.
- `ref_value_mtc`: valor referencial MTC.
- `zone`: norte, sur, centro u otra clasificación operativa.
- `route_type`: corta o larga.
- `default_viatic_amount`: viático sugerido.
- `expected_transit_hours`: horas esperadas de tránsito.

### C. Extensiones a `fleet.vehicle`

Odoo ya aporta placa, odómetro, conductor actual, contratos y servicios. Chanaltín debe extenderlo con campos operativos para transporte pesado:

- `internal_code`: código interno de Chanaltín.
- `is_trailer`: diferencia tractos de carretas.
- `vehicle_operation_type`: tracto, carreta, montacargas u otro activo operativo.
- `mtc_weight_format`: formato de pesos y medidas.
- `current_engine_hours`: horómetro actualizado por GPS, clave para montacargas.
- `soat_expiry`: vencimiento SOAT.
- `technical_review_expiry`: vencimiento de revisión técnica.
- `operational_status`: disponible, programado, en ruta, mantenimiento, bloqueado.
- `analytic_account_id`: cuenta analítica por placa o centro de costo.

Consideración: para kilometraje debe reutilizarse `fleet.vehicle.odometer`. Para costos de servicios existentes debe evaluarse `fleet.vehicle.log.services` antes de crear modelos paralelos.

### D. Extensiones a `hr.employee`

El conductor debe ser empleado cuando pertenece a Chanaltín. Para terceros, se puede evaluar `res.partner` o una extensión de homologación vinculada al proveedor.

Campos propuestos:

- `is_driver`: identifica empleados conductores.
- `license_type`: categoría de brevete.
- `license_expiry`: vencimiento de licencia.
- `sctr_expiry`: vencimiento de seguro complementario.
- `last_trip_type`: último viaje corto o largo.
- `last_long_trip_end`: fecha/hora de cierre del último viaje largo.
- `driver_status`: disponible, descansando, bloqueado, de vacaciones, suspendido.
- `incident_count`: contador de incidencias abiertas o relevantes.

### E. `chanaltin.checklist` (Checklist) - Nuevo o Integrado con FSM

Para checklist simple se puede crear modelo propio. Si requiere ejecución móvil, firma, evidencia y reporte al cliente, conviene integrarlo con `project.task` de `industry_fsm`.

Campos:

- `trip_id`: viaje asociado.
- `vehicle_id`: unidad inspeccionada.
- `driver_id`: responsable de la inspección.
- `type`: salida, retorno, entrega, recepción o auxilio.
- `observations`: JSON con luces, llantas, frenos, documentos, carrocería, EPP y otros controles.
- `evidence_photo`: evidencia fotográfica.
- `maintenance_required`: dispara solicitud a mantenimiento.
- `maintenance_request_id`: vínculo a `maintenance.request`.
- `fsm_task_id`: vínculo a `project.task` si se usa Field Service.

### F. `chanaltin.incident` (Incidencias/Papeletas) - Nuevo

Modelo para digitalizar papeletas, incidencias operativas, sanciones y responsabilidades económicas.

Campos:

- `vehicle_id`: placa involucrada.
- `driver_id`: conductor responsable.
- `trip_id`: viaje relacionado.
- `date_incident`: fecha de incidencia.
- `ticket_number`: papeleta o número de expediente.
- `amount_fine`: importe de multa.
- `severity`: baja, media, alta o crítica.
- `incident_source`: SUTRAN, SAT, municipal, cliente, taller u operación interna.
- `is_billable_to_client`: indica si debe reembolsarse por cliente.
- `state`: borrador, en revisión, apelado, confirmado, cerrado.

## Consideraciones de Lógica

### Validación Documental

Antes de pasar un viaje a `programado`, el sistema debe validar documentos del conductor y de la unidad:

- Brevete y SCTR desde `hr.employee`.
- SOAT, revisión técnica, formato MTC y documentos del activo desde `fleet.vehicle`.
- Para terceros, documentos del proveedor, conductor externo y unidad homologada.

Si un documento está vencido, el viaje debe quedar bloqueado con actividad automática para el responsable.

### Regla de Descanso

La regla mínima propuesta:

- Si `last_trip_type = larga`, el conductor no puede tomar otro viaje largo hasta cumplir la ventana de descanso definida por operación.
- La regla debe considerar conductor titular y asistente.
- Al cerrar un viaje largo, actualizar `last_trip_type` y `last_long_trip_end` en `hr.employee`.

### Bloqueo por Mantenimiento

El viaje debe consultar mantenimiento antes de confirmar:

- Si existe `maintenance.request` crítica abierta para la unidad, bloquear programación.
- Si el checklist marca `maintenance_required`, crear o vincular una solicitud de mantenimiento.
- Para montacargas, usar `current_engine_hours` y reglas por horas, por ejemplo mantenimiento cada 250h.
- Para tractos, usar odómetro de `fleet.vehicle.odometer` y alertas por km.

### Flujo Propio vs. Tercerizado

- Viaje propio: asigna unidad, carreta y conductores internos; genera costos internos y analítica por placa.
- Viaje tercerizado: genera o vincula `purchase.order` para el proveedor transportista y conserva margen contra la venta.
- En ambos casos, el viaje debe permitir medir rentabilidad por ruta, placa, cliente y tipo de operación.

### Analítica y Rentabilidad

La rentabilidad debe apoyarse en distribución analítica:

- Ingresos desde `sale.order` y factura de cliente.
- Costos de terceros desde `purchase.order` y factura de proveedor.
- Costos internos desde combustible, mantenimiento, viáticos y servicios.
- Distribución por placa, ruta o centro de costo usando `analytic_distribution` en líneas contables cuando aplique.

## Criterio de Diseño

El sistema debe consultar automáticamente el estado del conductor en `hr.employee` y de la unidad en `fleet.vehicle` al programar un viaje. La meta es garantizar que la operación sea segura, documentada, rentable y trazable antes de que el camión salga a ruta.
