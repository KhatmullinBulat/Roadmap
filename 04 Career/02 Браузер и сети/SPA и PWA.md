---
type: careers/interview
section: Браузер и сети
topic: Архитектура
status: ready
created: 2026-11-16
tags: [interview, spa, pwa, architecture]
---

# SPA и PWA

## Краткий ответ (30 секунд)

**SPA (Single Page Application)** — приложение, где при переходах между «страницами» не происходит полной перезагрузки: HTML приходит один раз, дальше роутинг и рендер делает JavaScript. Плюсы: мгновенные переходы, состояние сохраняется, удобно для приложений. Минусы: тяжёлый первый JS, SEO требует SSR/SSG, доступность и управление фокусом — на разработчике. **PWA (Progressive Web App)** — не «фреймворк», а набор технологий поверх обычного сайта: **manifest** (иконка, установка), **Service Worker** (офлайн, кэш, push), HTTPS, адаптивность. PWA можно ставить на домашний экран и запускать «как приложение».

## Подробно

### SPA

```html
<div id="root"></div>
<script type="module" src="/src/main.js"></script>
```

- **Роутинг** — `history.pushState`/`popstate`; сервер настраивают отдавать `index.html` на все маршруты (fallback).
- **Первая загрузка** — больше JS; LCP может страдать.
- **Состояние** — живёт в памяти, сохраняется при навигации.
- **SEO** — поисковики исполняют JS, но SSR/SSG даёт лучший результат (Next/Astro/Nuxt).
- **Доступность** — нужно управлять фокусом при навигации, `<title>`, `aria-live`.
- **Виды**: CSR (клиентский рендер), SSR (сервер + hydration), SSG (сборка), ISR (инкрементальная регенерация).

Когда SPA плох: контентные сайты, блоги, лендинги — быстрее и дешевле HTML/SSG. Когда хорош: админки, дашборды, сложные интерактивные редакторы.

### PWA

Минимально:

```json
// manifest.webmanifest
{
  "name": "Заметки",
  "short_name": "Notes",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [{ "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" }]
}
```

```html
<link rel="manifest" href="/manifest.webmanifest" />
```

+ Service Worker → офлайн-режим.

Характеристики (по [web.dev](https://web.dev/articles/what-are-pwas)):
- **Устанавливаемая** — manifest + SW + HTTPS.
- **Надёжная** — работает офлайн/на медленной сети.
- **Быстрая** — производительность.
- **Вовлекающая** — push-уведомления, install prompt.

### Установка PWA

- `beforeinstallprompt` (Chromium) → показать свой баннер и вызвать `prompt()`.
- iOS Safari не поддерживает `beforeinstallprompt`, но «Добавить на экран Домой» работает.
- Иконки и splash: для iOS — `apple-touch-icon`, `apple-mobile-web-app-capable`.

### Отличия

| | SPA | PWA |
|---|---|---|
| Что это | архитектура рендеринга | качество/функции сайта |
| Нужен ли SW | нет | да (в основном) |
| Может быть | на любом стеке | поверх любого сайта |
| Установка | нет | да |
| Офлайн | нет | да (через SW) |

SPA и PWA **не исключают друг друга**: SPA часто становится PWA.

> [!tip] Как отвечать
> «SPA — про то, как рендерим: один HTML + клиентский роутинг. PWA — про возможности: manifest, Service Worker, офлайн и установка. Можно и SPA без PWA, и PWA на MPA».

## Связанные вопросы

- [[Service Worker и Web Worker]]
- [[Оптимизация приложения и метрики]]
- [[Что такое Shadow DOM]]

## Ресурсы

- web.dev «What are PWAs»: https://web.dev/articles/what-are-pwas
- MDN Web App Manifest: https://developer.mozilla.org/ru/docs/Web/Manifest
