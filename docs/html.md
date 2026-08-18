# Tutorial: HTML na Prática — Estrutura e Tags Essenciais com Next.js

> Um guia progressivo — do documento mais simples possível até uma página completa e semântica — usando Next.js/JSX como ambiente de prática.

---

## Sumário

1. [Por que aprender HTML se eu já uso JSX?](#1-por-que-aprender-html-se-eu-já-uso-jsx)
2. [A estrutura mínima de um documento](#2-a-estrutura-mínima-de-um-documento)
3. [Como o Next.js gera essa estrutura pra você](#3-como-o-nextjs-gera-essa-estrutura-pra-você)
4. [Anatomia de uma tag](#4-anatomia-de-uma-tag)
5. [Tags de texto e conteúdo](#5-tags-de-texto-e-conteúdo)
6. [Tags de estrutura semântica](#6-tags-de-estrutura-semântica)
7. [Listas](#7-listas)
8. [Links e navegação](#8-links-e-navegação)
9. [Imagens e mídia](#9-imagens-e-mídia)
10. [Formulários](#10-formulários)
11. [Tabelas](#11-tabelas)
12. [Diferenças entre HTML e JSX que pegam todo mundo](#12-diferenças-entre-html-e-jsx-que-pegam-todo-mundo)
13. [Projeto prático: uma página "Sobre" completa](#13-projeto-prático-uma-página-sobre-completa)
14. [Boas práticas de semântica e acessibilidade](#14-boas-práticas-de-semântica-e-acessibilidade)
15. [Exercícios propostos](#15-exercícios-propostos)

---

## 1. Por que aprender HTML se eu já uso JSX?

Vale resolver essa dúvida antes de qualquer coisa, porque ela trava muita gente.

JSX **não substitui** HTML — ele é uma sintaxe que se parece com HTML e é traduzida para chamadas de função JavaScript (`React.createElement`) por baixo dos panos. Quando você escreve:

```jsx
<h1 className="title">Olá</h1>
```

O React entende isso como "crie um elemento `<h1>` de verdade no HTML final". Ou seja: **toda tag que existe em HTML continua existindo em JSX**, com o mesmo significado semântico. A diferença está só em alguns detalhes de sintaxe (que veremos na seção 12) e no fato de que o navegador nunca vê o JSX — ele vê o HTML gerado a partir dele.

> 💡 **Por isso este tutorial importa:** escolher `<section>` em vez de `<div>`, ou `<button>` em vez de `<div onClick={...}>`, é uma decisão de HTML — e ela afeta acessibilidade, SEO e comportamento do navegador, independente de você estar em Next.js, PHP ou HTML puro.

---

## 2. A estrutura mínima de um documento

Todo documento HTML válido segue um esqueleto fixo:

```html
<!DOCTYPE html>
<html lang="pt-BR">
  <head>
    <meta charset="UTF-8" />
    <title>Minha Página</title>
  </head>
  <body>
    <h1>Conteúdo visível aqui</h1>
  </body>
</html>
```

Decompondo cada peça:

| Tag | Para que serve |
|---|---|
| `<!DOCTYPE html>` | Diz ao navegador "isto é HTML5" — sem isso, alguns navegadores renderizam em "modo de compatibilidade" com bugs antigos |
| `<html lang="pt-BR">` | Elemento raiz. O atributo `lang` informa leitores de tela e buscadores qual o idioma do conteúdo |
| `<head>` | Metadados — nada aqui aparece diretamente na tela |
| `<meta charset="UTF-8">` | Define a codificação de caracteres (sem isso, acentos podem quebrar) |
| `<title>` | O texto que aparece na aba do navegador |
| `<body>` | Tudo que é visível ao usuário vive aqui |

> ✏️ **Teste mental:** se você remover `<meta charset="UTF-8">`, o que acontece com a palavra "codificação" na tela? (Resposta: os acentos provavelmente viram símbolos estranhos, tipo `codifica├º├úo` — o navegador tenta adivinhar a codificação e erra.)

---

## 3. Como o Next.js gera essa estrutura pra você

Se você olhar seu projeto, não existe um `<html>` escrito manualmente em `page.tsx` — ele vem do `app/layout.tsx`:

```tsx
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="pt-BR">
      <body>{children}</body>
    </html>
  );
}
```

Isso é literalmente a mesma estrutura da seção 2, só que escrita em JSX. O `{children}` é onde o conteúdo de cada página (`app/page.tsx`, `app/sobre/page.tsx`, etc.) é injetado dentro do `<body>`.

O `<head>` e o `<title>` funcionam de forma um pouco diferente — o Next.js gera automaticamente a partir de um `export const metadata`:

```tsx
export const metadata = {
  title: "Minha Página",
};
```

Nos bastidores, isso vira exatamente o `<title>Minha Página</title>` dentro do `<head>` que o navegador recebe. Você não escreve a tag — mas ela existe no HTML final, e é ela que faz o SEO e a aba do navegador funcionarem.

---

## 4. Anatomia de uma tag

Antes de decorar tags, entenda as peças que se repetem em todas elas:

```html
<a href="/sobre" target="_blank">Ir para Sobre</a>
 │  │              │                │            │
 │  └─ atributo     └─ atributo     conteúdo      └─ tag de fechamento
 tag de abertura
```

- **Tag de abertura/fechamento**: `<a>` e `</a>` — a maioria das tags vem em par.
- **Atributos**: pares `nome="valor"` dentro da tag de abertura, que configuram comportamento (`href` diz *para onde* o link vai).
- **Conteúdo**: o que fica entre abertura e fechamento.

Algumas tags não têm conteúdo nem fechamento — são **auto-fecháveis**, porque representam algo que não "contém" nada:

```html
<img src="/foto.jpg" alt="Descrição" />
<br />
<input type="text" />
```

> 🧠 **Em JSX, isso é regra obrigatória:** toda tag auto-fechável *precisa* da barra (`<img />`, nunca `<img>`), enquanto em HTML puro isso é opcional. Voltamos a esse ponto na seção 12.

---

## 5. Tags de texto e conteúdo

```html
<h1>Título principal — só um por página</h1>
<h2>Subtítulo de seção</h2>
<h3>Subtítulo de subseção</h3>

<p>Um parágrafo de texto corrido.</p>

<strong>Texto com importância forte (negrito semântico)</strong>
<em>Texto com ênfase (itálico semântico)</em>

<span>Um pedaço de texto sem significado próprio, só para estilizar</span>
```

O ponto que mais gera confusão aqui: **`<strong>` não é "deixar em negrito"**, é "isto é importante". O navegador escolhe renderizar em negrito por convenção, mas o significado é semântico — leitores de tela, por exemplo, mudam o tom de voz ao ler `<strong>`. Se você só quer o efeito visual de negrito sem significado, isso é trabalho do CSS (`font-weight: bold`), não do HTML.

A hierarquia de `<h1>` a `<h6>` também não é sobre tamanho de fonte — é sobre **estrutura do documento**, como um sumário de livro. Pular de `<h1>` direto pra `<h4>` "porque ficou do tamanho certo" quebra essa hierarquia para quem usa leitor de tela ou para buscadores.

---

## 6. Tags de estrutura semântica

Antes do HTML5, tudo era `<div>`. Hoje, tags semânticas descrevem *o que* cada bloco representa:

```html
<header>Cabeçalho do site ou de uma seção</header>

<nav>Bloco de navegação (menu de links)</nav>

<main>Conteúdo principal da página — só um por página</main>

<section>Uma seção temática dentro do conteúdo</section>

<article>Conteúdo independente e "auto-contido" — um post de blog, por exemplo</article>

<aside>Conteúdo relacionado, mas secundário — uma barra lateral</aside>

<footer>Rodapé do site ou de uma seção</footer>
```

**Como decidir entre `<div>` e uma tag semântica?** Pergunte: *"isto tem um papel identificável na página, ou é só um agrupador visual para CSS?"* Se tem papel (é o menu, é o rodapé, é um post), use a tag semântica. Se é só "preciso de uma caixa pra aplicar flexbox", `<div>` está certo.

```html
<body>
  <header>
    <nav>...</nav>
  </header>

  <main>
    <article>
      <h1>Título do post</h1>
      <p>Conteúdo...</p>
    </article>

    <aside>
      <h2>Posts relacionados</h2>
    </aside>
  </main>

  <footer>
    <p>© 2026</p>
  </footer>
</body>
```

> 💡 **Por que isso importa de verdade:** buscadores usam essa estrutura para entender o que é conteúdo principal versus decoração, e leitores de tela permitem que usuários "pulem" direto para `<main>` ou `<nav>` sem precisar navegar tag por tag. `<div>` em todo lugar não oferece nenhum desses atalhos.

---

## 7. Listas

```html
<!-- Lista não-ordenada: quando a ordem não importa -->
<ul>
  <li>Item</li>
  <li>Item</li>
</ul>

<!-- Lista ordenada: quando a sequência importa -->
<ol>
  <li>Primeiro passo</li>
  <li>Segundo passo</li>
</ol>

<!-- Lista de definição: pares termo/descrição -->
<dl>
  <dt>HTML</dt>
  <dd>Linguagem de marcação para estruturar conteúdo web.</dd>
</dl>
```

**Regra prática:** se você reordenaria os itens sem mudar o sentido (ex: itens de menu), use `<ul>`. Se a ordem carrega significado (ex: passo a passo de uma receita), use `<ol>`.

---

## 8. Links e navegação

```html
<a href="/sobre">Link interno (mesma origem)</a>
<a href="https://exemplo.com" target="_blank" rel="noopener noreferrer">Link externo, nova aba</a>
<a href="#secao-2">Link para uma âncora na mesma página</a>
<a href="mailto:contato@exemplo.com">Abrir cliente de e-mail</a>
```

`target="_blank"` sempre deve vir acompanhado de `rel="noopener noreferrer"` — sem isso, a página aberta tem acesso parcial à página de origem via `window.opener`, o que é um risco de segurança conhecido.

Em Next.js, links internos usam um componente próprio em vez de `<a>` puro:

```tsx
import Link from 'next/link';

<Link href="/sobre">Ir para Sobre</Link>
```

O `<Link>` do Next.js **renderiza um `<a>` de verdade no HTML final** — a diferença é que ele intercepta a navegação em JavaScript para trocar de página sem recarregar o site inteiro (navegação client-side), o que deixa a transição instantânea.

---

## 9. Imagens e mídia

```html
<img src="/gato.jpg" alt="Gato laranja dormindo em um sofá" width="400" height="300" />
```

O atributo `alt` **não é opcional na prática**, mesmo sendo tecnicamente possível omitir: é o texto que leitores de tela leem no lugar da imagem, e é o que aparece se a imagem falhar ao carregar. `alt=""` (vazio, mas presente) só é aceitável quando a imagem é puramente decorativa.

Em Next.js, a tag equivalente é `<Image>`, do pacote `next/image`:

```tsx
import Image from 'next/image';

<Image src="/gato.jpg" alt="Gato laranja dormindo em um sofá" width={400} height={300} />
```

Ela gera um `<img>` no HTML final, mas adiciona otimizações automáticas: redimensionamento, formatos modernos (WebP) e carregamento tardio (`lazy loading`) por padrão. `width` e `height` continuam obrigatórios — o Next.js usa esses valores para reservar o espaço da imagem antes dela carregar, evitando que o layout "pule" (isso se chama *layout shift*).

---

## 10. Formulários

```html
<form action="/enviar" method="POST">
  <label for="nome">Nome</label>
  <input type="text" id="nome" name="nome" required />

  <label for="email">E-mail</label>
  <input type="email" id="email" name="email" required />

  <label for="mensagem">Mensagem</label>
  <textarea id="mensagem" name="mensagem"></textarea>

  <button type="submit">Enviar</button>
</form>
```

Dois detalhes que fazem diferença real:

- **`<label for="nome">` + `<input id="nome">`**: o `for` precisa bater com o `id` do campo. Isso faz clicar no texto do label focar o input automaticamente, e é essencial para leitores de tela anunciarem o que cada campo espera.
- **`type="email"`**: o navegador já valida o formato sozinho, e em celulares troca o teclado exibido para um com `@` visível — tudo isso de graça, sem nenhum JavaScript.

Em componentes React controlados, a estrutura HTML continua a mesma — o que muda é que você amarra `value` e `onChange` ao estado:

```tsx
<input
  type="text"
  id="nome"
  value={nome}
  onChange={(e) => setNome(e.target.value)}
/>
```

---

## 11. Tabelas

Tabelas servem para **dados tabulares** (planilhas, comparativos) — nunca para layout de página, que é trabalho do CSS (flexbox/grid, como vimos no tutorial anterior).

```html
<table>
  <thead>
    <tr>
      <th>Nome</th>
      <th>Cargo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Ana</td>
      <td>Desenvolvedora</td>
    </tr>
  </tbody>
</table>
```

- `<thead>`/`<tbody>` separam cabeçalho de corpo — isso permite estilizar cada parte separadamente e ajuda leitores de tela a anunciar "isto é um cabeçalho de coluna" ao navegar pelas células.
- `<th>` (table header) versus `<td>` (table data): `<th>` é semanticamente um rótulo de coluna/linha, não só uma célula em negrito.

---

## 12. Diferenças entre HTML e JSX que pegam todo mundo

| HTML | JSX (Next.js/React) | Por quê |
|---|---|---|
| `class="card"` | `className="card"` | `class` é palavra reservada em JavaScript |
| `<img>` (fechamento opcional) | `<img />` (obrigatório) | JSX exige que toda tag seja fechada explicitamente |
| `for="nome"` (em `<label>`) | `htmlFor="nome"` | `for` também é palavra reservada em JS (usada em loops) |
| `onclick="..."` (string) | `onClick={funcao}` | Em JSX, eventos recebem uma referência de função JS, não uma string |
| `<!-- comentário -->` | `{/* comentário */}` | Comentários em JSX são expressões JavaScript |
| Atributos em `kebab-case` (`stroke-width`) | Atributos em `camelCase` (`strokeWidth`) | Convenção do JavaScript, aplicada a atributos de SVG dentro de JSX |

> 🧠 **Regra geral pra memorizar:** sempre que um atributo HTML colide com uma palavra reservada do JavaScript (`class`, `for`), o JSX usa uma variação em `camelCase`. Fora isso, a esmagadora maioria dos atributos (`href`, `src`, `id`, `type`, `placeholder`) é idêntica nos dois.

---

## 13. Projeto prático: uma página "Sobre" completa

Juntando as tags das seções anteriores em uma página real de Next.js:

```tsx
// app/sobre/page.tsx
import Image from 'next/image';
import Link from 'next/link';
import styles from './page.module.css';

export const metadata = {
  title: 'Sobre — Meu Site',
};

export default function Sobre() {
  return (
    <main className={styles.main}>
      <header>
        <h1>Sobre mim</h1>
      </header>

      <article>
        <Image
          src="/perfil.jpg"
          alt="Foto de perfil"
          width={200}
          height={200}
        />

        <section>
          <h2>Trajetória</h2>
          <p>
            Desenvolvedor com experiência em <strong>Next.js</strong> e{' '}
            <strong>Laravel</strong>.
          </p>

          <ul>
            <li>Componentização com React</li>
            <li>Templates com Blade</li>
            <li>Estilização com CSS Modules</li>
          </ul>
        </section>

        <section>
          <h2>Contato</h2>
          <form action="/enviar" method="POST">
            <label htmlFor="email">E-mail</label>
            <input type="email" id="email" name="email" required />
            <button type="submit">Enviar</button>
          </form>
        </section>
      </article>

      <aside>
        <h2>Veja também</h2>
        <Link href="/projetos">Meus projetos</Link>
      </aside>

      <footer>
        <p>© 2026</p>
      </footer>
    </main>
  );
}
```

Repare que **cada tag usada aqui tem um papel identificável** só de olhar a estrutura — `<header>` é o cabeçalho, `<article>` é o conteúdo principal auto-contido, `<aside>` é o conteúdo relacionado. Isso é o que "HTML semântico" significa na prática: a marcação já conta a história da página, antes mesmo do CSS entrar em cena.

---

## 14. Boas práticas de semântica e acessibilidade

- **Só um `<h1>` por página.** Ele representa o título principal do documento inteiro, como o título de um livro.
- **Nunca pule níveis de heading** (`<h2>` direto pra `<h4>`) só por causa do tamanho visual — ajuste o tamanho no CSS, não na tag.
- **`alt` em toda imagem que carrega significado.** Pergunte: "se eu não pudesse ver esta imagem, o que eu perderia de informação?" — essa frase é o `alt`.
- **`<button>` para ações, `<a>` para navegação.** Um erro comum é usar `<div onClick={...}>` para simular um botão — isso perde foco por teclado, leitura por leitor de tela e o comportamento nativo de `Enter`/`Espaço` ativarem o clique.
- **Todo `<input>` precisa de um `<label>` associado.** Um `placeholder` não substitui um label — ele some assim que o usuário começa a digitar.

---

## 15. Exercícios propostos

Pratique na ordem — cada exercício reaproveita conceitos do anterior:

1. **Estrutura básica:** crie uma página Next.js nova (`app/curriculo/page.tsx`) com `<header>`, `<main>` e `<footer>`, sem nenhum `<div>` — force-se a escolher a tag semântica certa para cada bloco.
2. **Texto e hierarquia:** dentro do `<main>`, adicione um `<h1>` e ao menos dois `<h2>`, com parágrafos usando `<strong>` para destacar palavras-chave.
3. **Lista de habilidades:** adicione uma `<ul>` com suas tecnologias, e uma `<ol>` descrevendo os passos de um processo (ex: seu fluxo de trabalho em um projeto).
4. **Formulário de contato:** construa um formulário com `label` + `input` corretamente associados via `htmlFor`/`id`, incluindo um campo `type="email"`.
5. **Imagem otimizada:** adicione uma imagem com `next/image`, escrevendo um `alt` que descreva a imagem para alguém que não pode vê-la.
6. **Projeto integrador:** monte uma página de "Projetos" combinando `<article>` (um por projeto), `<Image>`, `<Link>` para cada repositório, e uma `<aside>` com um resumo das tecnologias mais usadas.

Depois de cada exercício, revise sua marcação e pergunte: *"se eu remover todo o CSS agora, essa página ainda faz sentido de cima a baixo?"* — se a resposta for sim, seu HTML está bem estruturado.
