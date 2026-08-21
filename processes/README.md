# Processes

Одна папка = один канонический процесс или подпроцесс.

Рекомендуемая структура:

```text
processes/HR-HIRE-001/
├── process.yaml
├── evidence.yaml
├── as_is.yaml
├── issues.yaml
├── change_log.yaml
├── to_be.yaml
├── open_questions.yaml
├── documents/
├── diagrams/
│   ├── as_is/
│   └── to_be/
└── delivery/
    └── manifest.yaml
```

## Правила

- Не создавать копию процесса для каждого интервьюируемого.
- Не хранить разные названия одного процесса под разными ID без решения Coordinator.
- AS-IS и TO-BE существуют одновременно.
- Все новые предложенные документы лежат в `documents/`.
- Схема является представлением `process.yaml/as_is.yaml/to_be.yaml`, а не самостоятельным источником бизнес-логики.
- При декомпозиции L1 → L2 дочерний процесс получает собственный ID и `parent_process`.
