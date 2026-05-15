# Burgas Proveedores

Aplicacion mobile-first para cargar facturas y pagos de proveedores en Google Sheets a partir de una foto o imagen.

## Que Resuelve

- Un empleado saca una foto o sube una imagen de una factura, remito o comprobante de pago.
- Gemini extrae proveedor, CUIT, numero, fecha, tipo y monto.
- La app muestra un chequeo previo con los datos clave para revisar antes de confirmar.
- Al confirmar, se inserta una fila en la hoja del proveedor correspondiente con saldo corrido automatico.
- La imagen original se archiva en Google Drive.
- El registro queda tambien en la hoja interna `Facturas` para auditoria.

## Diferencia Con burgas-facturas

Esta app complementa a `burgas-facturas`. Mientras aquella maneja el stock de ingredientes,
esta maneja la cuenta corriente de cada proveedor: facturas, remitos, pagos, e-cheques y saldos.

| | burgas-facturas | burgas-proveedores |
|---|---|---|
| Que guarda | Facturas con items de stock | Movimientos de cuenta corriente |
| Destino en Sheets | Hojas de STOCK | Hojas por proveedor |
| Saldo corrido | No | Si, formula automatica |
| Tipos de comprobante | Factura | Factura, Remito, Pago, E-cheq |
| Chequeo previo | No | Si |

## Arquitectura

```
Celular / navegador
  -> Web App de Google Apps Script
  -> Gemini API
  -> Google Sheets (hoja del proveedor)
  -> Google Drive (imagen)
```

El frontend esta en `index.html` y el backend esta en `apps-script.gs`.

La app debe abrirse desde la URL `/exec` publicada por Apps Script. No se recomienda usar `localhost` para el flujo real porque Google Apps Script no habilita CORS para ese uso.

## Archivos Principales

- `index.html`: UI mobile para sacar foto, revisar datos con chequeo previo y guardar.
- `apps-script.gs`: backend de Apps Script; llama a Gemini, inserta en la hoja del proveedor y archiva la imagen.
- `CONFIGURACION_CLIENTE.md`: guia paso a paso para configurar la app en la planilla real.
- `AGENTS.md`: notas tecnicas para futuras sesiones de desarrollo.

## Google Sheets

La planilla debe tener una hoja por proveedor con esta estructura:

```
Fila 1: titulo (TheBurgas — Cuenta Corriente Proveedor)
Fila 2: nombre del proveedor
Fila 3: saldo inicial en columna E
Fila 4: vacia
Fila 5: encabezados de columnas
Fila 6+: datos de movimientos
```

Las columnas de datos son:

```
A: Fecha
B: Tipo (Factura, Remito, Pago, E-cheq)
C: N° Comprobante
D: Descripcion
E: Compra ($)
F: Pago ($)
G: Saldo ($)  <- formula automatica
H: Observaciones
```

La app busca la primera fila vacia desde la fila 6 e inserta ahi el nuevo movimiento.
La columna G se completa con una formula que toma el saldo anterior y suma o resta segun sea compra o pago.

Los proveedores conocidos y sus hojas correspondientes son:

```
AUGUSTO CARNES      -> AUGUSTO CARNES
SALVADOR SGRO       -> SALVADOR SGRO
LAS DINAS           -> LAS DINAS
SILVIO QUESOS       -> SILVIO QUESOS
OLLARI              -> OLLARI
MACOHUE             -> MACOHUE
DEL CAMPO PANES     -> DEL CAMPO PANES
CONURBANO LOGISTICA -> Conurbano bebidas
PROVEEDOR 10        -> Proveedor 10
FACUNDO QUESOS      -> facundo quesos
```

Para agregar un proveedor nuevo, agregar la entrada en `SHEET_MAP` dentro de `apps-script.gs` y publicar una nueva version.

## Chequeo Previo

Antes de guardar, la app muestra un banner amarillo con los datos que Gemini extrajo:

- Proveedor detectado (con chips para confirmar o cambiar)
- Tipo de comprobante
- Numero de comprobante
- Fecha
- Monto total

Si algo esta mal, el empleado lo corrige en el formulario antes de tocar Guardar. El banner se actualiza en tiempo real.

## Hojas Internas

La app crea y mantiene estas hojas para auditoria:

```
Facturas
Logs
```

Despues de guardar, el script las oculta para que el cliente trabaje principalmente con las hojas de proveedores.

## Configuracion Rapida

1. Abrir el Google Sheet de proveedores.
2. Ir a `Extensiones -> Apps Script`.
3. Pegar `apps-script.gs` en `Code.gs`.
4. Crear un archivo HTML llamado exactamente `index`.
5. Pegar el contenido de `index.html` en ese archivo HTML.
6. Configurar `Script Properties`:

```
GEMINI_API_KEY=tu_api_key_de_gemini
SPREADSHEET_ID=id_del_sheet_de_proveedores
```

Opcional:

```
DRIVE_FOLDER_ID=id_de_la_carpeta_de_drive
GEMINI_MODEL=gemini-2.5-flash
GEMINI_FALLBACK_MODELS=gemini-2.0-flash,gemini-1.5-flash
```

7. Publicar como Web App:

```
Deploy -> New deployment -> Web app
Execute as: Me
Who has access: Anyone
```

8. Abrir la URL generada que termina en `/exec`.

Para la guia completa, ver `CONFIGURACION_CLIENTE.md`.

## Uso Diario

1. Abrir la URL `/exec` desde el celular.
2. Tocar `Sacar foto` o `Subir imagen`.
3. Tocar `Extraer datos`.
4. Revisar el chequeo previo: proveedor, tipo, numero, fecha y monto.
5. Corregir cualquier dato mal leido.
6. Tocar `Guardar en planilla`.
7. Verificar el mensaje de confirmacion.

## Notas Operativas

- El proveedor debe coincidir con alguno de los conocidos para que la hoja se encuentre automaticamente.
- Si Gemini detecta mal el proveedor, usar los chips para elegir el correcto antes de guardar.
- Si es una factura o remito, completar el campo Compra. Si es un pago o e-cheq, completar el campo Pago.
- Si Gemini esta saturado, el script reintenta automaticamente y usa modelos fallback.
- La API key de Gemini nunca debe ir en `index.html`; va en Script Properties.
- La imagen se guarda en Drive con el ID interno del registro para trazabilidad.

## Desarrollo Local

No hay build, package manager ni test runner.

Para visualizar el HTML localmente:

```
python3 -m http.server 8000
```

Pero el flujo real debe probarse desde Apps Script (`/exec`) para evitar problemas de CORS.
