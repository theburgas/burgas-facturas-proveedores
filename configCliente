# Configuracion de Carga de Facturas de Proveedores

Este documento explica como configurar la aplicacion para que cualquier empleado pueda sacar una foto de una factura, remito o comprobante de pago desde el celular, extraer los datos con Gemini y guardarlos directamente en la hoja del proveedor en Google Sheets.

## Que Hace La Aplicacion

La aplicacion permite:

- Sacar o subir una foto de una factura, remito, pago o e-cheque desde el celular.
- Leer el comprobante automaticamente con Gemini.
- Mostrar un chequeo previo con los datos clave para revisar antes de confirmar.
- Guardar el movimiento en la hoja del proveedor correspondiente con saldo corrido automatico.
- Guardar la imagen original en Google Drive.
- Registrar cada movimiento en la hoja interna `Facturas` para auditoria.

El empleado no carga el comprobante manualmente: solo revisa los datos extraidos y confirma.

## Requisitos

Necesitas:

- Una cuenta de Google.
- El Google Sheet de proveedores de TheBurgas.
- Acceso a Google Apps Script (incluido en toda cuenta de Google).
- Una API key de Gemini desde Google AI Studio (gratis para empezar).
- El archivo `index.html` de la aplicacion.
- El archivo `apps-script.gs` con el codigo del backend.

## Paso 1: Preparar La Planilla De Proveedores

1. Abrir la planilla `Proveedores_TheBurgas` en Google Drive.
2. Si todavia es un archivo `.xlsx`, abrirlo con Google Sheets y convertirlo:

```
Archivo -> Guardar como Hojas de calculo de Google
```

3. Confirmar que la planilla convertida sea la que se va a usar en produccion.
4. Copiar el ID de la URL para usar en el paso de configuracion.

Ejemplo de URL:

```
https://docs.google.com/spreadsheets/d/1ABCDEF123456789/edit
```

El ID es:

```
1ABCDEF123456789
```

La planilla debe tener una hoja por proveedor. Los nombres de hoja que reconoce la app son:

```
AUGUSTO CARNES
SALVADOR SGRO
LAS DINAS
SILVIO QUESOS
OLLARI
MACOHUE
DEL CAMPO PANES
Conurbano bebidas
Proveedor 10
facundo quesos
```

Cada hoja debe tener esta estructura (que ya tienen las hojas de la planilla original):

```
Fila 1: titulo
Fila 2: nombre del proveedor
Fila 3: saldo inicial en columna E
Fila 4: vacia
Fila 5: encabezados (Fecha, Tipo, N° Comprobante, Descripcion, Compra, Pago, Saldo, Observaciones)
Fila 6+: datos de movimientos
```

No hace falta crear las hojas internas manualmente. El sistema las crea automaticamente cuando se guarda el primer registro.

## Paso 2: Crear El Apps Script

1. Abrir el Google Sheet de proveedores.
2. Ir a:

```
Extensiones -> Apps Script
```

3. Borrar cualquier codigo inicial que aparezca en `Code.gs`.
4. Pegar en `Code.gs` el contenido completo del archivo:

```
apps-script.gs
```

5. Crear un archivo HTML en Apps Script:

```
+ -> HTML
```

6. Ponerle este nombre exacto:

```
index
```

7. Pegar dentro de ese archivo HTML el contenido completo de:

```
index.html
```

8. Guardar el proyecto con `Ctrl+S` o el icono de guardar.

## Paso 3: Crear La API Key De Gemini

1. Entrar a Google AI Studio:

```
https://aistudio.google.com/app/apikey
```

2. Hacer clic en `Create API key`.
3. Elegir el proyecto de Google Cloud o crear uno nuevo.
4. Copiar la API key generada.

Importante: no pegar esta clave en el archivo `index.html`. La clave va solamente en Script Properties de Google Apps Script.

## Paso 4: Configurar Las Propiedades Del Script

En Google Apps Script:

1. Ir a `Project Settings` (icono de engranaje).
2. Bajar hasta la seccion `Script Properties`.
3. Hacer clic en `Add script property`.
4. Agregar las propiedades obligatorias:

```
GEMINI_API_KEY    -> pegar aca la API key de Gemini
SPREADSHEET_ID   -> pegar aca el ID de la planilla de proveedores
```

Propiedades opcionales:

```
DRIVE_FOLDER_ID  -> ID de una carpeta de Drive donde guardar las imagenes
```

Si no se configura `DRIVE_FOLDER_ID`, el sistema crea automaticamente una carpeta llamada:

```
Facturas Proveedores Burgas
```

Para cambiar el modelo de Gemini o los fallbacks:

```
GEMINI_MODEL              -> gemini-2.5-flash  (es el default)
GEMINI_FALLBACK_MODELS    -> gemini-2.0-flash,gemini-1.5-flash
```

Si Gemini responde con error por alta demanda, el script reintenta automaticamente y prueba los modelos de respaldo. Normalmente no hace falta configurar estas propiedades.

## Paso 5: Publicar El Apps Script

En Google Apps Script:

1. Ir a:

```
Deploy -> New deployment
```

2. Elegir tipo:

```
Web app
```

3. Configurar:

```
Execute as: Me
Who has access: Anyone
```

4. Hacer clic en `Deploy`.
5. Aceptar todos los permisos que pida Google (Sheets, Drive, internet).
6. Copiar la URL generada. Debe terminar en:

```
/exec
```

Esa URL es la aplicacion. Compartirla con los empleados que van a cargar facturas.

## Paso 6: Probar Desde Una Computadora

Abrir directamente la URL del Web App que termina en `/exec`:

```
https://script.google.com/macros/s/XXXXXXXXXXXX/exec
```

No probar desde `http://localhost` ni un servidor local para el flujo real. Google Apps Script no agrega headers CORS para ese uso.

Probar el flujo completo:

1. Subir una imagen de prueba de una factura.
2. Tocar `Extraer datos`.
3. Verificar que aparezca el chequeo previo con proveedor, tipo, numero, fecha y monto.
4. Revisar y corregir si algo esta mal.
5. Confirmar el proveedor usando los chips verdes.
6. Tocar `Guardar en planilla`.
7. Abrir el Google Sheet y verificar que se haya insertado una fila en la hoja del proveedor.
8. Verificar que la columna G (Saldo) tenga una formula y no un valor fijo.
9. Verificar que la imagen se haya guardado en Drive.

## Paso 7: Probar Desde Un Celular

Abrir desde el celular la misma URL del Web App que termina en `/exec`.

No hace falta que el celular este en la misma red WiFi, porque la app queda servida por Google Apps Script.

Desde el celular se muestran dos opciones:

- `Sacar foto`: abre la camara trasera del celular directamente.
- `Subir imagen`: permite elegir una imagen ya guardada en la galeria o archivos.

Recomendaciones para la foto:

- Sacar la foto con buena luz, sin sombras sobre la factura.
- Asegurarse de que se vea el numero de comprobante, fecha y total con claridad.
- Evitar fotos movidas o borrosas.
- Si la factura tiene varias hojas, la app procesa solo la imagen cargada.

## Paso 8: Publicar Cambios Nuevos

Cada vez que se cambie `apps-script.gs` o el archivo HTML `index`, hay que publicar una nueva version:

```
Deploy -> Manage deployments -> Edit -> Version -> New version -> Deploy
```

Si no se publica una nueva version, Google sigue sirviendo el codigo anterior.

## Uso Diario

El empleado debe hacer esto al recibir una factura o hacer un pago:

1. Abrir la URL de la aplicacion desde el celular.
2. Tocar `Sacar foto` o `Subir imagen`.
3. Tocar `Extraer datos`.
4. Revisar el chequeo previo:
   - Confirmar que el proveedor sea el correcto usando los chips. El chip en verde fue detectado automaticamente.
   - Verificar el tipo: Factura, Remito, Pago o E-cheq.
   - Verificar el numero de comprobante y la fecha.
   - Verificar el monto total.
5. Corregir cualquier dato mal leido en el formulario de abajo.
6. Completar el campo `Compra` si es una factura o remito.
7. Completar el campo `Pago` si es un pago o e-cheque.
8. Tocar `Guardar en planilla`.

## Hojas Internas

La app usa estas hojas como base de datos y auditoria:

```
Facturas
Logs
```

Estas hojas se ocultan automaticamente despues de guardar, para que el cliente trabaje principalmente con las hojas de proveedores.

## Que Datos Se Guardan

En la hoja del proveedor (por ejemplo `AUGUSTO CARNES`) se inserta una fila con:

- Fecha del comprobante.
- Tipo (Factura, Remito, Pago, E-cheq).
- Numero de comprobante.
- Descripcion o concepto.
- Monto de compra (si aplica).
- Monto de pago (si aplica).
- Saldo corrido calculado con formula automatica.
- Link a la imagen en Drive o CUIT del proveedor.

En `Facturas` se guarda para auditoria:

- ID interno del registro.
- Fecha de carga.
- Proveedor.
- CUIT.
- Tipo de comprobante.
- Numero de comprobante.
- Fecha del comprobante.
- Condicion de venta.
- Monto de compra.
- Monto de pago.
- Observaciones.
- Link a la imagen en Drive.

En `Logs` se guardan todos los eventos y errores para diagnostico.

## Como Funciona El Saldo Corrido

La columna G de cada hoja de proveedor contiene una formula que calcula el saldo automaticamente:

```
= saldo_anterior + compra - pago
```

Si el saldo inicial de la fila 3 esta correcto, todas las filas siguientes calculan el saldo acumulado sin necesidad de ingresarlo manualmente.

Si se necesita corregir el saldo, editar el valor en la celda `E3` de la hoja del proveedor. Las filas de movimientos no deben editarse directamente en la columna G, porque esa columna es formula.

## Agregar Un Proveedor Nuevo

1. Crear la hoja en el Google Sheet con el mismo formato que las hojas existentes:
   - Fila 1: titulo
   - Fila 2: nombre del proveedor
   - Fila 3: saldo inicial en E3
   - Fila 5: encabezados
2. Abrir `apps-script.gs` en Apps Script.
3. Agregar la entrada en `SHEET_MAP`:

```javascript
'NOMBRE EN MAYUSCULAS': 'Nombre exacto de la hoja'
```

Ejemplo:

```javascript
'LA PAMPA CARNES': 'La Pampa Carnes'
```

4. Publicar una nueva version del Apps Script.

## Seguridad

- La API key de Gemini no debe estar en `index.html`.
- La API key debe estar solamente en Script Properties de Google Apps Script.
- El Apps Script se ejecuta con la cuenta configurada en el deploy.
- Las imagenes quedan guardadas en Google Drive de esa cuenta.
- La planilla debe compartirse solo con las personas que necesiten ver o administrar los datos.
- La URL `/exec` es accesible por cualquier persona que la tenga. No publicarla abiertamente si los datos son confidenciales.

## Costos Y Cuotas

La aplicacion no necesita hosting propio.

Usa servicios de Google:

- Google Apps Script: gratuito con limites de uso diario.
- Google Sheets: gratuito.
- Google Drive: gratuito hasta el limite de almacenamiento de la cuenta.
- Gemini API: tiene capa gratuita. Revisar el consumo si se cargan muchas facturas por dia.

Para ver el consumo de Gemini:

```
https://aistudio.google.com/app/apikey
```

## Problemas Frecuentes

### No Extrae Datos

Revisar:

- Que `GEMINI_API_KEY` este configurada en Script Properties.
- Que el deploy del Apps Script sea la version mas reciente.
- Que la imagen sea clara y legible.
- Ver la hoja `Logs` para encontrar el error exacto.

### Proveedor No Reconocido

Gemini puede leer el nombre del proveedor de forma distinta a la que espera la app. En ese caso:

- Usar los chips de la pantalla de revision para seleccionar el proveedor correcto manualmente.
- Si el proveedor aparece muy frecuentemente mal leido, revisar el prompt en `invoicePrompt_()` dentro de `apps-script.gs`.

### Guarda Pero No Aparece En La Hoja Del Proveedor

Revisar:

- Que el nombre del proveedor seleccionado en la app coincida con alguna entrada en `SHEET_MAP`.
- Que la hoja exista en el Google Sheet con el nombre exacto que figura en `SHEET_MAP`.
- Ver la hoja `Logs` para encontrar el error.

### Saldo Incorrecto En La Hoja

Verificar:

- Que la celda `E3` de la hoja del proveedor tenga el saldo inicial correcto.
- Que la columna G no tenga valores sobreescritos a mano (debe ser formula en todas las filas de datos).
- Que los campos Compra y Pago se hayan cargado en las columnas correctas (E y F).

### No Guarda La Imagen En Drive

Revisar:

- Que el deploy tenga permisos de Drive aceptados. Si no, revocar permisos y volver a hacer el deploy.
- Si se usa `DRIVE_FOLDER_ID`, verificar que la carpeta exista y que la cuenta tenga acceso.

### Cambie El Codigo Pero Sigue Igual

Cada cambio en Apps Script requiere publicar una nueva version:

```
Deploy -> Manage deployments -> Edit -> New version -> Deploy
```

## Mantenimiento

Cuando Gemini lea mal facturas de algun proveedor en particular, guardar ejemplos y ajustar el prompt dentro de `apps-script.gs` en la funcion:

```
invoicePrompt_()
```

Despues de cambiar el prompt, publicar una nueva version del Apps Script.

Para ver todos los registros guardados y errores, mostrar la hoja `Logs` desde Google Sheets:

```
Click derecho sobre cualquier hoja -> Mostrar todas las hojas
```
