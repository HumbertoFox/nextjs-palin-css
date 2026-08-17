# Tutorial: Estilizando Next.js com CSS Puro (CSS Modules)

> Um guia progressivo — do "por quê" ao "como" — para dominar estilização em Next.js sem bibliotecas externas, usando apenas CSS Modules.

---

## Sumário

1. [Por que CSS Modules?](#1-por-que-css-modules)
2. [Configurando o projeto](#2-configurando-o-projeto)
3. [Sintaxe básica: seu primeiro módulo CSS](#3-sintaxe-básica-seu-primeiro-módulo-css)
4. [O Box Model — a base de tudo](#4-o-box-model--a-base-de-tudo)
5. [Seletores essenciais](#5-seletores-essenciais)
6. [Layout com Flexbox](#6-layout-com-flexbox)
7. [Layout com Grid](#7-layout-com-grid)
8. [Position: tirando elementos do fluxo normal](#8-position-tirando-elementos-do-fluxo-normal)
9. [Pseudo-classes e pseudo-elementos](#9-pseudo-classes-e-pseudo-elementos)
10. [Variáveis CSS (Custom Properties)](#10-variáveis-css-custom-properties)
11. [Composição de classes (`composes`)](#11-composição-de-classes-composes)
12. [Responsividade com Media Queries](#12-responsividade-com-media-queries)
13. [Projeto prático: um Card completo](#13-projeto-prático-um-card-completo)
14. [Boas práticas](#14-boas-práticas)
15. [Exercícios propostos](#15-exercícios-propostos)

---

## 1. Por que CSS Modules?

Antes de escrever uma linha de código, entenda o problema que estamos resolvendo.

Em CSS tradicional, todas as classes vivem no **mesmo escopo global**. Se você tem `.card` em dois arquivos diferentes, um vai sobrescrever o outro — é só uma questão de qual foi carregado por último. Em um projeto pequeno isso é irritante; em um projeto grande, é caótico.

**CSS Modules** resolve isso transformando cada classe em algo único automaticamente. Quando você escreve:

```css
/* Button.module.css */
.primary {
  background: blue;
}
```

O Next.js gera, nos bastidores, algo como `.Button_primary__a3f9k`. Você nunca vê esse nome — ele é gerenciado por trás dos panos — mas o resultado é que **sua classe `.primary` nunca vai colidir** com a `.primary` de outro componente.

> 💡 **Por que isso importa no Next.js especificamente:** o framework já vem com suporte nativo a CSS Modules, sem nenhuma configuração adicional. Basta nomear o arquivo com o sufixo `.module.css`.

---

## 2. Configurando o projeto

Se você ainda não tem um projeto Next.js:

```bash
npx create-next-app@latest meu-projeto
cd meu-projeto
```

Durante o setup, o Next.js pergunta se você quer Tailwind CSS. **Recuse** — vamos trabalhar com CSS puro para realmente entender o que está acontecendo por baixo dos frameworks utilitários.

Estrutura relevante que vamos usar:

```
meu-projeto/
├── app/
│   ├── page.js
│   ├── page.module.css      ← estilos da página inicial
│   └── layout.js
└── components/
    ├── Card.js
    └── Card.module.css       ← estilos do componente Card
```

**Regra de ouro:** cada componente que tem estilos próprios ganha seu próprio arquivo `.module.css`, sentado ao lado dele.

---

## 3. Sintaxe básica: seu primeiro módulo CSS

Crie `app/page.module.css`:

```css
.container {
  padding: 24px;
}

.title {
  color: #1a1a1a;
  font-size: 2rem;
}
```

E em `app/page.js`:

```jsx
import styles from './page.module.css';

export default function Home() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Olá, Next.js!</h1>
    </div>
  );
}
```

Repare em três detalhes que costumam confundir quem vem do CSS tradicional:

| Em CSS puro | Em CSS Modules dentro do JSX |
|---|---|
| `class="container"` | `className={styles.container}` |
| Nomes com hífen (`my-class`) | Prefira `camelCase` (`myClass`) — hífens exigem `styles['my-class']` |
| O CSS é importado no HTML via `<link>` | O CSS é **importado como objeto JS**: `import styles from '...'` |

> 🧠 **Por que `styles.container` e não `"container"`?** O objeto `styles` é um mapa: a chave `container` (o nome que você escreveu no CSS) aponta para o nome único gerado (`page_container__xY2z`). Se você escrever `styles.contaner` (erro de digitação), o React simplesmente não aplica nenhuma classe — vale a pena conferir no DevTools quando um estilo "sumir".

---

## 4. O Box Model — a base de tudo

Todo elemento HTML é, visualmente, uma caixa retangular. Entender as quatro camadas dessa caixa é o que te permite prever como qualquer layout vai se comportar.

```
┌──────────────────────────────────────┐
│              margin                  │  ← espaço FORA do elemento
│  ┌─────────────────────────────────┐ │
│  │            border               │ │  ← a "borda" visível
│  │  ┌───────────────────────────┐  │ │
│  │  │         padding           │  │ │  ← espaço DENTRO, antes do conteúdo
│  │  │  ┌─────────────────────┐  │  │ │
│  │  │  │      conteúdo       │  │  │ │
│  │  │  └─────────────────────┘  │  │ │
│  │  └───────────────────────────┘  │ │
│  └─────────────────────────────────┘ │
└──────────────────────────────────────┘
```

```css
.box {
  width: 300px;
  padding: 16px;      /* espaço interno entre a borda e o conteúdo */
  border: 1px solid #ddd;
  margin: 24px;        /* espaço externo, entre esta caixa e as vizinhas */
}
```

**A armadilha clássica:** por padrão, `width: 300px` mede só o *conteúdo*. Se você soma `padding` e `border`, a caixa fica visualmente maior que 300px. É por isso que praticamente todo projeto profissional começa com:

```css
* {
  box-sizing: border-box;
}
```

Isso muda a regra: `width: 300px` passa a incluir padding e border dentro dos 300px, o que é muito mais intuitivo. Coloque essa regra no seu CSS global (`app/globals.css`), nunca dentro de um módulo (ela precisa afetar tudo).

> ✏️ **Teste mental:** se `.box` tem `width: 300px`, `padding: 20px` e `border: 2px`, qual a largura total renderizada *sem* `box-sizing: border-box`? (Resposta: 300 + 20+20 + 2+2 = 344px)

---

## 5. Seletores essenciais

| Seletor | Exemplo | Quando usar |
|---|---|---|
| Classe | `.card { }` | 95% do seu CSS em React/Next.js — é o que `className` referencia |
| Descendente | `.card p { }` | Estilizar todo `<p>` dentro de `.card`, sem precisar de classe própria |
| Filho direto | `.list > li { }` | Só os `<li>` filhos imediatos, ignora netos |
| Múltiplas classes | `.btn.disabled { }` | Elemento que tem as duas classes ao mesmo tempo |
| Agrupamento | `.title, .subtitle { }` | Aplicar a mesma regra a seletores diferentes |

Dentro de CSS Modules, o seletor descendente continua funcionando normalmente:

```css
/* Card.module.css */
.card p {
  color: #666;
  line-height: 1.5;
}
```

```jsx
<div className={styles.card}>
  <p>Este parágrafo herda o estilo acima, sem precisar de className.</p>
</div>
```

> 💡 Isso é útil quando o conteúdo vem de fora (ex: Markdown renderizado) e você não controla as classes internas.

---

## 6. Layout com Flexbox

Flexbox resolve um problema específico: **alinhar itens em uma única direção** (linha ou coluna), distribuindo espaço entre eles.

```css
.nav {
  display: flex;
  justify-content: space-between; /* eixo principal: espaça os itens */
  align-items: center;            /* eixo cruzado: centraliza verticalmente */
  gap: 16px;                       /* espaço entre os itens, sem precisar de margin */
}
```

O segredo para não se perder em Flexbox é sempre identificar dois eixos:

- **`justify-content`** controla o eixo em que os itens *fluem* (horizontal, por padrão).
- **`align-items`** controla o eixo perpendicular a esse.

Se você mudar `flex-direction: column`, os dois trocam de papel — `justify-content` passa a controlar o vertical.

```css
.sidebar {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
```

> ✏️ **Regra prática:** se você está tentando centralizar algo e não sabe por onde começar, `display: flex; justify-content: center; align-items: center;` resolve 90% dos casos.

---

## 7. Layout com Grid

Enquanto Flexbox pensa em **uma linha ou coluna por vez**, Grid pensa em **linhas e colunas simultaneamente** — ideal para layouts de página inteira ou galerias.

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}
```

`repeat(3, 1fr)` cria três colunas de largura igual (`1fr` = uma "fração" do espaço disponível). Isso é mais poderoso do que parece: combine com `auto-fit` para um grid que se reorganiza sozinho conforme a tela encolhe:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 24px;
}
```

Traduzindo: "encaixe quantas colunas de no mínimo 200px couberem, e distribua o espaço sobrando igualmente." Isso já é responsivo, sem nenhuma media query.

**Quando usar Flexbox vs. Grid?**

| Situação | Ferramenta |
|---|---|
| Barra de navegação, lista de botões | Flexbox |
| Grade de cards, layout de página com header/sidebar/main | Grid |
| Centralizar um único elemento | Flexbox (mais simples) |

---

## 8. Position: tirando elementos do fluxo normal

Por padrão, todo elemento é `position: static` — ele simplesmente segue o fluxo do documento. As outras opções mudam esse comportamento:

```css
.badge {
  position: absolute;
  top: 8px;
  right: 8px;
}

.cardWrapper {
  position: relative; /* necessário para o .badge se posicionar em relação a ele */
}
```

- **`relative`**: o elemento continua no fluxo normal, mas serve como "âncora" para filhos `absolute`.
- **`absolute`**: sai do fluxo e se posiciona em relação ao ancestral `relative` mais próximo (ou à página inteira, se não houver nenhum).
- **`fixed`**: se posiciona em relação à *janela do navegador* — útil para headers que não somem ao rolar a página.
- **`sticky`**: comporta-se como `relative` até atingir um limite de rolagem, então "gruda" como `fixed`.

> 🧠 **Padrão comum:** um badge de "novo" no canto de um card sempre usa a dupla `relative` (no container) + `absolute` (no badge). Memorize essa dupla — ela aparece o tempo todo.

---

## 9. Pseudo-classes e pseudo-elementos

Pseudo-classes descrevem um **estado** do elemento; pseudo-elementos criam algo que **não existe no HTML**.

```css
/* Pseudo-classes: reagem a um estado */
.button:hover {
  background: #2563eb;
}

.input:focus {
  outline: 2px solid #2563eb;
}

.listItem:first-child {
  border-top: none;
}

.listItem:nth-child(even) {
  background: #f9fafb;
}

/* Pseudo-elementos: criam conteúdo virtual */
.quote::before {
  content: '"';
  color: #999;
}

.tooltip::after {
  content: attr(data-label); /* pega o valor do atributo data-label */
}
```

A diferença de sintaxe (`:` vs `::`) não é acidental: `::before`/`::after` são tecnicamente elementos gerados, então o padrão moderno usa dois-pontos duplos para distingui-los das pseudo-classes.

---

## 10. Variáveis CSS (Custom Properties)

Variáveis eliminam a repetição de valores mágicos espalhados pelo código. Diferente de variáveis de pré-processador (Sass), estas são **nativas do CSS** e funcionam em tempo de execução no navegador.

```css
/* app/globals.css */
:root {
  --color-primary: #2563eb;
  --color-text: #1a1a1a;
  --spacing-md: 16px;
  --radius: 8px;
}
```

```css
/* Card.module.css */
.card {
  padding: var(--spacing-md);
  border-radius: var(--radius);
  color: var(--color-text);
}

.card:hover {
  border-color: var(--color-primary);
}
```

`:root` é o seletor da raiz do documento — declarar variáveis ali as torna disponíveis em **qualquer** arquivo `.module.css` do projeto, sem precisar importar nada. É a forma "CSS puro" de ter um design system centralizado.

---

## 11. Composição de classes (`composes`)

Um recurso exclusivo de CSS Modules (não existe em CSS comum): você pode fazer uma classe "herdar" de outra.

```css
/* Button.module.css */
.base {
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 600;
  border: none;
  cursor: pointer;
}

.primary {
  composes: base;
  background: var(--color-primary);
  color: white;
}

.secondary {
  composes: base;
  background: transparent;
  border: 1px solid var(--color-primary);
}
```

```jsx
<button className={styles.primary}>Confirmar</button>
```

O elemento recebe **as duas classes** (`base` e `primary`) automaticamente — sem precisar escrever `className={`${styles.base} ${styles.primary}`}` no JSX. Isso mantém a lógica de estilo dentro do CSS, onde ela pertence.

---

## 12. Responsividade com Media Queries

Media queries aplicam regras condicionalmente, baseadas no tamanho da tela:

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr; /* uma coluna em telas pequenas */
  }
}
```

**Abordagem recomendada: mobile-first.** Escreva o CSS base pensando em telas pequenas, e use `min-width` para *adicionar* complexidade conforme a tela cresce:

```css
.grid {
  display: grid;
  grid-template-columns: 1fr; /* padrão: celular */
  gap: 16px;
}

@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr); /* tablet */
  }
}

@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr); /* desktop */
    gap: 24px;
  }
}
```

> 💡 Mobile-first evita que você fique "desfazendo" estilos de desktop em telas pequenas — você só adiciona, nunca sobrescreve para trás.

---

## 13. Projeto prático: um Card completo

Juntando tudo que vimos, aqui está um componente real:

```css
/* Card.module.css */
.card {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: var(--spacing-md);
  border: 1px solid #e5e7eb;
  border-radius: var(--radius);
  transition: border-color 0.2s ease;
}

.card:hover {
  border-color: var(--color-primary);
}

.badge {
  position: absolute;
  top: -8px;
  right: 12px;
  background: var(--color-primary);
  color: white;
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 999px;
}

.title {
  font-size: 1.125rem;
  font-weight: 600;
}

.title::after {
  content: '';
  display: block;
  width: 32px;
  height: 2px;
  background: var(--color-primary);
  margin-top: 4px;
}

@media (max-width: 480px) {
  .card {
    padding: 12px;
  }
}
```

```jsx
// Card.js
import styles from './Card.module.css';

export default function Card({ title, children, isNew }) {
  return (
    <div className={styles.card}>
      {isNew && <span className={styles.badge}>Novo</span>}
      <h3 className={styles.title}>{title}</h3>
      <p>{children}</p>
    </div>
  );
}
```

Cada decisão aqui remete a uma seção anterior: `position: relative` + `.badge` absoluto (seção 8), `::after` decorativo (seção 9), `var(--color-primary)` (seção 10), media query mobile (seção 12).

---

## 14. Boas práticas

- **Um arquivo `.module.css` por componente.** Evita que um arquivo cresça descontroladamente.
- **Nomes de classe descrevem função, não aparência.** Prefira `.errorMessage` a `.redText` — o texto pode deixar de ser vermelho um dia, mas continuará sendo uma mensagem de erro.
- **Centralize valores repetidos em `:root`.** Cores, espaçamentos e raios de borda usados em 3+ lugares viram variável.
- **Evite `!important`.** Se você precisa dele, geralmente é sinal de que a especificidade do seletor está mal planejada.
- **Prefira `gap` a `margin` entre itens de flex/grid.** Menos propenso a criar espaçamentos duplicados nas bordas.

---

## 15. Exercícios propostos

Pratique na ordem — cada exercício usa o que veio antes:

1. **Box Model:** crie um `.module.css` para um botão com padding, border-radius e uma borda de 2px. Adicione `box-sizing: border-box` e observe a diferença comentando/descomentando essa linha.
2. **Flexbox:** construa uma barra de navegação com um logo à esquerda e três links à direita, todos alinhados verticalmente ao centro.
3. **Grid responsivo:** crie uma galeria de 6 cards que mostra 1 coluna no celular, 2 no tablet e 3 no desktop — sem usar `auto-fit` (para praticar media queries manualmente).
4. **Pseudo-classes:** estilize uma lista onde itens pares têm fundo cinza-claro e o item sob o mouse muda de cor.
5. **Composição:** crie três variantes de botão (`primary`, `danger`, `ghost`) que compartilham uma classe `base` via `composes`.
6. **Projeto integrador:** monte um card de perfil de usuário (foto, nome, cargo, badge "online") combinando Flexbox, `position: absolute` para o badge, e uma variável CSS para a cor de destaque.

Depois de cada exercício, pergunte-se: *"Se eu precisasse explicar essa regra CSS para outra pessoa, eu conseguiria dizer o porquê, não só o como?"* — essa é a diferença entre decorar propriedades e realmente entender CSS.
