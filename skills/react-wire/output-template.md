# Шаблоны

Шаблоны кода — ориентир; способ стилей, роутер и структура берутся из базы.

## Скоуп запуска (фаза 0)

```text
Собрано экранов: <N> (<список>). Оживлено ранее: <список / нет>.
Edge cases из дизайна: <список из composition maps и screens-audit / нет>.

Что оживляем в этот запуск?
1. Всё приложение
2. Сценарий: <например, «создание сервера: Dashboard → Create → Dashboard»>
3. Экран: <имя>
```

## Вопросы по логике (фаза 3)

```text
В макете не видно, что должно происходить в этих местах. Ответь коротко, я реализую:

1. <Экран> → «<кнопка/действие>»: что происходит? (например: создаётся запись и переход на список / открывается модалка / …)
2. <Форма>: ограничения поля «<поле>»? (мин / макс, формат)
3. <Действие>: что при недостатке <ресурса>? (блокировать / предупреждать / разрешить)
4. Тексты успеха и ошибки для «<действие>» — свои или предложить?
```

## Дополнительные edge cases (фаза 4)

```text
Сверх макета предлагаю заложить:
- [A] <Экран / блок>: <ситуация> → <как показываем>
- [B] Нулевой баланс: баннер + кнопка «Создать» заблокирована с подсказкой
- [C] Поиск без результатов: «Ничего не найдено» + «Сбросить фильтр»
Какие добавляем?
```

---

## Хранилище (ориентир)

```tsx
// src/data/store.tsx
import { createContext, useContext, useReducer, type ReactNode } from 'react';
import type { Server } from './types';
import { initialState } from './fixtures';

type State = { servers: Server[]; balance: number };
type Action =
  | { type: 'server/created'; server: Server }
  | { type: 'server/deleted'; id: string }
  | { type: 'balance/toppedUp'; amount: number };

function reducer(state: State, a: Action): State {
  switch (a.type) {
    case 'server/created': return { ...state, servers: [a.server, ...state.servers] };
    case 'server/deleted': return { ...state, servers: state.servers.filter(s => s.id !== a.id) };
    case 'balance/toppedUp': return { ...state, balance: state.balance + a.amount };
  }
}

const Ctx = createContext<{ state: State; dispatch: (a: Action) => void } | null>(null);
export function DataProvider({ children, seed = initialState }: { children: ReactNode; seed?: State }) {
  const [state, dispatch] = useReducer(reducer, seed);
  return <Ctx.Provider value={{ state, dispatch }}>{children}</Ctx.Provider>;
}
export const useData = () => {
  const v = useContext(Ctx);
  if (!v) throw new Error('useData outside DataProvider');
  return v;
};
```

## Мок-API (ориентир)

```ts
// src/data/api.ts — только мок, без сети
import { currentScenario } from './scenarios';

const delay = (ms: number) => new Promise(r => setTimeout(r, ms));

export async function mockRequest<T>(result: () => T): Promise<T> {
  const sc = currentScenario();
  await delay(sc === 'slow' ? 2500 : 300 + Math.random() * 500);
  if (sc === 'error') throw new Error('<текст ошибки из продукта>');
  return result();
}
```

---

## Раздел заметки экрана

```markdown
## Поведение
_Оживлено: <YYYY-MM-DD> (wire)_

### Навигация
- «<кнопка>» → /<route>

### Данные
- Читает: servers, balance (`src/data/`)
- Меняет: createServer, deleteServer

### Действия и логика (по решению пользователя)
- «Создать сервер»: валидация имени (1–40 символов, уникальное) → createServer → Toast «Сервер создан» → список
- Баланс ≤ 0: баннер, «Создать» заблокирована

### Состояния
- Загрузка: skeleton списка; loading у кнопки
- Успех / ошибка: Toast; ошибка — «Повторить»
- Пусто: EmptyState «Серверов пока нет» + «Создать сервер»
- Edge cases: <список>

### Сценарии для проверки
- `?scenario=empty`, `?scenario=error`, `?scenario=slow`, `?scenario=zero-balance`

### Не определено
- <действия без логики — ждут решения>
```

## Финальный отчёт

```text
Оживлено.
- Навигация: <что заработало>
- Мок-данные: workspace/product/code/src/data/ — экраны читают и меняют (<сущности>)
- Формы: <какие> с валидацией
- Состояния: загрузка / успех / ошибка / пусто + edge cases (<перечень>)
- Сценарии для проверки: <?scenario=…>
- Анимации: <добавлены / предложены / нет>
- Новые зависимости: <нет / какие, по согласию>

Оживлено в этот запуск: <экраны / сценарии>. Осталось статичным: <список / нет>.
Не определено (ждёт решения): <список / нет>.
Бэкенд, деплой и метрики не трогал — отдельные шаги (публикация — скилл deploy).
Запуск: cd workspace/product/code && npm run dev → <адрес>
```
