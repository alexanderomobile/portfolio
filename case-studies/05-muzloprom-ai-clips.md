# Case Study: MuzloProm AI Clips

**Showcase:** [muzloprom-ai-clips](https://github.com/alexanderomobile/muzloprom-ai-clips)
**Статус:** ✅ Production
**Исходники:** приватный репозиторий

## Задача

Telegram-бот: фото + музыкальный фрагмент → AI-сцена → image-to-video → субтитры → готовый клип.

## Стек

FastAPI · aiogram · MySQL · OpenAI · Qwen WAN · ffmpeg · APScheduler

## Функциональные блоки

Fragment Selection · Scene Gen · WAN Pipeline · Subtitles · Workflow Resume · Billing

## Скриншоты

![Архитектура](../assets/muzloprom-diagram.svg)

## Запуск

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8100
```

[← К портфолио](../README.md)
