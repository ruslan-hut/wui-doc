# Політика дизайну для мобільних пристроїв

Цей документ визначає стандарти дизайну для мобільних макетів у застосунку WUI. Дотримуйтесь цих рекомендацій, щоб підтримувати візуальну узгодженість усіх компонентів.

## Основні принципи

1. **Компактність та ефективність** — максимізуйте видимість контенту за мінімального візуального шуму
2. **Орієнтація на бізнес** — чистий, професійний вигляд, придатний для бізнес-клієнтів
3. **Зручність для дотику** — усі інтерактивні елементи відповідають стандартам доступності
4. **Узгоджені відступи** — використовуйте стандартизовані проміжки та padding у всіх компонентах

---

## Система типографіки

> Базується на [Material Design 3 Type Scale](https://m3.material.io/styles/typography/applying-type) з підтримкою української кирилиці

### Сімейства шрифтів

Застосунок використовує **Montserrat** для заголовків та **Commissioner** для основного тексту, обидва з відмінною підтримкою української кирилиці.

| Призначення | Сімейство шрифту | Доступні накреслення | Підтримка кирилиці |
|---------|-------------|-------------------|------------------|
| **Заголовки** | Montserrat | 300-900 (Thin до Black) | ✅ Відмінна (8 640+ гліфів) |
| **Основний текст** | Commissioner | 100-900 (Thin до Black) | ✅ Відмінна (виправлення лігатури ії) |

**Чому саме це поєднання:**
- Montserrat: геометричний гротеск, схожий на Space Grotesk, професійна B2B-естетика
- Commissioner: розв'язує проблему щільних українських акцентів (її) завдяки розумним лігатурам
- Обидва доступні як варіативні шрифти для оптимальної продуктивності
- Повна підтримка українського алфавіту, включно з рідкісними діакритичними знаками

### Завантаження шрифтів

Додайте до `frontend/src/index.html` у секцію `<head>`:

```html
<!-- Preconnect for performance -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Variable fonts for optimal file size -->
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300..900&family=Commissioner:wght@100..900&display=swap&subset=latin,cyrillic" rel="stylesheet">
```

### CSS-змінні

Визначте у `frontend/src/styles.scss`:

```scss
:root {
  // Font families
  --font-heading: 'Montserrat', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-body: 'Commissioner', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

  // Font weights
  --font-weight-light: 300;
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  --font-weight-extrabold: 800;
  --font-weight-black: 900;
}
```

### Базові стилі типографіки

```scss
body {
  font-family: var(--font-body);
  font-weight: var(--font-weight-regular);
  font-size: 14px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

// Headings use Montserrat
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: var(--font-weight-bold);
  line-height: 1.25;
  margin: 0;
}

// Buttons use Commissioner
button, .button {
  font-family: var(--font-body);
  font-weight: var(--font-weight-medium);
}
```

### Тестування української типографіки

Тестуйте на цих українських рядках, щоб перевірити коректність відображення:

```
// Accent-heavy (tests ї, і combinations)
"Київ, Україна"
"Їжачок і їжа"
"Замовлення №12345"
"Продукти: 150 ₴"
```

---

## Стандарти відступів

> Базується на [Material Design 3 8dp Grid System](https://m3.material.io/foundations/layout/understanding-layout/spacing)

### Шкала відступів (сітка 8dp)

Використовуйте кратні 8px значення для узгоджених відступів. Інкремент у 4px доступний для випадків, що потребують щільного розташування.

| Token | Значення | Використання |
|-------|-------|-------|
| `--spacing-xs` | 4px | Щільні відступи, padding іконок |
| `--spacing-sm` | 8px | Стандартний padding інпутів, малі проміжки |
| `--spacing-md` | 16px | Стандартний відступ між елементами |
| `--spacing-lg` | 24px | Відступи між секціями |
| `--spacing-xl` | 32px | Великі проміжки між секціями |
| `--spacing-2xl` | 40px | Розділення основних секцій |
| `--spacing-3xl` | 48px | Відступи рівня сторінки |

### Мобільні пристрої (max-width: 768px)

| Тип елемента | Padding | Проміжок | Margin |
|-------------|---------|-----|--------|
| Контейнер/Сторінка | 12px (1.5×) по горизонталі | - | - |
| Секція/Картка | 12px (1.5×) | - | 8-16px знизу |
| Елементи списку | 12px (1.5×) | 8px (1×) між | - |
| Кнопки | 12px (1.5×) | - | - |
| Поля форми | 8px (1×) (`--spacing-sm`) | - | 12px (1.5×) знизу |

### Дуже малі екрани (max-width: 480px)

| Тип елемента | Padding | Проміжок | Margin |
|-------------|---------|-----|--------|
| Контейнер/Сторінка | 12px (1.5×) по горизонталі | - | - |
| Секція/Картка | 8px (1×) | - | 8px (1×) знизу |
| Елементи списку | 8px (1×) | 8px (1×) між | - |

**Примітка:** числа в дужках (наприклад, «1×», «1.5×») вказують на кратність базової сітки 8px.

---

## Стандарти типографіки

> Базується на [Material Design 3 Type Scale](https://m3.material.io/styles/typography/applying-type)

### Шкала типів Material Design 3 (довідка)

Material Design 3 визначає вичерпну шкалу типів. Нижче наведено розміри, адаптовані для B2B-застосунків на мобільних пристроях:

| Роль | Розмір / Висота рядка | Накреслення | Використання на мобільних |
|------|-------------------|--------|--------------|
| **Display Large** | 57px / 64px | 400 | Hero-секції (рідко на мобільних) |
| **Display Medium** | 45px / 52px | 400 | Маркетингові заголовки (рідко на мобільних) |
| **Display Small** | 36px / 44px | 400 | Великий промоконтент |
| **Headline Large** | 32px / 40px | 400 | Не рекомендується для мобільних |
| **Headline Medium** | 28px / 36px | 400 | Не рекомендується для мобільних |
| **Headline Small** | 24px / 32px | 400 | Великі заголовки сторінок (помірно) |
| **Title Large** | 22px / 28px | 400 | **Заголовки сторінок (h1)** ✓ |
| **Title Medium** | 16px / 24px | 500 | **Заголовки секцій (h2)** ✓ |
| **Title Small** | 14px / 20px | 500 | **Підрозділи (h3), картки** ✓ |
| **Body Large** | 16px / 24px | 400 | Великий основний текст, описи |
| **Body Medium** | 14px / 20px | 400 | **Стандартний основний текст** ✓ |
| **Body Small** | 12px / 16px | 400 | **Другорядний текст, підписи** ✓ |
| **Label Large** | 14px / 20px | 500 | **Великі кнопки** ✓ |
| **Label Medium** | 12px / 16px | 500 | **Стандартні кнопки, мітки** ✓ |
| **Label Small** | 11px / 16px | 500 | **Малі мітки, бейджі** ✓ |

### Розміри шрифтів для мобільних (оптимізовано для B2B)

Наш B2B-застосунок використовує дещо компактнішу шкалу для більшої щільності інформації:

| Елемент | Розмір / Висота рядка | Накреслення | Сімейство шрифту | Роль MD3 |
|---------|-------------------|--------|-------------|----------|
| Заголовок сторінки (h1) | 22px / 28px | 700 | **Montserrat** | Title Large |
| Заголовок секції (h2) | 16px / 24px | 700 | **Montserrat** | Title Medium |
| Підрозділ (h3) | 15px / 22px | 700 | **Montserrat** | Title Small+ |
| Основний текст | 14px / 20px | 400 | **Commissioner** | Body Medium |
| Другорядний текст | 12px / 16px | 400 | **Commissioner** | Body Small |
| Мітки | 12px / 16px | 500 | **Commissioner** | Label Medium |
| Бейджі | 11px / 16px | 600 | **Commissioner** | Label Small |
| Малий текст | 11px / 16px | 400 | **Commissioner** | Label Small |
| Кнопки | 14px / 20px | 500 | **Commissioner** | Label Large |

### Найкращі практики типографіки

1. **Висота рядка**: підтримуйте співвідношення 1.4-1.5× для зручності читання (наприклад, шрифт 14px → висота рядка 20px)
2. **Накреслення шрифтів**:
   - Заголовки (Montserrat): 700 (bold), 800 (extra-bold для h1 на десктопі)
   - Основний текст (Commissioner): 400 (regular), 500 (medium для міток/кнопок), 600 (semi-bold для акценту)
3. **Ієрархія**: максимум 3 рівні ієрархії на екран на мобільних пристроях
4. **Контраст**: мінімум 4.5:1 для основного тексту, 3:1 для великого тексту (18px+)
5. **Міжлітерний інтервал**: використовуйте типовий трекінг; уникайте власного letter-spacing без потреби
6. **Українські акценти**: лігатура ії у Commissioner забезпечує коректні інтервали в українському тексті
7. **Завантаження шрифтів**: використовуйте `font-display: swap`, щоб уникнути невидимого тексту під час завантаження шрифтів

---

## Зони дотику (Touch Targets)

Це внутрішній адміністративний інструмент, оптимізований радше для десктопної щільності, ніж для суворого розміру зон дотику за WCAG. Стандартні CTA мають повний розмір; вбудовані кнопки-іконки та елементи керування панелі дій жертвують ідеальними 44px з WCAG заради візуальної щільності.

- **Основні / CTA-кнопки**: min-height 48px
- **Кнопки панелі дій**: min-height 40px
- **Кнопки-іконки**: **40×40** (іконка 24px + padding 8px) — стандарт проєкту
- **Елементи списку**: min-height 44-52px
- **Поля форми**: min-height 40px (відповідає кнопкам-іконкам у рядках інпутів)
- **Примітка щодо WCAG**: 40px нижче за мінімум 44px з WCAG; прийнятно для десктоп-орієнтованого адміністративного контексту. Публічні поверхні слід збільшити до 48.

---

## Приклади реалізації типографіки

### Повний приклад компонента

```scss
// Card component with proper typography
.product-card {
  padding: 12px;
  border-radius: var(--radius-md);
  background: var(--card-bg);

  .card-title {
    font-family: var(--font-heading);  // Montserrat
    font-size: 16px;
    font-weight: var(--font-weight-bold);  // 700
    line-height: 24px;
    color: var(--color-text-primary);
    margin-bottom: 8px;
  }

  .card-description {
    font-family: var(--font-body);  // Commissioner
    font-size: 14px;
    font-weight: var(--font-weight-regular);  // 400
    line-height: 20px;
    color: var(--color-text-secondary);
    margin-bottom: 12px;
  }

  .card-price {
    font-family: var(--font-heading);  // Montserrat for emphasis
    font-size: 18px;
    font-weight: var(--font-weight-extrabold);  // 800
    color: var(--color-primary);
  }

  .card-label {
    font-family: var(--font-body);  // Commissioner
    font-size: 12px;
    font-weight: var(--font-weight-medium);  // 500
    line-height: 16px;
    color: var(--color-text-muted);
  }

  .card-button {
    font-family: var(--font-body);  // Commissioner
    font-size: 14px;
    font-weight: var(--font-weight-medium);  // 500
    padding: 12px 24px;
    border-radius: var(--radius-md);
  }
}
```

### Ієрархія заголовків

```scss
// Montserrat for all headings
h1 {
  font-family: var(--font-heading);
  font-size: 22px;
  font-weight: var(--font-weight-bold);  // 700
  line-height: 28px;

  @media (min-width: 768px) {
    font-size: 28px;
    line-height: 36px;
  }

  @media (min-width: 1024px) {
    font-size: 32px;
    font-weight: var(--font-weight-extrabold);  // 800
    line-height: 40px;
  }
}

h2 {
  font-family: var(--font-heading);
  font-size: 16px;
  font-weight: var(--font-weight-bold);
  line-height: 24px;

  @media (min-width: 768px) {
    font-size: 20px;
    line-height: 28px;
  }
}

h3 {
  font-family: var(--font-heading);
  font-size: 15px;
  font-weight: var(--font-weight-semibold);  // 600
  line-height: 22px;

  @media (min-width: 768px) {
    font-size: 18px;
    line-height: 26px;
  }
}
```

### Елементи форми

```scss
// Labels, inputs, and helper text
.form-group {
  margin-bottom: 16px;

  label {
    font-family: var(--font-body);  // Commissioner
    font-size: 12px;
    font-weight: var(--font-weight-medium);  // 500
    line-height: 16px;
    color: var(--color-text-secondary);
    display: block;
    margin-bottom: 4px;
  }

  input,
  select,
  textarea {
    font-family: var(--font-body);  // Commissioner
    font-size: 15px;
    font-weight: var(--font-weight-regular);  // 400
    line-height: 20px;
    padding: var(--spacing-sm);
    border: 2px solid var(--color-border);
    border-radius: var(--radius-md);

    &::placeholder {
      font-family: var(--font-body);  // Commissioner
      color: var(--color-text-muted);
    }
  }

  .helper-text {
    font-family: var(--font-body);  // Commissioner
    font-size: 11px;
    font-weight: var(--font-weight-regular);  // 400
    line-height: 16px;
    color: var(--color-text-muted);
    margin-top: 4px;
  }

  .error-message {
    font-family: var(--font-body);  // Commissioner
    font-size: 11px;
    font-weight: var(--font-weight-medium);  // 500
    line-height: 16px;
    color: var(--color-danger);
    margin-top: 4px;
  }
}
```

### Кнопки

```scss
// All buttons use Commissioner
.button {
  font-family: var(--font-body);  // Commissioner
  font-size: 14px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 20px;
  padding: 12px 24px;
  border-radius: var(--radius-md);
  transition: all var(--duration-medium) var(--easing-standard);

  // Button variants
  &.button-large {
    font-size: 15px;
    padding: 14px 28px;
  }

  &.button-small {
    font-size: 12px;
    padding: 8px 16px;
  }
}
```

### Бейджі та чипи

```scss
.badge {
  font-family: var(--font-body);  // Commissioner
  font-size: 11px;
  font-weight: var(--font-weight-semibold);  // 600
  line-height: 16px;
  padding: 2px 8px;
  border-radius: var(--radius-full);
  display: inline-block;
}

.chip {
  font-family: var(--font-body);  // Commissioner
  font-size: 12px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 16px;
  padding: 6px 12px;
  border-radius: var(--radius-full);
  display: inline-flex;
  align-items: center;
  gap: 6px;
}
```

---

## Система кнопок

> Базується на [Material Design 3 Buttons](https://m3.material.io/components/buttons/guidelines) з B2B-кастомізаціями

### Ієрархія та типи кнопок

Material Design 3 визначає 5 типів кнопок з різними рівнями акценту. Для нашого застосунку ми застосовуємо гібридний підхід: **плоскі кнопки в панелях дій/тулбарах** та **градієнтні кнопки для основних CTA**.

#### Рівні акценту кнопок (від найвищого до найнижчого)

| Тип | Акцент | Використання | Застосування в B2B |
|------|----------|-------|-----------------|
| **Filled (Gradient)** | Найвищий | Основні дії, головні CTA | ✅ **Окремі CTA** (з градієнтом) |
| **Filled (Flat)** | Високий | Важливі дії | ⚠️ Використовувати помірно |
| **Filled Tonal** | Середньо-високий | Другорядні важливі дії | ✅ Альтернативні основні дії |
| **Outlined** | Середній | Панелі дій, тулбари, форми | ✅ **Панелі дій/тулбари** (плоскі) |
| **Text** | Низький | Третинні дії, навігація | ✅ Скасування, закриття, необов'язкові дії |

### Рекомендації щодо стилів кнопок

#### 1. Основні CTA-кнопки (градієнтні, окремі)

Використовуйте для найважливішої дії на екрані (оформлення замовлення, надсилання, підтвердження покупки).

```scss
.button-primary {
  // Font
  font-family: var(--font-body);  // Commissioner
  font-size: 14px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 20px;

  // Layout
  min-height: 48px;
  padding: 14px 24px;
  border-radius: var(--radius-md);  // 8px
  border: none;

  // Gradient background (primary brand gradient)
  background: var(--gradient-primary);  // Solid emerald green (#10b981)
  color: var(--color-on-primary);  // White

  // Shadow for depth
  box-shadow: var(--elevation-2);
  transition: all var(--duration-medium) var(--easing-standard);

  // Hover state
  &:hover:not(:disabled) {
    box-shadow: var(--elevation-3);
    transform: translateY(-1px);
    // Slightly darker gradient overlay
    background: linear-gradient(135deg, #5568d3 0%, #6a4091 100%);
  }

  // Active/pressed state
  &:active:not(:disabled) {
    box-shadow: var(--elevation-1);
    transform: translateY(0);
  }

  // Focus state
  &:focus:not(:disabled) {
    box-shadow: var(--elevation-2), var(--focus-ring);
  }

  // Disabled state
  &:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    box-shadow: none;
    cursor: not-allowed;
    opacity: 0.6;
  }
}
```

**Використання:**
- Максимум **1 градієнтна кнопка на екран**
- Зарезервуйте для дії з найвищим пріоритетом (наприклад, «Додати в кошик», «Оформити замовлення», «Надіслати»)
- Має бути найбільш візуально помітним елементом у своєму контексті

#### 2. Кнопки панелі дій (плоскі, з обведенням)

Використовуйте для дій у тулбарах, апбарах та навігаційних зонах.

```scss
.button-action-bar {
  // Font
  font-family: var(--font-body);  // Commissioner
  font-size: 14px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 20px;

  // Layout
  min-height: 40px;
  padding: 10px 16px;
  border-radius: var(--radius-md);  // 8px

  // Flat style with border
  background: transparent;
  border: 2px solid var(--color-border);
  color: var(--color-text-primary);

  // No shadow (flat design)
  box-shadow: none;
  transition: all var(--duration-medium) var(--easing-standard);

  // Hover state (subtle background)
  &:hover:not(:disabled) {
    background: var(--color-surface-variant);  // Very light fill
    border-color: var(--color-primary);
    color: var(--color-primary);
  }

  // Active/pressed state
  &:active:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 12%, transparent);
  }

  // Focus state
  &:focus:not(:disabled) {
    border-color: var(--color-primary);
    box-shadow: var(--focus-ring);
  }

  // Disabled state
  &:disabled {
    background: transparent;
    border-color: var(--color-border-light);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.6;
  }
}
```

**Використання:**
- Дії тулбара (зберегти, редагувати, видалити, фільтрувати)
- Елементи керування навігацією
- Декілька дій у панелях дій
- Дії форми (поряд з основним CTA)

#### 3. Другорядні кнопки (Filled Tonal, плоскі)

Використовуйте для важливих, але не основних дій.

```scss
.button-secondary {
  // Font
  font-family: var(--font-body);  // Commissioner
  font-size: 14px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 20px;

  // Layout
  min-height: 48px;
  padding: 14px 24px;
  border-radius: var(--radius-md);  // 8px
  border: none;

  // Tonal background (subtle primary color)
  background: var(--color-primary-container);  // Light primary tint
  color: var(--color-on-primary-container);  // Dark text

  // Subtle shadow
  box-shadow: var(--elevation-1);
  transition: all var(--duration-medium) var(--easing-standard);

  // Hover state
  &:hover:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 20%, var(--color-primary-container));
    box-shadow: var(--elevation-2);
  }

  // Active state
  &:active:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 25%, var(--color-primary-container));
    box-shadow: var(--elevation-1);
  }

  // Focus state
  &:focus:not(:disabled) {
    box-shadow: var(--elevation-1), var(--focus-ring);
  }

  // Disabled state
  &:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    box-shadow: none;
    cursor: not-allowed;
    opacity: 0.6;
  }
}
```

**Використання:**
- Альтернативні дії (наприклад, «Зберегти чернетку» поряд з «Опублікувати»)
- Кнопки фільтрування/сортування в тулбарах
- Другорядні CTA, яким потрібна помітність

#### 4. Текстові кнопки (низький акцент)

Використовуйте для третинних дій, скасувань та необов'язкових сценаріїв.

```scss
.button-text {
  // Font
  font-family: var(--font-body);  // Commissioner
  font-size: 14px;
  font-weight: var(--font-weight-medium);  // 500
  line-height: 20px;

  // Layout
  min-height: 40px;
  padding: 10px 16px;
  border-radius: var(--radius-md);  // 8px

  // Transparent (text only)
  background: transparent;
  border: none;
  color: var(--color-primary);

  // No shadow
  box-shadow: none;
  transition: all var(--duration-short) var(--easing-standard);

  // Hover state
  &:hover:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 8%, transparent);
  }

  // Active state
  &:active:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 12%, transparent);
  }

  // Focus state
  &:focus:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 12%, transparent);
    box-shadow: var(--focus-ring);
  }

  // Disabled state
  &:disabled {
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.38;
  }
}
```

**Використання:**
- Кнопки «Скасувати»
- Навігація «Пропустити» або «Назад»
- Діалоги, які можна закрити
- Необов'язкові дії

#### 5. Кнопки-іконки (компактні дії)

Використовуйте в тулбарах, списках та компактних інтерфейсах.

```scss
.button-icon {
  // Layout
  width: 40px;
  height: 40px;
  min-width: 40px;
  min-height: 40px;
  padding: 8px;  // 40px - 24px icon = 8px padding each side
  border-radius: var(--radius-md);  // 8px or 50% for circular

  // Flat, transparent
  background: transparent;
  border: none;
  color: var(--color-text-secondary);

  // Icon sizing
  .icon {
    width: 24px;
    height: 24px;
  }

  // No shadow
  box-shadow: none;
  transition: all var(--duration-short) var(--easing-standard);

  // Hover state
  &:hover:not(:disabled) {
    background: var(--color-surface-variant);
    color: var(--color-primary);
  }

  // Active state
  &:active:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 12%, transparent);
  }

  // Focus state
  &:focus:not(:disabled) {
    box-shadow: var(--focus-ring);
  }

  // Disabled state
  &:disabled {
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.38;
  }

  // Filled variant (for active states)
  &.filled {
    background: var(--color-primary);
    color: var(--color-on-primary);

    &:hover:not(:disabled) {
      background: color-mix(in srgb, #000 8%, var(--color-primary));
    }
  }
}
```

**Використання:**
- Дії тулбара (меню, пошук, налаштування, закриття)
- Дії елемента списку (редагувати, видалити, поділитися)
- Навігаційні іконки

### Розміри кнопок

| Розмір | Висота | Padding (H) | Розмір шрифту | Використання |
|------|--------|-------------|-----------|-------|
| **Large** | 56px | 20px | 16px | Hero-CTA, лендинги |
| **Standard** | 48px | 14-16px | 14px | **За замовчуванням** — основні дії |
| **Medium** | 40px | 10-12px | 14px | Панелі дій, тулбари |
| **Small** | 36px | 8-10px | 12px | Компактні UI, щільні таблиці |

### Реалізація фону кнопок

**Основна кнопка (суцільний смарагдово-зелений):**
```scss
:root {
  --gradient-primary: #10b981;        // Solid emerald green
  --gradient-primary-hover: #059669;  // Darker emerald for hover
}
```

> **Примітка:** попри назву змінної `--gradient-primary`, ми використовуємо **суцільні кольори** для чистого, сучасного плоского дизайну. Назву змінної збережено для сумісності.

**Найкращі практики для основних кнопок:**
1. Використовуйте **суцільний смарагдово-зелений** (#10b981) для основних CTA
2. Використовуйте **темніший відтінок** (#059669) для станів hover
3. Забезпечте **контраст 4.5:1** для білого тексту на зеленому фоні
4. Обмежте **однією основною кнопкою на екран** для чіткої ієрархії
5. Зарезервуйте суцільний зелений для найважливішої дії

**Коли НЕ використовувати основний зелений:**
- ❌ Панелі дій і тулбари (використовуйте плоскі кнопки з обведенням)
- ❌ Декілька кнопок поряд (використовуйте другорядні/текстові кнопки)
- ❌ Стани disabled (використовуйте сірий: `--color-border`)

---

## Панелі дій і тулбари

> Базується на [Material Design 3 Top App Bar](https://m3.material.io/components/app-bars/guidelines)

### Правила узгодженості панелі дій

#### 1. Стиль кнопок у панелях дій: **завжди плоский**

```scss
.app-bar,
.toolbar,
.action-bar {
  // Container
  height: 56px;  // Standard Material Design app bar height
  padding: 0 16px;
  background: var(--card-bg);
  border-bottom: 1px solid var(--color-border);
  display: flex;
  align-items: center;
  justify-content: space-between;

  // All buttons in action bar are flat/outlined
  .button,
  button {
    @extend .button-action-bar;  // Flat outlined style
    min-height: 40px;
    padding: 10px 16px;
  }

  // Icon buttons
  .button-icon {
    width: 40px;
    height: 40px;
    padding: 8px;
    background: transparent;

    &:hover {
      background: var(--color-surface-variant);
    }
  }

  // Title
  .app-bar-title {
    font-family: var(--font-heading);  // Montserrat
    font-size: 18px;
    font-weight: var(--font-weight-bold);
    color: var(--color-text-primary);
    flex: 1;
    margin: 0 16px;
  }
}
```

#### 2. Мобільний апбар (компактний)

```scss
@media (max-width: 768px) {
  .app-bar,
  .toolbar {
    height: 56px;
    padding: 0 12px;

    .app-bar-title {
      font-size: 16px;
    }

    // Icon-only buttons on mobile
    .button {
      min-width: 40px;
      padding: 8px;

      // Hide text, show icon only
      .button-text {
        display: none;
      }
    }
  }
}
```

#### 3. Дії панелі дій

**Максимум 3 видимі дії:**
- Основні дії мають бути лише з іконками для економії місця
- Використовуйте оверфлоу-меню «Більше» (⋮) для додаткових дій
- Дії впорядковано за важливістю (зліва направо)

```html
<!-- Example action bar -->
<div class="app-bar">
  <button class="button-icon" aria-label="Menu">
    <i class="icon-menu"></i>
  </button>

  <h1 class="app-bar-title">Product Catalog</h1>

  <div class="app-bar-actions">
    <button class="button-icon" aria-label="Search">
      <i class="icon-search"></i>
    </button>
    <button class="button-icon" aria-label="Filter">
      <i class="icon-filter"></i>
    </button>
    <button class="button-icon" aria-label="More options">
      <i class="icon-more-vert"></i>
    </button>
  </div>
</div>
```

### Нижня панель дій (мобільні)

Для мобільних екранів з основними CTA:

```scss
.bottom-action-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 72px;  // Taller for prominent CTA
  padding: 12px 16px calc(12px + env(safe-area-inset-bottom, 0));
  background: var(--card-bg);
  border-top: 1px solid var(--color-border);
  box-shadow: var(--elevation-3);
  z-index: 100;

  // Primary CTA button (gradient allowed here)
  .button-primary {
    width: 100%;
    min-height: 48px;
  }
}
```

### Узгодженість меню

#### Випадаючі меню

```scss
.dropdown-menu {
  background: var(--card-bg);
  border-radius: var(--radius-lg);  // 12px
  box-shadow: var(--elevation-3);
  padding: 8px 0;
  min-width: 200px;

  .menu-item {
    // Text button style
    font-family: var(--font-body);
    font-size: 14px;
    font-weight: var(--font-weight-regular);
    padding: 12px 16px;
    background: transparent;
    border: none;
    color: var(--color-text-primary);
    text-align: left;
    width: 100%;
    cursor: pointer;
    transition: background var(--duration-short);

    &:hover {
      background: var(--color-surface-variant);
    }

    &:active {
      background: color-mix(in srgb, var(--color-primary) 12%, transparent);
    }

    // With icon
    .icon {
      margin-right: 12px;
      width: 20px;
      height: 20px;
      color: var(--color-text-secondary);
    }

    // Destructive action
    &.destructive {
      color: var(--color-danger);

      .icon {
        color: var(--color-danger);
      }
    }
  }

  .menu-divider {
    height: 1px;
    background: var(--color-border);
    margin: 8px 0;
  }
}
```

### Навігаційне меню (бічна панель)

```scss
.navigation-menu {
  width: 280px;
  height: 100vh;
  background: var(--card-bg);
  border-right: 1px solid var(--color-border);
  padding: 16px 0;

  .nav-item {
    // Text button with left alignment
    font-family: var(--font-body);
    font-size: 14px;
    font-weight: var(--font-weight-medium);
    padding: 12px 24px;
    background: transparent;
    border: none;
    border-left: 3px solid transparent;
    color: var(--color-text-primary);
    text-align: left;
    width: 100%;
    display: flex;
    align-items: center;
    gap: 12px;
    transition: all var(--duration-short);

    .icon {
      width: 24px;
      height: 24px;
      color: var(--color-text-secondary);
    }

    &:hover {
      background: var(--color-surface-variant);
    }

    &.active {
      background: var(--color-primary-container);
      border-left-color: var(--color-primary);
      color: var(--color-primary);

      .icon {
        color: var(--color-primary);
      }
    }
  }
}
```

---

## Патерни компонентів

### Картки (згортувані)

```scss
.card {
  padding: 10px 12px;
  border-radius: 8px;
  border: 1px solid #e1e8ed;

  &.expanded {
    border-color: #10b981;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }
}
```

### Списки з «Показати більше»

- Типовий ліміт: 5 елементів
- Показуйте кількість тих, що залишилися, у кнопці: «Показати більше (X)»
- Стиль кнопки: пунктирне обведення, на всю ширину, текст по центру

### Діалоги-шторки (Bottom Sheet)

```scss
.dialog {
  border-radius: 12px 12px 0 0;
  max-height: 80-85vh;
  margin-bottom: env(safe-area-inset-bottom, 0);
}
```

### Поля форми (стандартний патерн)

Використовуйте цей патерн для всіх текстових інпутів, select та textarea:

```scss
input,
select,
textarea {
  width: 100%;
  padding: var(--spacing-sm);  // 8px - standard input padding
  border: 2px solid var(--color-border);
  border-radius: var(--radius-md);  // 8px
  font-size: 15px;
  background: var(--input-bg);
  color: var(--input-text);
  transition: border-color 0.3s ease, box-shadow 0.3s ease;

  &:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: var(--focus-ring);
  }

  &.invalid,
  &.is-invalid {
    border-color: var(--color-danger);
  }

  &::placeholder {
    color: var(--color-text-muted);
  }
}
```

> [!IMPORTANT]
> **Завжди використовуйте CSS-змінні** (custom properties) для кольорів замість захардкоджених hex-значень. Це забезпечує належну підтримку світлої/темної теми.

**Ключові специфікації:**
- **Padding**: `var(--spacing-sm)` (8px) — забезпечує оптимальний інтервал тексту всередині інпутів
- **Border**: 2px solid для чіткої візуальної межі
- **Border radius**: `var(--radius-md)` (8px) для узгодженості
- **Стан focus**: обведення основного кольору + тінь focus ring
- **Стан invalid**: обведення кольору danger для зворотного зв'язку валідації

### Пошуковий фільтр з підсвічуванням

Використовуйте цей патерн, коли інпут пошуку/фільтра фільтрує рядки таблиці на боці клієнта та підсвічує збіги тексту вбудовано. Інпут фільтра розміщується всередині згортуваної панелі пошуку поряд з іншими елементами керування пошуком (наприклад, інпутами серверних запитів).

**Макет:** декілька інпутів пошуку поділяють один контейнер `.search-bar` з горизонтальним flex-макетом (`flex-wrap: wrap`). Кожен інпут обгорнуто в `.search-input-group`. Інпути клієнтського фільтра використовують провідну іконку (наприклад, `filter_list`) без кнопки надсилання — фільтрація відбувається миттєво через реактивне зв'язування. Інпути серверних запитів зберігають свою кнопку надсилання.

```html
<div class="search-bar">
  <!-- Client-side filter (instant, no button) -->
  <div class="search-input-group">
    <span class="material-symbols-outlined" style="font-size: 20px; color: var(--color-text-muted);">filter_list</span>
    <input
      type="text"
      [ngModel]="filterSignal()"
      (ngModelChange)="filterSignal.set($event)"
      class="search-input"
      placeholder="Filter placeholder text" />
  </div>

  <!-- Backend query (with submit button) -->
  <div class="search-input-group">
    <input type="text" class="search-input" placeholder="Query placeholder" />
    <button class="search-button">
      <span class="material-symbols-outlined">search</span>
    </button>
  </div>
</div>
```

**Реалізація підсвічування:** використовуйте зв'язування `[innerHTML]` з методом компонента, який екранує HTML та обгортає підрядки збігів у теги `<mark class="search-highlight">`. Метод має:
1. Повертати `'—'` для null/порожнього тексту
2. Екранувати HTML-сутності (`&`, `<`, `>`) перед заміною
3. Екранувати спеціальні символи регулярних виразів у рядку фільтра
4. Використовувати заміну регулярним виразом без урахування регістру

```typescript
highlightMatch(text: string | null | undefined): string {
  if (!text) return '—';
  const filter = this.posFilter().trim().toLowerCase();
  if (!filter) return this.escapeHtml(text);

  const escaped = this.escapeHtml(text);
  const filterEscaped = this.escapeHtml(filter);
  const regex = new RegExp(
    `(${filterEscaped.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi'
  );
  return escaped.replace(regex, '<mark class="search-highlight">$1</mark>');
}
```

**Стилізація підсвічування:** використовуйте `::ng-deep` зі скоупом `:host`. Фон використовує `color-mix()` з основним кольором для автоматичної підтримки світлої/темної теми.

```scss
:host ::ng-deep .search-highlight {
  background: color-mix(in srgb, var(--color-primary) 25%, transparent);
  color: inherit;
  border-radius: 2px;
  padding: 0 1px;
}
```

**Ключові специфікації:**
- **Фон**: 25% основного кольору через `color-mix()` — адаптується до обох тем
- **Колір**: `inherit` — зберігає колір навколишнього тексту
- **Border radius**: 2px — ненав'язливе заокруглення для вбудованого підсвічування
- **Padding**: 0 1px — запобігає обрізанню тексту
- **Скоуп**: `:host ::ng-deep` обов'язковий, оскільки `<mark>` вставляється через `[innerHTML]`

**Макет панелі пошуку:**
- **Напрямок**: рядок (`flex-wrap: wrap`) — інпути розташовані поряд, переносяться на вузьких екранах
- **Проміжок**: `var(--spacing-sm)` (8px) між групами інпутів
- **Повідомлення про помилки**: використовуйте `width: 100%`, щоб винести їх в окремий рядок під інпутами

---

## Палітра кольорів

> [!WARNING]
> **Ніколи не використовуйте захардкоджені hex-кольори безпосередньо у стилях компонентів.** Завжди посилайтеся на CSS-змінні, визначені у `styles.scss`, щоб забезпечити належну підтримку світлої/темної теми.

### Кольорові ролі Material Design 3

> Базується на [Material Design 3 Color System](https://m3.material.io/styles/color/roles)

Material Design 3 використовує систему семантичних кольорових ролей. Наш B2B-застосунок зіставляє ці ролі так:

#### Основні кольори (смарагдово-зелений)
| Роль MD3 | CSS-змінна | Значення (Light) | Використання |
|----------|--------------|-------------|-------|
| Primary | `--color-primary` | #10b981 | Основний колір бренду, основні дії, FAB |
| On Primary | `--color-on-primary` | #ffffff | Текст/іконки на основних фонах |
| Primary Container | `--color-primary-container` | #d1fae5 | Тоновані фони, заливка чипів |
| On Primary Container | `--color-on-primary-container` | #064e3b | Текст на primary container |

#### Другорядні кольори (бірюзовий)
| Роль MD3 | CSS-змінна | Значення (Light) | Використання |
|----------|--------------|-------------|-------|
| Secondary | `--color-secondary` | #14b8a6 | Менш помітні дії, доповнювальні |
| On Secondary | `--color-on-secondary` | #ffffff | Текст/іконки на secondary |
| Secondary Container | `--color-secondary-container` | #ccfbf1 | Фони другорядних чипів |
| On Secondary Container | `--color-on-secondary-container` | #115e59 | Текст на secondary container |

#### Кольори поверхонь
| Роль MD3 | CSS-змінна | Значення (Light) | Використання |
|----------|--------------|-------------|-------|
| Background | `--color-background` | #f5f7fa | Фон сторінки |
| Surface | `--card-bg` | #ffffff | Картки, шторки, меню |
| Surface Variant | `--color-surface-variant` | #e8edff | Альтернативний колір поверхні |
| On Surface | `--color-text-primary` | #333333 | Основний текст на поверхнях |
| On Surface Variant | `--color-text-secondary` | #666666 | Другорядний текст на поверхнях |

#### Семантичні кольори
| Роль MD3 | CSS-змінна | Значення (Light) | Використання |
|----------|--------------|-------------|-------|
| Error | `--color-danger` | #dc3545 | Стани помилок, деструктивні дії |
| On Error | `--color-on-error` | #ffffff | Текст на фонах помилок |
| Error Container | `--color-error-container` | #f9dedc | Фони повідомлень про помилки |
| On Error Container | `--color-on-error-container` | #490909 | Текст на error container |
| Success | `--color-success` | #28a745 | Стани успіху, підтвердження |
| Warning | `--color-warning` | #ffc107 | Стани попередження, обережність |
| Info | `--color-info` | #17a2b8 | Інформаційні повідомлення |

#### Кольори обведення та меж
| Роль MD3 | CSS-змінна | Значення (Light) | Використання |
|----------|--------------|-------------|-------|
| Outline | `--color-border` | #e1e8ed | Межі компонентів, роздільники |
| Outline Variant | `--color-border-light` | #f0f4f8 | Ненав'язливі роздільники |

#### Застарілі зіставлення (наявні B2B-змінні)
| CSS-змінна | Значення (Light) | Використання |
|--------------|-------------|-------|
| `--color-primary-dark` | #5568d3 | Стани hover (використовуйте `:hover` з прозорістю натомість) |
| `--color-text-muted` | #999999 | Плейсхолдери, стани disabled |

### Вимоги до контрасту кольорів

Дотримуйтесь стандартів WCAG 2.1 AA:
- **Основний текст (< 18px)**: мінімальне співвідношення контрасту 4.5:1
- **Великий текст (≥ 18px або ≥ 14px bold)**: мінімальне співвідношення контрасту 3:1
- **UI-компоненти та межі**: мінімальне співвідношення контрасту 3:1
- **Активні елементи**: забезпечте чітку відмінність від неактивних станів

**Приклад — правильне використання:**
```scss
.my-component {
  background: var(--card-bg);  // ✅ Adapts to theme
  color: var(--color-text-primary);  // ✅ Adapts to theme
  border: 1px solid var(--color-border);  // ✅ Adapts to theme
}
```

**Приклад — неправильне використання:**
```scss
.my-component {
  background: #ffffff;  // ❌ Breaks dark mode
  color: #333;  // ❌ Breaks dark mode
  border: 1px solid #e1e8ed;  // ❌ Breaks dark mode
}
```

---

## Система іконок

> Базується на [Material Design 3 Icons](https://m3.material.io/styles/icons/designing-icons)

### Розміри іконок

| Розмір | Габарити | Використання |
|------|-----------|-------|
| **Extra Small** | 16×16px | Вбудовані іконки, бейджі, щільні списки |
| **Small** | 20×20px | Компактний UI, іконки таблиць |
| **Medium** (стандартний) | 24×24px | **Типовий розмір іконки** — кнопки, інпути, списки |
| **Large** | 32×32px | Помітні дії, великі картки |
| **Extra Large** | 48×48px | Виділення функцій, порожні стани |

### Padding зони дотику

Кнопки-іконки в цьому проєкті мають розмір **40×40** (іконка 24px + padding 8px). Див. розділ [Зони дотику](#зони-дотику-touch-targets) щодо обґрунтування використання розміру, нижчого за ідеальні 44px з WCAG.

```scss
.icon-button {
  // Icon visual size
  .icon {
    width: 24px;
    height: 24px;
  }

  // Touch target (includes padding)
  min-width: 40px;
  min-height: 40px;
  padding: 8px; // (40 - 24) / 2
}
```

### Сімейства іконок

**Material Symbols** (рекомендовано):
- **Outlined**: за замовчуванням, чистий вигляд для більшості елементів UI
- **Filled**: активні стани, вибрані елементи
- **Rounded**: дружній, привітний відчуття
- **Sharp**: сучасна, технічна естетика

### Рекомендації щодо використання іконок

> **ВАЖЛИВО: використовуйте лише Material Symbols/Icons. Ніколи не використовуйте емодзі-іконки (наприклад, ✏️, 🗑️, 💳) в UI.**

1. **Лише Material Icons**: завжди використовуйте Material Symbols Outlined для всіх іконок. Символи емодзі заборонені, оскільки вони відображаються неузгоджено на різних платформах і порушують візуальну узгодженість.
2. **Семантична ясність**: іконки мають бути миттєво впізнаваними
3. **Узгодженість**: використовуйте одне сімейство іконок у всьому застосунку (Material Symbols Outlined)
4. **Доступність**: завжди надавайте `aria-label` для кнопок лише з іконками
5. **Вирівнювання**: центруйте іконки оптично, а не математично
6. **Колір**: використовуйте `currentColor` для успадкування кольору тексту
7. **Інтервал**: проміжок 8-12px між іконкою та текстом

### Завантаження Material Symbols

Додайте до `index.html`:
```html
<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200&display=swap" rel="stylesheet">
```

### Використання у шаблонах

```html
<!-- Correct: Material Symbol -->
<span class="material-symbols-outlined">edit</span>
<span class="material-symbols-outlined">delete</span>
<span class="material-symbols-outlined">add</span>

<!-- Incorrect: Emoji (NEVER use) -->
<span>✏️</span>
<span>🗑️</span>
<span>➕</span>
```

### Патерн «іконка + текст»

```scss
.button-with-icon {
  display: flex;
  align-items: center;
  gap: 8px; // Material Design standard

  .icon {
    width: 20px;  // Slightly smaller than standalone
    height: 20px;
  }
}
```

### Стани іконок

```scss
.icon {
  color: var(--color-text-secondary);
  transition: color 0.2s ease;

  &:hover {
    color: var(--color-primary);
  }

  &.active {
    color: var(--color-primary);
  }

  &.disabled {
    color: var(--color-text-muted);
    opacity: 0.38; // Material Design disabled opacity
  }
}
```

---

## Стандарти border-radius

> Базується на системі форм Material Design 3

| Елемент | Радіус | CSS-змінна | Використання |
|---------|--------|--------------|-------|
| Малі компоненти | 4px | `--radius-sm` | Чипи, малі бейджі |
| Середні компоненти | 8px | `--radius-md` | **Кнопки, картки, інпути** (за замовчуванням) |
| Великі компоненти | 12px | `--radius-lg` | Великі картки, діалоги |
| Дуже великі | 16px | `--radius-xl` | Шторки, модальні вікна |
| Повне заокруглення | 24px+ | `--radius-full` | Пігулки, круглі кнопки |

### Токени форм

```scss
:root {
  --radius-none: 0;
  --radius-sm: 4px;
  --radius-md: 8px;   // Default
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-2xl: 20px;
  --radius-full: 9999px; // Pill shape
}
```

### Радіуси для конкретних компонентів

| Компонент | Радіус | Примітка |
|-----------|--------|------|
| Кнопки | 8px (`--radius-md`) | Стандартні, на всю ширину |
| Текстові інпути | 8px (`--radius-md`) | Узгодженість з кнопками |
| Картки | 8px (`--radius-md`) | Стандартна елевація |
| Підняті картки | 12px (`--radius-lg`) | Помітніші |
| Бейджі/Чипи | 20px (`--radius-full`) | Форма пігулки |
| Шторки | 12px (лише верх) | `border-radius: 12px 12px 0 0` |
| Діалоги/Модальні | 12px (`--radius-lg`) | Усі кути |
| Аватар | 50% / `--radius-full` | Круглий |

---

## Елевація та тіні

> Базується на системі елевації Material Design 3

### Рівні елевації

Material Design використовує 5 рівнів елевації для ієрархії глибини:

| Рівень | Елевація | CSS-змінна | Специфікація тіні | Використання |
|-------|-----------|--------------|-------------|-------|
| **0** | 0dp | `--elevation-0` | Немає | Плоскі поверхні, фони |
| **1** | 1dp | `--elevation-1` | 0 1px 2px rgba(0,0,0,0.08) | Картки, підняті елементи |
| **2** | 3dp | `--elevation-2` | 0 2px 8px rgba(0,0,0,0.12) | Картки при hover, випадаючі меню |
| **3** | 6dp | `--elevation-3` | 0 4px 16px rgba(0,0,0,0.16) | Діалоги, навігаційні шухляди |
| **4** | 8dp | `--elevation-4` | 0 8px 24px rgba(0,0,0,0.20) | Модальні оверлеї |
| **5** | 12dp | `--elevation-5` | 0 12px 32px rgba(0,0,0,0.24) | FAB, тултіпи при hover |

### Визначення тіней

```scss
:root {
  // Elevation shadows
  --elevation-0: none;
  --elevation-1: 0 1px 2px 0 rgba(0, 0, 0, 0.08);
  --elevation-2: 0 2px 8px 0 rgba(0, 0, 0, 0.12);
  --elevation-3: 0 4px 16px 0 rgba(0, 0, 0, 0.16);
  --elevation-4: 0 8px 24px 0 rgba(0, 0, 0, 0.20);
  --elevation-5: 0 12px 32px 0 rgba(0, 0, 0, 0.24);

  // Focus ring
  --focus-ring: 0 0 0 3px rgba(16, 185, 129, 0.2);
}
```

### Рекомендації щодо використання

**Перевага застосунку**: використовуйте **ненав'язливі межі** замість важких тіней для чистого, професійного вигляду:

```scss
// ✓ Preferred 
.card {
  border: 1px solid var(--color-border);
  box-shadow: none; // Or very subtle --elevation-1
}

// ✗ Avoid heavy shadows
.card {
  box-shadow: 0 10px 40px rgba(0,0,0,0.3); // Too dramatic
}
```

**Коли використовувати тіні**:
- **Підняті стани**: hover, focus, active
- **Плаваючі елементи**: модальні вікна, тултіпи, випадаючі меню
- **Чітке нашарування**: коли ієрархія z-index потребує візуального підкріплення

**Патерн переходу**:
```scss
.card {
  box-shadow: var(--elevation-1);
  transition: box-shadow 0.2s ease;

  &:hover {
    box-shadow: var(--elevation-2);
  }
}
```

---

## Адаптивні брейкпоінти

> Базується на [Material Design 3 Layout System](https://m3.material.io/foundations/layout/applying-layout)

### Класи розмірів вікна Material Design 3

Material Design 3 використовує **класи розмірів вікна** замість традиційних піксельних брейкпоінтів:

| Клас | Діапазон ширини | Колонки | Margins | Gutters | Опис |
|-------|-------------|---------|---------|---------|-------------|
| **Compact** | < 600px | 4 | 16px | 16px | Телефони (портретна) |
| **Medium** | 600-839px | 8 | 24px | 24px | Планшети (портретна), великі телефони |
| **Expanded** | ≥ 840px | 12 | 24px | 24px | Планшети (альбомна), десктоп |

### Брейкпоінти B2B-застосунку

Наш застосунок використовує такі практичні брейкпоінти:

| Брейкпоінт | Ширина | Клас вікна | Використання |
|------------|-------|--------------|-------|
| **xs** | < 480px | Compact | Малі телефони |
| **sm** | 480-767px | Compact | Стандартні телефони |
| **md** | 768-1023px | Medium | Планшети (портретна) |
| **lg** | 1024-1279px | Expanded | Планшети (альбомна), малі десктопи |
| **xl** | ≥ 1280px | Expanded | Десктоп |

### Використання брейкпоінтів у SCSS

```scss
// Mobile-first approach (Material Design recommended)
.component {
  // Base styles (mobile)
  padding: 12px;
  font-size: 14px;

  // Tablet
  @media (min-width: 768px) {
    padding: 16px;
    font-size: 15px;
  }

  // Desktop
  @media (min-width: 1024px) {
    padding: 24px;
    font-size: 16px;
  }
}
```

### Специфікації сітки за брейкпоінтами

```scss
// Compact (< 600px) - Phones
@media (max-width: 599px) {
  .container {
    padding: 0 16px;           // 16px margins
    gap: 16px;                 // 16px gutters
    grid-template-columns: repeat(4, 1fr);
  }
}

// Medium (600-839px) - Large phones, tablets
@media (min-width: 600px) and (max-width: 839px) {
  .container {
    padding: 0 24px;           // 24px margins
    gap: 24px;                 // 24px gutters
    grid-template-columns: repeat(8, 1fr);
  }
}

// Expanded (≥ 840px) - Desktop
@media (min-width: 840px) {
  .container {
    padding: 0 24px;           // 24px margins
    gap: 24px;                 // 24px gutters
    grid-template-columns: repeat(12, 1fr);
  }
}
```

### Адаптивна типографіка

```scss
// Adjust font sizes across breakpoints
h1 {
  font-size: 22px;             // Mobile
  line-height: 28px;

  @media (min-width: 768px) {
    font-size: 28px;           // Tablet
    line-height: 36px;
  }

  @media (min-width: 1024px) {
    font-size: 32px;           // Desktop
    line-height: 40px;
  }
}
```

---

## Обробка безпечних зон (Safe Area)

Завжди враховуйте пристрої з вирізами (notch):

```scss
.footer, .bottom-sheet {
  padding-bottom: env(safe-area-inset-bottom, 0);
  // Or add to existing padding:
  padding-bottom: calc(12px + env(safe-area-inset-bottom, 0));
}
```

---

## Стани компонентів та взаємодії

> Базується на станах взаємодії Material Design 3

### Система шарів стану (State Layer)

Material Design використовує **шари стану** — напівпрозорі оверлеї, що позначають інтерактивні стани:

| Стан | Прозорість | Тривалість | Використання |
|-------|---------|----------|-------|
| **Hover** | 8% (0.08) | 200ms | Наведення вказівника на десктопі |
| **Focus** | 12% (0.12) | Миттєво | Фокус клавіатури/доступності |
| **Pressed** | 12% (0.12) | 100ms | Активне натискання/тап |
| **Dragged** | 16% (0.16) | 200ms | Операції drag-and-drop |
| **Disabled** | 38% (0.38) | - | Неактивні елементи (знижена прозорість) |

### Стани інтерактивних елементів

```scss
.interactive-element {
  position: relative;
  transition: background-color 0.2s ease, box-shadow 0.2s ease;

  // Default state
  background: var(--card-bg);
  color: var(--color-text-primary);

  // Hover state (8% overlay)
  &:hover:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 8%, var(--card-bg));
  }

  // Focus state (12% overlay + focus ring)
  &:focus:not(:disabled) {
    outline: none;
    background: color-mix(in srgb, var(--color-primary) 12%, var(--card-bg));
    box-shadow: var(--focus-ring);
  }

  // Active/Pressed state (12% overlay)
  &:active:not(:disabled) {
    background: color-mix(in srgb, var(--color-primary) 12%, var(--card-bg));
  }

  // Disabled state (38% opacity)
  &:disabled {
    opacity: 0.38;
    cursor: not-allowed;
    pointer-events: none;
  }
}
```

### Стани кнопок (повний патерн)

```scss
.button {
  // Base styles
  padding: 12px 24px;
  border: none;
  border-radius: var(--radius-md);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s ease, box-shadow 0.2s ease;

  // Primary button
  &.primary {
    background: var(--color-primary);
    color: var(--color-on-primary);

    &:hover:not(:disabled) {
      background: color-mix(in srgb, #000 8%, var(--color-primary));
      box-shadow: var(--elevation-2);
    }

    &:focus:not(:disabled) {
      box-shadow: var(--focus-ring);
    }

    &:active:not(:disabled) {
      background: color-mix(in srgb, #000 12%, var(--color-primary));
      box-shadow: var(--elevation-1);
    }
  }

  // Secondary button (outlined)
  &.secondary {
    background: transparent;
    color: var(--color-primary);
    border: 2px solid var(--color-primary);

    &:hover:not(:disabled) {
      background: color-mix(in srgb, var(--color-primary) 8%, transparent);
    }

    &:active:not(:disabled) {
      background: color-mix(in srgb, var(--color-primary) 12%, transparent);
    }
  }

  // Disabled state
  &:disabled {
    background: var(--color-border);
    color: var(--color-text-muted);
    cursor: not-allowed;
    box-shadow: none;
  }
}
```

### Стани полів вводу

```scss
input,
select,
textarea {
  // Base styles
  border: 2px solid var(--color-border);
  transition: border-color 0.2s ease, box-shadow 0.2s ease;

  // Hover state
  &:hover:not(:disabled):not(:focus) {
    border-color: color-mix(in srgb, var(--color-primary) 40%, var(--color-border));
  }

  // Focus state
  &:focus:not(:disabled) {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: var(--focus-ring);
  }

  // Error state
  &.invalid,
  &.is-invalid {
    border-color: var(--color-danger);

    &:focus {
      box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.15);
    }
  }

  // Disabled state
  &:disabled {
    background: var(--color-surface-variant);
    border-color: var(--color-border-light);
    color: var(--color-text-muted);
    cursor: not-allowed;
    opacity: 0.6;
  }
}
```

### Стандарти переходів

```scss
// Standard timing functions
:root {
  --duration-short: 100ms;    // Quick actions (press)
  --duration-medium: 200ms;   // Standard transitions (hover)
  --duration-long: 300ms;     // Complex animations (panel slide)
  --duration-xl: 400ms;       // Page transitions

  --easing-standard: cubic-bezier(0.4, 0.0, 0.2, 1);      // Default
  --easing-decelerate: cubic-bezier(0.0, 0.0, 0.2, 1);   // Enter screen
  --easing-accelerate: cubic-bezier(0.4, 0.0, 1, 1);     // Exit screen
}

// Usage
.element {
  transition: all var(--duration-medium) var(--easing-standard);
}
```

### Вимоги до доступності

1. **Індикатори фокуса**: завжди видимі, мінімальний контраст 3:1
2. **Зміни стану**: повідомляються скрінрідерам через ARIA
3. **Зони дотику**: мінімум 40×40px для кнопок-іконок, 48×48px для основних CTA (див. [Зони дотику](#зони-дотику-touch-targets))
4. **Незалежність від кольору**: ніколи не покладайтеся лише на колір для передачі стану
5. **Навігація клавіатурою**: усі інтерактивні елементи мають бути доступними з клавіатури

---

## Що робити і чого уникати

### Узгодженість кнопок та панелі дій ✓

**Робіть:**
- ✓ Використовуйте **плоскі кнопки з обведенням** в усіх панелях дій та тулбарах
- ✓ Використовуйте **градієнтні кнопки** лише для основних CTA (максимум 1 на екран)
- ✓ Використовуйте **filled tonal кнопки** для другорядних важливих дій
- ✓ Використовуйте **текстові кнопки** для дій скасування/закриття/третинних
- ✓ Використовуйте **кнопки-іконки** для дій тулбара та компактних UI
- ✓ Підтримуйте узгоджені висоти кнопок: 48px (основний CTA), 40px (панель дій, кнопки-іконки)
- ✓ Зберігайте чітку ієрархію кнопок: видима лише 1 дія з високим акцентом
- ✓ Використовуйте 2-кольорові градієнти під кутами 45-135°
- ✓ Забезпечте контраст тексту 4.5:1 на градієнтних фонах
- ✓ Тестуйте градієнти на мобільних пристроях
- ✓ Обмежте панель дій 3 видимими діями (використовуйте оверфлоу-меню)
- ✓ Використовуйте кнопки лише з іконками в мобільних панелях дій
- ✓ Підтримуйте узгоджений інтервал: проміжки 8-12px між кнопками панелі дій
- ✓ Додавайте належні ARIA-мітки до кнопок-іконок

**Не робіть:**
- ✗ Не використовуйте градієнтні кнопки в панелях дій або тулбарах
- ✗ Не використовуйте декілька градієнтних кнопок на одному екрані
- ✗ Не змішуйте стилі кнопок в одному контексті (усі кнопки панелі дій мають бути плоскими)
- ✗ Не використовуйте градієнти для станів disabled
- ✗ Не створюйте градієнти з більш ніж 3 кольорами
- ✗ Не використовуйте конфліктні кольори градієнта
- ✗ Не робіть кнопки панелі дій вищими за 40px
- ✗ Не використовуйте кнопки лише з текстом для основних дій
- ✗ Не розміщуйте більше ніж 3 дії в мобільній панелі дій без оверфлоу
- ✗ Не використовуйте різні стилі кнопок для однакового типу дії на різних екранах
- ✗ Не прибирайте індикатори фокуса з кнопок
- ✗ Не використовуйте градієнти на малих кнопках (висота <40px)

### Макет та відступи ✓

**Робіть:**
- ✓ Використовуйте кратні 8px значення сітки (8, 16, 24, 32, 40, 48px)
- ✓ Використовуйте проміжки 8px між елементами списку на мобільних
- ✓ Тримайте padding секцій на рівні 12-16px
- ✓ Використовуйте CSS-змінні `--spacing-*` замість захардкоджених значень
- ✓ Застосовуйте узгоджені відступи для схожих компонентів
- ✓ Використовуйте властивість `gap` для flex/grid-макетів
- ✓ Підтримуйте візуальний ритм через узгоджені відступи

**Не робіть:**
- ✗ Не використовуйте довільні значення відступів (наприклад, 13px, 27px)
- ✗ Не використовуйте проміжки більші за 16px на мобільних
- ✗ Не додавайте надмірний padding (>24px) до мобільних карток
- ✗ Не змішуйте різні системи відступів
- ✗ Не використовуйте від'ємні margin для коригування макета

### Типографіка ✓

**Робіть:**
- ✓ Використовуйте Montserrat для всіх заголовків (h1-h6)
- ✓ Використовуйте Commissioner для всього основного тексту, міток, кнопок
- ✓ Дотримуйтесь шкали типів Material Design (Title Large, Body Medium тощо)
- ✓ Підтримуйте співвідношення висоти рядка 1.4-1.5× (наприклад, 14px/20px)
- ✓ Використовуйте мінімум 14px для основного тексту на мобільних (Commissioner)
- ✓ Обмежтеся 3 рівнями заголовків на екран
- ✓ Забезпечте контраст 4.5:1 для основного тексту
- ✓ Використовуйте накреслення шрифтів узгоджено:
  - Montserrat: 700 (bold), 800 (extra-bold для h1 на десктопі)
  - Commissioner: 400 (regular), 500 (medium), 600 (semi-bold)
- ✓ Тестуйте український текст для перевірки відображення лігатури ії
- ✓ Використовуйте варіативні шрифти для оптимальної продуктивності

**Не робіть:**
- ✗ Не використовуйте розміри шрифтів менші за 11px
- ✗ Не створюйте більше ніж 4 варіанти типографіки на екран
- ✗ Не використовуйте власний letter-spacing без потреби
- ✗ Не змішуйте різні шкали накреслень шрифтів
- ✗ Не використовуйте все великими літерами для довгих блоків тексту
- ✗ Не використовуйте Montserrat для основного тексту (надто важкий для читання)
- ✗ Не використовуйте Commissioner для великих заголовків (бракує виразності)
- ✗ Не завантажуйте шрифти без параметра `&subset=latin,cyrillic`
- ✗ Не використовуйте накреслення важчі за 800 для українського тексту (щільні акценти)

### Колір та теми ✓

**Робіть:**
- ✓ Завжди використовуйте CSS-змінні (`var(--color-primary)`)
- ✓ Дотримуйтесь кольорових ролей Material Design 3
- ✓ Забезпечте мінімальний контраст 3:1 для UI-компонентів
- ✓ Використовуйте семантичні назви кольорів (primary, error, success)
- ✓ Тестуйте дизайн у світлій та темній темах
- ✓ Використовуйте `color-mix()` для оверлеїв станів (hover, focus)

**Не робіть:**
- ✗ Не хардкодьте hex-кольори у стилях компонентів
- ✗ Не використовуйте колір як єдиний засіб передачі інформації
- ✗ Не створюйте інтерфейси з низьким контрастом
- ✗ Не використовуйте більше ніж 3 кольори бренду
- ✗ Не перевизначайте кольори теми за допомогою `!important`

### Зони дотику та доступність ✓

**Робіть:**
- ✓ Використовуйте 40×40 для кнопок-іконок, 48×48 для основних CTA
- ✓ Додавайте padding 8px навколо іконок 24px (40 - 24 = 16, розділіть на кожну сторону)
- ✓ Розділяйте зони дотику принаймні на 8px
- ✓ Надавайте індикатори фокуса (мінімум 3px, контраст 3:1)
- ✓ Використовуйте `aria-label` для кнопок лише з іконками
- ✓ Забезпечте роботу навігації клавіатурою

**Не робіть:**
- ✗ Не створюйте кнопки-іконки менші за 40px (action-panel-button з 38px — нещодавній приклад, якого слід уникати)
- ✗ Не розміщуйте інтерактивні елементи надто близько один до одного (<8px)
- ✗ Не прибирайте обведення фокуса без заміни
- ✗ Не використовуйте іконки без текстових міток у критичних діях
- ✗ Не покладайтеся на стани hover для мобільних інтерфейсів

### Візуальний стиль ✓

**Робіть:**
- ✓ Використовуйте border-radius 8px узгоджено для карток/кнопок
- ✓ Надавайте перевагу ненав'язливим межам перед важкими тінями (B2B)
- ✓ Використовуйте `--elevation-1` або `--elevation-2` для карток
- ✓ Застосовуйте плавні переходи (стандартно 200ms)
- ✓ Використовуйте шари стану для інтерактивного зворотного зв'язку
- ✓ Підтримуйте візуальну узгодженість компонентів

**Не робіть:**
- ✗ Не використовуйте важкі тіні (> `--elevation-3`)
- ✗ Не змішуйте різні значення border-radius довільно
- ✗ Не переанімовуйте інтерфейси (відволікає)
- ✗ Не використовуйте градієнти надмірно
- ✗ Не створюйте власні тіні замість використання токенів елевації

### Іконки ✓

**Робіть:**
- ✓ Використовуйте **Material Symbols Outlined** виключно для всіх іконок
- ✓ Використовуйте 24px як типовий розмір іконки
- ✓ Використовуйте узгоджене сімейство іконок (outlined, filled, rounded)
- ✓ Центруйте іконки оптично, а не математично
- ✓ Використовуйте `currentColor` для заливки іконок
- ✓ Додавайте проміжок 8-12px між іконкою та текстом
- ✓ Використовуйте filled-іконки для активних/вибраних станів

**Не робіть:**
- ✗ **НІКОЛИ не використовуйте емодзі-іконки** (наприклад, ✏️, 🗑️, 💳, ➕) — вони відображаються неузгоджено
- ✗ Не змішуйте різні стилі іконок в одному інтерфейсі
- ✗ Не використовуйте іконки менші за 16px
- ✗ Не створюйте власні іконки, що не відповідають системі
- ✗ Не використовуйте іконки без чіткого значення
- ✗ Не розміщуйте іконки неузгоджено (іноді зліва, іноді справа)

### Форми та інпути ✓

**Робіть:**
- ✓ Використовуйте стандартний padding інпутів: 8px (`--spacing-sm`)
- ✓ Використовуйте межі 2px для чітких меж
- ✓ Показуйте чіткі стани фокуса з обведенням + тінню ring
- ✓ Надавайте зворотний зв'язок валідації миттєво
- ✓ Використовуйте плейсхолдер-текст помірно
- ✓ Групуйте пов'язані інпути логічно

**Не робіть:**
- ✗ Не використовуйте межі 1px (надто ненав'язливі на мобільних)
- ✗ Не прибирайте обведення при фокусі (знижує чіткість)
- ✗ Не приховуйте мітки на користь лише плейсхолдерів
- ✗ Не валідуйте на кожне натискання клавіші (дратує)
- ✗ Не використовуйте червоні обведення без повідомлень про помилки

### Підсвічування пошуку ✓

**Робіть:**
- ✓ Використовуйте `color-mix(in srgb, var(--color-primary) 25%, transparent)` для фону підсвічування
- ✓ Використовуйте `color: inherit`, щоб зберегти колір навколишнього тексту
- ✓ Екрануйте HTML перед обгортанням збігів у теги `<mark>`
- ✓ Екрануйте спеціальні символи регулярних виразів у рядку фільтра
- ✓ Використовуйте `:host ::ng-deep` для стилізації розмітки, вставленої через `[innerHTML]`
- ✓ Використовуйте провідну іконку `filter_list` для інпутів клієнтського фільтра (без кнопки надсилання)
- ✓ Розміщуйте декілька інпутів пошуку горизонтально з `flex-wrap: wrap`

**Не робіть:**
- ✗ Не використовуйте захардкоджені кольори підсвічування (порушує темну тему)
- ✗ Не використовуйте парсинг у стилі `new Date()` для тексту підсвічування (використовуйте зіставлення сирих рядків)
- ✗ Не вставляйте неекранований текст користувача в `[innerHTML]` (ризик XSS)
- ✗ Не додавайте кнопку надсилання до інпутів клієнтського фільтра (фільтрація має бути миттєвою)

---

## Система тем

> Повна підтримка світлої/темної теми з CSS-змінними та визначенням системних налаштувань

### Огляд

Застосунок підтримує три режими теми:
- **Light**: типова світла тема
- **Dark**: темна тема для умов слабкого освітлення
- **System**: автоматично слідує за налаштуваннями операційної системи

### Архітектура

Токени теми організовані у два файли:
- `src/styles/_themes.scss`: усі кольорові токени для світлої та темної теми
- `src/styles/_variables.scss`: токени, що не залежать від теми (типографіка, відступи, макет)

### Категорії токенів

| Категорія | Приклади токенів | Використання |
|----------|---------------|-------|
| **Primary** | `--color-primary`, `--color-on-primary` | Кольори бренду, основні дії |
| **Surface** | `--color-background`, `--card-bg` | Фони сторінок і компонентів |
| **Text** | `--color-text-primary`, `--color-text-muted` | Кольори типографіки |
| **Semantic** | `--color-success`, `--color-error` | Статус і зворотний зв'язок |
| **Badge** | `--badge-success-bg`, `--badge-success-text` | Бейджі статусу |
| **Status** | `--status-online-bg`, `--status-online-text` | Статус каси/зміни |
| **Sidebar** | `--sidebar-bg`, `--sidebar-text` | Навігаційна бічна панель |
| **Header** | `--header-bg`, `--header-text` | Шапка застосунку |
| **Brand** | `--brand-viber`, `--brand-telegram` | Кольори сторонніх брендів |

### Використання ThemeService

```typescript
import { ThemeService } from '@app/core/services/theme.service';

@Component({...})
export class MyComponent {
  readonly themeService = inject(ThemeService);

  // Check current theme
  isDark = this.themeService.isDarkMode(); // signal: boolean
  resolved = this.themeService.resolvedTheme(); // signal: 'light' | 'dark'
  preference = this.themeService.themePreference(); // signal: 'light' | 'dark' | 'system'

  // Change theme
  setLight() { this.themeService.setTheme('light'); }
  setDark() { this.themeService.setTheme('dark'); }
  setSystem() { this.themeService.setTheme('system'); }
  toggle() { this.themeService.toggleTheme(); }
  cycle() { this.themeService.cycleTheme(); } // light -> dark -> system -> light

  // Get icon/label for UI
  icon = this.themeService.getThemeIcon(); // 'light_mode' | 'dark_mode' | 'contrast'
  label = this.themeService.getThemeLabel(); // 'Light' | 'Dark' | 'System'
}
```

### Додавання кнопки перемикання теми

```html
<button
  class="theme-toggle"
  (click)="themeService.cycleTheme()"
  [attr.aria-label]="'Theme: ' + themeService.getThemeLabel()"
>
  <span class="material-symbols-outlined">{{ themeService.getThemeIcon() }}</span>
</button>
```

### Додавання нових кольорів

Додаючи нові кольори, **завжди визначайте їх для обох тем** у `_themes.scss`:

```scss
// In :root (light theme)
:root {
  --my-new-color-bg: #e0f2fe;
  --my-new-color-text: #0369a1;
}

// In [data-theme="dark"]
[data-theme="dark"] {
  --my-new-color-bg: #0c4a6e;
  --my-new-color-text: #7dd3fc;
}

// In @media (prefers-color-scheme: dark) :root:not([data-theme])
// (same values as [data-theme="dark"])
```

### Правила тематизації

**РОБІТЬ:**
- Завжди використовуйте CSS-змінні для кольорів: `var(--color-primary)`
- Визначайте нові кольори для обох тем — світлої та темної
- Використовуйте семантичні назви токенів (`--color-success`, а не `--color-green`)
- Використовуйте `color-mix()` для варіацій прозорості: `color-mix(in srgb, var(--color-primary) 15%, transparent)`
- Тестуйте всі компоненти в обох темах

**НЕ РОБІТЬ:**
- Ніколи не хардкодьте hex-кольори: `#10b981` (використовуйте `var(--color-primary)`)
- Ніколи не використовуйте `rgba()` із захардкодженими кольорами
- Ніколи не пропускайте темну тему під час додавання нових кольорів
- Ніколи не використовуйте `!important` для перевизначення кольорів теми

### Запобігання FOUC

Тема застосовується до гідрації Angular через вбудований скрипт у `index.html`:

```html
<script>
  (function() {
    var stored = localStorage.getItem('webcheck-theme-preference');
    var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    if (stored === 'dark' || (stored === 'system' && prefersDark) || (!stored && prefersDark)) {
      document.documentElement.setAttribute('data-theme', 'dark');
    }
  })();
</script>
```

### Довідка ключових токенів дизайну

| Токен | Light | Dark |
|-------|-------|------|
| `--color-primary` | #10b981 | #10b981 |
| `--color-background` | #f5f7fa | #0f172a |
| `--card-bg` | #ffffff | #1e293b |
| `--color-text-primary` | #1e293b | #f1f5f9 |
| `--sidebar-bg` | #1e293b | #0f172a |
| `--header-bg` | #10b981 | #1e293b |
| `--badge-success-bg` | #d1fae5 | #064e3b |
| `--badge-success-text` | #065f46 | #6ee7b7 |

### Стани фокуса

Обведення фокуса адаптуються до теми:
- Light: `0 0 0 3px rgba(16, 185, 129, 0.2)`
- Dark: `0 0 0 3px rgba(16, 185, 129, 0.4)` (яскравіше для видимості)

### Елевація (тіні)

Тіні сильніші в темній темі для видимості:
- Light `--elevation-2`: `0 2px 8px 0 rgba(0, 0, 0, 0.1)`
- Dark `--elevation-2`: `0 2px 8px 0 rgba(0, 0, 0, 0.4)`

---

## Сучасні можливості CSS та найкращі практики

### CSS-змінні (Custom Properties)

Завжди використовуйте CSS-змінні для значень, що залежать від теми:

```scss
:root {
  // Spacing scale
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;

  // Colors (semantic naming)
  --color-primary: #10b981;
  --color-on-primary: #ffffff;
  --color-surface: #ffffff;
  --color-on-surface: #333333;

  // Typography
  --font-size-body-m: 14px;
  --line-height-body-m: 20px;
  --font-weight-regular: 400;
  --font-weight-medium: 500;

  // Elevation
  --elevation-1: 0 1px 2px 0 rgba(0, 0, 0, 0.08);

  // Border radius
  --radius-md: 8px;

  // Transitions
  --duration-short: 100ms;
  --duration-medium: 200ms;
  --easing-standard: cubic-bezier(0.4, 0.0, 0.2, 1);
}
```

### Сучасний макет: Flexbox та Grid

```scss
// Flexbox - for one-dimensional layouts
.flex-container {
  display: flex;
  gap: var(--spacing-md);              // Modern gap property
  align-items: center;
  justify-content: space-between;
}

// Grid - for two-dimensional layouts
.grid-container {
  display: grid;
  gap: var(--spacing-md);              // Consistent spacing
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
}

// Avoid floats and positioning hacks
```

### Змішування кольорів (сучасні оверлеї станів)

Використовуйте `color-mix()` для шарів стану (підтримується сучасними браузерами):

```scss
.button:hover {
  // Mix 8% black into primary color
  background: color-mix(in srgb, #000 8%, var(--color-primary));
}

// Fallback for older browsers
@supports not (background: color-mix(in srgb, #000 8%, white)) {
  .button:hover {
    background: var(--color-primary-dark);
  }
}
```

### Логічні властивості (інтернаціоналізація)

Використовуйте логічні властивості для кращої підтримки RTL:

```scss
// ✓ Logical (direction-agnostic)
.element {
  margin-inline-start: 16px;     // Left in LTR, right in RTL
  margin-inline-end: 16px;
  padding-block: 12px;           // Top and bottom
  border-inline-start: 2px solid;
}

// ✗ Physical (direction-specific)
.element {
  margin-left: 16px;             // Always left
  margin-right: 16px;
  padding-top: 12px;
  padding-bottom: 12px;
  border-left: 2px solid;
}
```

### Контейнерні запити (просунуте)

Для адаптивного дизайну на рівні компонента:

```scss
.card-container {
  container-type: inline-size;
  container-name: card;
}

.card {
  padding: var(--spacing-sm);

  // Respond to container size, not viewport
  @container card (min-width: 400px) {
    padding: var(--spacing-md);
    display: grid;
    grid-template-columns: auto 1fr;
  }
}
```

### Шари каскаду (організація)

Керуйте специфічністю CSS за допомогою `@layer`:

```scss
@layer reset, base, components, utilities;

@layer base {
  h1 { font-size: 22px; }
}

@layer components {
  .button { /* Component styles */ }
}

@layer utilities {
  .text-center { text-align: center; }
}
```

### Сучасні одиниці

```scss
// ✓ Use relative units
font-size: 1rem;           // Respects user's browser settings
width: clamp(280px, 90vw, 1200px);  // Fluid sizing with limits
gap: 1rem;                 // Scales with root font size

// ✓ Viewport units
height: 100vh;             // Full viewport height
height: 100dvh;            // Dynamic viewport (accounts for mobile bars)

// ✗ Avoid fixed pixel sizing for everything
width: 320px;              // Doesn't scale
```

### CSS-вкладеність (сучасна альтернатива SCSS)

Нативна CSS-вкладеність (підтримується сучасними браузерами):

```css
.button {
  padding: 12px 24px;
  background: var(--color-primary);

  &:hover {
    background: var(--color-primary-dark);
  }

  &:disabled {
    opacity: 0.38;
  }

  .icon {
    margin-inline-end: 8px;
  }
}
```

### Найкращі практики продуктивності

```scss
// ✓ Use transforms for animations (GPU-accelerated)
.element {
  transition: transform 0.2s ease;
}
.element:hover {
  transform: translateY(-2px);
}

// ✗ Avoid animating layout properties
.element {
  transition: margin 0.2s ease;  // Triggers layout reflow
}
.element:hover {
  margin-top: -2px;
}

// ✓ Use will-change for heavy animations
.animated-element {
  will-change: transform;
  animation: slide-in 0.3s ease;
}

// ✓ Contain layout where possible
.isolated-component {
  contain: layout style paint;
}
```

---

## Контрольний список реалізації

Під час створення нових мобільних макетів:

### Макет та відступи
- [ ] Використовуйте кратні сітці 8dp значення (8, 16, 24, 32, 40, 48px)
- [ ] Padding контейнера: 12-16px по горизонталі
- [ ] Padding секції/картки: 12-16px (8px для компактних)
- [ ] Проміжок списку: 8px між елементами
- [ ] Використовуйте CSS-змінні для відступів (`--spacing-sm`, `--spacing-md`)
- [ ] Застосовуйте властивість `gap` для flex/grid-макетів

### Типографіка
- [ ] Використовуйте Montserrat для всіх заголовків (h1-h6)
- [ ] Використовуйте Commissioner для всього основного тексту, міток та кнопок
- [ ] Дотримуйтесь шкали типів Material Design (Title Large, Body Medium, Label Small)
- [ ] Основний текст: мінімум 14px (Body Medium, Commissioner)
- [ ] Мітки: мінімум 12px (Label Medium, Commissioner)
- [ ] Висота рядка: співвідношення 1.4-1.5× (наприклад, 14px/20px)
- [ ] Максимум 3 рівні заголовків на екран
- [ ] Забезпечте контраст 4.5:1 для основного тексту
- [ ] Тестуйте український текст на коректне відображення ії (лігатура Commissioner)

### Кольори та теми
- [ ] Використовуйте CSS-змінні (без захардкоджених hex-кольорів)
- [ ] Дотримуйтесь кольорових ролей Material Design (primary, on-primary, surface)
- [ ] Тестуйте у світлій та темній темах
- [ ] Забезпечте мінімальний контраст 3:1 для UI-компонентів
- [ ] Використовуйте семантичні назви кольорів

### Зони дотику та доступність
- [ ] Кнопки-іконки: іконка 24px + padding 8px = разом 40×40
- [ ] Основні CTA: мінімум 48×48
- [ ] Розділяйте інтерактивні елементи на >= 8px
- [ ] Індикатори фокуса видимі (ring 3px, контраст 3:1)
- [ ] Усі кнопки доступні з клавіатури
- [ ] Кнопки лише з іконками мають `aria-label`

### Візуальний стиль
- [ ] Border-radius: 8px (`--radius-md`) для кнопок, карток, інпутів
- [ ] Border-radius: 12px (`--radius-lg`) для шторок, діалогів
- [ ] Використовуйте ненав'язливі межі замість важких тіней (перевага B2B)
- [ ] Елевація: `--elevation-1` для карток (за потреби)
- [ ] Плавні переходи: стандартно 200ms (`--duration-medium`)

### Іконки
- [ ] Типовий розмір іконки: 24×24px
- [ ] Узгоджене сімейство іконок (outlined/filled/rounded)
- [ ] Іконки використовують `currentColor` для заливки
- [ ] Проміжок 8-12px між іконкою та текстом
- [ ] Padding зони дотику навколо іконок

### Кнопки та дії
- [ ] Використовуйте градієнтні кнопки лише для основних CTA (максимум 1 на екран)
- [ ] Використовуйте плоскі кнопки з обведенням в усіх панелях дій/тулбарах
- [ ] Використовуйте filled tonal кнопки для другорядних важливих дій
- [ ] Використовуйте текстові кнопки для дій скасування/закриття/третинних
- [ ] Використовуйте кнопки-іконки (40×40) для дій тулбара
- [ ] Висоти кнопок: 48px (основний CTA), 40px (панель дій, кнопки-іконки)
- [ ] Шрифт кнопки: Commissioner, накреслення 500, 14px
- [ ] Градієнт: 2 кольори, кут 135°, контраст тексту 4.5:1
- [ ] Висота панелі дій: 56px лише з плоскими кнопками
- [ ] Максимум 3 видимі дії в мобільній панелі дій
- [ ] Узгоджені стилі кнопок для однакових типів дій
- [ ] Належні ARIA-мітки на кнопках-іконках

### Форми та інпути
- [ ] Padding інпута: 8px (`--spacing-sm`)
- [ ] Межа: 2px solid для чіткості
- [ ] Стан focus: колір обведення + тінь focus ring
- [ ] Стан invalid: обведення кольору danger
- [ ] Стан disabled: прозорість 0.6 + приглушені кольори
- [ ] Мітки видимі (не лише плейсхолдери)

### Стани компонентів
- [ ] Стан hover: оверлей 8% (лише десктоп)
- [ ] Стан focus: оверлей 12% + focus ring
- [ ] Стан active/pressed: оверлей 12%
- [ ] Стан disabled: прозорість 38%
- [ ] Переходи станів: 200ms ease

### Адаптивний дизайн
- [ ] Підхід mobile-first
- [ ] Брейкпоінти: 480px (xs), 768px (md), 1024px (lg)
- [ ] Тестуйте на compact (< 600px), medium (600-839px), expanded (≥ 840px)
- [ ] Коригуйте типографіку за брейкпоінтами
- [ ] Використовуйте класи розмірів вікна для зміщень макета

### Безпечні зони та особливі випадки
- [ ] Шторки: обробка безпечної зони `env(safe-area-inset-bottom)`
- [ ] Згортувані списки для > 5 елементів
- [ ] Кнопки «Показати більше» з відображенням кількості
- [ ] Належні стани завантаження та помилок

### Пошук та фільтр
- [ ] Інпути клієнтського фільтра використовують провідну іконку, без кнопки надсилання
- [ ] Декілька інпутів пошуку в горизонтальному рядку (`flex-wrap: wrap`)
- [ ] Підсвічування використовує `color-mix()` з `--color-primary` (безпечно для теми)
- [ ] HTML екранований перед вставкою `<mark>` (запобігання XSS)
- [ ] Спеціальні символи регулярних виразів екрановані в рядку фільтра
- [ ] `::ng-deep` зі скоупом `:host` для стилів `[innerHTML]`
- [ ] Фільтр очищується, коли панель пошуку закривається

### Якість коду
- [ ] Використовуйте сучасні можливості CSS (flexbox, grid, gap)
- [ ] Уникайте float та clearfix-хаків
- [ ] Використовуйте логічні властивості для підтримки RTL
- [ ] Належний семантичний HTML (заголовки, кнопки, інпути)
- [ ] Без inline-стилів

---

## Картка швидкої довідки

| Аспект | Специфікація |
|--------|---------------|
| **Шрифт (заголовки)** | Montserrat, накреслення 700 |
| **Шрифт (основний текст)** | Commissioner, накреслення 400 |
| **Шрифт (кнопки/мітки)** | Commissioner, накреслення 500 |
| **Кнопка (основний CTA)** | Градієнт, висота 48px, максимум 1 на екран |
| **Кнопка (панель дій)** | Плоска з обведенням, висота 40px |
| **Кнопка (другорядна)** | Filled tonal, висота 48px |
| **Кнопка (третинна)** | Лише текст, висота 40px |
| **Кнопка (іконка)** | 40×40px, плоска, прозора |
| **Основний колір** | Суцільний смарагдово-зелений `#10b981` |
| **Висота панелі дій** | 56px з плоскими кнопками |
| **Базовий відступ** | Сітка 8px (4, 8, 16, 24, 32, 40, 48) |
| **Основний текст** | 14px / висота рядка 20px (Body Medium) |
| **Зона дотику** | 40×40 (кнопки-іконки), 48×48 (основні CTA) |
| **Border radius** | 8px (стандарт), 12px (діалоги) |
| **Розмір іконки** | 24×24px (за замовчуванням) |
| **Padding інпута** | 8px (`--spacing-sm`) |
| **Межа інпута** | 2px solid |
| **Перехід** | 200ms ease |
| **Оверлей hover** | прозорість 8% |
| **Focus ring** | 3px, прозорість 15% |
| **Прозорість disabled** | 38% (0.38) |
| **Елевація (картки)** | `--elevation-1` (ненав'язлива) |
| **Padding контейнера** | 12-16px по горизонталі |

---

*Останнє оновлення: січень 2026 (Material Design 3)*
