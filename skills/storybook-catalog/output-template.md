# Шаблоны

## Канонический шаблон story

Равняйся на него для всех компонентов. Имена пропсов и значения — из базы и `ds/components.md`, контент — из контекста продукта.

```tsx
import type { Meta, StoryObj } from '@storybook/react-vite';
import { Fragment } from 'react';
import { Plus } from 'lucide-react';
import { Button } from './Button';

/** Основная кнопка действий. Варианты и состояния — по ds/components.md. */
const meta = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],                       // страница с таблицей пропсов, контролами и Copy code
  args: { type: 'primary', children: '<действие из продукта>' },
  argTypes: {
    type:     { control: 'inline-radio', options: ['primary', 'secondary', 'ghost'] },
    state:    { control: 'inline-radio', options: ['default', 'hover', 'disabled', 'loading'] },
    size:     { control: 'inline-radio', options: ['sm', 'md', 'lg'] },
    children: { control: 'text' },
    onClick:  { action: 'clicked' },
  },
  parameters: { layout: 'centered' },
} satisfies Meta<typeof Button>;
export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story   = { args: { type: 'primary' } };
export const Secondary: Story = { args: { type: 'secondary' } };
export const Ghost: Story     = { args: { type: 'ghost' } };
export const Disabled: Story  = { args: { state: 'disabled' } };
/** С иконкой слева. */
export const WithIcon: Story  = { args: { children: <><Plus size={16} /> <действие></> } };

/** Матрица Type × State — все варианты рядом для сверки с Figma. */
export const AllVariants: Story = {
  parameters: { layout: 'padded' },
  render: (args) => {
    const types = ['primary', 'secondary', 'ghost'] as const;
    const states = ['default', 'hover', 'disabled', 'loading'] as const;
    return (
      <div style={{ display: 'grid', gridTemplateColumns: `auto repeat(${states.length}, auto)`, gap: 'var(--space-4)', alignItems: 'center' }}>
        <span />
        {states.map(s => <span key={s}>{s}</span>)}
        {types.map(t => (
          <Fragment key={t}>
            <span>{t}</span>
            {states.map(s => <Button key={s} {...args} type={t} state={s} />)}
          </Fragment>
        ))}
      </div>
    );
  },
};
```

## Шаблон песочницы

```tsx
import type { Meta, StoryObj } from '@storybook/react-vite';
import { useState } from 'react';
import { Modal, Input, Badge, Button } from '../components';

/** Модальное окно: компоненты в связке. */
const meta = { title: 'Sandboxes/Modal', parameters: { layout: 'fullscreen' } } satisfies Meta;
export default meta;

export const Default: StoryObj = {
  render: () => {
    const [open, setOpen] = useState(true);
    return (
      <>
        <Button type="primary" onClick={() => setOpen(true)}>Открыть</Button>
        {open && (
          <Modal title="<заголовок из продукта>" onClose={() => setOpen(false)}>
            {/* Input, Badge, Button secondary + primary */}
          </Modal>
        )}
      </>
    );
  },
};
```

## Вопрос про сложные кейсы (фаза 6)

```text
Базовый каталог готов. По сложным компонентам можно добавить отдельные кейсы:
- Table: длинный контент, пусто, 100+ строк
- Select / Dropdown: длинные опции, много опций, ошибка
- Toast / Modal: микро-анимации появления
Сделать? Для каких?
```

## Финальный отчёт

```text
Каталог в Storybook собран.
- Код: workspace/product/code/
- Foundation: Primitive (<N>) + Semantic (<M>) токены, типографика
- Компоненты: <N>/<всего в базе> со всеми вариантами и состояниями, autodocs (контролы + Copy code), AllVariants
- Живые переходы: <компоненты>; правки базы: <файлы / нет>
- Песочницы: <Modal, Form, Navigation, Data & Feedback>
- Сложные кейсы: <добавлено / пропущено по ответу>
- Запуск: cd workspace/product/code && npm run storybook → <адрес>

Не попало / требует внимания:
- <есть в components.md, нет в базе: …>
- <расхождения матриц база ↔ components.md: …>
- <или «нет»>

Дальше: пройди каталог рядом с Figma и назови расхождения — правки внесу.
Передача разработке — скилл github-handoff (handoff).
```
