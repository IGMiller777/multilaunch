# 🚀 Multi-Brand Site Platform

> Масштабируемая фронтенд-платформа для запуска 15+ мультибрендовых сайтов в месяц

## 📋 Содержание

- [Технологический стек](#-технологический-стек)
- [Архитектура проекта](#-архитектура-проекта)
- [Структура проекта](#-структура-проекта)
- [Конфигурация](#-конфигурация)
- [UI-Kit & Design System](#-ui-kit--design-system)
- [Развертывание](#-развертывание)
- [Быстрый старт](#-быстрый-старт)

---

## 🛠 Технологический стек

### Основные технологии

| Технология | Версия | Назначение |
|------------|--------|------------|
| **NX** | 21.6.3 | Monorepo orchestration, кэширование сборок, граф зависимостей |
| **Next.js** | 15.5.4 | React framework, App Router, SSR/SSG |
| **React** | 19.2.0 | UI библиотека |
| **TypeScript** | 5.9.2 | Type safety, DX |
| **pnpm** | latest | Package manager с workspace support |

### Вспомогательные инструменты

| Инструмент | Назначение |
|------------|------------|
| **@nx/react** | Генерация React библиотек |
| **@nx/next** | Генерация Next.js приложений |
| **@swc/core** | Быстрая компиляция TS/JSX |
| **Playwright** | E2E тестирование |
| **Prettier** | Форматирование кода |
| **ESLint** | Линтинг кода |

### Планируемые зависимости

```json
{
  "zod": "^3.22.0",              // Валидация схем конфигурации
  "tailwindcss": "^3.4.0",       // Utility-first CSS
  "zustand": "^4.5.0",           // State management для editor
  "react-dnd": "^16.0.0",        // Drag & Drop в visual builder
  "clsx": "^2.1.0",              // Условные CSS классы
  "framer-motion": "^11.0.0"     // Анимации
}
```

---

## 🏗 Архитектура проекта

### Структура проекта: NX Monorepo

**Выбор: Monorepo (NX + pnpm workspaces)**

#### Почему Monorepo

✅ **Атомарные изменения** — один commit изменяет UI компонент во всех 15+ сайтах одновременно  
✅ **Граф зависимостей** — NX автоматически понимает что пересобрать при изменении библиотеки  
✅ **Кэширование** — повторные сборки неизмененного кода мгновенные  
✅ **Единая кодовая база** — нет версионного дрейфа между сайтами  
✅ **Affected builds** — собираем только то, что изменилось в PR

#### Альтернативы отвергнуты

❌ **Polyrepo** — 15 репозиториев требуют 15 синхронизаций зависимостей  
❌ **Multirepo** — изменение UI компонента = 15 отдельных PR в разные репо

---

### Переиспользование компонентов

```
┌─────────────────────────────────────────────────────────┐
│                    apps/admin-platform                  │
│  (импортирует все libs для управления и preview)        │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼────────┐       ┌────────▼──────────┐
│  libs/ui-kit   │◄──────┤ libs/theme-engine │
│  (компоненты)  │       │  (темизация)      │
└───────┬────────┘       └───────────────────┘
        │
        │ extends
        │
┌───────▼──────────────┐
│ libs/business-blocks │
│ (VPN/SaaS/Steam)     │
└──────────────────────┘
        │
        │ используют все сайты
        │
┌───────▼──────────────┐
│   apps/site-vpn      │
│   apps/site-saas     │
│   apps/site-steam    │
└──────────────────────┘
```

**Механизм:**

1. Все компоненты экспортируются из `libs/ui-kit/src/index.ts`
2. Импорт в приложения через алиасы: `import { Button } from '@platform/ui-kit'`
3. NX отслеживает изменения: изменил `Button.tsx` → пересобрал зависимые apps
4. TypeScript paths в `tsconfig.base.json` обеспечивают автокомплит

---

### Основные компоненты по типам бизнеса

#### Общие компоненты (все типы)

**Primitives (базовые):**
- Button, Input, Card, Badge, Modal, Dropdown, Tabs, Checkbox, Radio, Switch, Tooltip

**Compositions (маркетинговые):**
- Hero, Pricing, Features, FAQ, Testimonials, CTA, Stats, LogoCloud, Newsletter

**Layouts:**
- Header, Footer, Sidebar, Container, Grid, Stack

**Forms:**
- ContactForm, SubscribeForm, CheckoutForm, SearchBar

#### VPN-специфичные

```
libs/business-blocks/src/vpn/
├── ServerSelector/        # Интерактивная карта серверов с пингом
├── SpeedTest/            # Виджет проверки скорости соединения
├── ProtocolInfo/         # Сравнительная таблица OpenVPN/WireGuard/IKEv2
├── EncryptionBadge/      # Визуальный индикатор уровня шифрования
├── ConnectionStatus/     # Real-time статус подключения
└── DeviceCompatibility/  # Список поддерживаемых платформ
```

#### SaaS-специфичные

```
libs/business-blocks/src/saas/
├── FeatureComparison/    # Таблица сравнения тарифов (Free/Pro/Enterprise)
├── IntegrationsList/     # Сетка логотипов интеграций с поиском
├── UsageMetrics/         # Dashboard preview с графиками
├── APIDocumentation/     # Интерактивная документация API
├── StatusPage/           # Uptime monitor виджет
└── TrialCounter/         # Таймер пробного периода
```

#### Steam Keys-специфичные

```
libs/business-blocks/src/steam/
├── GameKeyCard/          # Карточка игры с ключом (платформа, цена, наличие)
├── PlatformBadges/       # Иконки Steam/Epic/Origin/GOG
├── InstantDelivery/      # Таймер доставки ключа (1-5 минут)
├── RegionLock/           # Индикатор региональных ограничений
├── KeyActivation/        # Инструкция по активации с шагами
└── GameGallery/          # Слайдер скриншотов игры
```

---

### Изоляция сайтов

#### 1. Уровень кода

```typescript
// Каждый сайт = отдельное Next.js приложение
apps/
├── site-vpn/           # Независимый деплой, env, routing
├── site-saas/          # Отдельная конфигурация, domain
└── site-steam/         # Изолированные зависимости
```

#### 2. Уровень конфигурации

```typescript
// apps/site-vpn/site.config.ts
export default {
  meta: {
    name: 'SecureVPN',
    domain: 'securevpn.com',
  },
  theme: {
    preset: 'vpn-dark',
    colors: { primary: '#6366f1' }
  },
  features: {
    analytics: true,
    chatSupport: true,
  }
}
```

#### 3. Уровень деплоя

```
Production:
├── securevpn.com       → apps/site-vpn (Vercel Project #1)
├── cloudkeys.io        → apps/site-steam (Vercel Project #2)
└── protools.app        → apps/site-saas (Vercel Project #3)

# Каждый сайт деплоится независимо
# Падение одного не влияет на другие
```

#### 4. Уровень данных

```
# Переменные окружения изолированы
apps/site-vpn/.env.local
apps/site-saas/.env.local

# API keys, database URLs, analytics tokens - отдельно для каждого
```

---

## 📁 Структура проекта

### `/apps` — Приложения

#### `apps/admin-platform/` — Админ-панель

Единая панель управления всеми сайтами.

```
apps/admin-platform/
├── src/app/
│   ├── (auth)/              # Авторизация
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/         # Основная панель
│   │   ├── sites/           # CRUD сайтов
│   │   │   ├── page.tsx           # Список всех сайтов
│   │   │   ├── [id]/page.tsx      # Детали сайта
│   │   │   ├── [id]/edit/         # Редактор настроек
│   │   │   └── new/               # Создание нового сайта
│   │   ├── editor/[siteId]/       # Visual Page Builder
│   │   │   ├── page.tsx           # Canvas для drag-drop
│   │   │   └── components/
│   │   │       ├── BlockPalette.tsx   # Библиотека блоков
│   │   │       ├── PropertyPanel.tsx  # Настройки блока
│   │   │       └── PreviewMode.tsx    # Превью страницы
│   │   ├── templates/         # Marketplace шаблонов
│   │   │   └── page.tsx           # VPN/SaaS/Steam темплейты
│   │   ├── themes/            # Редактор тем
│   │   │   └── page.tsx           # Customizer цветов/шрифтов
│   │   └── settings/          # Глобальные настройки
│   └── api/                 # API routes (Next.js Route Handlers)
├── components/              # UI компоненты админки
└── next.config.js
```

**Ключевые фичи:**
- Drag-and-drop редактор страниц
- Live preview изменений
- Клонирование сайтов из шаблонов
- Управление темами и компонентами
- Публикация/unpublish сайтов

---

### `/libs` — Библиотеки

#### `libs/ui-kit/` — Design System

Универсальная библиотека UI компонентов.

```
libs/ui-kit/src/
├── primitives/                # Атомарные компоненты
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx    # Storybook документация
│   │   ├── Button.test.tsx       # Unit тесты
│   │   └── index.ts
│   ├── Input/
│   ├── Card/
│   └── ...
├── compositions/              # Композитные компоненты
│   ├── Hero/
│   │   ├── Hero.tsx
│   │   ├── variants/             # HeroSplit, HeroCentered, HeroVideo
│   │   └── index.ts
│   ├── Pricing/
│   │   ├── Pricing.tsx
│   │   ├── PricingCard.tsx
│   │   └── index.ts
│   └── ...
├── layouts/                   # Layout компоненты
│   ├── Header/
│   ├── Footer/
│   └── ...
├── forms/                     # Форм-компоненты
└── styles/
    ├── globals.css            # Global styles
    └── tokens.css             # CSS переменные (--color-primary, etc)
```

**Экспорт:**

```typescript
// libs/ui-kit/src/index.ts
export * from './primitives';
export * from './compositions';
export * from './layouts';
export * from './forms';
```

---

#### `libs/theme-engine/` — Система тем

Динамическая темизация без пересборки компонентов.

```
libs/theme-engine/src/
├── types/
│   └── theme.types.ts         # TypeScript интерфейсы
├── provider/
│   └── ThemeProvider.tsx      # React Context для темы
├── presets/                   # Готовые темы
│   ├── vpn-dark.ts
│   ├── saas-light.ts
│   ├── gaming-neon.ts
│   └── index.ts
├── utils/
│   ├── cssVarGenerator.ts     # theme → CSS variables
│   └── themeValidator.ts      # Zod валидация
└── hooks/
    └── useTheme.ts            # Hook для доступа к теме
```

**Принцип работы:**

```typescript
// 1. Определение темы
const vpnTheme = {
  colors: {
    primary: '#6366f1',
    secondary: '#8b5cf6',
    background: '#0f172a',
  },
  typography: {
    fontFamily: 'Inter',
    scale: { base: '1rem', lg: '1.25rem' }
  }
};

// 2. Применение в runtime
<ThemeProvider theme={vpnTheme}>
  <App />
</ThemeProvider>

// 3. Генерация CSS variables
:root {
  --color-primary: #6366f1;
  --color-secondary: #8b5cf6;
  --font-family: Inter;
}

// 4. Использование в компонентах
.button-primary {
  background: var(--color-primary);
  font-family: var(--font-family);
}
```

---

#### `libs/business-blocks/` — Бизнес-специфичные блоки

```
libs/business-blocks/src/
├── vpn/
│   ├── ServerSelector/
│   ├── SpeedTest/
│   └── ...
├── steam/
│   ├── GameKeyCard/
│   └── ...
└── saas/
    ├── FeatureComparison/
    └── ...
```

**Зависимости:**
- Импортирует `@platform/ui-kit` для базовых компонентов
- Импортирует `@platform/theme-engine` для темизации

---

#### `libs/site-generator/` — Генератор сайтов

Программное создание новых сайтов.

```
libs/site-generator/src/
├── templates/
│   ├── vpn-template.ts        # Конфиг + структура страниц VPN сайта
│   ├── saas-template.ts
│   └── steam-template.ts
├── generators/
│   ├── create-site.ts         # Создание apps/site-{name}
│   └── scaffold-pages.ts      # Генерация страниц из шаблона
└── utils/
    └── file-writer.ts
```

---

#### `libs/config-schema/` — Схемы конфигурации

Валидация через Zod.

```
libs/config-schema/src/
├── site-config.schema.ts      # Zod схема для site.config.ts
├── theme-config.schema.ts     # Валидация тем
└── validators.ts
```

---

#### `libs/shared-utils/` — Утилиты

```
libs/shared-utils/src/
├── hooks/
│   ├── useMediaQuery.ts
│   ├── useLocalStorage.ts
│   └── ...
├── helpers/
│   ├── cn.ts                  # classnames helper
│   ├── formatDate.ts
│   └── ...
└── types/
    └── common.types.ts
```

---

### `/tools` — Инструменты

#### `tools/generators/create-site/` — NX генератор

```
tools/generators/create-site/
├── index.ts                   # Логика генератора
├── schema.json                # JSON Schema параметров
└── files/                     # Шаблонные файлы
    ├── site.config.ts.template
    └── next.config.js.template
```

**Использование:**

```bash
pnpm nx g @platform/tools:create-site premium-vpn --businessType=vpn
```

---

## ⚙️ Конфигурация

### Добавление нового сайта

#### Способ 1: Через NX генератор (рекомендуется)

```bash
# 1. Генерация структуры
pnpm nx g @platform/tools:create-site my-vpn-site --businessType=vpn

# Что создается:
# - apps/site-my-vpn-site/
# - apps/site-my-vpn-site/site.config.ts
# - apps/site-my-vpn-site/app/layout.tsx
# - apps/site-my-vpn-site/app/page.tsx
# - Регистрация в nx.json

# 2. Запуск разработки
pnpm nx serve site-my-vpn-site

# 3. Билд
pnpm nx build site-my-vpn-site
```

#### Способ 2: Через админ-панель

```
1. Админка → Sites → New Site
2. Выбор шаблона: VPN / SaaS / Steam
3. Конфигурация:
   - Название сайта
   - Домен
   - Тема (preset или custom)
   - Активные фичи
4. Генерация → автоматическое создание через API
5. Деплой на Vercel через API
```

---

### Система конфигураций

#### `site.config.ts` — Главный конфиг сайта

```typescript
// apps/site-vpn/site.config.ts
import { defineConfig } from '@platform/config-schema';

export default defineConfig({
  meta: {
    name: 'SecureVPN',
    domain: 'securevpn.com',
    language: 'en',
    timezone: 'UTC',
    
    // SEO
    title: 'SecureVPN - Fast & Private VPN Service',
    description: 'Protect your privacy with military-grade encryption',
    ogImage: '/og-image.png',
  },

  theme: {
    preset: 'vpn-dark',
    
    // Переопределение цветов preset
    customColors: {
      primary: '#6366f1',
      accent: '#22d3ee',
    },
    
    // Кастомная типографика
    typography: {
      fontFamily: 'Inter',
      headingFontFamily: 'Space Grotesk',
    },
  },

  features: {
    analytics: {
      enabled: true,
      provider: 'vercel',
    },
    chatSupport: {
      enabled: true,
      provider: 'intercom',
      appId: 'xxx',
    },
    multiLanguage: false,
    darkMode: true,
    cookieConsent: true,
  },

  business: {
    type: 'vpn',
    
    payments: {
      providers: ['stripe', 'paypal'],
      currency: 'USD',
      stripePk: process.env.NEXT_PUBLIC_STRIPE_PK,
    },
    
    vpn: {
      serverLocations: ['US', 'UK', 'DE', 'JP'],
      protocols: ['OpenVPN', 'WireGuard'],
    },
  },

  routing: {
    pages: [
      { path: '/', component: 'HomePage' },
      { path: '/pricing', component: 'PricingPage' },
      { path: '/servers', component: 'ServersPage' },
      { path: '/download', component: 'DownloadPage' },
    ],
    
    redirects: [
      { from: '/old-pricing', to: '/pricing', permanent: true },
    ],
  },
});
```

---

#### Feature Flags — Поэтапный роллаут фич

```typescript
// apps/site-vpn/feature-flags.ts
export const FEATURE_FLAGS = {
  // Новый UI pricing таблицы (A/B тест на 50% пользователей)
  NEW_PRICING_UI: {
    enabled: true,
    rollout: 0.5,  // 50% пользователей
  },
  
  // Крипто-платежи (только для админов)
  CRYPTO_PAYMENTS: {
    enabled: false,
    rollout: 0,
    allowlist: ['admin@securevpn.com'],
  },
  
  // AI-чат поддержка (включен для всех)
  AI_CHAT: {
    enabled: true,
    rollout: 1.0,
  },
};

// Использование в компонентах
import { useFeature } from '@platform/feature-flags';

function PricingPage() {
  const showNewUI = useFeature('NEW_PRICING_UI');
  
  return showNewUI ? <NewPricingTable /> : <OldPricingTable />;
}
```

---

#### Роутинг — Динамическая генерация страниц

```typescript
// apps/site-vpn/app/[...slug]/page.tsx
import { notFound } from 'next/navigation';
import siteConfig from '../../site.config';

export async function generateStaticParams() {
  return siteConfig.routing.pages.map(page => ({
    slug: page.path.split('/').filter(Boolean),
  }));
}

export default function DynamicPage({ params }) {
  const route = siteConfig.routing.pages.find(
    p => p.path === `/${params.slug.join('/')}`
  );
  
  if (!route) notFound();
  
  const Component = await import(`@/pages/${route.component}`);
  return <Component.default />;
}
```

---

### Конфигурируемые параметры

| Категория | Параметры |
|-----------|-----------|
| **Meta** | Название, домен, язык, SEO |
| **Theme** | Цвета, шрифты, spacing, border-radius |
| **Features** | Аналитика, чат, мультиязычность, dark mode |
| **Business** | Тип бизнеса, платежи, специфичные настройки |
| **Routing** | Страницы, редиректы, sitemap |
| **Integrations** | Analytics, CRM, Email, Payment providers |

---

## 🎨 UI-Kit & Design System

### Поддержка уникального дизайна

#### Подход: Theme Engine + CSS Variables

**Архитектура темизации:**

```
┌─────────────────────────────────────────────────┐
│         site.config.ts (theme preset)           │
│              { preset: 'vpn-dark' }             │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│          ThemeProvider (React Context)          │
│     Инжектит CSS variables в :root              │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│       UI Components (используют var())          │
│   background: var(--color-primary)              │
└─────────────────────────────────────────────────┘
```


**Результат:**
- Один набор компонентов
- Каждый сайт имеет уникальные цвета/шрифты
- Смена темы без пересборки компонентов

---

### Минимальный набор компонентов UI-кита

#### Primitives (18 компонентов)

```
✓ Button          ✓ Input           ✓ Textarea
✓ Checkbox        ✓ Radio           ✓ Switch
✓ Select          ✓ Dropdown        ✓ Modal
✓ Card            ✓ Badge           ✓ Avatar
✓ Tabs            ✓ Accordion       ✓ Tooltip
✓ Spinner         ✓ Progress        ✓ Skeleton
```

#### Compositions (10 компонентов)

```
✓ Hero (3 варианта)       # HeroSplit, HeroCentered, HeroVideo
✓ Pricing                 # PricingTable с feature comparison
✓ Features                # FeatureGrid 2x3, 3x3
✓ FAQ                     # Accordion list
✓ Testimonials            # Slider с отзывами
✓ CTA                     # Call-to-action секция
✓ Stats                   # Метрики компании (1M+ users)
✓ LogoCloud               # Логотипы клиентов
✓ Newsletter              # Email subscription форма
✓ BlogCard                # Карточка статьи
```

#### Layouts (4 компонента)

```
✓ Header         # Навигация + мобильное меню
✓ Footer         # Футер с ссылками
✓ Sidebar        # Боковая панель
✓ Container      # Max-width wrapper
```

#### Forms (3 компонента)

```
✓ ContactForm    # Имя, Email, Сообщение
✓ SubscribeForm  # Только Email
✓ CheckoutForm   # Payment форма
```

### Масштабируемость UI-кита и его развитие

**Подход к масштабируемости:**

Для обеспечения масштабируемости UI-кита мы используем модульную архитектуру на базе NX monorepo, где `libs/ui-kit` является отдельной библиотекой. Это позволяет:

- **Атомарное развитие:** Компоненты разделены на primitives (базовые), compositions (составные), layouts и forms. Новые компоненты добавляются без влияния на существующие.
- **Версионирование:** Используем semantic versioning (semver) для библиотеки. Изменения в UI-kit автоматически триггерят пересборку зависимых приложений через NX graph.
- **Тестирование:** Каждый компонент имеет unit-тесты (Jest), stories (Storybook) и E2E (Playwright). Coverage >80%.
- **Документация:** Storybook как живой каталог компонентов с примерами, пропсами и вариациями.
- **Производительность:** Компоненты оптимизированы для tree-shaking (ES modules), lazy-loading и мемоизации (React.memo).

**Процесс развития:**

1. **Добавление компонента:**
    - Генерация: `pnpm nx g @nx/react:component Button --project=ui-kit --export`
    - Реализация: TSX + CSS (Tailwind).
    - Тестирование: Добавить .test.tsx и .stories.tsx.
    - Review: PR с changelog.

2. **Масштабирование:**
    - **Variants:** Для каждого компонента — пропс `variant` (e.g., primary/secondary для Button).
    - **Extensibility:** Компоненты принимают `className` для overrides через clsx.
    - **Theming:** Все стили через CSS vars из theme-engine — легко добавлять новые токены.
    - **Accessibility:** ARIA атрибуты по умолчанию, тестирование с axe-core.
    - **I18n:** Поддержка RTL/LTR через theme.

3. **Мониторинг и улучшения:**
    - Bundle analyzer (NX plugin) для размера.
    - Feedback loop: Интеграция с админ-панелью для сбора метрик использования.
    - Quarterly review: Аудит на deprecation старых компонентов.

**Сторонние библиотеки:** Собственный UI-kit на базе TailwindCSS (utility-first) + clsx для классов. Не используем большие фреймворки вроде MUI/AntD, чтобы избежать overhead и сохранить контроль над стилями. Для анимаций — Framer Motion (минимально).

---

### Развёртывание и инфраструктура

#### Процесс деплоя

**Подход: Раздельный билд под каждый сайт + единый для админ-панели.**

- **Платформа:** Vercel (Next.js native) для всех apps. Автоматический preview на каждый PR.
- **Организация:**
    - Каждый сайт (`apps/site-*`) — отдельный Vercel Project с своим domain.
    - Админ-панель (`apps/admin-platform`) — отдельный проект на subdomain (e.g., admin.platform.com).
    - Libs собираются on-demand через NX cache — нет глобального билда.
- **Шаги деплоя:**
    1. Push/PR в main → GitHub Actions/NX Cloud: `pnpm nx affected:build` (только изменившиеся apps/libs).
    2. Vercel webhook: Деплой каждого app как статического сайта/SSR.
    3. Rollout: Blue-green deployment на Vercel (0 downtime).
- **Преимущества:** Падение одного сайта не влияет на другие. Масштабирование по трафику индивидуально.

**Альтернатива:** AWS Amplify/Netlify для multi-app, но Vercel лучше интегрируется с Next.js.

---

#### Dev/Staging/Prod Pipeline

**Pipeline на базе GitHub Actions + Vercel.**

- **Environments:**
    - **Dev:** Локально (`pnpm nx serve site-*`) + Storybook для UI-kit.
    - **Staging:** Branch deploy на Vercel (e.g., staging-securevpn.vercel.app). Авто-деплой на feature branches.
    - **Prod:** Main branch → автоматический деплой после тестов.



#### Автоматизация запуска нового сайта из шаблона

**Подход: CLI + API.**

- **CLI (NX генератор):** `pnpm nx g create-site new-vpn --template=vpn` — создает app, config, pages из шаблона в libs/site-generator/templates/.
- **Через админ-панель:** API endpoint `/api/sites/create` вызывает генератор на сервере (Next.js API route) → git commit/push → Vercel deploy.
- **Шаги автоматизации:**
    1. Выбор шаблона (VPN/SaaS/Steam).
    2. Генерация: Копирование файлов, замена placeholders (e.g., {{siteName}}).
    3. Деплой: Vercel API создает новый project + domain linking.
    4. Post-setup: Добавление в админ-DB, настройка env vars.

**Интеграции:** GitHub API для repo updates, Vercel API для deployments.

---

#### Автоматизация типовых задач

**1. Создание сайта:** Как выше — CLI/API с шаблонами.

**2. Добавление перевода:**
- **Автоматизация:** Интеграция с i18next/next-i18next. Команда: `pnpm nx run site-*:extract-i18n` — извлекает строки в JSON.
- **Процесс:**
    - Добавление locale: `site.config.ts` → `languages: ['en', 'ru']`.
    - Генерация файлов: `public/locales/ru/common.json`.
    - Auto-translate: GitHub Action с DeepL/Google Translate API для draft.
    - Review: Через админ-editor с live preview.

**3. SEO-настройки:**
- **Автоматизация:** Next.js metadata API + site.config.ts.
- **Процесс:**
    - Глобально: `meta.title/description` в config → применяется в layout.tsx.
    - Per-page: `export const metadata = { title: 'Page' }` в page.tsx.
    - Генерация: CLI `pnpm nx g seo-config site-*` — обновляет sitemap.xml/robots.txt.
    - Динамика: Админ-панель для редактирования meta + auto-submit to Google Search Console via API.

**Другие задачи:**
- **Миграции:** NX migrations для deps updates.
- **Backups:** Vercel + Git для code, Supabase/Airtable для админ-data.
- **CI/CD:** Все задачи в workflows — lint/format/test on PR.