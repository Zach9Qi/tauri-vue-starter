# 样式规范(Tailwind CSS 4 + 三层设计令牌)

> 唯一的全局样式文件是 `src/index.css`;组件只写工具类。本仓库选 Tailwind 4(CSS-first 配置、`light-dark()` 原生深浅色),以下全部基于本仓库真实实现。

---

## 1. 三层令牌(`src/index.css`)

| 层 | 位置 | 允许出现的东西 | 禁止 |
|---|---|---|---|
| 原始层 | `:root` | 唯一允许写具体值的地方:颜色用 `light-dark(var(--color-zinc-*), …)`、透明用 `--alpha(… / n%)`、圆角基准 `--radius`、根字号、字体栈 | 字面色值(`#fff`、`rgb()`) |
| 语义层 | `@theme inline` | 只做映射与派生:`--color-background: var(--background)`、`--radius-sm: calc(var(--radius) - 4px)` | 任何具体值 |
| 基础层 | `@layer base` | 全局行为策略:根字号、`html { color-scheme: light dark }`、`body` 消费语义令牌设文档底色 / 文字色 / 字体族(`var(--color-background)` 等)、字体平滑、`button { cursor: pointer }`、默认 `border-color` | 具体色值、原始色变量(`var(--color-zinc-*)`) |
| 消费层 | `.vue` 组件 | 语义工具类:`bg-background`、`text-muted-foreground`、`bg-accent text-accent-foreground`、`border`、`focus-visible:ring-ring` | 原始色工具类 `bg-zinc-900`、任意值 `bg-[#123]` |

现有语义令牌:`background` / `foreground` / `muted` / `muted-foreground` / `accent` / `accent-foreground` / `border` / `ring`;圆角 `radius-sm|md|lg|xl|2xl`;字体 `font-sans` / `font-mono`。

新增令牌的流程:先在 `:root` 加原始变量(带一行中文注释说明用途)→ 在 `@theme inline` 映射为 `--color-*` / `--radius-*` → 组件使用。**成对的前景/背景**(`accent` + `accent-foreground`)必须同时加。

## 2. 深浅色

- 靠 `light-dark()` + `@layer base` 里的 `html { color-scheme: light dark }`,跟随系统,无 JS、无 `dark:` 变体。
- `color-scheme` **必须和 `:root` 上的 `light-dark()` 令牌声明在同一元素(html)上**,不要在组件里用 `scheme-*` 工具类充当主题根。原因见 `index.css` 注释:一是 Teleport 到 body 的弹层会脱离组件子树;二是构建时 Lightning CSS 会 polyfill `light-dark()`,开关变量不在 `:root` 上时所有颜色令牌都会算成非法值。
- 文档级底色 / 文字色 / 字体族由 `@layer base` 的 `body` 承担(`var(--color-background)` / `var(--color-foreground)` / `var(--font-sans)`),这样 overscroll、Teleport 弹层等脱离布局容器的区域不会露出 webview 白底。布局组件(`App.vue` 的 `<main>`)不再写 `bg-background text-foreground font-sans`;需要局部换底时才在容器上用 `bg-muted` 等令牌。
- 需要手动切换主题时,在 `html` 上覆盖 `color-scheme`(如加 `scheme-dark` 类),仍不写 `dark:`。
- 不使用 `dark:` 前缀写双份样式;要新颜色就加令牌。

## 3. 组件内写法

- 工具类顺序由 `prettier-plugin-tailwindcss` 排序(`.prettierrc` 已配置 `tailwindStylesheet: ./src/index.css`),提交前跑 `bun run format`,不手动排。
- 尺寸用 `size-*` / `gap-*` / `p-*` 等 rem 工具类;不写 `px` 任意值。
- 交互态用 `hover:opacity-80`、`disabled:cursor-not-allowed disabled:opacity-50`、`focus-visible:ring-2 focus-visible:ring-ring`,与 `HelloWorld.vue` 一致。
- 需要复用的一组类名:优先抽组件(带 props 的按钮组件),其次才是 `@utility`;`@apply` 仅在组件无法抽取时使用(Tailwind 官方文档建议优先组件化而非 `@apply`)。
- `<style scoped>` 仅在工具类确实表达不了(复杂动画、第三方组件深层选择器 `:deep()`)时使用,并写注释说明为何不能用工具类。

## 4. 字体与图标

- 字体族只在 `:root` 定义一次(`--font-family-sans` / `--font-family-mono`);`body` 默认 `font-sans`,组件只在需要等宽时写 `font-mono`。
- 图标是 SVG 组件(`~icons/lucide/*`),尺寸 `size-4` 等,颜色随 `currentColor`;不设 `fill` / `stroke`。

## 5. 禁止

- 组件里出现字面色值、`bg-zinc-*` 等原始色工具类、`bg-[...]` 任意色值。
- `dark:` 变体。
- 全局 `<style>`、在组件里 `@import` CSS。
- 新建第二个全局 CSS 文件(主题相关只改 `index.css` 的 `:root`)。
- `!important` / `!` 修饰符:需要它通常意味着令牌或层叠层设计有问题,先修根因;确实需要(覆盖第三方内联样式)时写注释。
