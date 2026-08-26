🇷🇺 [Русский](../ru/10-autofunnel.md) · 🇬🇧 [English](../en/10-autofunnel.md) · 🇪🇸 [Español](../es/10-autofunnel.md)

# Case Study: AutoFunnel

**Live:** https://autofunnel.optimum-ltd.com/
**Status:** ✅ Production · primer cliente en producción, con clientes finales reales

---

## Problema

Las pequeñas empresas necesitan embudos de venta dentro de los mensajeros, pero cada plataforma
funciona de otra manera: sus propios webhooks, botones, ventanas de envío y límites de medios.
Montar un embudo significa pagar siete servicios distintos o escribir las integraciones a mano.

AutoFunnel es un solo editor visual, un solo motor y una sola bandeja para siete canales.

## Funcionalidad

Editor visual de embudos (nueve tipos de nodo: mensaje, condición, acción, recogida de datos,
pago, reserva, test A/B, fórmula, nota), disparadores por palabras clave, comentarios, historias,
etiqueta de origen, horario y pago, CRM con segmentos, campañas y series de mensajes, imanes de
leads, base de conocimiento, venta de entradas con control por QR, cobros mediante ocho
proveedores y devoluciones, bandeja unificada con respuesta del operador y panel en trece idiomas.

## Valor de negocio

Sustituye siete integraciones por un solo panel: el embudo se dibuja con el ratón y se comporta
igual en Telegram, Instagram y el resto. El dinero va directo a la cuenta del propietario — la
plataforma no cobra comisión sobre la facturación ni retiene fondos.

## Stack

Python 3.11 · FastAPI · MariaDB · JS sin empaquetador · systemd · LXC · Cloudflare Tunnel ·
Telegram Bot API · Meta Graph API (Instagram, Messenger, WhatsApp) · VK Callback API · MAX API ·
MTProto (Telethon) para historias

## Bloques funcionales

Canales · Editor de embudos · Motor · CRM · Campañas · Entradas · Pagos · Analítica · Bandeja

## Decisiones de ingeniería

**Escalera de precedencia de disparadores.** El evento va al disparador que más dice sobre él:
etiqueta de origen → palabras clave → genérico. Sin ella un único disparador genérico se traga
todos los demás escenarios — que es justo lo que ocurrió una vez con datos reales.

**Los medios se envían por identificador, no por archivo.** Tras el primer envío la plataforma
devuelve su propio id del archivo ya subido. Medido con un vídeo real de 50 MB: 94 s → 0,5 s,
cero bytes de subida.

**Las decisiones leen propiedades, no texto.** Meta traduce los mensajes de error, así que
comparar cadenas funciona en inglés y falla en silencio en árabe. Esas ramas leen códigos numéricos.

**Trinquete de tamaño por módulo.** Cada módulo tiene su tope; cuando se agota, el dominio se
extrae a un módulo nuevo y el umbral nunca sube. Dos veces se *bajó* el tope tras una extracción.

**El testing de mutación como condición de aceptación.** Veintiún mil tests no prueban nada por
sí solos: un test verificado sólo en verde es una descripción, no un guardián. 422 scripts rompen
una regla cada uno y deben ponerse en rojo.

## Escala

| | |
|---|---|
| Módulos | 460 de servidor + 217 de interfaz |
| Tests | más de 21 800 en 691 ficheros |
| Scripts de mutación | 422 |
| Tablas de base de datos | 60 |
| Documentos de diseño | 193 |
| Idiomas de interfaz | 13 |

## Capturas

![Architecture](../../assets/autofunnel-diagram.svg)

## En curso

Verificación de empresa en Meta para levantar los límites de WhatsApp y Messenger · escaparate
público de entradas · informe por evento y exportación de invitados · control de entradas sin
conexión · entradas nominativas · página de entrada en todos los idiomas del panel.

---

[← Volver al portafolio](https://github.com/alexanderomobile/portfolio/blob/main/README.md)
