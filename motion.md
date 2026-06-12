# Motion — живые микроинтеракции, не «пластик»

Цель: убрать ощущение бездушного AI-дизайна через сдержанную естественную анимацию.
Принцип Ковальски — физика + сдержанность. Конкретные паттерны ниже копируй, не интерпретируй.

## Стек (гибрид)
- **Простые интеракции** (press, hover, focus) — CSS/Tailwind, ноль зависимостей.
- **Сложные** (вход элемента, stagger, spring, layout) — `motion` (framer-motion).
  Ставь библиотеку только когда реально нужна сложная анимация, не ради press-эффекта.

## Что анимировать по умолчанию
- нажатие интерактивного элемента (кнопка, карточка-ссылка)
- hover на интерактивном
- появление/исчезание оверлеев (модалка, дропдаун, тост)
- смену состояния (loading → done)

## Что НЕ анимировать (это slop)
- декоративное движение без интеракции
- бесконечные лупы (пульсация, «дыхание») вне loading
- парящие/качающиеся элементы
- въезды каждого блока при скролле «потому что можно»
- длительности >300ms на UI-интеракциях (ощущается медленным)

---

## Сниппеты — CSS/Tailwind (простое)

Press + hover на кнопке:
```jsx
<button className="transition active:scale-95 hover:bg-surface-200 duration-150 ease-out">
  Запустить
</button>
```

Hover-lift на карточке:
```jsx
<div className="transition duration-200 ease-out hover:-translate-y-0.5 hover:shadow-md">
  ...
</div>
```

Базовые токены: длительность 150–250ms, `ease-out` (не linear), трансформации (`scale`/`translate`) дешевле для GPU чем `box-shadow`/`width`.

Токены в сниппетах (`bg-surface-200` и т.п.) — плейсхолдеры: подставь реальные токены проекта, не копируй как есть.

## Сниппеты — motion (сложное)

Вход модалки/дропдауна (spring, ощущается естественно):
```jsx
import { motion } from "motion/react";

<motion.div
  initial={{ opacity: 0, scale: 0.96, y: 8 }}
  animate={{ opacity: 1, scale: 1, y: 0 }}
  exit={{ opacity: 0, scale: 0.96, y: 8 }}
  transition={{ type: "spring", stiffness: 300, damping: 25 }}
>
  ...
</motion.div>
```

Stagger списка (элементы появляются каскадом, не разом):
```jsx
<motion.ul
  initial="hidden" animate="show"
  variants={{ show: { transition: { staggerChildren: 0.05 } } }}
>
  {items.map((it) => (
    <motion.li key={it.id}
      variants={{ hidden: { opacity: 0, y: 8 }, show: { opacity: 1, y: 0 } }}>
      {it.label}
    </motion.li>
  ))}
</motion.ul>
```

---

## Обязательно: prefers-reduced-motion
Уважай системную настройку — иначе анимация = барьер доступности.

CSS:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

motion — оборачивай в `useReducedMotion()` и отключай смещения/спринги при `true`.
