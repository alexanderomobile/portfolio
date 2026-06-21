# Case Study: RAG Documentation Assistant

**Showcase:** [rag-documentation-assistant](https://github.com/alexanderomobile/rag-documentation-assistant)
**Статус:** ✅ Завершён
**Исходники:** приватный репозиторий

## Задача

Консольный QA-ассистент по технической документации: ответы только из базы знаний, кэш повторов, метрики качества RAGAS.

## Решение

Ingest → ChromaDB → semantic search → GPT-4o-mini → SQLite cache.

## Стек

Python · OpenAI · ChromaDB · SQLite · RAGAS

## Функциональные блоки

1. Smart Chunking
2. Retriever (TOP_K)
3. Prompt Layer
4. Cache Layer
5. RAGAS evaluation

## Скриншоты

![Архитектура](../assets/rag-assistant-diagram.svg)

## Запуск

```powershell
pip install -r requirements.txt
python app.py
```

[← К портфолио](../README.md)
