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

Procesa documentos de tarjetas de novobanco (Alterações / Pedidos, Pagamentos, Reclamações) con **Claude** y guarda las firmas en **Google Drive**.

1. En el navegador: abre el TIF (o PDF/imagen) y lee los códigos de barras de cada página (ITF de 20 dígitos = **CB**, Code 39 `SEPnn` = **ETIQUETA**).
2. Envía las páginas (JPEG, lado mayor 1568 px) al webhook `/webhook/acartoes` del workflow de n8n **«ACartoes · Claude + firmas a Drive y Google Sheet»**. Claude extrae los datos y devuelve la caja de cada firma o sello.
3. La web recorta cada firma del original a resolución completa y la envía con la fila a `/webhook/acartoes-guardar`: n8n sube los PNG a Google Drive y añade la fila a la pestaña **«Datos»** del Google Sheet **ACartoes** (https://docs.google.com/spreadsheets/d/1s2K873ZdJHj-oTi5e7ZrkHoaEgDkwb0h67EYfQVPAFI); la pestaña «Campos» describe cada columna.

| NOMBRE PDF | TIPO | CB | DATACB | DATA | ETIQUETA | NCONTA | NCARTAO | NCONTACARTAO | N FIRMAS | FIRMAS | FUENTE CB | OTROS DATOS |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

- **CB / ETIQUETA**: los leídos en la web; si no se pueden leer, los de Claude (`FUENTE CB`).
- **DATACB**: dígitos 9-14 del CB en formato DDMMAA (`0007 0493 210926 007361` → `21/09/2026`).
- **TIPO**: por la etiqueta (SEP2, SEP14, SEP27) o por el código de formulario del CB (1429 / 0493 / 0656).
- **FIRMAS**: enlaces a los PNG en Drive, nombrados `CB_pPAGINA_TIPO_N.png` (tipo: cliente, banco, sello, otro).
- `acartoes/prueba/pagina_prueba.png`: página de prueba con datos inventados.
