# Ascensores

Aplicación web para registrar incidentes de los ascensores de un edificio y consultar su historial en un calendario. Permite identificar en qué pisos se repiten las puertas abiertas, comparar ascensores y analizar los registros de un período.

El edificio tiene dos ascensores, identificados como **Par** e **Impar**. Ambos paran en todos los pisos, desde **PB (0) hasta el piso 10**.

## Funcionalidades

- Calendario mensual con un indicador por ascensor y día.
- Múltiples registros por día, con tipo, hora, piso y observación.
- Edición y eliminación de registros.
- Reportes por período y por ascensor.
- Ranking de pisos con más registros de puerta abierta.
- Estadísticas por tipo de incidente, franja horaria y mes.
- Inicio de sesión y registro de cuentas mediante Supabase Auth.
- Guardado compartido en Supabase, con detección de ediciones simultáneas.
- Interfaz adaptable a celular y escritorio, con tema claro y oscuro.

## Tecnologías

- HTML, CSS y JavaScript, sin framework ni proceso de compilación.
- Supabase: PostgreSQL y autenticación.
- Cliente JavaScript de Supabase cargado desde CDN.

## Archivos

| Archivo | Descripción |
| --- | --- |
| `ascensores.html` | Interfaz, estilos y lógica de la aplicación. |
| `actualizar-base.sql` | Extensión de la tabla existente y función de guardado con control de versiones. |
| `README.md` | Documentación del proyecto. |

## Configuración

### 1. Preparar Supabase

La aplicación requiere un proyecto Supabase con autenticación por email y contraseña y una tabla `public.registros_ascensores`.

**El archivo SQL incluido presupone que esa tabla ya existe; no configura una base vacía.** La tabla debe contar con los siguientes campos:

| Campo | Requisito |
| --- | --- |
| `fecha` | Fecha del registro, única por día. |
| `par_estado`, `impar_estado` | Estados en texto; permiten valores nulos. |
| `par_detalle`, `impar_detalle` | Detalles en texto; permiten valores nulos. |
| `updated_at` | Fecha y hora de actualización. |

Ejecutá `actualizar-base.sql` desde el SQL Editor del proyecto. Agrega la columna `incidentes` de tipo JSONB, el contador `revision` y la función `guardar_dia_ascensores`.

El acceso depende de los permisos y las políticas RLS configurados en Supabase. Para el funcionamiento previsto, la consulta del calendario requiere lectura; los usuarios autorizados a registrar necesitan lectura, inserción y actualización. La función de guardado exige una sesión iniciada y respeta esos permisos. El SQL incluido no crea ni amplía las políticas de acceso.

### 2. Configurar la conexión

En `ascensores.html`, definí las constantes `SUPABASE_URL` y `SUPABASE_KEY` con la URL y la clave pública de tu proyecto.

Usá una clave pública (*publishable*), nunca una clave secreta o `service_role` en el HTML. La autorización de acceso a los datos se configura en Supabase.

Si la confirmación de email está habilitada, configurá también las URL permitidas de tu aplicación en Supabase Auth.

### 3. Ejecutar o publicar

Serví los archivos mediante un servidor estático. Para desarrollo local, con Python instalado:

```bash
python3 -m http.server 8000
```

Abrí [la aplicación local](http://localhost:8000/ascensores.html).

También puede alojarse en un servicio de archivos estáticos, como GitHub Pages. Si querés que abra directamente en la raíz del sitio, nombrá el archivo principal `index.html`. Usá HTTPS al publicarlo.

## Uso

1. Iniciá sesión o creá una cuenta.
2. Seleccioná un día del calendario.
3. En el ascensor correspondiente, presioná **Agregar registro**.
4. Elegí el estado y completá la hora, el piso y la observación según corresponda.
5. Presioná **Listo** para incorporar el registro al borrador del día.
6. Presioná **Guardar cambios** para guardarlo en Supabase.

Podés agregar varios registros antes de guardar. Si desconocés la hora, dejala vacía; si desconocés el piso, elegí **No sé el piso**. Los registros pueden editarse o eliminarse desde el detalle del día.

El calendario y los reportes se pueden consultar sin iniciar sesión cuando las políticas de lectura de la base lo permiten. Para abrir el editor de un día se solicita autenticación.

## Estados e indicadores

| Estado | Color | Uso |
| --- | --- | --- |
| Funcionando | Verde | Observación de funcionamiento; no cuenta como incidente. |
| FS | Rojo | Código de incidente FS. |
| PA | Amarillo | Puerta abierta. |
| EE | Violeta | Código EE, conservado tal como aparece en el ascensor. |
| Otro | Gris | Incidente con descripción libre. |

Debajo de cada día aparecen dos círculos: **Par a la izquierda e Impar a la derecha**. Un círculo vacío indica que no hay registros para ese ascensor.

Si hay incidentes, el círculo conserva el color del último incidente del día, aunque luego se registre «Funcionando». No representa necesariamente el estado actual del ascensor ni su incidente más grave.

Los incidentes con hora se ordenan cronológicamente. Los nuevos registros sin hora se ubican después, en orden de carga; los registros antiguos importados sin hora se ubican primero. En caso de empate de hora, se conserva el orden de la lista.

## Reportes y estadísticas

Desde **Generar reporte**, elegí un período y filtrá por Par, Impar o ambos. Las fechas de inicio y fin están incluidas.

El reporte muestra:

- Total de incidentes y cantidad de días afectados.
- Distribución por ascensor y tipo de incidente.
- Ranking de pisos para los registros PA.
- Distribución por franjas horarias y categoría «Sin hora».
- Evolución mensual e historial de registros del período.

Los porcentajes del ranking de pisos se calculan sobre **todos los PA del período filtrado**, incluidos los que no tienen piso informado. Los demás gráficos usan el total de incidentes filtrados. «Funcionando» se contabiliza por separado.

Las estadísticas describen lo registrado: no miden la probabilidad de falla por viaje, porque no se registra la cantidad de viajes o paradas. Un período sin registros tampoco confirma que no hubo problemas. Los meses inicial y final pueden ser parciales según las fechas seleccionadas.

## Datos y guardado

Cada fecha tiene una lista de registros almacenada en `incidentes`. Cada registro incluye identificador, ascensor, estado, hora, piso, observación y fecha de creación cuando está disponible.

El guardado del día es transaccional y usa un número de revisión. Si otra persona modifica la misma fecha antes de que guardes, la aplicación rechaza el reemplazo y conserva tu borrador en pantalla. Copiá lo necesario, cerrá y recargá para revisar la versión reciente antes de volver a guardar.

La sincronización se realiza al cargar los datos y guardar cambios; no hay una suscripción a actualizaciones en tiempo real. La aplicación requiere conexión para consultar y guardar en Supabase.

## Compatibilidad de registros

Los registros de la estructura anterior se muestran sin inventar horarios. Cuando se guarda una fecha, su historial se almacena en la columna `incidentes`; los campos anteriores se conservan como respaldo y dejan de representar los cambios posteriores de esa fecha.

Cuando existen registros locales del formato `ascensores-v1`, aparece una opción para importarlos. Se importan las fechas que no existen en la base; las coincidentes se conservan localmente para su revisión, sin reemplazar el historial remoto.

Una vez utilizado el historial múltiple, las cargas deben hacerse con esta aplicación: los clientes que solo escriben los campos anteriores no son compatibles con el nuevo historial.
