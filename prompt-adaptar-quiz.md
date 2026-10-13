# Prompt para adaptar tu quiz con Claude
*IA Founders Accelerator · Fase 2*

> **Cómo usarlo:** abre el proyecto de Claude de la Fase 2 (el que tiene tu oferta de la Fase 1), pega este prompt y cambia solo lo que está entre corchetes.

---

```
Tengo esta plantilla de quiz: [pega el HTML].

Quiero adaptarla a mi nicho: [tu nicho]. Mi oferta está en el contexto de este proyecto.

Cambia solo las preguntas y el texto de resultado.
No toques la estructura del HTML, el script del píxel ni las funciones de scoring y envío del webhook.
```

---

## Qué NO se toca
- La estructura del HTML.
- El script del píxel. El Dataset ID se pega en las 3 páginas, dos veces por archivo: en el `script` y en el `noscript`.
- Las funciones de scoring y de envío al webhook.

## Antes de darlo por bueno
- Rellena tú el quiz entero: tiene que completarse en **menos de 2 minutos**.
- Ábrelo en el móvil.
- Con Meta Pixel Helper (extensión de Chrome), comprueba que se dispara el PageView.
