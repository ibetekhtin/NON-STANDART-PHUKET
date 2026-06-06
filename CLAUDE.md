# Нестандартный Отдых — Project Guide

Турбизнес на Пхукете и Паттайе. PWA + Telegram Mini App + AI-бот КотЭ.

## Стек

| Слой | Технология |
|------|-----------|
| Frontend | Single HTML file (`public/index.html`), Tailwind CSS CDN |
| Хостинг | Cloudflare Workers (статика через `[assets]`) |
| База данных | Supabase (PostgreSQL) |
| AI-бот | n8n workflow + Google Gemini 2.0 Flash |
| CI/CD | GitHub → Cloudflare Workers (автодеплой) |

## Структура файлов

```
public/
  index.html          ← Всё приложение (3200+ строк)
kote/
  workflow.json       ← n8n воркфлоу КотЭ (13 нод)
  prompt.txt          ← Системный промпт КотЭ
wrangler.toml         ← Cloudflare конфиг
CLAUDE.md             ← Этот файл
```

## Supabase

**Project ID:** `cmmdrhususjuadqzyssc`  
**URL:** `https://cmmdrhususjuadqzyssc.supabase.co`

### Таблицы
- `tours` — каталог туров (16 записей, Пхукет + Паттайя)
- `clients` — клиенты CRM (tg_chat_id, name, phone, email, stage)
- `bookings` — бронирования (client_id FK, tour_id FK, date_start, adults, children, total)
- `payments` — платежи (booking_id FK, provider, status)
- `conversations` — история диалогов с КотЭ
- `client_memory` — долгосрочная память КотЭ (interests, budget_level, tours_viewed)
- `reviews` — отзывы
- `action_history` — лог действий

### RPC функции
- `get_kote_context(p_tg_chat_id)` — контекст клиента для КотЭ
- `upsert_client_memory(...)` — обновить память
- `update_client_stage(p_tg_chat_id, p_stage)` — обновить стадию воронки

## Ключевые константы в index.html

```javascript
const SB_URL = 'https://cmmdrhususjuadqzyssc.supabase.co';
const SB_KEY = '...';  // anon key (публичный)
const PATTAYA_ENABLED = true;
const PAYMENT_CONFIG = { mode: 'demo' }; // ← поменять на 'web' для реальных платежей
const OAUTH_CONFIG = { google: { client_id: '' } }; // ← заполнить для соцсетей
```

## n8n КотЭ

**URL:** `https://ibetekhtin.app.n8n.cloud`  
**Webhook:** `https://ibetekhtin.app.n8n.cloud/webhook/kote`  
**Telegram бот:** `@phuket_nestandart_bot`

### Схема воркфлоу
```
📱 Telegram → 📝 Извлечь → 👤 Upsert клиент → 🧠 Загрузить контекст
→ ✍️ Собрать промпт → 🤖 Gemini → 💬 Достать ответ
→ 📤 Отправить (Telegram)
→ 💾 Сохранить диалог → 🔍 Интент → ❓ Есть обновления?
→ 📊 Обновить память + 🎯 Обновить стадию
```

## Правила разработки

### Ветки
- `claude/...` — ветки Claude Code
- `cline/...` — ветки Cline
- `main` — продакшен, только через PR

### Перед началом работы
```bash
git pull origin main
git checkout -b claude/my-feature  # или cline/my-feature
```

### Деплой
Автоматический: push в любую ветку → Cloudflare строит preview URL.  
Продакшен: merge в `main` → деплой на `non-standart-phuket.ibetekhtin.workers.dev`.

## Что нужно сделать (TODO)

### Критично
- [ ] Подключить реальный платёжный провайдер (ЮKassa) — заменить `mode: 'demo'`
- [ ] Активировать n8n воркфлоу КотЭ (проблема с Telegram credential при Publish)

### Важно  
- [ ] Настроить OAuth (Google/VK/OK) — заполнить `client_id` в `OAUTH_CONFIG`
- [ ] Реальные фото туров вместо Unsplash placeholders
- [ ] Добавить недостающие туры Пхукета в Supabase (сейчас 10, нужно 20)

### Желательно
- [ ] Email-уведомления после бронирования (`PAYMENT_CONFIG.receiptEndpoint`)
- [ ] Отзывы клиентов (таблица `reviews` есть, UI нет)
- [ ] Страница для менеджера (просмотр броней и клиентов)
