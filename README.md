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
