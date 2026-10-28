# Design System e Componentes (@lib/ui)

> Submódulo Git compartilhado (`packages/lib-ui`) com os tokens do design system e os componentes do shadcn. Instruções de uso em outro projeto no `README.md`.

---

## 1. O Que Fica Aqui

| Caminho | Conteúdo |
|---|---|
| `src/components/` | Componentes do shadcn (`base-nova`, primitives do `@base-ui/react`). Gerados pelo CLI, não crie arquivos à mão. |
| `src/lib/utils.ts` | `cn()` (`clsx` + `tailwind-merge`). |
| `src/hooks/` | Hooks que o CLI do shadcn instalar junto com algum componente. |
| `src/styles/globals.css` | Tailwind v4, `tw-animate-css`, `shadcn/tailwind.css`, tokens oklch de `:root` e `.dark`, `@theme inline` e `@layer base`. É o único lugar onde os tokens são definidos. |

Só entra aqui o que é **genérico**: nada de feature, chamada de API, `next/*` ou regra de negócio de um app. Componentes que dependem de dados do app continuam no app (`src/shared/components/<Nome>`).

## 2. Regras

- O pacote é publicado como código-fonte (`exports` apontando para `src/`). Não há build: quem consome usa `transpilePackages: ['@lib/ui']`.
- Imports internos usam o próprio nome do pacote (`@lib/ui/lib/utils`, `@lib/ui/components/button`), nunca `@/` nem caminho relativo. É o que o CLI do shadcn gera com o `components.json` daqui, e resolve igual no lib (`paths` do `tsconfig.json`) e no app.
- `react`, `react-dom`, `next-themes`, `sonner` e `tailwindcss` são **peerDependencies**: o lib precisa usar a mesma instância do app (contexto do tema, fila de toasts do `sonner`, um único Tailwind).
- Versões ficam fixadas no `package.json` (sem `catalog:`), porque o submódulo é usado em repositórios que não têm o mesmo catálogo. Mantenha as mesmas faixas do app; `pnpm check-versions` aponta divergências.
- `@source "../**/*.{ts,tsx}"` no `globals.css` faz o Tailwind gerar as classes usadas aqui; o app continua com a detecção automática dos próprios arquivos.
- A fonte vem da variável `--font-inter`, definida pelo app (`next/font`).

## 3. Adicionar ou Atualizar um Componente

Rode o CLI a partir do app consumidor, nunca de dentro deste pacote:

```bash
cd apps/web
pnpm dlx shadcn@latest add <componente>
```

O componente é gravado em `packages/lib-ui/src/components/`. Formate com Biome e confira o typecheck:

```bash
pnpm --filter @lib/ui typecheck
```

Alterações aqui são commitadas no repositório do submódulo; depois o projeto principal faz commit do novo ponteiro.
