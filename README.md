# Product Design Agent

Шаблон агента продуктового дизайна: workflows, skills, knowledge и рабочая папка одного продукта с множеством задач.

Для каждого продукта делается своя копия шаблона. Как завести продукт, обновить его до новой версии шаблона и перенести улучшения обратно в шаблон — в [UPDATING.md](UPDATING.md). История версий — в [CHANGELOG.md](CHANGELOG.md).

## Структура

```text
product-design-agent/
├── .agents/       # Codex и совместимые агенты        ┐
├── .claude/       # Claude Code                        │
├── workflows/     # Сценарии работы                    │ файлы шаблона:
├── skills/        # Продуктовые навыки                 │ приходят из шаблона,
├── knowledge/     # База знаний                        │ в продукте не меняются
├── AGENTS.md      # Инструкции агента                  ┘
├── audit/         # Аудит и журналы
├── workspace/
│   └── product/   # Данные продукта: context/, tasks/, ds/  ← работа над продуктом
├── UPDATING.md
└── CHANGELOG.md
```
