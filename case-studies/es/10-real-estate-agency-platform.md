🇷🇺 [Русский](../ru/10-real-estate-agency-platform.md) · 🇬🇧 [English](../en/10-real-estate-agency-platform.md) · 🇪🇸 [Español](../es/10-real-estate-agency-platform.md)

# Case Study: Plataforma de agencia inmobiliaria

**Status:** ✅ Production

---

## Problema

Una agencia inmobiliaria en la costa del mar Rojo vendía propiedades por mensajería y carpetas de fotos. El sitio web existía sobre el papel: una plantilla comprada con su contenido de demostración intacto — una dirección en Miami, seiscientos anuncios inventados y un formulario que escribía al autor de la plantilla. Ni una sola propiedad real.

Todo debía resolverse en una sola pasada: trasladar el conjunto a infraestructura propia, construir un catálogo real a partir de material en bruto, dejar el sitio en un estado que las plataformas publicitarias acepten a revisión — y conservar la posibilidad de seguir desarrollando sin convertir el sitio en producción en un banco de pruebas.

Una condición más: la agencia vende a compradores de cuatro grupos lingüísticos, el árabe entre ellos con escritura de derecha a izquierda, y negocia en tres monedas.

## Funcionalidad

**Catálogo.** 96 propiedades, 552 fotografías, 17 vídeos y un punto en el mapa para cada una. Galería, vídeo con fotograma de portada, características y precio. El propietario lo edita todo desde las pantallas normales de administración, sin necesidad de un programador.

**Búsqueda.** Filtros por zona, tipo, estado, habitaciones, superficie y precio, más una página propia de «todo en el mapa» con agrupación de marcadores y una lista sincronizada.

**Cuatro idiomas.** Interfaz, páginas legales, correos y etiquetas en inglés, ruso, alemán y árabe, con maquetación RTL completa. Las propiedades, en cambio, no se traducen a propósito: un único catálogo sirve a las cuatro versiones.

**Multidivisa.** Una moneda base, tipos de cambio actualizados a diario y precios que el visitante lee en la suya. Cada ficha advierte que la cifra convertida es orientativa y que el contrato se firma en la moneda base.

**Canales de contacto.** Teléfono, correo y tres mensajerías — en la cabecera, el pie, la ficha de la propiedad y el menú de compartir, con enlaces que funcionan en los cuatro idiomas.

**Analítica tras el consentimiento.** Ni una sola petición a terceros sale del navegador antes de que el visitante responda al aviso. En paralelo, las visitas se cuentan desde el propio registro del servidor web: sin cookies y sin enviar nada fuera.

**Copia de preproducción.** Una copia completa del sitio en su propio subdominio, con su base de datos, protegida por contraseña, cerrada a los buscadores y con el envío de correo desactivado — para probar cambios sobre datos reales sin que puedan indexarse ni escribir a un cliente.

## Valor de negocio

La agencia dispone de un escaparate que puede enseñar a un comprador y presentar a revisión de plataforma: un catálogo real en lugar de uno de demostración, contactos que alguien atiende y páginas legales que corresponden a lo que el sitio hace de verdad. El catálogo se mantiene internamente. La infraestructura es propia: sin cuota mensual de alojamiento, sin depender del servicio de correo de un tercero y sin un solo puerto de entrada abierto a internet.

## Stack

**Plataforma.** Proxmox, un contenedor por rol: web, base de datos, correo, túnel y tareas de servicio. Instantáneas diarias, restauración probada.

**Web.** Apache con mod_php, PHP 8.3 y un CMS con plantilla sectorial. Todo el desarrollo vive en un tema hijo y un plugin propio, de modo que el tema padre sigue siendo actualizable.

**Perímetro.** Es el propio servidor quien abre la conexión hacia fuera mediante un túnel; no hay puertos de entrada. TLS y DNS en el nodo de borde, y la configuración de rutas se valida antes de aplicarse: un error en ese único archivo tumba a la vez todos los sitios del nodo.

**Correo.** El proveedor bloquea el puerto 25 y el servidor está tras NAT. El correo entrante llega al MX de borde, un manejador lo firma con un secreto compartido y lo entrega por HTTPS a través del mismo túnel a un receptor; sin ese secreto el receptor responde 403. Después: Maildir, IMAP y webmail. El saliente sale por un servicio transaccional, firmado con DKIM.

**Datos.** MariaDB en un nodo aparte, accesible solo a través de dos capas de cortafuegos independientes: una dentro del contenedor y otra en el hipervisor.

**Procesado del material.** Python: Pillow para fotografías y documentos, ffmpeg para extraer fotogramas del vídeo, y scripts de importación en PHP mediante la CLI de la plataforma.

## Decisiones de ingeniería

**El catálogo se construyó con lo que había.** El origen eran 53 carpetas: fotos mezcladas con vídeo, precios enterrados en archivos de texto y en pies de publicaciones, y la mitad de las carpetas conteniendo dos o tres propiedades distintas a la vez. El análisis se hizo como un abanico de pasadas independientes: cada una lee su carpeta, separa los objetos y extrae tipo, superficie, habitaciones, zona y precio. Las trampas se nombraron de antemano: una entrada no es el precio, un alquiler mensual no es una venta, un precio por metro cuadrado no es el precio de la propiedad. El resultado fueron 96 fichas en lugar de 24, y ni una característica inventada.

**Las fotografías se sanearon antes de publicarlas.** Se redimensionaron, se les quitó el EXIF por completo — de lo contrario la geoetiqueta de la cámara sale al mundo junto con la dirección de la vivienda de un tercero — y se marcaron con el logotipo. Las propiedades sin fotos recibieron una portada extraída de su vídeo a un cuarto de su duración: no del primer segundo, donde la cámara aún tiembla.

**La galería estaba guardada en una forma que nada lee.** La plantilla lee la galería como *varias filas* de un campo meta; la importación había escrito una sola fila con una lista separada por comas, de modo que una propiedad con 28 fotografías mostraba exactamente una. Diagnosticarlo costó más que arreglarlo: la lectura normal del campo pasa por un filtro de compatibilidad que convierte el valor a entero y devuelve el primer identificador, así que desde fuera parecía que la base de datos contenía de verdad una sola imagen. Hubo que leer la tabla directamente.

**Una propiedad no es contenido dependiente del idioma.** El plugin multiidioma trataba las propiedades como traducibles, las 96 figuraban como inglesas y en tres de las cuatro versiones el catálogo, todos los archivos y el mapa salían vacíos. Desmarcar la casilla en los ajustes no bastaba: el tema padre declara sus propios tipos como traducibles en su archivo de configuración, y el plugin obedece al archivo antes que a los ajustes. Las listas se filtran en el tema hijo, con una prioridad que se ejecuta con seguridad la última.

**Una capa de traducción propia en lugar de la incluida.** Los catálogos que acompañan a la plantilla están traducidos a medias y a máquina: «Currency Switcher» acaba en algo que ningún hablante escribiría, y los marcadores de printf quedan rotos como «% 2 $ s» — una traducción así estropea la página más que dejarla en inglés. En su lugar hay 253 cadenas por idioma que cubren cuatro fuentes de etiquetas a la vez: las cadenas del propio tema, los valores guardados en sus ajustes, los nombres de las taxonomías y ciertos campos meta.

**Cuándo se aplica la traducción importa más que el diccionario.** La primera versión de esa capa se enganchaba a la lectura de los ajustes, es decir, se ejecutaba durante el arranque, antes de saber el idioma de la página. Su idioma de reserva era la configuración regional del motor, que coincidía con el idioma en que el propietario lee la administración. Las fichas en inglés salieron con títulos en ruso, y los scripts auxiliares que leían y guardaban ajustes escribieron el idioma equivocado en la base como si fuera el original. El diagnóstico llevó una tarde; la recuperación vino de invertir los propios diccionarios y de leer los valores por defecto del tema. La sustitución se aplica ahora cuando el idioma ya está resuelto. La lección merecía anotarse: una traducción tiene diccionario *y* momento.

**Un `meta_key` es un INNER JOIN.** El catálogo ordena «destacados primero», y la plantilla lo implementa con un `meta_key`. WordPress convierte un `meta_key` en un inner join: una propiedad sin esa fila no queda la última, desaparece del resultado. 15 de 96 no la tenían: el archivo por ciudad decía «no se han encontrado propiedades» mientras el término contaba tres, y el archivo por estado mostraba 75 de 90. El síntoma parecía un problema de índices; la causa era un campo ausente.

**La búsqueda por precio solo habla la moneda base.** La plantilla convierte los límites del deslizador para mostrarlos y luego envía esos mismos números convertidos a una consulta que los compara con precios almacenados en la moneda base. Con dólares seleccionados, la búsqueda no encontraba nada, y con razón. Ahora los límites se reconvierten antes de ejecutar la consulta, con los mismos tipos de cambio diarios.

**Una promesa en el texto forma parte del código.** La política de privacidad decía con todas las letras que nada se mide en el navegador del visitante y que el texto se actualizaría *antes* de que eso cambiara. Añadir analítica hizo vencer esa promesa: ocho páginas legales en cuatro idiomas se reescribieron en el mismo cambio que activó el contador.

**Las tipografías, traídas a casa.** Cada página — incluidas las que el visitante ve antes de responder al aviso — pedía sus tipografías a una CDN de terceros y le entregaba de paso la IP del visitante. 38 archivos viven ahora en el servidor propio, y las referencias a la fuente externa no se reescriben sino que se descartan en el punto en que se registran las hojas de estilo, para que un plugin añadido más adelante no reabra el canal en silencio.

**El control de calidad se montó como un banco adversarial.** La preparación no se comprobó pinchando por ahí. Revisores independientes recorrieron cada dimensión — galerías, moneda, idiomas, analítica, SEO, contactos — y cada hallazgo se entregó a un escéptico distinto cuya tarea era refutarlo. Solo contaba lo reproducido. En dos rondas: 40 defectos confirmados y corregidos, 4 descartados como no-defectos — incluido un «texto truncado» que resultó ser la plantilla funcionando exactamente como se diseñó.

## Bloques funcionales

Catálogo · Procesado de medios · Búsqueda y mapa · Capa multiidioma · Multidivisa · Canales de contacto · Consentimiento y analítica · Superficie legal · Perímetro y correo · Copia de preproducción · QA adversarial

## Métricas

| indicador | valor |
|---|---|
| Propiedades publicadas | **96** a partir de 53 carpetas de material en bruto |
| Fotografías · vídeos · puntos en el mapa | 552 · 17 · 96 |
| Idiomas de la interfaz | **4**, RTL incluido |
| Puertos de entrada abiertos | **0** |
| Peticiones a terceros antes del consentimiento | **0** |
| Defectos hallados y corregidos por el banco | **40** confirmados, 4 descartados |
| Propiedades que desaparecían del resultado | 15 de 96 |
| Archivos que devolvían lista vacía | 3 |
| Archivos de tipografía trasladados al propio servidor | 38 |
| URLs en el sitemap, todas responden 200 | 165 |

## Roadmap

Esta sección describe lo **diseñado, todavía no construido**.

**Búsqueda de propiedades por voz en mensajería.** El comprador dicta su petición — «un dos habitaciones en tal zona por menos de cien mil dólares» — y recibe una selección. El recorrido: mensaje de voz → reconocimiento del habla en cuatro idiomas → un modelo de lenguaje rellena los campos de la consulta (presupuesto y moneda, zona, tipo, habitaciones, venta o alquiler) → llamada al catálogo a través de la capa REST que ya existe → respuesta con fichas, foto, precio en la moneda mencionada y enlace al mapa → síntesis de una respuesta hablada. Contenedor propio, webhook por el mismo túnel, cero puertos de entrada igual que ahora. Dos decisiones quedan tomadas de antemano: ante una petición ambigua el bot hace una única pregunta de aclaración en lugar de adivinar, y los precios se convierten con los mismos tipos diarios que el sitio, para que bot y escaparate nunca discrepen en una cifra.

**Búsquedas guardadas y avisos.** Suscribirse a unos criterios y recibir aviso, por correo o mensaje, cuando aparezca algo que encaje.

**Embudo de contactos.** Solicitudes de formularios y mensajerías en una sola cola con estados, en lugar de una conversación repartida entre cuatro aplicaciones.

**Integración con la API de negocio de mensajería.** Mensajes de plantilla y confirmaciones de visita; exige la verificación de plataforma completada, que está en curso.

**Puntuación y deduplicación de fotos.** Descartar automáticamente duplicados y tomas de calidad manifiestamente baja al incorporar una carpeta nueva.

**Comprobaciones continuas del tema hijo.** Linter, verificación de tipos y pruebas de humo de las páginas clave en cada cambio: hoy ese papel lo cumple el banco adversarial, lanzado a mano.

**Content-Security-Policy.** Deliberadamente aún sin activar: sobre esta combinación de plantilla y editor visual no puede encenderse a ciegas y requiere una pasada propia con cada pantalla verificada.

## Capturas

![Architecture](../../assets/real-estate-platform-diagram.svg)

## Ejecución

La infraestructura corre sobre un nodo Proxmox propio. Configuración, direcciones y credenciales no se publican: el sistema da servicio a un negocio en funcionamiento.

---

[← Volver al portafolio](https://github.com/alexanderomobile/portfolio/blob/main/README.md)
