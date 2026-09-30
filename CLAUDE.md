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
4. **`<script>`** (end of body) — starts with a **CONFIGURACIÓN** block holding all event data: `EVENTO` (date, Puebla time `-06:00`), `WHATSAPP` (RSVP number), `DIRECCION` (used for Google Maps/Waze links), `AUDIO_SRC`, `FOTOS`, `FONDO` (theme), `IMAGEN_PORTADA`. Change event details here rather than in scattered markup — though some text (date, venue name) is also duplicated in the HTML, meta tags, and the RSVP WhatsApp message in `updateLink()`.

## Key behaviors

- **Entry flow:** `openDoor()` plays a creak + bat flutter, then after ~2.1s hides the gate, starts music, shows the mute button, enables the cursor-following ghost, and animates the lead text. Audio must start from this user gesture (autoplay policies) — `primeSong()` is called synchronously in the click handler for that reason.
- **Music fallback:** tries `cancion.mp3`; if it fails, `startSynth()` plays a generated Web Audio melody.
- **Graceful missing assets:** the hero image and gallery photos fall back to placeholders on `error`, so the page works with assets absent.
- **Personalized links:** `?invitado=<nombre>&pases=<n>` fills the greeting, pre-fills the RSVP name, and shows reserved seats.
- **RSVP:** no backend — the form builds a `wa.me/<WHATSAPP>?text=...` link that is regenerated on every input change.

## Assets

`cancion.mp3`, `portada.png` (hero), `vista-previa.jpg` (OG preview), `fotos/foto1..11`. `FOTOS` builds paths as `fotos/foto${n}.jpg` — keep new photos lowercase `.jpg`; Netlify is case-sensitive, so `.JPG` files 404 and show placeholders.
