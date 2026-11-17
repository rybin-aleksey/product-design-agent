# Шаблоны

Шаблоны кода — ориентир; способ стилей, роутер и стиль импорта берутся из базы.

## Реестр экранов

```ts
// src/screens/registry.ts
import { lazy, type ComponentType, type LazyExoticComponent } from 'react';

export type ScreenEntry = {
  id: string;            // 'dashboard'
  name: string;          // название для плитки
  description: string;   // одна строка: ключевые блоки экрана
  route: string;         // '/dashboard'
  component: LazyExoticComponent<ComponentType>;
};

export const screens: ScreenEntry[] = [
  {
    id: 'dashboard',
    name: 'Dashboard',
    description: '<ключевые блоки: шапка, метрики, список, активность>',
    route: '/dashboard',
    component: lazy(() => import('./dashboard/Dashboard')),
  },
];
```

## Индекс «/»

```tsx
// src/screens/ScreensIndex.tsx
import { Link } from 'react-router-dom';      // или ссылка из принятого роутера
import { Card } from '../components';
import { screens } from './registry';
import s from './ScreensIndex.module.css';

export default function ScreensIndex() {
  return (
    <main className={s.page}>
      <header className={s.header}>
        <h1>{'<Продукт>'} — экраны</h1>
        <p>Статичные экраны, собранные из дизайн-системы. Логика и данные — отдельный шаг.</p>
      </header>
      {screens.length === 0 ? (
        <p className={s.empty}>Экранов пока нет</p>
      ) : (
        <ul className={s.grid}>
          {screens.map(sc => (
            <li key={sc.id}>
              <Link to={sc.route} className={s.tile}>
                <Card>
                  <strong>{sc.name}</strong>
                  <span>{sc.description}</span>
                  <code>{sc.route}</code>
                </Card>
              </Link>
            </li>
          ))}
        </ul>
      )}
      {/* Ссылка на Storybook — только если каталог есть в проекте */}
    </main>
  );
}
```

```css
/* ScreensIndex.module.css — только токены */
.page { min-height: 100vh; padding: var(--space-12) var(--space-6); background: var(--surface-subtle); }
.header { display: flex; flex-direction: column; gap: var(--space-2); margin-bottom: var(--space-8); }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: var(--space-6); list-style: none; padding: 0; }
.tile { display: block; color: inherit; text-decoration: none; border-radius: var(--radius-md); }
.tile:focus-visible { outline: 2px solid var(--border-focus-ring); outline-offset: 2px; }
```

## Заметка экрана

```markdown
# <Name>

**Роут:** /<name>
**Figma:** <ссылка на фрейм> (node `<id>`)
**Composition map:** <путь или «нет»>
**Собран / обновлён:** <YYYY-MM-DD>

## Из базы
- Navbar
- Card (Type=default)
- Button (Type=primary, Size=md)

## Локальные доменные блоки (`parts/`)
- ServerCard — <из чего собран>; кандидат на вынос в базу: <да — уже есть на <экран> / пока нет>

## Статика
- Контент перенесён из макета как есть. Данных, состояния и обработчиков нет — это шаг оживления.
- Плейсхолдеры: <логотип, изображения>
- Скрытые состояния из макета (empty / loading): <не собирались — оживление>

## Замечания
- <FIXED-размеры без токена, детачнутые элементы в Figma, расхождения с composition map>
```

## Финальный отчёт

```text
Экран <Name> собран и подключён.
- Роут: /<name> · файлы: workspace/product/code/src/screens/<name>/
- Из базы: <компоненты>
- Локальные доменные блоки: <список / нет> (кандидаты на вынос: <…>)
- Заметка: src/screens/<name>/<name>.md
- Индекс: открой «/» — экран плиткой (название + описание + роут), переход работает
- Проверка раскладки (ширина / длинный контент): <ок / замечания>
- Запуск: cd workspace/product/code && npm run dev → <адрес>

Логику и данные не трогал — экран статичный (оживление — скилл react-wire, команда wire).
Не собралось / требует внимания: <список / нет>
```
