# @lib/ui

Design system compartilhado: tokens de cor/raio/tipografia (Tailwind v4) e os componentes do shadcn (`base-nova`, sobre Base UI).

O pacote é distribuído como **código-fonte** (sem build): o app que consome compila os `.tsx` (`transpilePackages`) e o Tailwind lê as classes direto de `src/`.

## Estrutura

```
src/
├── components/
│   ├── button.tsx, dialog.tsx, ...   # shadcn (arquivo solto, kebab-case)
│   └── ConfirmDeleteDialog/index.tsx # customizados (pasta PascalCase + index.tsx)
├── providers/       # Providers de contexto (theme-provider.tsx)
├── hooks/           # Hooks que os componentes do shadcn instalarem
├── lib/utils.ts     # cn()
└── styles/
    └── globals.css  # Tailwind, tokens (claro/escuro), @theme e camada base
```

| Import | Arquivo |
|---|---|
| `@lib/ui/components/<nome>` | `src/components/<nome>.tsx` (shadcn) |
| `@lib/ui/components/<Nome>` | `src/components/<Nome>/index.tsx` (customizado, registrado em `exports`) |
| `@lib/ui/providers/<nome>` | `src/providers/<nome>.tsx` |
| `@lib/ui/lib/utils` | `src/lib/utils.ts` |
| `@lib/ui/hooks/<nome>` | `src/hooks/<nome>.ts` |
| `@lib/ui/globals.css` | `src/styles/globals.css` |

Componentes customizados disponíveis: `ConfirmDeleteDialog`, `FilterSelect` e `ModeToggle` (alternância claro/escuro/sistema).

## Usando em um projeto (Next.js + pnpm workspace)

1. Adicione o submódulo:

```bash
git submodule add -b main https://github.com/apps-home/lib-ui packages/lib-ui
```

2. No `package.json` do app:

```json
{
  "dependencies": {
    "@lib/ui": "workspace:*"
  }
}
```

O app precisa ter os peers instalados: `react`, `react-dom`, `next-themes`, `sonner` e `tailwindcss` (com `@tailwindcss/postcss`).

3. `next.config.ts`:

```ts
transpilePackages: ['@lib/ui']
```

4. `tsconfig.json` do app (o CLI do shadcn usa esse alias para achar a pasta do lib):

```json
"paths": {
  "@lib/ui/*": ["../../packages/lib-ui/src/*"]
}
```

5. CSS global do app (o Tailwind já vem do lib e continua lendo as classes do app automaticamente):

```css
@import "@lib/ui/globals.css";
```

6. A fonte é a Inter, via variável `--font-inter`:

```tsx
const inter = Inter({ variable: '--font-inter', subsets: ['latin'] })

<body className={`${inter.variable} antialiased`}>
```

7. Providers: o modo escuro usa a classe `.dark` no `<html>`.

```tsx
import { Toaster } from '@lib/ui/components/sonner'
import { TooltipProvider } from '@lib/ui/components/tooltip'
import { ThemeProvider } from '@lib/ui/providers/theme-provider'

<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  <TooltipProvider>{children}</TooltipProvider>
  <Toaster richColors />
</ThemeProvider>
```

8. `components.json` do app, para o CLI do shadcn instalar os componentes no lib:

```json
{
  "style": "base-nova",
  "tailwind": {
    "config": "",
    "css": "../../packages/lib-ui/src/styles/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "iconLibrary": "lucide",
  "aliases": {
    "components": "@/components",
    "ui": "@lib/ui/components",
    "utils": "@lib/ui/lib/utils",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

## Adicionando componentes do shadcn

Rode o CLI **de dentro do app** (não do lib):

```bash
cd apps/web
pnpm dlx shadcn@latest add <componente>
```

O CLI lê os dois `components.json` e grava o componente em `packages/lib-ui/src/components`, com os imports já apontando para `@lib/ui/...`. Dependências novas do componente entram no `package.json` do lib. Depois, faça o commit **no repositório do lib** e atualize o ponteiro do submódulo no projeto.

## Criando um componente customizado

1. Crie `src/components/<Nome>/index.tsx` (PascalCase), compondo os componentes do shadcn com imports `@lib/ui/...`.
2. Registre no `exports` do `package.json`, **antes** do padrão `./components/*` (que só resolve arquivos `.tsx` soltos):

```json
"./components/<Nome>": "./src/components/<Nome>/index.tsx",
```

Sem esse registro, o import funciona neste monorepo por causa do `paths` do `tsconfig`, mas quebra em ferramentas e projetos que resolvem só pelo pacote (Node, Vitest sem alias, etc.).

## Atualizando o lib em outro projeto

```bash
git submodule update --remote packages/lib-ui
pnpm install
```
