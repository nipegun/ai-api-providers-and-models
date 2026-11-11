# AI API Providers and Models

This README is also available in:\
[en-GB](README.en-GB.md), [en-US](../README.md), [es-AR](README.es-AR.md), es-ES

Catálogo compartido de proveedores y modelos de API de inteligencia artificial para aplicaciones que necesitan una fuente común de configuración de conexión, capacidades de los modelos, límites de tokens y precios de referencia.

El proyecto mantiene dos archivos JSON. Las aplicaciones pueden utilizarlos para rellenar selectores de proveedores y modelos o configurar sus integraciones con las API, centralizando las actualizaciones del catálogo.

## Contenido

| Archivo | Finalidad |
| --- | --- |
| [api-providers.json](../api-providers.json) | Nombres de proveedores, URL de las API, formatos de petición y opciones de integración. |
| [api-models.json](../api-models.json) | Modelos agrupados por proveedor, con identificadores, capacidades, límites, configuración de razonamiento, precios y enlaces a las fuentes. |
| [MANUAL.es-ES.md](MANUAL.es-ES.md) | Instrucciones para consultar, utilizar y actualizar el catálogo. |
| [CODE.es-ES.md](CODE.es-ES.md) | Referencia de la estructura de datos y su integración. |

El catálogo incluye proveedores directos, pasarelas y servicios locales. Algunos ejemplos son Anthropic, Gemini, OpenAI, DeepSeek, OpenRouter, Vercel AI Gateway, Ollama, llama.cpp y vLLM. Algunos proveedores tienen una lista de modelos vacía.

La ubicación del repositorio compartido indicada para este proyecto es [nipegun/ai-api-providers-and-models](https://github.com/nipegun/ai-api-providers-and-models).

## Primeros pasos en Debian

1. Guarda `api-providers.json` y `api-models.json` en la misma carpeta, conservando sus nombres.
2. Configura la aplicación que los vaya a utilizar para que lea ambos archivos desde esa carpeta.
3. Para explorar los archivos desde una terminal, puedes instalar `jq` de forma opcional. Ejecuta estos comandos de instalación de paquetes como `root`:

```sh
apt-get update
apt-get install -y jq
```

El catálogo no necesita paquetes, compilación, base de datos ni un servicio que arrancar. Cualquier aplicación con un lector de JSON puede utilizarlo. Los comandos siguientes presuponen que la terminal está en la carpeta que contiene ambos archivos y se pueden ejecutar como usuario normal.

## Uso

Lista los identificadores de los proveedores:

```sh
jq -r 'keys[]' api-providers.json
```

Consulta la configuración de conexión de un proveedor:

```sh
jq '.openai' api-providers.json
```

Lista los identificadores de los modelos registrados para ese proveedor:

```sh
jq -r '.openai[].name' api-models.json
```

Sustituye `openai` por la clave de otro proveedor cuando lo necesites. Estos comandos leen datos locales; no envían peticiones a los modelos.

Para utilizar un modelo en una aplicación, relaciona los dos archivos por la clave del proveedor, selecciona su `name` exacto y utiliza el `format` del proveedor junto con el `api_type` del modelo para elegir la integración adecuada. Las credenciales, las peticiones HTTP, el tratamiento de respuestas y la instalación de modelos locales corresponden a la aplicación que consume los datos. Este proyecto no incluye un cliente de API ni un servidor de inferencia.

## Interpretación de los datos

- **Las claves de proveedor relacionan los archivos.** Por ejemplo, `api-providers.json["openai"]` describe el proveedor cuyos modelos aparecen en `api-models.json["openai"]`. El identificador de un modelo pertenece a su proveedor; las entradas de proveedores distintos son registros independientes.
- **La mayoría de los valores de los modelos son cadenas de texto.** Esto incluye cantidades de tokens, precios e indicadores como `"true"` y `"false"`. Conserva sus tipos en los archivos. El campo opcional `reasoning_effort_map` es un objeto con valores numéricos.
- **Una cadena vacía indica que no hay un valor registrado.** No significa cero, acceso gratuito, capacidad ilimitada ni ausencia de soporte. Una lista de modelos vacía indica que no hay modelos incluidos para ese proveedor.
- **La configuración de razonamiento requiere interpretación.** Los niveles de esfuerzo se guardan en una cadena separada por comas. Puede haber ajustes de razonamiento vacíos junto con `reasoning: "true"`.
- **Los precios son valores de referencia en USD por millón de tokens.** El esquema guarda una tarifa de entrada, entrada en caché y salida por cada modelo. No representa todos los tramos, rutas, periodos promocionales, condiciones de acceso ni cargos adicionales. Un `"0"` registrado no implica por sí solo acceso gratuito sin restricciones.
- **Los datos de conexión son metadatos.** Las URL locales necesitan un servicio que se ejecute por separado. La aplicación que consume los datos debe resolver los marcadores de las URL, como `{account_id}`.

Consulta la [referencia técnica](CODE.es-ES.md) para conocer todos los campos y el [manual](MANUAL.es-ES.md) para ver ejemplos prácticos.

## Mantenimiento

Las actualizaciones se mantienen de forma centralizada y se comparten con las aplicaciones que utilizan los datos. Conserva los nombres de archivo, las claves de proveedor, los nombres de los campos, la estructura JSON y los tipos de los valores al editar el catálogo.

Utiliza los enlaces a las fuentes registrados en los modelos para confirmar los cambios. Cuando un proveedor anuncie la deprecación o retirada de un modelo, elimina esa entrada inmediatamente, aunque la fecha efectiva sea posterior. Aplica el anuncio al proveedor afectado; no confirma automáticamente la retirada en otros proveedores.

Las aplicaciones consumidoras deben actualizar sus copias de forma deliberada y revisar los modelos seleccionados después de cada actualización. El repositorio no contiene un actualizador automático, un servicio de sincronización ni comprobaciones programadas de disponibilidad.
