# Power Automate Workflows - Sistema de Operaciones Comerciales

Este directorio contiene la documentación completa de los 7 flujos de trabajo de Power Automate que orquestan el Sistema de Operaciones Comerciales (Prototipo PPL).

## 📋 Descripción General

Los flujos de Power Automate son responsables de:
- **Captura de datos**: Reciben datos de Power Apps cuando un usuario completa un formulario
- **Validación**: Aplican reglas de negocio y validaciones de integridad
- **Transformación**: Convierten datos de entrada en el formato requerido por Excel
- **Almacenamiento**: Escriben registros en las tablas normalizadas de Excel en OneDrive
- **Notificaciones**: Envían alertas por email en caso de errores o eventos críticos

## 🔄 Arquitectura de Flujos

```
Power Apps (Entrada)
        ↓
        Power Automate (Orquestación)
                ├─ Validación de datos
                        ├─ Transformación
                                └─ Manejo de errores
                                        ↓
                                        Excel on OneDrive (Persistencia)
                                                ↓
                                                Power BI (Consumo y análisis)
                                                ```

                                                ## 📁 Estructura de Carpetas

                                                ```
                                                power-automate/
                                                ├── README.md (este archivo)
                                                ├── CONNECTIONS_GUIDE.md (configuración de conexiones)
                                                ├── flows/ (documentación de flujos)
                                                │   ├── 01-RegistroVentasInventario.md
                                                │   ├── 02-RegistrodeCompras.md
                                                │   ├── 03-MOVInventario.md
                                                │   ├── 04-RegistroEstacionamiento.md
                                                │   ├── 05-RegistroBaños.md
                                                │   ├── 06-AgregarProducto.md
                                                │   └── 07-Educacion.md
                                                ```

                                                ## 📊 Resumen de los 7 Flujos Principales

                                                ### 1️⃣ RegistroVentasInventario
                                                **Propósito**: Registrar transacciones de ventas e inventario  
                                                **Disparador**: Power Apps (cuando se completa el formulario de venta)  
                                                **Tabla destino**: Ventas en Excel  
                                                **Acciones**: Validación de producto, registro de movimiento, actualización de inventario

                                                ### 2️⃣ RegistrodeCompras
                                                **Propósito**: Registrar órdenes de compra a proveedores  
                                                **Disparador**: Power Apps (cuando se crea una nueva compra)  
                                                **Tabla destino**: Compras en Excel  
                                                **Acciones**: Validación de proveedor, búsqueda de precios, creación de registro

                                                ### 3️⃣ MOVInventario
                                                **Propósito**: Procesar movimientos y ajustes de inventario  
                                                **Disparador**: Power Apps (ajustes manuales de stock)  
                                                **Tabla destino**: MovimientosInventario en Excel  
                                                **Acciones**: Validación de cantidad, actualización de saldos, auditoría

                                                ### 4️⃣ RegistroEstacionamiento
                                                **Propósito**: Registrar ingresos por servicios de estacionamiento  
                                                **Disparador**: Power Apps (entrada de vehículos)  
                                                **Tabla destino**: Estacionamiento en Excel  
                                                **Acciones**: Cálculo de tarifa, generación de comprobante, registro de ingreso

                                                ### 5️⃣ RegistroBaños
                                                **Propósito**: Registrar datos operacionales de baños  
                                                **Disparador**: Power Apps (checklist diario de baños)  
                                                **Tabla destino**: RegistroBaños en Excel  
                                                **Acciones**: Validación de métricas, registro de limpieza, alertas de mantenimiento

                                                ### 6️⃣ AgregarProducto
                                                **Propósito**: Agregar nuevos productos al catálogo  
                                                **Disparador**: Power Apps (creación de producto)  
                                                **Tabla destino**: Productos en Excel  
                                                **Acciones**: Generación de SKU, validación de unicidad, categorización

                                                ### 7️⃣ Educación
                                                **Propósito**: Registrar actividades educativas y de sostenibilidad ambiental  
                                                **Disparador**: Power Apps (evento educativo)  
                                                **Tabla destino**: RegistroEducacion en Excel  
                                                **Acciones**: Validación de participantes, registro de asistencia, métricas de impacto

                                                ## 🔌 Conexiones Requeridas

                                                - **Excel Online (Empresas)**: Conexión a libro Excel en OneDrive
                                                - **Office 365 Outlook**: Para notificaciones por email
                                                - **Power Apps**: Para recibir datos de formularios

                                                Consulta `CONNECTIONS_GUIDE.md` para instrucciones de configuración.

                                                ## 🔒 Seguridad y Auditoría

                                                - Todos los flujos registran timestamp de ejecución
                                                - Se validan entradas contra esquema definido en DATABASE_DESIGN.md
                                                - Los errores se notifican por email al administrador
                                                - Cada registro incluye usuario que realizó la operación (desde Power Apps)

                                                ## 📈 Monitoreo y Mantenimiento

                                                ### Verificación Semanal
                                                - Revisar el historial de ejecuciones en Power Automate
                                                - Identificar flujos fallidos y revisar logs de error
                                                - Validar que los datos lleguen correctamente a Excel

                                                ### Verificación Mensual
                                                - Auditar integridad referencial entre tablas
                                                - Validar que los cálculos de Power BI coincidan con Excel
                                                - Revisar rendimiento de flujos (tiempo de ejecución)

                                                ## 🆘 Troubleshooting

                                                ### El flujo falla al intentar agregar fila a Excel
                                                1. Verificar que el archivo Excel esté abierto en OneDrive
                                                2. Validar que la tabla existe y tiene la estructura correcta
                                                3. Revisar permisos de acceso al archivo

                                                ### Los datos no aparecen en Power BI
                                                1. Verificar que el flujo completó exitosamente
                                                2. Revisar conexión DirectQuery en Power BI
                                                3. Actualizar dataset en Power BI manualmente

                                                ### Errores de validación en Power Apps
                                                1. Revisar que los campos requeridos estén completos
                                                2. Validar formatos de entrada (fechas, números, etc.)
                                                3. Consultar mensaje de error específico en Power Automate

                                                ## 📚 Documentación Relacionada

                                                - [DATABASE_DESIGN.md](../DATABASE_DESIGN.md) - Esquema de tablas Excel
                                                - [ARCHITECTURE.md](../ARCHITECTURE.md) - Arquitectura general del sistema
                                                - [DEPLOYMENT.md](../DEPLOYMENT.md) - Guía de implementación

                                                ---
                                                **Última actualización**: 2025-04-20  
                                                **Versión**: 1.0.1  
                                                **Autor**: Cristopher Eduardo Rojas Carrasco
