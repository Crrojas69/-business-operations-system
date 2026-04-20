# Power Automate - Guía de Conexiones

Este documento proporciona instrucciones paso a paso para configurar las conexiones necesarias en los flujos de Power Automate del Sistema de Operaciones Comerciales.

## 🔌 Conexiones Requeridas

### 1. Excel Online (Empresas)
**Propósito**: Acceder y actualizar tablas en el workbook de Excel alojado en OneDrive

#### Pasos de Configuración:
1. Abre Power Automate
2. Haz clic en "Mi flujo" → selecciona el flujo
3. Haz clic en "Editar"
4. En el paso que dice "Obtener una fila de tabla" o "Agregar una fila a tabla"
5. En el campo "Ubicación", selecciona tu sitio de OneDrive
6. En el campo "Biblioteca de documentos", selecciona "Documents"
7. En el campo "Archivo", busca y selecciona tu archivo Excel (ej: `OperacionesComerciales.xlsx`)
8. En el campo "Tabla", selecciona la tabla correspondiente

#### Tablas Disponibles:
- **Ventas**: Registros de transacciones de venta
- **Compras**: Órdenes de compra a proveedores
- **Inventario**: Estado actual de inventario
- **Productos**: Catálogo de productos
- **Movimientos Inventario**: Historial de movimientos de stock
- **Estacionamiento**: Registros de ingresos por estacionamiento
- **RegistroBaños**: Datos operacionales de baños
- **RegistroEducacion**: Actividades educativas/ambientales

### 2. Office 365 Outlook
**Propósito**: Enviar notificaciones por email cuando ocurren eventos

#### Pasos de Configuración:
1. En el paso "Enviar correo electrónico"
2. Haz clic en "Más opciones" para expandir
3. Configurar campos:
   - **Para**: Email del destinatario (ej: admin@universidad.edu)
      - **Asunto**: Mensaje descriptivo (ej: "Error en Flujo de Ventas")
         - **Cuerpo**: HTML o texto con detalles del evento
         4. Puedes usar variables dinámicas del flujo con el botón "Agregar contenido dinámico"

         #### Ejemplo de Email:
         ```html
         <h2>Notificación de Venta Registrada</h2>
         <p><strong>Referencia:</strong> @{outputs('Obtener_Fila')?['body/ID']}</p>
         <p><strong>Monto:</strong> @{triggerBody()['Precio']}</p>
         <p><strong>Fecha:</strong> @{utcNow()}</p>
         ```

         ### 3. Power Apps
         **Propósito**: Recibir datos del formulario cuando el usuario completa una entrada

         #### Configuración Automática:
         - La mayoría de flujos tienen el trigger "Cuando Power Apps llama a un flujo (V2)" configurado automáticamente
         - Este trigger se configura cuando Power Apps invoca el flujo

         #### Verificar Conexión:
         1. Ve al flujo en Power Automate
         2. Mira el trigger "Cuando Power Apps llama a un flujo"
         3. Debería mostrar parámetros que coincidan con los campos del formulario

         ## ✅ Verificación de Conexiones

         ### Checklist Post-Configuración:
         - [ ] El archivo Excel está accesible en OneDrive
         - [ ] Todas las tablas requeridas existen en el workbook
         - [ ] Los nombres de las tablas coinciden exactamente
         - [ ] La cuenta de Office 365 está autenticada
         - [ ] Los emails de notificación son válidos
         - [ ] Los flujos pueden guardar cambios sin errores de conexión

         ### Prueba de Conexión:
         1. Haz clic en "Probar" en Power Automate
         2. Completa los parámetros de entrada con datos de prueba
         3. Haz clic en "Ejecutar flujo"
         4. Verifica que:
            - El flujo se ejecute sin errores
               - Los datos aparezcan en Excel
                  - Se envíe un email si está configurado

                  ##  Troubleshooting Común

                  ### Error: "No se encuentra el archivo"
                  **Solución**: 
                  - Verifica que el archivo Excel está en OneDrive
                  - Asegúrate de que el nombre del archivo es exacto
                  - Comprueba permisos de acceso al archivo

                  ### Error: "Tabla no encontrada"
                  **Solución**:
                  - Confirma que la tabla existe en Excel
                  - Verifica que el nombre de la tabla es exacto (Excel es sensible a mayúsculas)
                  - Asegúrate de que la tabla tiene encabezados

                  ### Error: "No autorizado"
                  **Solución**:
                  - Reconecta la conexión de Outlook
                  - Verifica que tu cuenta tiene permisos de envío de email
                  - Prueba con una dirección de email diferente

                  ### Error: "El flujo no recibe parámetros"
                  **Solución**:
                  - Verifica que Power Apps está invocando el flujo correctamente
                  - Revisa que los nombres de parámetros coinciden en Power Apps y Power Automate
                  - Prueba el flujo manualmente desde Power Automate

                  ## 🔐 Mejores Prácticas de Seguridad

                  1. **Credenciales**: Nunca hardcodees emails o contraseñas
                  2. **Permisos**: Usa cuentas con permisos mínimos necesarios
                  3. **Auditoría**: Registra acciones importantes en Excel
                  4. **Validación**: Valida todos los datos de entrada
                  5. **Notificaciones**: Notifica sobre errores críticos inmediatamente

                  ## 📊 Matriz de Conexiones por Flujo

                  | Flujo | Excel | Outlook | Power Apps |
                  |-------|-------|---------|------------|
                  | RegistroVentasInventario | ✓ | ✓ | ✓ |
                  | RegistrodeCompras | ✓ | ✓ | ✓ |
                  | MOVInventario | ✓ | ✓ | ✓ |
                  | RegistroEstacionamiento | ✓ | ✓ | ✓ |
                  | RegistroBaños | ✓ | ✓ | ✓ |
                  | AgregarProducto | ✓ | ✓ | ✓ |
                  | Educación | ✓ | ✓ | ✓ |

                  ## 🔗 Links Útiles

                  - [Documentación Power Automate](https://learn.microsoft.com/en-us/power-automate/)
                  - [Conector Excel Online](https://learn.microsoft.com/en-us/connectors/excelonline/)
                  - [Conector Outlook](https://learn.microsoft.com/en-us/connectors/outlook/)
                  - [Power Apps to Flow](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-powerbi)

                  ---
                  **Última actualización**: 2025-04-20  
                  **Versión**: 1.0  
                  **Contacto**: Cristopher Eduardo Rojas Carrasco
