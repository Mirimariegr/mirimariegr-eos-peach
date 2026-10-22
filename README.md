# EOS Peach · Extracción de datos de contratos con Peach

Variante de [EOS](https://github.com/Mirimariegr/EOS) que procesa los contratos con la API de **Peach** en lugar de Claude.

**Web:** https://mirimariegr.github.io/mirimariegr-eos-peach/ (GitHub Pages desde `main`, carpeta raíz)

## Cómo funciona

1. La página envía cada archivo al webhook del workflow de n8n **«EOS Peach · Documento a Google Sheet»** (`/webhook/eos-peach`).
2. n8n llama a Peach (`https://dev.api.peachwisdom.com`):
   - `POST /v1/documents/presign` con `pipelineId` y `documentTypeId` → URL prefirmada de S3.
   - `PUT` del archivo a esa URL.
   - `GET /v1/documents/{id}` cada 5 s hasta que el estado es `completed` (máx. 5 min).
3. Los `extractedData` de Peach se convierten en **una fila por interviniente** en la pestaña **«Peach»** del Sheet:

| NOMBRE PDF | DNI/NIE | NOMBRE INTERVINIENTE | DIRECCION PARTICULAR COMPLETA | POBLACION | PROVINCIA | PAIS | CODIGO POSTAL |
| --- | --- | --- | --- | --- | --- | --- | --- |

La web muestra los intervinientes, el tiempo de procesado en Peach y los datos originales devueltos por Peach.

## Configuración

- URL base, `pipelineId` y `documentTypeId`: al principio del nodo «Normalizar archivo y config Peach» del workflow.
- Token: credencial **Peach API** de n8n (tipo Header Auth, header `Authorization`, valor = token sin `Bearer`).

---

# ACartoes · Documentos de tarjetas (carpeta `acartoes/`)

**Web:** https://mirimariegr.github.io/mirimariegr-eos-peach/acartoes/

Copia de la web anterior para los documentos de tarjetas de novobanco (Alterações / Pedidos, Pagamentos, Reclamações).

1. En el navegador: abre el TIF (o PDF/imagen), lee los códigos de barras de cada página (ITF de 20 dígitos = **CB**, Code 39 `SEPnn` = **ETIQUETA**) y convierte el TIF a PDF, porque Peach no acepta TIF.
2. Envía el PDF y los códigos al webhook `/webhook/acartoes` del workflow de n8n **«ACartoes · Peach + código de barras a Google Sheet»** (campos `file`, `cb`, `etiqueta`, `codigos`, `nombre_original`).
3. n8n procesa el documento con Peach y añade una fila a la pestaña **«ACartoes»** del Sheet:

| NOMBRE PDF | URL DOCUMENTO | TIPO | CB | DATACB | DATA | ETIQUETA | NCONTA | NCARTAO | NCONTACARTAO | FUENTE CB | OTROS DATOS |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

- **CB / ETIQUETA**: los del código de barras leído en la web; si no se pueden leer, los de Peach (`FUENTE CB` indica cuál).
- **DATACB**: dígitos 9-14 del CB en formato DDMMAA (`0007 0493 210926 007361` → `21/09/2026`).
- **TIPO**: por la etiqueta (SEP2 Alterações Pedidos, SEP14 Pagamentos, SEP27 Reclamações) o por el código de formulario del CB (1429 / 0493 / 0656).
- **NCARTAO** y **NCONTACARTAO** se devuelven siempre por separado.
- **OTROS DATOS**: el resto de lo que devuelva Peach, en JSON.

Configuración: `PIPELINE_ID` y `DOCUMENT_TYPE_ID` en el nodo «Normalizar archivo y config Peach» del workflow ACartoes.
