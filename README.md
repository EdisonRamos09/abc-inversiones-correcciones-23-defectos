# Correcciones de calidad de ABC Inversiones

Este repositorio reúne los parches aplicados al frontend y a la API. Cada parche corresponde a un único commit sobre la versión original. El texto de propuestas del PDF se conserva en `CODIGO_DEL_PDF.txt` para consultar el planteamiento inicial.

| Componente | Versión original | Commit corregido | Código de los cambios |
|---|---|---|---|
| Frontend | `807fca1` | `66df02c` | [Parche del frontend](parches/frontend.patch) |
| Backend | `74b62c1` | `f7cbd6b` | [Parche del backend](parches/backend.patch) |

## 7.1.3 Resolución de problemas comunes

La evaluación original registró nueve fallos en 118 pruebas unitarias. Tres defectos explicaban esos nueve fallos. En el frontend, la revisión de préstamos no distinguía la etapa del inversionista de la administrativa. En la API, las validaciones de pagos admitían montos infinitos y rechazaban fechas válidas con desfase horario. El lector de comprobantes también interpretaba de forma incorrecta algunas referencias, bancos, importes y fechas. Después de aplicar las correcciones, los 118 casos unitarios pasaron con la zona horaria de negocio configurada en America/Lima.

La tabla vincula las incidencias de las cuatro suites y del análisis estático con la modificación realizada. La columna de casos identifica las pruebas de referencia. La evidencia de ejecución se detalla al final.

| Ref. | Problema observado y resolución aplicada | Casos de referencia |
|---|---|---|
| DEF-01 | La falta de credenciales de Google Vision interrumpía el escaneo. La API espera a que el cliente se inicialice y, si falla Vision, analiza la imagen con Tesseract y un modelo instalado localmente. Una indisponibilidad de ambos lectores devuelve 503 y permite completar los datos manualmente. | CP-I62, CP-S32, CP-A12 |
| DEF-02 | Un inversionista aún veía la opción de aprobar después de su revisión. El permiso requiere que esté asignado al préstamo y que el estado sea pendiente de revisión del inversionista. | CP-U89, CP-S21 |
| DEF-03 | Se podía rechazar un préstamo activo. La API restringe la transición a estados revisables y comprueba el estado de nuevo al escribir para detectar cambios concurrentes. | CP-I25 |
| DEF-04 | La aprobación solicitada por un cliente devolvía 400. La ruta comprueba el rol antes de procesar el préstamo y responde 403 cuando corresponde. | CP-I23 |
| DEF-05 | El historial seguía disponible tras la eliminación lógica del préstamo. La consulta excluye préstamos eliminados antes de devolver transacciones. | CP-I26 |
| DEF-06 | El principal y la deuda del cliente incluían préstamos rechazados. El resumen excluye los rechazados y los eliminados. | CP-I27 |
| DEF-07 | El resumen del inversionista omitía abonos de préstamos activos. La suma incluye pagos confirmados vigentes de sus préstamos, incluso antes de su liquidación. | CP-I28 |
| DEF-08 | El límite de intentos de autenticación estaba desactivado. El límite se aplica por defecto y la excepción requiere una variable explícita en modo de pruebas. | CP-I40 |
| DEF-09 | Un administrador podía desactivar su propia cuenta mediante la operación de eliminación. La API compara el usuario autenticado con el destinatario y devuelve 403 si coinciden. | CP-I54 |
| DEF-10 | Un usuario reactivado conservaba la fecha de eliminación. Al reactivarlo se limpia esa fecha en la misma actualización. | CP-I53 |
| DEF-11 | Consultar un usuario con un UUID mal formado producía un error interno. La ruta valida el identificador antes de consultar la base de datos y devuelve 400. | CP-I52 |
| DEF-12 | Un correo ya registrado producía un error genérico. El registro interpreta el conflicto comunicado por Supabase y devuelve 409. | CP-I11 |
| DEF-13 | Se aceptaban montos infinitos y se rechazaban fechas de pago con desfase horario. Los esquemas exigen montos positivos finitos y admiten fechas ISO con desfase. | CP-U53, CP-U54, CP-U56 |
| DEF-14 | El lector perdía referencias con ceros iniciales, omitía bancos, confundía millares con decimales y aceptaba fechas imposibles. El analizador conserva la referencia como texto, reconoce BANBIF y Banco de Crédito, normaliza importes y verifica el calendario. | CP-U35, CP-U37 a CP-U40 |
| DEF-15 | La foto de perfil guardaba una URL temporal del navegador. La edición utiliza la URL devuelta por la API, libera la vista previa y vuelve a consultar el perfil. | CP-S38 |
| DEF-16 | El detalle de una transacción recibía campos con nombres distintos a los que usaba la interfaz para comprobar la propiedad del voucher. La API devuelve los campos en camelCase para que los controles existentes evalúen el cliente y el estado. | CP-S31 |
| DEF-17 | El aviso de vouchers por revisar siempre recibía una lista vacía. El detalle del préstamo incluye el número real de comprobantes pendientes del inversionista. El botón lleva a la sección de pagos. | CP-S40 |
| DEF-18 | La bandeja administrativa mezclaba pendientes e historial. La API filtra los comprobantes que esperan decisión administrativa antes de paginarlos. El contador y el listado utilizan los mismos estados. | CP-S41 |
| DEF-19 | El indicador de mora del inversionista estaba fijo en cero. El portafolio cuenta los préstamos en incumplimiento o con cuotas vencidas y la interfaz muestra ese resultado. | CP-S43 |
| DEF-20 | Una cuenta inactiva recibía un aviso genérico. El backend distingue la desactivación y el formulario presenta la causa resuelta por la API. | CP-S04 |
| DEF-21 | La ruta de acceso, las consultas administrativas y la pestaña de solicitudes ignoraban el contexto de navegación. Una sesión existente redirige al inicio, las consultas comprueban el rol y la ruta de solicitudes tiene prioridad sobre la preferencia guardada. | CP-S09, CP-S10, CP-S16 |
| DEF-22 | Al buscar un préstamo, la lista conservaba el número de página anterior. Cambiar el término lleva la paginación a la primera página. | CP-S23 |
| DEF-23 | ESLint detectaba dos conversiones a `any` y una función sin uso en la API. Se eliminaron ambas causas. En el frontend se distribuyeron las dependencias de React y animación en archivos separados. El archivo principal pasó de 654,64 a 387,72 kB. | ESLint y compilación de producción |

### Evidencia de verificación

- Pruebas unitarias de [qa-tests](https://github.com/iZeroKey/qa-tests/tree/15be9d0d72ec87ed581f87e8055f380d15d0a459). En la repetición sobre las versiones originales pasaron 109 de 118 y fallaron los nueve casos documentados. En los commits corregidos pasaron 118 de 118, distribuidos en 19 archivos. Se ejecutaron las pruebas originales con `TZ` y `APP_TIMEZONE` configuradas en `America/Lima`.
- Frontend. ESLint y la compilación de producción finalizaron correctamente. Ningún archivo JavaScript generado superó 500 kB. La pantalla de inicio de sesión de esa compilación se abrió en Chrome sin excepciones de JavaScript.
- Backend. ESLint, la comprobación de tipos y la compilación finalizaron correctamente.
- OCR. Una comprobación HTTP aislada del controlador con la imagen de QA `voucher-small.png` devolvió 200, el proveedor `tesseract` y el objeto `extracted` sin credenciales de Google. Se ejercitó el análisis real de la imagen. La prueba aislada omitió autenticación y no certifica la exactitud de todos los campos ni el flujo completo.
- Queda pendiente repetir las 65 pruebas de integración, 47 de sistema y 18 de aceptación en un entorno de QA con PostgreSQL, Supabase y usuarios de prueba. Los casos citados en esas suites identifican qué verificar antes de declarar el cierre de cada incidencia.
