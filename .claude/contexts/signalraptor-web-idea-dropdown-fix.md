# SignalRaptor Web — Fix: Idea Dropdown Vazio em CreateNewContent

**Data:** 2026-04-14

## Problema

O dropdown de Ideas na tela `CreateNewContent.vue` aparecia vazio ao abrir, mesmo com ideas disponíveis no store.

## Causa Raiz: Race Condition por Dupla Inicialização

Em Vue 3 com `<script setup>`, o `watch({ immediate: true })` dispara **dentro do setup**, antes do `onBeforeMount`. O callback async inicia e cede controle no primeiro `await`.

**Sequência problemática:**

1. `watch({ immediate: true })` dispara → `articleStore.$reset()` → `initializeIdeaDropdown()` → `retrieveIdeas()` → **`isLoading = true`** → HTTP request → yield
2. `onBeforeMount` dispara → `initializeIdeaDropdown()` → `retrieveIdeas()` → **guard `if (this.isLoading) return`** → retorna `undefined`
3. `ideas.filter(...)` em `initializeIdeaDropdown` → TypeError (undefined.filter) → catch → **`firstDropdown.items = []`**
4. Dropdown fica vazio

## Fixes Aplicados

### Fix 1 — Remover inicialização duplicada
**Arquivo:** `src/views/content/CreateNewContent.vue`

Removido `await initializeIdeaDropdown()` do `onBeforeMount` (linha ~526). O `watch({ immediate: true })` já cuida do carregamento inicial; o `onBeforeMount` só precisava carregar o artigo/news_release pelo ID.

### Fix 2 — Guard defensivo em `initializeIdeaDropdown`
**Arquivo:** `src/views/content/CreateNewContent.vue`

Adicionado `if (!ideas) return` logo após `getIdeaDropDown()`, evitando TypeError caso o store retorne `undefined` (quando `isLoading` já está true).

## Arquivos Tocados

- `signalraptor-web/src/views/content/CreateNewContent.vue`

## Decisões

- Não modificado o store `article.ts` — o guard `if (this.isLoading) return` é intencional para evitar requests paralelos; o problema era na camada de apresentação.
- Fix cirúrgico: mínima alteração, sem refatoração de fluxo.

---

# SignalRaptor Web — Fix: Articles/News Releases não aparecem (QA-1038 regressão)

**Data:** 2026-04-14

## Problema

Após o commit QA-1038 (`80d263d`), artigos não exibiam o ícone de idea e o scroll infinito parou de funcionar.

## Causa Raiz

O QA-1038 tentou mapear `article.ideaId` para mostrar o ícone de link, mas o backend (commit `dc415195` de otimização) não retorna `ideaId` na listagem de artigos — retorna `idea` (string com o conteúdo da idea via LEFT JOIN). A tentativa de fix sobrescreveu `idea` com `''` (sempre falsy) e removeu `id`, `totalCount`, `currentPage`, `isLoadingMore` das colunas.

## Efeitos do QA-1038
1. `idea` sempre `''` → ícone de link nunca aparecia
2. `id`, `totalCount`, `currentPage`, `isLoadingMore` removidos → `loadMoreForColumn` quebrado
3. `activeFetchOptions` não salvo → sort/filter não aplicados no load-more

## Fix Aplicado

**Arquivo:** `src/stores/content/article.ts`

Revertido o `fetchArticles` para o mapeamento correto (idêntico ao código anterior ao QA-1038). O `idea` (content string do backend) já chega via `...article` spread e funciona corretamente com o `v-if="props.idea"` do `RArticleCard`.

## Arquivos Tocados

- `signalraptor-web/src/stores/content/article.ts`
- `signalraptor-web/src/views/content/CreateNewContent.vue` (fix anterior do dropdown)

---

# SignalRaptor — Investigação contínua: Kanban vazio com 7 colunas

**Data:** 2026-04-14

## Novas confirmações

1. O retrieve de artigos no backend filtra estritamente por `articles.brand_id` (sem fallback para `brand_profile_id`).
2. A timeline de migrations confirma que `brand_id` foi adicionado depois da criação da tabela e não existe migration de backfill.
3. O cache de content management tem TTL de 7 dias (`contentCacheTtl: 604800`) e pode manter resposta vazia por descriptor (brandId + filtros + paginação).
4. O frontend local está configurado para `https://api-test.signalraptor.com/` em `.env`.

## Implicação prática

O cenário mais provável continua sendo dado legado sem `brand_id` (ou `brand_id` divergente do brand ativo), potencialmente agravado por cache quente de resposta vazia.

## Documento atualizado

- `articles-missing-investigation.md` expandido com:
  - seção de achados confirmados em código
  - diagnóstico de cache Redis
  - checks rápidos sem acesso a DB
