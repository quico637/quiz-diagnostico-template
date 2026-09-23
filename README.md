# Plantilla: funnel de diagnóstico (quiz → resultado) para Vercel

Plantilla de un funnel tipo "diagnóstico gratuito": quiz de varias preguntas →
página de resultado con score, coste estimado y CTA a calendario/formulario →
página de privacidad. HTML/CSS/JS estático, sin build, pensado para forkear y
desplegar en Vercel en un click.

Incluye tracking de Meta Ads (Píxel + preparado para API de Conversiones vía
webhook), captura de UTM/fbclid, y envío de leads a tu CRM por webhook.

## Estructura

```
index.html       → landing de ejemplo, con CTA a /quiz (sustitúyela por la tuya)
quiz.html        → el cuestionario, en /quiz
resultado.html   → la página de resultado, en /resultado
privacidad.html  → política de privacidad, en /privacidad
vercel.json      → cleanUrls activado, para que /quiz sirva quiz.html sin ".html"
```

No hay build ni dependencias de servidor: son 4 archivos HTML autocontenidos
(CSS y JS inline, salvo Google Fonts y la librería de teléfono `intl-tel-input`
vía CDN).

## Deploy en 0

1. Haz **Fork** de este repositorio.
2. En Vercel: **Add New → Project** → importa tu fork. Vercel detecta que es
   un proyecto estático sin framework; no hace falta configurar nada más.
3. Deploy. Ya tienes `tu-proyecto.vercel.app`, `/quiz`, `/resultado` y
   `/privacidad` funcionando de punta a punta — con datos de ejemplo.
4. Ahora sí, personaliza (ver checklist abajo) y vuelve a desplegar (push a
   `main` = deploy automático).

### Usarlo con Claude Code

Puedes pegarle este repo a Claude Code y pedirle que lo adapte por ti:
dile el nombre de tu marca, tu Pixel ID, tu webhook, y qué preguntas quieres
en el quiz (con qué opciones y qué debería costar cada una), y que aplique
los cambios siguiendo el checklist de este README.

## Checklist de personalización

### 1. Marca
- `quiz.html`, `resultado.html`, `index.html`: el bloque `<div class="nav-orb">T</div>`
  y los textos `Tu Marca` / `Tu categoría de negocio` en el `<nav>`. Sustituye
  la "T" por tu logo (`<img>` o SVG) y los textos por tu nombre real.
- `privacidad.html`: mismo cambio en el `<nav>` y en el `<footer>`, más los
  enlaces del menú (`Planes`, `Testimonios`, `Contacto`, el link de
  `cal.com/tu-usuario/tu-evento`).
- Colores: cada archivo define su paleta en `:root` dentro de `<style>`
  (`--gold`, `--lila`, `--cream`...). Es un tema oscuro dorado/lila de
  ejemplo — cámbialo por los colores de tu marca.
- Favicon: no hay ninguno puesto (se quitó el de la plantilla original).
  Añade `<link rel="icon" ...>` en el `<head>` de cada página si quieres uno.

### 2. Meta Pixel (Facebook/Instagram Ads)
Busca `TU_PIXEL_ID_AQUI` en `index.html`, `quiz.html` y `resultado.html`
(dos apariciones por archivo: el script y el `<noscript>`) y sustitúyelo por
tu Pixel ID real. Es el mismo ID en las tres páginas.

Eventos que ya dispara el front:
- `PageView` en las tres páginas.
- `InitiateCheckout` en `quiz.html`, al responder la primera pregunta.
- `trackCustom('VeCalendario', ...)` en `resultado.html`, cuando se muestra
  el calendario (rama "encaja").

**`CompleteRegistration` / `Schedule` NO se disparan desde el navegador.**
Estos eventos de conversión real están pensados para dispararse desde tu
backend (n8n, GoHighLevel, Zapier...) vía la API de Conversiones de Meta
(CAPI), usando los campos `fbp`, `fbc` y `fbclid` que ya van en el payload
del webhook. Si los disparas también desde el front, cuentas la conversión
dos veces.

### 3. Webhooks (envío de leads a tu CRM)
- `quiz.html` → constante `WEBHOOK_URL`: recibe el lead que completa el quiz
  (encaje alto, medio o bajo).
- `resultado.html` → constante `WEBHOOK_URL_NOENCAJE`: recibe el lead que,
  pese a no encajar, rellena el formulario de "¿aun así quieres que lo
  miremos?". Es un webhook distinto a propósito, para poder mandarlo a otra
  columna/pipeline en tu CRM.

Ambos aceptan un `POST` con JSON. Revisa el objeto `payload` en cada archivo
para ver los campos exactos que se envían (nombre, email, teléfono, score,
coste estimado, respuestas, UTMs, `fbp`/`fbc`/`fbclid`, tags...).

Si dejas alguna de las dos constantes vacía (`''`), el quiz/resultado
funciona igual, simplemente no envía nada a ningún sitio.

### 4. Preguntas del quiz
Todo está en `quiz.html`, dentro del `<script>` final, dos bloques:

- `questions`: array de preguntas. Cada opción lleva `score` (0-100, cuánto
  encaja esa respuesta con tu cliente ideal), `flag` (`null` / `'orange'`
  aviso / `'red'` mal encaje), y opcionalmente `disqualify: true` para
  descartar del todo. Las preguntas `frecuencia`, `minutos` y `quien` además
  alimentan el cálculo del coste anual (`vecesSemana`, `minutos`, `perfil`).
- `PESOS`: cuánto pesa cada pregunta en el score final ponderado.

Puedes añadir, quitar o reordenar preguntas libremente; solo asegúrate de
que las que usa el cálculo del coste (ver punto 5) sigan existiendo con esos
mismos `id`, o ajusta `startAnalysis()` si las cambias de sitio.

Las categorías del desglose que se ve en `/resultado` (las barras de
"Frecuencia y volumen", "Tiempo por tarea"...) se generan en el objeto
`cats` dentro de `startAnalysis()`, en `quiz.html`. Los nombres que pongas
ahí son los que se muestran tal cual en el resultado.

### 5. Cálculo del coste anual
En `quiz.html`:
- `TARIFAS`: €/hora por perfil (quién hace hoy la tarea).
- `SEMANAS_ANIO`: semanas trabajadas al año, para pasar de horas/semana a
  horas/año.

La fórmula (en `startAnalysis()`) es: `horasAnio = vecesSemana × minutos/60 × SEMANAS_ANIO`,
y `costeAnual = horasAnio × tarifaHora`. Es intencionadamente simple; si
necesitas algo más elaborado (coste de contratación, rotación, distintos
perfiles a la vez...), es el sitio donde ampliarlo.

### 6. Textos del resultado
En `resultado.html`, al final del `<script>`:
- `getVerdict(score, disqualifier)`: título y subtítulo bajo el círculo de
  porcentaje.
- `generateAnalysis(...)`: el párrafo de "Lectura de tu caso".
- `generateImprovementSteps(...)`: los pasos sugeridos cuando el lead no
  encaja o queda en zona gris.

Todo son plantillas de texto (template literals) con datos inventados de
ejemplo — reescríbelos con la voz y el contexto de tu propio negocio.

### 7. Calendario y vídeo (VSL) en el resultado
En `resultado.html`:
- `CALENDAR_EMBED_URL`: URL de embed de tu calendario (Calendly,
  GoHighLevel, Cal.com...). Vacío = se muestra una caja con instrucciones en
  su lugar, así el flujo no se rompe si aún no lo tienes.
- `VSL_EMBED_URL`: URL de embed de tu vídeo (Loom, YouTube, Vimeo...). Vacío
  = simplemente no se muestra ningún vídeo.

### 8. Página de privacidad
`privacidad.html` tiene datos de ejemplo: `Tu Empresa, S.L.`, `Tu ciudad, Tu
país`, `hola@tudominio.com`, fecha de última actualización, y los enlaces
del menú/footer. Sustitúyelos por los reales — y revisa el contenido legal
con quien corresponda antes de publicarlo; el texto es una plantilla
genérica, no asesoría legal.

### 9. Landing (`index.html`)
Es un placeholder minimalista con un único CTA a `/quiz`, para que el fork
funcione de punta a punta nada más desplegarlo. En un proyecto real esta
página normalmente se sustituye por tu landing de verdad (la que ya tengas
en Webflow, GoHighLevel, Framer, o esta misma reescrita).

## Cómo viajan los datos entre páginas

El quiz guarda las respuestas y el resultado calculado en
`localStorage.diag_result`, y `resultado.html` los lee de ahí (si no hay
nada, redirige de vuelta a `/quiz`). Los parámetros de atribución (UTMs,
`fbclid`...) se guardan aparte en `localStorage.diag_attrib` desde que la
persona aterriza en cualquiera de las tres páginas, para no perderlos si
navega entre ellas. Si renombras esas claves, hazlo a la vez en los tres
archivos que las usan (`index.html`, `quiz.html`, `resultado.html`).
