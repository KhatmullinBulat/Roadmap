---
type: careers/interview
section: Браузер и сети
topic: Реальное время
status: ready
created: 2026-11-16
tags: [interview, websocket, webrtc, realtime]
---

# WebSocket и WebRTC

## Краткий ответ (30 секунд)

**WebSocket** — постоянное **двустороннее** соединение клиент-сервер поверх TCP (`ws://`/`wss://`). После HTTP-хендшейка (`101 Switching Protocols`) данные идут кадрами в обе стороны с минимальным overhead. Для чатов, лент, курсов, совместной работы. **WebRTC** — стандарт для **прямого peer-to-peer** соединения: медиа (аудио/видео) и данные (DataChannel), с шифрованием через DTLS/SRTP. Для звонков, видеоконференций, screen-sharing, P2P-передачи файлов. Для установления соединения всё равно нужен сигнальный сервер, и часто TURN-релей, если прямой путь не работает.

## Подробно

### WebSocket

```js
const ws = new WebSocket("wss://api.example.com/chat");

ws.addEventListener("open", () => ws.send(JSON.stringify({ type: "join", room: "1" })));
ws.addEventListener("message", (e) => console.log(JSON.parse(e.data)));
ws.addEventListener("close", () => reconnect());
ws.addEventListener("error", (e) => console.error(e));

// отправка после закрытия бросает; проверять ws.readyState === WebSocket.OPEN
```

- **Хендшейк** — обычный HTTP `Upgrade: websocket` + `Sec-WebSocket-Key`; сервер отвечает `101`.
- **Протокол** — бинарные фреймы с маской, без HTTP-заголовков на каждое сообщение.
- **Пинг/понг** — встроенный механизм keep-alive на уровне протокола; на уровне приложения часто добавляют heartbeat, потому что прокси могут «усыплять» простаивающие соединения.
- **Не работает поверх HTTP/2** в браузере (только HTTP/1.1 Upgrade, `wss` порт 443).
- **Автопереподключение** — руками (экспоненциальная задержка, `navigator.onLine`).
- **Альтернативы:** Server-Sent Events (`EventSource`, только сервер→клиент, авто-reconnect, текст), long-polling, WebTransport (HTTP/3, с потоками и ненадёжной доставкой).

### Когда что

| Задача | Выбор |
|---|---|
| Чаты, уведомления, курсы | WebSocket |
| Только поток сервер→клиент | SSE |
| Один запрос/ответ | REST/fetch |
| Звонок/видеоконференция | WebRTC |
| P2P-передача файлов | WebRTC DataChannel |
| Синхронизация состояния в реальном времени | WebSocket (+ CRDT) |

### WebRTC

Компоненты:
- **`RTCPeerConnection`** — соединение.
- **`getUserMedia`** — доступ к камере/микрофону.
- **`getDisplayMedia`** — screen-sharing.
- **`RTCDataChannel`** — произвольные данные, настраиваемая надёжность (по умолчанию SCTP над DTLS).
- **ICE** — поиск сетевых путей (host/srflx/relay кандидаты).
- **STUN** — узнать публичный IP/порт.
- **TURN** — релей, если P2P невозможен (NAT/фаервол).
- **Сигнализация** — свой сервер (WebSocket) для обмена `offer`/`answer`/`candidate`.
- **SDP** — описание медиа-сессии.
- **Шифрование** — DTLS/SRTP обязательны.
- **`getStats()`** — метрики качества (RTT, jitter, loss) для диагностики.

```js
const pc = new RTCPeerConnection({ iceServers: [{ urls: "stun:stun.example.com" }] });

const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
// offer отправляем через сигнальный сервер → ответ (answer) → setRemoteDescription
pc.onicecandidate = (e) => e.candidate && signal.send(e.candidate);
```

### Мифы и подвохи

- «WebRTC — только для видео» — нет, DataChannel для любых данных.
- «WebSocket — это HTTP/2» — нет, отдельный протокол поверх TCP.
- Обратное прокси может «резать» WebSocket; нужна поддержка `Upgrade`.
- WebRTC всегда шифрован — но сигнализация и TURN — не обязательно.
- WebRTC работает и в мобильных Safari/Chrome, но кодек/поведение отличаются.

> [!tip] Формулировка
> «WebSocket — двунаправленный канал поверх одного TCP-соединения через HTTP Upgrade. WebRTC — P2P для медиа/данных, с STUN/TURN и обязательным шифрованием. Для установления WebRTC-сессии нужен сигнальный сервер».

## Связанные вопросы

- [[Из чего состоит HTTP запрос]]
- [[Service Worker и Web Worker]]
- [[SPA и PWA]]

## Ресурсы

- MDN WebSocket: https://developer.mozilla.org/ru/docs/Web/API/WebSocket
- MDN WebRTC: https://developer.mozilla.org/ru/docs/Web/API/WebRTC_API
