# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page digital Halloween party invitation (Spanish, es-MX) meant to be shared over WhatsApp and hosted as a static site on Netlify (`halloween-chely.netlify.app`). There is no build step, package manager, linter, or test suite — everything lives in `index.html` (inline CSS + HTML + JS). Code comments and all user-facing text are in Spanish; keep it that way.

## Running locally

Open `index.html` directly, or serve the folder so audio/images load like in production:

```
python3 -m http.server 8000   # then open http://localhost:8000/?invitado=Familia%20López&pases=4
```

## Structure of `index.html`

1. **`<head>`** — Open Graph / Twitter meta tags for the WhatsApp link preview. They hardcode absolute URLs to `https://halloween-chely.netlify.app/` (`vista-previa.jpg`, 1200×630). If the site name changes, update both `og:url` and `og:image`.
2. **`<style>`** — Dark, mobile-first single-column design. Colors are CSS custom properties on `:root`; the default `:root` is the "brasa" palette, overridden by `:root[data-fondo="noche|pantano|medianoche"]`. Fonts come from Google Fonts (Griffy, Creepster, Butcherman, Cinzel, Fredoka).
3. **HTML** — an entry "gate" (door to knock on) covering the page, then sections: hero, countdown, costume contest, games, gallery, "dulce o truco" pumpkin, tips, location, RSVP form.
4. **`<script>`** (end of body) — starts with a **CONFIGURACIÓN** block holding all event data: `EVENTO` (date, Puebla time `-06:00`), `DURACION_HORAS` (event length for the calendar button), `WHATSAPP` (RSVP number; also rendered as the fallback phone text), `DIRECCION` (used for Google Maps/Waze/calendar links), `AUDIO_SRC`, `FOTOS` + `TEXTOS_FOTOS` (polaroid captions), `FONDO` (theme), `IMAGEN_PORTADA` / `IMAGEN_PORTADA_RESPALDO`. Change event details here rather than in scattered markup — though some text (date, venue name) is also duplicated in the HTML, meta tags, and the RSVP WhatsApp message in `updateLink()`.

## Key behaviors

- **Entry flow:** `openDoor()` plays a creak + bat flutter, then after ~2.1s hides the gate, starts music, shows the mute button, starts the cursor-following ghost (`ghostFollow()`, skipped under `prefers-reduced-motion`), and animates the lead text. Audio must start from this user gesture (autoplay policies) — `primeSong()` is called synchronously in the click handler for that reason.
- **Music fallback:** tries `cancion.mp3`; if it fails, `startSynth()` plays a generated Web Audio melody.
- **Graceful missing assets:** the hero tries `portada.webp`, then `portada.png`, then a placeholder; gallery photos fall back to placeholders on `error` and are skipped by the lightbox.
- **Personalized links:** `?invitado=<nombre>&pases=<n>` fills the greeting ("Familia …" → plural "están invitados", otherwise gender-neutral "te esperamos"), pre-fills the RSVP name, and shows reserved seats. With `pases`, adults + children are capped at that number (`LIMITE`) and the WhatsApp message includes it; without it the cap is 20.
- **RSVP:** no backend — the form builds a `wa.me/<WHATSAPP>?text=...` link that is regenerated on every input change.
- **Gallery lightbox:** prev/next buttons, ←/→/Esc, and swipe (`deslizo` flag stops a swipe from also closing it). While open, `body.sin-scroll` blocks page scroll — don't reuse `body.locked`, which also sets `height: 100dvh` and would jump the page to the top.
- **Guía de scroll:** tras abrir la puerta aparecen una barra de progreso fija arriba (`#progreso`) y una flecha "Desliza" (`#scroll-hint`) que se oculta al pasar de 80px de scroll; ambas se actualizan en un solo listener `scroll` (`actualizarScroll()`).
- **Chips de la portada:** fecha/hora son enlaces al calendario (copian el `href` de `#calendar`) y el lugar abre Google Maps (copia el de `#maps`).
- **Calendar button:** Google Calendar link on Android/desktop; on iPhone/iPad it downloads a generated `.ics` (blob URL).

## Assets

`cancion.mp3`, `portada.webp` (hero, 32 KB) with `portada.png` as fallback, `fondo-final.jpg` (background of the closing "Te esperamos… si te atreves" footer), `vista-previa.jpg` (OG preview), `fotos/foto1..12`. `FOTOS` builds paths as `fotos/foto${n}.jpg` — keep new photos lowercase `.jpg`; Netlify is case-sensitive, so `.JPG` files 404 and show placeholders. Resize new photos before adding them (e.g. `mogrify -auto-orient -resize '1200x1200>' -quality 80 -strip`); guests open the page on mobile data.
