# Referencia técnica del catálogo

This CODE is also available in:\
[en-GB](CODE.en-GB.md), [en-US](CODE.md), es-AR, [es-ES](CODE.es-ES.md)

## Índice

- [Arquitectura y diseño](#arquitectura-y-diseño)
- [Mapa de módulos](#mapa-de-módulos)
- [Símbolos de datos clave](#símbolos-de-datos-clave)
- [Campos de proveedor](#campos-de-proveedor)
- [Campos de modelo](#campos-de-modelo)
- [Flujos principales](#flujos-principales)
- [Puntos de entrada](#puntos-de-entrada)
- [Análisis de impacto](#análisis-de-impacto)
- [Puntos de extensión](#puntos-de-extensión)

## Arquitectura y diseño

Este es un repositorio de datos. No contiene código de aplicación ejecutable, manejadores HTTP, clases ni funciones públicas. Su interfaz consiste en dos documentos JSON:

- `api-providers.json`: un objeto que relaciona claves de proveedor con objetos de configuración.
- `api-models.json`: un objeto que relaciona las mismas claves de proveedor con listas de objetos de modelo.

Separar la configuración de conexión de los modelos evita repetir los ajustes del proveedor en cada entrada de modelo. Mantener ambos archivos como catálogo compartido permite que aplicaciones independientes reciban las mismas actualizaciones sin mantener fuentes de datos independientes.

Los nombres de archivo, las claves, la estructura de objetos y listas y los tipos de los valores son requisitos de compatibilidad. Una aplicación relaciona los archivos mediante la clave de proveedor y selecciona un modelo por su `name` dentro de ese proveedor. No se deben fusionar entradas de proveedores distintos solo porque hagan referencia al mismo modelo subyacente.

No se proporciona un archivo de esquema, una implementación de lector, un cliente de API, una tarea de sincronización ni un almacén de credenciales. El comportamiento de integración descrito a continuación corresponde a la aplicación que consume los datos.

## Mapa de módulos

| Módulo | Ruta | Responsabilidad | Depende de | Usado por |
| --- | --- | --- | --- | --- |
| Proveedores | `api-providers.json` | Metadatos de conexión e indicaciones específicas del proveedor | Documentación del proveedor | Adaptadores y selectores externos |
| Modelos | `api-models.json` | Identificadores y metadatos de modelos por proveedor | Claves compartidas de proveedor y fuentes de los modelos | Selectores y configuración de peticiones externos |
| Presentación | `README.md`, `doc/README.*.md` | Finalidad, preparación en Debian e inicio rápido | Ambos catálogos | Usuarios e integradores |
| Manual | `doc/MANUAL*.md` | Ejemplos de uso y solución de problemas | Ambos catálogos | Usuarios |
| Referencia técnica | `doc/CODE*.md` | Contrato de datos y responsabilidades de integración | Ambos catálogos | Desarrolladores y asistentes de programación |

## Símbolos de datos clave

No hay símbolos de código que indexar. Estas claves JSON son los principales puntos de navegación; los números de línea corresponden a los archivos al crear esta documentación y pueden cambiar al modificar las entradas.

| Símbolo de datos | Archivo:línea | Función |
| --- | --- | --- |
| Objeto raíz de proveedores | `api-providers.json:1` | Consulta por clave de proveedor |
| Objeto raíz de modelos | `api-models.json:1` | Consulta de la lista de modelos de un proveedor |
| `anthropic` | `api-providers.json:2`, `api-models.json:2` | Ejemplo de formato de petición nativo |
| `llamacpp` | `api-providers.json:35`, `api-models.json:625` | Ejemplo de servicio local |
| `openai` | `api-providers.json:46`, `api-models.json:1133` | Ejemplo de URL principal y de Responses |
| `vercel` | `api-providers.json:151`, `api-models.json:7204` | Indicación de formato de razonamiento anidado |
| `cloudflare` | `api-providers.json:159`, `api-models.json:9054` | Marcador de cuenta y formato de credenciales |
| `freetoken` | `api-providers.json:165`, `api-models.json:9182` | Proveedor configurado con lista de modelos vacía |

## Campos de proveedor

Todos los valores de los campos de proveedor son actualmente cadenas. `name`, `url` y `format` aparecen en todos los objetos de proveedor; los demás campos son opcionales.

| Campo | Significado |
| --- | --- |
| `name` | Nombre visible. La clave JSON que contiene el objeto es el identificador del proveedor. |
| `url` | URL principal de la API. Algunos proveedores incluyen el endpoint completo de la operación; Gemini incluye una URL base. |
| `format` | Etiqueta del adaptador. Los valores registrados son `anthropic`, `cohere`, `gemini`, `llamacpp` y `openai`. |
| `url_responses` | Endpoint alternativo para modelos marcados con `api_type: "responses"`. |
| `max_tokens_field` | Nombre del parámetro de límite de salida, como `max_tokens` o `max_completion_tokens`. |
| `tool_choice_mode` | Indicación de selección de herramientas. Los valores registrados son `auto`, `any` y `omit`; el adaptador debe implementar su tratamiento específico por proveedor. |
| `reasoning_format` | Indicación de serialización del razonamiento; Vercel tiene registrado `nested`. |
| `credential_format` | Indicación de estructura de credenciales; Cloudflare registra `account_id:api_token`, con `{account_id}` en su URL. |

La ausencia de un campo opcional no define un valor alternativo universal. El adaptador debe aportar un comportamiento coherente con su proveedor. Las credenciales se proporcionan por separado y los metadatos de su formato no se deben enviar literalmente como token de autorización.

## Campos de modelo

Todos los campos ordinarios de modelo siguientes son cadenas, incluidos los números y los indicadores de tipo booleano. El campo opcional `reasoning_effort_map` es un objeto cuyos valores registrados son enteros.

| Campo | Significado |
| --- | --- |
| `name` | Identificador exacto del modelo dentro del proveedor seleccionado. |
| `creation_date` | Fecha registrada con formato `yYYYYmMMdDD`, por ejemplo `y2026m10d19`, o cadena vacía. La fecha de incorporación a un proveedor puede diferir de la publicación original del modelo. |
| `tokens_max_context_window` | Tamaño registrado de la ventana de contexto en tokens. |
| `tokens_max_output` | Límite registrado de tokens de salida. |
| `price_per_million_tokens_input` | Precio de referencia de entrada en USD por millón de tokens. |
| `price_per_million_tokens_cached_input` | Precio de referencia de entrada en caché en USD por millón de tokens. |
| `price_per_million_tokens_output` | Precio de referencia de salida en USD por millón de tokens. |
| `model_info_url` | Fuente de los detalles del modelo. |
| `provider_models_url` | Fuente del catálogo de modelos o documentación del proveedor. |
| `is_mixture_of_experts` | Indicador registrado de arquitectura MoE: `"true"`, `"false"` o vacío. |
| `max_chars_per_request` | Límite registrado de caracteres, independiente del número de tokens. |
| `reasoning` | Indicador de soporte de razonamiento expresado como cadena. |
| `tool_calling` | Indicador de soporte de herramientas expresado como cadena; no identifica una herramienta alojada ni un servicio de búsqueda concretos. |
| `accepts_temperature` | Indicador de soporte de temperatura configurable expresado como cadena. |
| `reasoning_mode` | Etiqueta de configuración del catálogo; véanse los modos más abajo. |
| `reasoning_effort_levels` | Etiquetas de esfuerzo separadas por comas; no es una lista JSON. |
| `reasoning_effort_default` | Etiqueta de esfuerzo por defecto registrada. |
| `reasoning_effort_source` | Marcador de fuente; actualmente `docs` o vacío. |
| `api_type` | Etiqueta de familia de operación: `completions` o `responses`. Se interpreta junto con el `format` del proveedor. |
| `reasoning_effort_map` | Correspondencia opcional entre etiquetas de esfuerzo y presupuestos numéricos de tokens. |

Las etiquetas registradas de modo de razonamiento son `adaptive`, `budget`, `effort`, `effort_toggle`, `toggle` y la cadena vacía. Describen estilos de configuración que la aplicación consumidora debe interpretar; no son un cuerpo de petición universal. En las entradas `budget`, un objeto opcional aporta valores como `"low": 1024`. Estos valores se conservan como números dentro del objeto.

Una cadena vacía indica que no hay un valor registrado. No se debe convertir en `0`, `false`, un límite ilimitado ni un valor por defecto inferido. Del mismo modo, `"false"` debe interpretarse explícitamente en lugar de evaluar si la cadena contiene texto. Las cadenas numéricas se convierten únicamente en los datos de trabajo de la aplicación, sin cambiar la representación de origen.

Los precios describen una tarifa registrada por campo. No se representan los tramos de contexto, la selección de rutas, las fechas promocionales, los modos rápidos o por lotes, el almacenamiento de caché ni los cargos por petición o audio. El precio de una entrada no representa sus condiciones de acceso.

## Flujos principales

Estos son flujos de integración, no secuencias de llamadas implementadas en este repositorio.

1. **Consulta:** leer proveedores → seleccionar clave de proveedor → leer la lista de modelos correspondiente → mostrar nombres y metadatos.
2. **Configuración de una petición:** seleccionar pareja proveedor/modelo → interpretar `format` y `api_type` → elegir o construir la URL → aplicar opciones admitidas → proporcionar credenciales → enviar mediante un cliente externo.
3. **Actualización de datos:** revisar fuentes del proveedor → editar las entradas afectadas → conservar el contrato de datos → distribuir ambos archivos conjuntamente → actualizar las aplicaciones consumidoras y revisar las selecciones existentes.
4. **Retirada de un modelo:** confirmar proveedor e identificador afectados → eliminar la entrada al anunciarse → conservar las de otros proveedores salvo confirmación independiente → seleccionar un sustituto en las aplicaciones cuando sea necesario.

## Puntos de entrada

| Ruta o comando | Manejador | Archivo |
| --- | --- | --- |
| Leer `api-providers.json` | Lector JSON de la aplicación consumidora | `api-providers.json` |
| Leer `api-models.json` | Lector JSON de la aplicación consumidora | `api-models.json` |
| `jq -r 'keys[]' api-providers.json` | Utilidad externa `jq` | `api-providers.json` |
| `jq -r '.openai[].name' api-models.json` | Utilidad externa `jq` | `api-models.json` |

El repositorio no expone rutas HTTP ni una CLI de aplicación. Las URL del catálogo de proveedores describen servicios externos o locales; este repositorio no los arranca.

## Análisis de impacto

| Cambio | Consumidores afectados |
| --- | --- |
| Renombrar un archivo, clave o campo, o cambiar un tipo de valor | Lectores, relaciones entre archivos, selecciones guardadas y carga de configuración; incompatible con el contrato actual |
| Cambiar una URL o indicación de integración de un proveedor | Todas las peticiones de modelos que utilicen su adaptador |
| Añadir o retirar un modelo | Selectores de modelos y selecciones guardadas |
| Cambiar precios o límites | Visualizaciones, estimaciones y dimensionamiento de peticiones |
| Cambiar metadatos de razonamiento, herramientas o temperatura | Construcción de peticiones y controles de configuración |
| Cambiar una URL de fuente | Mantenimiento y trazabilidad |

## Puntos de extensión

Para añadir un proveedor, se crea su objeto de configuración en `api-providers.json` y una lista con la misma clave en `api-models.json`; la lista puede estar vacía. Solo se incluyen las indicaciones de integración admitidas. Cualquier comportamiento de adaptador nuevo debe implementarse en las aplicaciones consumidoras.

Para añadir un modelo, se utiliza el identificador exacto de ese proveedor y se conserva la estructura de modelo existente. Se rellenan los valores documentados, se dejan vacías las cadenas sin valor registrado y se incluyen enlaces a las fuentes. `reasoning_effort_map` solo se añade cuando la correspondencia numérica está respaldada.

Para revisar una entrada, se actualizan los datos del proveedor afectado sin extender suposiciones sin confirmar a otros servicios. Los identificadores cuya deprecación o retirada se anuncie se eliminan inmediatamente. Las funciones que no se pueden representar con el esquema existente necesitan un diseño independiente; no se deben presentar como modelos de chat o precios por token.

Esta referencia, sus traducciones, los README y los manuales deben mantenerse alineados con los cambios de estructura. Las entradas de módulos, símbolos y flujos afectadas se actualizan cuando cambien sus ubicaciones o comportamiento.
