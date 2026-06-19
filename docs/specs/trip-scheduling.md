# 1. Sistema Avanzado de Programación de Viajes (Trip Scheduling)

El Sistema Avanzado de Programación de Viajes para Chanaltín centraliza la planificación, validación de seguridad y control de costos. Su objetivo es reemplazar el uso de Excel y procesos manuales por un flujo integrado en Odoo v18.

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

