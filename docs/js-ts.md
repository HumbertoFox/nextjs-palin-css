# Tutorial: JavaScript e TypeScript na Prática com Next.js

> Um guia progressivo — dos fundamentos da linguagem até tipagem de componentes reais — usando Next.js como ambiente de prática.

---

## Sumário

1. [Por que aprender os dois juntos?](#1-por-que-aprender-os-dois-juntos)
2. [Variáveis e tipos primitivos](#2-variáveis-e-tipos-primitivos)
3. [Funções](#3-funções)
4. [Arrays e seus métodos essenciais](#4-arrays-e-seus-métodos-essenciais)
5. [Objetos e tipos compostos](#5-objetos-e-tipos-compostos)
6. [Condicionais e operadores](#6-condicionais-e-operadores)
7. [Estruturas de repetição](#7-estruturas-de-repetição)
8. [Assincronia: Promises e async/await](#8-assincronia-promises-e-asyncawait)
9. [Módulos: import e export](#9-módulos-import-e-export)
10. [Tipando props de componentes](#10-tipando-props-de-componentes)
11. [Union types, type narrowing e generics](#11-union-types-type-narrowing-e-generics)
12. [Diferenças entre JS e TS que pegam todo mundo](#12-diferenças-entre-js-e-ts-que-pegam-todo-mundo)
13. [Projeto prático: um hook customizado com fetch tipado](#13-projeto-prático-um-hook-customizado-com-fetch-tipado)
14. [Boas práticas](#14-boas-práticas)
15. [Exercícios propostos](#15-exercícios-propostos)

---

## 1. Por que aprender os dois juntos?

Vale alinhar essa relação antes de ver uma linha de código.

**TypeScript não é uma linguagem separada do JavaScript** — é uma camada que adiciona tipos estáticos por cima dele. Todo código JavaScript válido também é código TypeScript válido; o que o TS acrescenta é a possibilidade de declarar *o que cada coisa deveria ser*, e ter isso checado **antes** de rodar, direto no seu editor.

```js
// JavaScript puro: o erro só aparece em tempo de execução
function somar(a, b) {
  return a + b;
}
somar(2, "3"); // retorna "23" (concatenação!), e ninguém avisa
```

```ts
// TypeScript: o erro aparece enquanto você digita
function somar(a: number, b: number): number {
  return a + b;
}
somar(2, "3"); // erro: Argument of type 'string' is not assignable to parameter of type 'number'
```

> 💡 **Por que isso importa no teu contexto:** seu projeto `nextjs-palin-css` já usa `.tsx`, então cada componente que você escreve está, tecnicamente, usando as duas linguagens ao mesmo tempo — a lógica é JavaScript, e as anotações de tipo (`: string`, `: number`, interfaces) são a camada TypeScript por cima.

---

## 2. Variáveis e tipos primitivos

```js
// Declaração — igual em JS e TS
let idade = 25;        // pode ser reatribuído
const nome = "Ana";    // não pode ser reatribuído
var antigo = "evite";  // forma antiga, com problemas de escopo — não use
```

**Regra prática:** use `const` por padrão. Só troque para `let` quando o valor genuinamente precisar mudar (um contador, um acumulador). `var` não deve aparecer em código novo — ele "vaza" para fora de blocos `{ }`, causando bugs difíceis de rastrear.

Os tipos primitivos do JavaScript:

```js
"texto"        // string
42             // number (não existe int/float separados)
true           // boolean
null           // ausência intencional de valor
undefined      // valor não definido
```

Em TypeScript, você pode **anotar** explicitamente qual tipo uma variável deve ter:

```ts
let idade: number = 25;
let nome: string = "Ana";
let ativo: boolean = true;
```

Na prática, você raramente escreve isso para variáveis simples — o TypeScript **infere** o tipo sozinho a partir do valor inicial:

```ts
let idade = 25; // TS já sabe: isto é number, para sempre
idade = "25";   // erro — mesmo sem anotação explícita
```

> ✏️ **Teste mental:** se você escreve `let x = 10;` e depois tenta `x = "dez"`, o TypeScript acusa erro mesmo sem você ter escrito `: number` em lugar nenhum. Por quê? (Resposta: é a **inferência de tipo** — o TS fixa o tipo pelo primeiro valor atribuído.)

---

## 3. Funções

```js
// Função declarada
function saudar(nome) {
  return `Olá, ${nome}!`;
}

// Função de expressão / arrow function — a forma mais usada em React
const saudar = (nome) => {
  return `Olá, ${nome}!`;
};

// Arrow function com retorno implícito (uma linha, sem chaves)
const saudar = (nome) => `Olá, ${nome}!`;
```

As três formas fazem a mesma coisa. Em componentes Next.js/React, a convenção é usar **arrow functions**, principalmente porque elas não criam seu próprio `this` — evitando uma classe inteira de bugs comuns em funções tradicionais dentro de callbacks.

Com TypeScript, você tipa parâmetros e o retorno:

```ts
const saudar = (nome: string): string => `Olá, ${nome}!`;

const somar = (a: number, b: number): number => a + b;
```

Parâmetros opcionais usam `?`, e podem ter valor padrão:

```ts
const saudar = (nome: string, sobrenome?: string): string => {
  return sobrenome ? `Olá, ${nome} ${sobrenome}!` : `Olá, ${nome}!`;
};

const saudarComPadrao = (nome: string = "visitante"): string => `Olá, ${nome}!`;
```

> 🧠 **Diferença sutil:** `sobrenome?: string` aceita `undefined` (o parâmetro pode simplesmente não ser passado). `nome: string = "visitante"` sempre tem um valor — se você não passar nada, o padrão entra em ação. São ferramentas para situações diferentes.

---

## 4. Arrays e seus métodos essenciais

```js
const numeros = [1, 2, 3, 4, 5];
```

Três métodos que aparecem o tempo todo em componentes React/Next.js:

```js
// map: transforma cada item, retorna um NOVO array do mesmo tamanho
const dobrados = numeros.map((n) => n * 2);
// [2, 4, 6, 8, 10]

// filter: mantém só os itens que passam no teste, retorna um array (talvez menor)
const pares = numeros.filter((n) => n % 2 === 0);
// [2, 4]

// reduce: "reduz" o array inteiro a um único valor
const soma = numeros.reduce((acumulador, n) => acumulador + n, 0);
// 15
```

`map` é, de longe, o mais comum em JSX — é como você transforma uma lista de dados em uma lista de elementos:

```tsx
const produtos = ["Caneta", "Caderno", "Lápis"];

<ul>
  {produtos.map((produto) => (
    <li key={produto}>{produto}</li>
  ))}
</ul>
```

> 💡 **Sobre o `key`:** o React exige uma `key` única em listas geradas por `map` para saber qual item mudou entre renderizações, sem precisar comparar a lista inteira. Usar o índice do array (`key={index}`) funciona, mas é desaconselhado quando a lista pode reordenar ou remover itens — prefira um identificador estável (id, nome único).

Em TypeScript, arrays são tipados com `tipo[]`:

```ts
const numeros: number[] = [1, 2, 3];
const nomes: string[] = ["Ana", "Bruno"];
```

---

## 5. Objetos e tipos compostos

```js
const usuario = {
  nome: "Ana",
  idade: 25,
  ativo: true,
};

console.log(usuario.nome);   // acesso por ponto
console.log(usuario["nome"]); // acesso por colchete — útil quando a chave é dinâmica
```

Em TypeScript, você descreve o "formato" de um objeto com `interface` ou `type`:

```ts
interface Usuario {
  nome: string;
  idade: number;
  ativo: boolean;
}

const usuario: Usuario = {
  nome: "Ana",
  idade: 25,
  ativo: true,
};
```

`interface` e `type` fazem, na prática, quase a mesma coisa para descrever objetos. A convenção mais comum em projetos React/Next.js: use `interface` para formatos de objeto (especialmente props de componentes), e `type` quando precisar combinar tipos (união, interseção):

```ts
type Status = "ativo" | "inativo" | "pendente"; // union type — só esses três valores são aceitos
```

Desestruturação — extrair propriedades diretamente em variáveis — é essencial em componentes React:

```ts
const { nome, idade } = usuario;
// equivalente, mais verboso:
// const nome = usuario.nome;
// const idade = usuario.idade;
```

---

## 6. Condicionais e operadores

```js
if (idade >= 18) {
  console.log("Maior de idade");
} else if (idade >= 12) {
  console.log("Adolescente");
} else {
  console.log("Criança");
}
```

Dentro de JSX, `if/else` tradicional não pode ser usado diretamente no meio do markup — usa-se o **operador ternário** ou o **`&&`**:

```tsx
// Ternário: quando há duas possibilidades
<p>{idade >= 18 ? "Maior de idade" : "Menor de idade"}</p>

// && : quando só quer renderizar algo SE a condição for verdadeira
{isLogado && <p>Bem-vindo de volta!</p>}
```

> 🧠 **Cuidado com o `&&` e números:** `{quantidade && <p>Itens: {quantidade}</p>}` parece inofensivo, mas se `quantidade` for `0`, o React renderiza literalmente o número `0` na tela (porque `0` é "falsy", e o `&&` retorna o próprio `0`, não `false`). O jeito seguro: `{quantidade > 0 && <p>...</p>}`.

Operadores que valem memorizar:

```js
=== // igualdade estrita (compara valor E tipo) — sempre prefira este
==  // igualdade "frouxa" (converte tipos antes de comparar) — evite

??  // nullish coalescing: usa o valor da direita SÓ se a esquerda for null/undefined
const nome = usuario.nome ?? "Anônimo";

?.  // optional chaining: acessa propriedades sem quebrar se algo no meio for null/undefined
const cidade = usuario.endereco?.cidade;
```

---

## 7. Estruturas de repetição

```js
// for tradicional
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// for...of — percorre VALORES de um array (o mais usado em código moderno)
for (const numero of numeros) {
  console.log(numero);
}

// for...in — percorre CHAVES de um objeto (menos comum, cuidado com arrays)
for (const chave in usuario) {
  console.log(chave, usuario[chave]);
}
```

Em componentes React, você **quase nunca** escreve um loop `for` diretamente no JSX — o padrão é usar `.map()` (seção 4), porque `map` retorna um array de elementos, que é exatamente o que o JSX consegue renderizar. Um `for` tradicional não retorna nada, então não tem como "encaixar" no meio do markup.

---

## 8. Assincronia: Promises e async/await

Buscar dados de uma API é uma operação que **leva tempo** — o JavaScript não pode travar esperando a resposta. É para isso que existem Promises.

```js
// Uma Promise representa um valor que estará disponível NO FUTURO
fetch("/api/usuarios")
  .then((resposta) => resposta.json())
  .then((dados) => console.log(dados))
  .catch((erro) => console.error(erro));
```

`async/await` é uma forma de escrever o mesmo código, mas com aparência síncrona — muito mais fácil de ler:

```js
async function buscarUsuarios() {
  try {
    const resposta = await fetch("/api/usuarios");
    const dados = await resposta.json();
    console.log(dados);
  } catch (erro) {
    console.error(erro);
  }
}
```

`await` "pausa" a função até a Promise resolver, sem travar o resto da aplicação. Toda função que usa `await` precisa ser declarada com `async` antes.

Com TypeScript, você tipa o que a função assíncrona retorna:

```ts
interface Usuario {
  id: number;
  nome: string;
}

async function buscarUsuarios(): Promise<Usuario[]> {
  const resposta = await fetch("/api/usuarios");
  const dados: Usuario[] = await resposta.json();
  return dados;
}
```

> 💡 **No Next.js especificamente:** Server Components (o padrão na pasta `app/`) permitem usar `await` **diretamente no corpo do componente**, sem precisar de `useEffect`:
> ```tsx
> export default async function Pagina() {
>   const usuarios = await buscarUsuarios();
>   return <ul>{usuarios.map((u) => <li key={u.id}>{u.nome}</li>)}</ul>;
> }
> ```
> Isso só funciona em Server Components (sem `"use client"` no topo do arquivo) — em Client Components, você volta a precisar de `useEffect` + `useState` para buscar dados.

---

## 9. Módulos: import e export

```js
// arquivo utils.js
export function formatarData(data) {
  return data.toLocaleDateString("pt-BR");
}

export const IMPOSTO = 0.1;

// export default: um único "principal" por arquivo
export default function Calculadora() { /* ... */ }
```

```js
// outro arquivo
import Calculadora, { formatarData, IMPOSTO } from "./utils";
```

A diferença chave: **`export default`** não precisa de chaves na importação, e você pode renomear livremente ao importar. **`export` nomeado** exige chaves `{ }`, e o nome deve bater (a menos que você use `as` para renomear).

```js
import { formatarData as formatar } from "./utils"; // renomeando na importação
```

> 🧠 **Convenção comum em Next.js:** componentes geralmente usam `export default` (um componente principal por arquivo), enquanto funções utilitárias e tipos compartilhados usam `export` nomeado — assim você pode importar só o que precisa de um arquivo com várias funções.

---

## 10. Tipando props de componentes

Esta é, provavelmente, a aplicação mais frequente de TypeScript no seu dia a dia com Next.js.

```tsx
interface CardProps {
  titulo: string;
  descricao: string;
  isNovo?: boolean; // opcional
  onClick: () => void; // função que não recebe nem retorna nada
}

export default function Card({ titulo, descricao, isNovo, onClick }: CardProps) {
  return (
    <div onClick={onClick}>
      {isNovo && <span>Novo</span>}
      <h3>{titulo}</h3>
      <p>{descricao}</p>
    </div>
  );
}
```

Usando o componente em outro arquivo, o editor agora **avisa em tempo real** se você esquecer uma prop obrigatória ou passar o tipo errado:

```tsx
<Card
  titulo="Meu produto"
  descricao="Descrição aqui"
  onClick={() => console.log("clicado")}
/>
// Se você esquecer "descricao", o TypeScript acusa erro ANTES de rodar
```

Para tipar `children` (quando um componente envolve outro conteúdo):

```tsx
interface LayoutProps {
  children: React.ReactNode; // aceita qualquer coisa renderizável: texto, JSX, array de elementos
}

export default function Layout({ children }: LayoutProps) {
  return <div className="wrapper">{children}</div>;
}
```

---

## 11. Union types, type narrowing e generics

**Union types** (vistos rapidamente na seção 5) descrevem "um valor entre estas opções":

```ts
type Status = "carregando" | "sucesso" | "erro";

function mensagem(status: Status): string {
  if (status === "carregando") return "Aguarde...";
  if (status === "erro") return "Algo deu errado";
  return "Pronto!";
}
```

**Type narrowing** é o processo de "estreitar" um tipo genérico para um mais específico, dentro de um `if`:

```ts
function processar(valor: string | number) {
  if (typeof valor === "string") {
    return valor.toUpperCase(); // aqui, o TS já sabe que "valor" é string
  }
  return valor.toFixed(2); // aqui, o TS já sabe que é number
}
```

**Generics** permitem escrever código reutilizável sem perder a segurança de tipos — pense neles como "parâmetros para tipos":

```ts
function primeiro<T>(lista: T[]): T {
  return lista[0];
}

primeiro<number>([1, 2, 3]);      // retorna number
primeiro<string>(["a", "b"]);     // retorna string
```

Você provavelmente já usa generics sem perceber — o hook `useState` do React é genérico:

```tsx
const [contador, setContador] = useState<number>(0);
const [usuario, setUsuario] = useState<Usuario | null>(null);
```

`useState<Usuario | null>(null)` diz ao TypeScript: "este estado vai ser um `Usuario`, ou `null` enquanto ainda não carregou" — e o editor passa a te avisar se você tentar acessar `usuario.nome` sem antes checar se `usuario` não é `null`.

---

## 12. Diferenças entre JS e TS que pegam todo mundo

| Situação | JavaScript | TypeScript |
|---|---|---|
| Extensão de arquivo | `.js` / `.jsx` | `.ts` / `.tsx` |
| Tipar uma variável | Não existe — o tipo é só do valor | `let nome: string` |
| Formato de objeto | Nenhuma verificação | `interface` ou `type` |
| Erro de tipo | Só aparece rodando o código | Aparece no editor, antes de rodar |
| Propriedade que pode não existir | `usuario.endereco.cidade` quebra se `endereco` for `undefined` | TS avisa; use `usuario.endereco?.cidade` |
| Compilação | Roda direto no navegador/Node | Precisa ser "transpilado" para JS antes (o Next.js faz isso automaticamente) |

> 💡 **Ponto tranquilizador:** você não precisa saber TypeScript "de cabeça" para começar — pode escrever a lógica em JavaScript puro dentro de um arquivo `.tsx` e ir adicionando tipos aos poucos. O `tsconfig.json` do seu projeto controla o quão rígida é essa checagem (a flag `strict: true` é a mais exigente, e a recomendada a longo prazo).

---

## 13. Projeto prático: um hook customizado com fetch tipado

Juntando assincronia (seção 8), tipagem de objetos (seção 5) e generics (seção 11) em algo reutilizável:

```ts
// hooks/useFetch.ts
"use client";

import { useState, useEffect } from "react";

interface UseFetchResult<T> {
  dados: T | null;
  carregando: boolean;
  erro: string | null;
}

export function useFetch<T>(url: string): UseFetchResult<T> {
  const [dados, setDados] = useState<T | null>(null);
  const [carregando, setCarregando] = useState(true);
  const [erro, setErro] = useState<string | null>(null);

  useEffect(() => {
    async function buscar() {
      try {
        const resposta = await fetch(url);
        if (!resposta.ok) throw new Error("Falha ao buscar dados");
        const json: T = await resposta.json();
        setDados(json);
      } catch (e) {
        setErro(e instanceof Error ? e.message : "Erro desconhecido");
      } finally {
        setCarregando(false);
      }
    }
    buscar();
  }, [url]);

  return { dados, carregando, erro };
}
```

Usando o hook em um componente, com o tipo específico dos dados esperados:

```tsx
// app/usuarios/page.tsx
"use client";

import { useFetch } from "@/hooks/useFetch";

interface Usuario {
  id: number;
  nome: string;
}

export default function Usuarios() {
  const { dados: usuarios, carregando, erro } = useFetch<Usuario[]>("/api/usuarios");

  if (carregando) return <p>Carregando...</p>;
  if (erro) return <p>Erro: {erro}</p>;

  return (
    <ul>
      {usuarios?.map((usuario) => (
        <li key={usuario.id}>{usuario.nome}</li>
      ))}
    </ul>
  );
}
```

Repare que `useFetch<Usuario[]>(...)` faz o TypeScript entender, em toda a cadeia, que `usuarios` é `Usuario[] | null` — e o `?.` na hora de renderizar (seção 6) protege contra o caso de ainda ser `null`. Cada peça deste hook remonta a uma seção anterior do tutorial.

---

## 14. Boas práticas

- **Prefira `const`, evite `var`.** Use `let` só quando o valor realmente precisa mudar.
- **Ative `strict: true` no `tsconfig.json`.** É mais chato no começo, mas evita a maioria dos bugs de `null`/`undefined` em produção.
- **Não use `any` como atalho.** `any` desliga a checagem de tipos naquele ponto — se você não sabe o tipo ainda, `unknown` é mais seguro (ele obriga a fazer uma verificação antes de usar o valor).
- **Tipe o retorno de funções assíncronas explicitamente** (`Promise<Usuario[]>`) — ajuda quem lê o código a saber o que esperar sem precisar rastrear a implementação inteira.
- **Prefira `interface` para props e formatos de objeto**, `type` para unions e composições — não é regra rígida, mas mantém consistência no projeto.
- **Nunca ignore o aviso de `key` faltando em listas.** Ele parece cosmético, mas evita bugs reais de re-renderização.

---

## 15. Exercícios propostos

Pratique na ordem — cada exercício reaproveita conceitos do anterior:

1. **Funções tipadas:** escreva uma função `calcularDesconto(preco: number, percentual: number): number` e teste passando um `string` de propósito para ver o erro do TypeScript no editor.
2. **Array e map:** crie um array de objetos `{ nome: string; preco: number }[]` e renderize uma lista em JSX usando `.map()`, exibindo nome e preço formatado.
3. **Interface de props:** construa um componente `Botao` com `interface BotaoProps { texto: string; variante: "primario" | "secundario"; onClick: () => void }`.
4. **Async/await:** crie uma função `async` que busca dados de uma API pública (ex: `https://jsonplaceholder.typicode.com/users`) e tipe o retorno com uma `interface`.
5. **Hook customizado:** adapte o `useFetch` da seção 13 para aceitar um segundo genérico de erro customizado, em vez de `string`.
6. **Projeto integrador:** monte uma página que usa o `useFetch` para carregar uma lista de posts, com estados de carregamento e erro tratados, e cada post renderizado dentro de um `<Card>` tipado.

Depois de cada exercício, pergunte-se: *"se eu apagasse toda a lógica e deixasse só as assinaturas de tipo, alguém entenderia o que esta função faz?"* — essa é a marca de uma tipagem bem pensada, não só "para o TypeScript parar de reclamar".
