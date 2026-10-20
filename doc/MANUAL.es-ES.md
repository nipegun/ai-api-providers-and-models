# Manual de uso del catálogo

This MANUAL is also available in:\
[en-GB](MANUAL.en-GB.md), [en-US](MANUAL.md), [es-AR](MANUAL.es-AR.md), es-ES

## Primeros pasos

Guarda `api-providers.json` y `api-models.json` juntos. Puedes abrirlos en un editor de texto o leerlos desde una aplicación. Para los ejemplos de terminal, instala `jq` siguiendo las [instrucciones para Debian](README.es-ES.md#primeros-pasos-en-debian) y abre una terminal en la carpeta que contiene ambos archivos.

Leer el catálogo no requiere una clave de API. Las llamadas a un modelo alojado necesitan las credenciales y condiciones de acceso del servicio seleccionado. Los servicios locales deben instalarse y arrancarse por separado.

## Buscar un proveedor y sus modelos

Muestra las claves de los proveedores junto con sus nombres visibles:

```sh
jq -r 'to_entries[] | [.key, .value.name] | @tsv' api-providers.json
```

Utiliza una clave de la primera columna para consultar su configuración y listar sus modelos:

```sh
jq '.openai' api-providers.json
jq -r '.openai[].name' api-models.json
```

Para mostrar todos los detalles registrados de los modelos de ese proveedor:

```sh
jq '.openai[]' api-models.json
```

Conserva exactamente la clave del proveedor y el `name` del modelo, incluidas las mayúsculas, la puntuación, las barras y los sufijos. El mismo modelo ofrecido por otro proveedor puede tener un identificador, precio, límite o configuración diferente.

## Explorar capacidades y límites

Muestra el proveedor, el modelo, la ventana de contexto y la salida máxima en columnas separadas por tabulaciones:

```sh
jq -r 'to_entries[] | .key as $vProvider | .value[] | [$vProvider, .name, .tokens_max_context_window, .tokens_max_output] | @tsv' api-models.json
```

Lista los modelos marcados como compatibles con herramientas:

```sh
jq -r 'to_entries[] | .key as $vProvider | .value[] | select(.tool_calling == "true") | [$vProvider, .name] | @tsv' api-models.json
```

Compara los indicadores con la cadena `"true"`, no con el booleano JSON `true`. Un límite vacío no está especificado; no es ilimitado. Los límites de contexto y de salida son distintos; no presupongas que se pueden utilizar a la vez toda la ventana de contexto y toda la salida permitida.

Para configurar el razonamiento, consulta `reasoning_mode`, `reasoning_effort_levels`, `reasoning_effort_default` y el campo opcional `reasoning_effort_map`. Los ajustes vacíos no anulan un `reasoning: "true"` registrado. La aplicación que consume los datos debe traducir esos valores al formato de petición del proveedor elegido.

## Leer los precios

Muestra el identificador del modelo seguido de los precios de entrada, entrada en caché y salida de un proveedor:

```sh
jq -r '.openrouter[] | [.name, .price_per_million_tokens_input, .price_per_million_tokens_cached_input, .price_per_million_tokens_output] | @tsv' api-models.json
```

Los precios registrados se expresan en USD por millón de tokens. Un campo vacío es desconocido o no está registrado; `"0"` es un cero explícito. Los precios de entrada en caché no representan todos los cargos relacionados con la caché.

El catálogo no representa todos los tramos de contexto, diferencias entre rutas, fechas de fin de promociones, modos de petición o condiciones de acceso. Consulta `model_info_url` y `provider_models_url` antes de basarte en un precio o límite para una petición concreta.

## Conectar una aplicación

1. Carga ambos archivos JSON y selecciona una clave de proveedor que exista en los dos.
2. Elige un modelo de la lista de ese proveedor. Una lista vacía no ofrece modelos para seleccionar.
3. Lee el `format` del proveedor y el `api_type` del modelo. Cuando corresponda, `url_responses` contiene el endpoint de Responses; `url` contiene la URL principal del proveedor. Algunas integraciones necesitan construir una URL completa a partir de ella.
4. Introduce las credenciales mediante la configuración de la propia aplicación. Resuelve los marcadores como `{account_id}` de Cloudflare. El valor de `credential_format` describe la estructura esperada de las credenciales; no es una credencial.
5. Aplica los ajustes admitidos de límites, razonamiento, temperatura y herramientas mediante el adaptador del proveedor de la aplicación.
6. Deja que la aplicación envíe la petición y procese la respuesta.

Consulta la [referencia técnica](CODE.es-ES.md) para conocer el significado de los campos. El repositorio proporciona los datos para este flujo; no implementa el flujo.

## Mantener una copia actualizada

Obtén ambos archivos del catálogo de la misma actualización y sustituye conjuntamente las copias de la aplicación que los utiliza. Recarga o reinicia esa aplicación si mantiene los archivos en caché; el mecanismo depende de la aplicación.

Si un modelo seleccionado anteriormente desaparece, elige otro modelo incluido para ese proveedor y actualiza la selección de la aplicación. Las deprecaciones y retiradas anunciadas se aplican inmediatamente, aunque un servicio redirija temporalmente las peticiones del identificador antiguo.

Mantén los datos compartidos en este proyecto, conserva el esquema existente y distribuye las actualizaciones a las aplicaciones consumidoras. Guarda las credenciales y los ajustes de URL locales de cada despliegue en la configuración de la aplicación correspondiente.

## Solución de problemas

| Síntoma | Explicación o siguiente paso |
| --- | --- |
| `jq: command not found` | Instala el paquete opcional `jq` siguiendo el README. |
| Archivo no encontrado | Abre la terminal en la carpeta de los JSON o utiliza sus rutas completas. |
| No se muestran modelos | Consulta la clave del proveedor y su lista. Un proveedor existente puede tener una lista vacía. |
| Un valor o precio está vacío | No hay un valor registrado. Consulta las fuentes en lugar de tratarlo como cero. |
| Una URL local no responde | Arranca el servicio local correspondiente y ajusta su dirección en la configuración de la aplicación consumidora. |
| Falla una petición a un modelo | Revisa el identificador exacto, el acceso al proveedor, las credenciales, el endpoint y el formato de petición en la aplicación consumidora. |
