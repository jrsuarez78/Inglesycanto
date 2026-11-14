---
name: videoscurso
description: Busca videos de karaoke en inglés en YouTube, los muestra como lista con enlace, deja que el usuario escoja cuál quiere trabajar y guarda esa selección para que todo el programa (dashboard) la use. Úsalo cuando el usuario diga "videos", "karaoke", "escoger canción/video", "cambiar video" o pida trabajar con un video de YouTube.
---

# Skill: videoscurso (karaoke en inglés vía YouTube)

Este skill es la **fuente única** de "el video activo" para todo el programa. Cualquier otra parte del dashboard debe leer la selección desde `videoscurso/seleccion.json` en lugar de pedirla de nuevo.

## Flujo

0. **Preguntar cómo quiere buscar**. Al empezar, ofrece siempre estas opciones:
   - **Buscar por nombre de canción** (ej. "Perfect")
   - **Buscar por artista** (ej. "Ed Sheeran")
   - **Canción + artista** (ej. "Perfect Ed Sheeran")
   - **Ver sugeridos** (populares en pop, rock, baladas)

   Si el usuario ya escribió un nombre o artista, úsalo directamente sin preguntar. Después de cada lista ofrece "buscar otra canción o artista" para repetir sin salir del skill.
1. **Buscar**. Usa WebSearch (o el navegador integrado en `https://www.youtube.com/results?search_query=...`) con consultas como:
   - `karaoke english <canción>` / `karaoke english <artista>` / `karaoke english <canción> <artista>` según lo que el usuario escribió
   - Si no dio nada, usa sugeridos populares (pop, rock, baladas)
2. **Mostrar opciones**. Presenta 5–8 resultados numerados en una tabla:

   | # | Título | Canal | Duración | Enlace |
   |---|--------|-------|----------|--------|

   - Enlace siempre en formato `https://www.youtube.com/watch?v=<ID>`.
   - Prefiere canales de karaoke reconocidos (Sing King, KaraFun, Karaoke Version, Stingray Karaoke) y descarta covers sin letra en pantalla.
   - Nunca inventes un ID de video. Si no puedes verificar el enlace, dilo y no lo listes.
3. **Escoger**. Pregunta cuál quiere trabajar (por número). Acepta "otro", "más resultados" o una nueva búsqueda.
4. **Guardar la selección** en `videoscurso/seleccion.json`:

   ```json
   {
     "activo": {
       "id": "VIDEO_ID",
       "titulo": "Título",
       "canal": "Canal",
       "url": "https://www.youtube.com/watch?v=VIDEO_ID",
       "idioma": "en",
       "tipo": "karaoke",
       "seleccionado_en": "2026-11-14T00:00:00Z"
     },
     "historial": []
   }
   ```

   Al cambiar de video, mueve el anterior a `historial` (máx. 20) y no lo pierdas.
5. **Confirmar** al usuario: título + enlace clicable + "este es ahora el video activo para todo el programa".

## Buscador con sugerencias en vivo (dashboard)

En el chat no se puede autocompletar mientras se escribe; eso vive en la interfaz. Usa `videoscurso/buscador.html` como componente del dashboard:

- Mientras el usuario escribe (desde 2 letras) muestra hasta 8 sugerencias de nombres/artistas; se navegan con flechas y Enter.
- Al escoger una sugerencia busca "<texto> karaoke". Con `YT_API_KEY` (YouTube Data API v3) lista los resultados con miniatura y botón "Trabajar este"; sin clave abre la búsqueda en YouTube y el usuario pega el enlace.
- El video escogido se guarda como activo (`localStorage` clave `videos.activo`, misma forma que `activo` en `seleccion.json`) y se reproduce incrustado.
- Si integras el dashboard con un backend, sincroniza esa clave con `seleccion.json`.

## Reglas para el resto del programa

- Cualquier módulo que necesite un video lee `videoscurso/seleccion.json` → `activo`.
- Si `activo` está vacío, invoca este skill para que el usuario escoja antes de continuar.
- Para incrustar en el dashboard usa `https://www.youtube.com/embed/<ID>`; el botón "Abrir en YouTube" usa `url`.
- Responde en el idioma del usuario (español), pero deja títulos de canciones y letras en inglés.
- Respeta los derechos de autor: enlaza o incrusta desde YouTube, no descargues ni copies los videos ni las letras completas.

