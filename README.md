# Ignite Shop

Languages: [Português (BR)](#pt-br) | [English](#english)

---

## Português (BR) <a id="pt-br"></a>

Aplicação de e-commerce construída com Next.js e Stripe para exibir um catálogo de produtos e realizar checkout via Stripe Checkout. O projeto faz parte de um desafio/prática do Ignite (Rocketseat), com foco em SSR/SSG, estilização com Stitches e integração de pagamentos.

### Demonstração

- Catálogo com carrossel de produtos (Keen Slider)
- Página de detalhes do produto com botão de compra
- Redirecionamento para o Stripe Checkout
- Página de sucesso exibindo dados do pedido (nome do cliente e imagem do produto)

### Sumário

- Visão Geral
- Tecnologias e Stack
- Pré‑requisitos
- Configuração do Ambiente (.env)
- Como rodar o projeto
- Scripts disponíveis
- Build e Deploy
- Estrutura de Pastas
- Funcionalidades
- API interna (Next API Routes)
- Boas práticas e notas
- Licença

### Visão Geral

O Ignite Shop lista produtos provenientes da API do Stripe, permitindo que o usuário visualize o catálogo (gerado estaticamente via SSG com revalidação) e acesse a página de cada produto. A compra é realizada por meio do Stripe Checkout, e ao finalizar o pagamento o usuário é redirecionado para a página de sucesso, onde são exibidas informações básicas da compra.

### Tecnologias e Stack

- Next.js 13 (pasta pages)
- React 18
- TypeScript
- Stitches (@stitches/react) para estilização
- Stripe (SDK Server-side e Checkout)
- Axios para requisições client-side
- Keen Slider para o carrossel na Home

### Pré‑requisitos

- Node.js 18+ (recomendado)
- Yarn ou npm/pnpm
- Conta no Stripe com produtos e preços configurados

### Configuração do Ambiente (.env)

Crie um arquivo `.env.local` na raiz do projeto com as variáveis abaixo:

```
# URL base da aplicação (sem barra no final). Ex.: http://localhost:3000 em dev ou a URL do deploy
NEXT_URL=http://localhost:3000

# Chave secreta do Stripe (Server-side). Ex.: sk_live_...
STRIPE_SECRET_KEY=sk_test_XXXXXXXXXXXXXXXXXXXXXXXX
```

Observações:
- A variável `STRIPE_SECRET_KEY` é utilizada apenas no server (lib/stripe.ts e API Routes). Não exponha essa chave no cliente.
- `NEXT_URL` é utilizada para compor as URLs de sucesso e cancelamento do checkout no endpoint `/api/checkout`.

### Como rodar o projeto

1. Instale as dependências:
   - npm: `npm install`
   - yarn: `yarn`
   - pnpm: `pnpm install`

2. Configure o `.env.local` conforme a seção acima.

3. Rode em desenvolvimento:
   - npm: `npm run dev`
   - yarn: `yarn dev`
   - pnpm: `pnpm dev`

O app iniciará em `http://localhost:3000`.

### Scripts disponíveis

- `dev`: inicia o servidor de desenvolvimento do Next.js.
- `build`: gera a build de produção.
- `start`: inicia o servidor em modo produção (requer `build` prévio).
- `lint`: executa o ESLint com o preset `next/core-web-vitals`.

### Build e Deploy

1. Build de produção: `npm run build`
2. Start em produção: `npm run start`

Em provedores como Vercel:
- Defina as environment variables (`NEXT_URL`, `STRIPE_SECRET_KEY`).
- Certifique-se de que `NEXT_URL` aponta para a URL pública do deploy.

Imagens remotas (Next Image):
- Já está configurado no `next.config.js` o domínio `files.stripe.com` para servir imagens de produtos.

### Estrutura de Pastas

```
src/
├─ assets/
│  └─ logo.svg
├─ lib/
│  └─ stripe.ts        # Instância do Stripe (server-side)
├─ pages/
│  ├─ _app.tsx         # Layout raiz e estilos globais
│  ├─ _document.tsx    # Fonte Roboto e injeção do CSS do Stitches
│  ├─ api/
│  │  └─ checkout.ts   # API Route para criar sessão do Stripe Checkout
│  ├─ index.tsx        # Home com carrossel (Keen Slider) e SSG
│  ├─ product/
│  │  └─ [id].tsx      # Página de produto (SSG + fallback blocking)
│  └─ success.tsx      # Página de sucesso (SSR via getServerSideProps)
└─ styles/
   ├─ global.ts        # Estilos globais (reset e fontes)
   ├─ index.ts         # Configuração do Stitches (tema, tokens)
   └─ pages/           # Estilos por página (app, home, product, success)
```

### Funcionalidades

- Listagem de produtos na Home:
  - `getStaticProps` consulta `stripe.products.list` e expande o `default_price`.
  - Formatação de preço em BRL.
  - Carrossel com Keen Slider (3 itens por view).

- Página de Produto:
  - `getStaticPaths` com 1 exemplo pré-renderizado e `fallback: 'blocking'`.
  - `getStaticProps` consulta `stripe.products.retrieve` com `expand: ['default_price']`.
  - Botão “Comprar agora” que cria sessão de checkout via `/api/checkout` e redireciona o usuário para o Stripe Checkout.

- Página de Sucesso:
  - `getServerSideProps` busca a sessão de checkout pelo `session_id` e expande `line_items` para obter o produto.
  - Exibe nome do cliente e a imagem do produto comprado.
  - `meta name="robots" content="noindex"` para evitar indexação.

### API interna (Next API Routes)

`POST /api/checkout`

- Body: `{ priceId: string }`
- Valida método e `priceId`.
- Monta `success_url` e `cancel_url` usando `NEXT_URL`.
- Cria sessão do checkout com `mode: 'payment'` e `line_items`.
- Retorna: `{ checkoutUrl: string }` para redirecionar o usuário.

Erros possíveis:
- 405 Method not allowed (quando método !== POST)
- 400 Price not found (quando `priceId` não informado)

### Boas práticas e notas

- Variáveis sensíveis (STRIPE_SECRET_KEY) nunca devem ser expostas ao cliente. Use-as apenas em `lib/stripe.ts` e rotas de API.
- Para ambientes de produção, configure webhooks do Stripe se desejar sincronização/automação pós-pagamento (não implementado neste projeto, mas recomendável em apps reais).
- A Home utiliza SSG com `revalidate` para atualizar o catálogo periodicamente sem rebuild manual (2 horas).
- O domínio `files.stripe.com` está permitido para imagens remotas de produtos no Next Image.
- O carrossel depende do CSS de `keen-slider`. Garanta a importação de `keen-slider/keen-slider.min.css` na Home.

### Licença

Este projeto é de estudo/prática. Ajuste a licença conforme sua necessidade.

---

## English <a id="english"></a>

E-commerce application built with Next.js and Stripe to display a product catalog and handle checkout via Stripe Checkout. This project is part of an Ignite (Rocketseat) challenge/practice, focusing on SSR/SSG, Stitches styling, and payment integration.

### Demo

- Catalog with product carousel (Keen Slider)
- Product details page with a purchase button
- Redirect to Stripe Checkout
- Success page showing order details (customer name and product image)

### Table of Contents

- Overview
- Tech Stack
- Prerequisites
- Environment Setup (.env)
- Running Locally
- Available Scripts
- Build & Deploy
- Folder Structure
- Features
- Internal API (Next API Routes)
- Best Practices & Notes
- License

### Overview

Ignite Shop lists products from the Stripe API, allowing users to browse the catalog (statically generated via SSG with revalidation) and access each product page. Purchases are completed through Stripe Checkout, and after payment the user is redirected to a success page with basic order information.

### Tech Stack

- Next.js 13 (pages directory)
- React 18
- TypeScript
- Stitches (@stitches/react) for styling
- Stripe (Server-side SDK and Checkout)
- Axios for client-side requests
- Keen Slider for the carousel on the Home page

### Prerequisites

- Node.js 18+ (recommended)
- Yarn or npm/pnpm
- A Stripe account with products and prices configured

### Environment Setup (.env)

Create a `.env.local` file at the project root with the following variables:

```
# Base application URL (no trailing slash). E.g., http://localhost:3000 in dev or your deployed URL
NEXT_URL=http://localhost:3000

# Stripe secret key (Server-side). E.g., sk_live_...
STRIPE_SECRET_KEY=sk_test_XXXXXXXXXXXXXXXXXXXXXXXX
```

Notes:
- `STRIPE_SECRET_KEY` is used only on the server (lib/stripe.ts and API Routes). Do not expose this key on the client.
- `NEXT_URL` is used to build success and cancel URLs for checkout in the `/api/checkout` endpoint.

### Running Locally

1. Install dependencies:
   - npm: `npm install`
   - yarn: `yarn`
   - pnpm: `pnpm install`

2. Configure `.env.local` as described above.

3. Start the development server:
   - npm: `npm run dev`
   - yarn: `yarn dev`
   - pnpm: `pnpm dev`

The app will be available at `http://localhost:3000`.

### Available Scripts

- `dev`: starts the Next.js development server
- `build`: creates the production build
- `start`: starts the server in production mode (requires prior `build`)
- `lint`: runs ESLint using the `next/core-web-vitals` preset

### Build & Deploy

1. Production build: `npm run build`
2. Start in production: `npm run start`

On platforms like Vercel:
- Set environment variables (`NEXT_URL`, `STRIPE_SECRET_KEY`).
- Ensure `NEXT_URL` points to your public deployment URL.

Remote Images (Next Image):
- `files.stripe.com` is already allowed in `next.config.js` to serve product images.

### Folder Structure

```
src/
├─ assets/
│  └─ logo.svg
├─ lib/
│  └─ stripe.ts        # Stripe instance (server-side)
├─ pages/
│  ├─ _app.tsx         # Root layout and global styles
│  ├─ _document.tsx    # Roboto font and Stitches CSS injection
│  ├─ api/
│  │  └─ checkout.ts   # API Route to create a Stripe Checkout session
│  ├─ index.tsx        # Home with carousel (Keen Slider) and SSG
│  ├─ product/
│  │  └─ [id].tsx      # Product page (SSG + fallback blocking)
│  └─ success.tsx      # Success page (SSR via getServerSideProps)
└─ styles/
   ├─ global.ts        # Global styles (reset and fonts)
   ├─ index.ts         # Stitches configuration (theme, tokens)
   └─ pages/           # Per-page styles (app, home, product, success)
```

### Features

- Home (Catalog):
  - `getStaticProps` calls `stripe.products.list` and expands the default price.
  - Price formatted to BRL.
  - Carousel with Keen Slider (3 items per view).

- Product Page:
  - `getStaticPaths` with one pre-rendered example and `fallback: 'blocking'`.
  - `getStaticProps` calls `stripe.products.retrieve` with `expand: ['default_price']`.
  - “Buy now” button creates a checkout session via `/api/checkout` and redirects the user to Stripe Checkout.

- Success Page:
  - `getServerSideProps` retrieves the checkout session by `session_id` and expands `line_items` to get the product.
  - Displays the customer name and the product image.
  - `meta name="robots" content="noindex"` to avoid indexing.

### Internal API (Next API Routes)

`POST /api/checkout`

- Body: `{ priceId: string }`
- Validates HTTP method and `priceId`.
- Builds `success_url` and `cancel_url` using `NEXT_URL`.
- Creates a checkout session with `mode: 'payment'` and `line_items`.
- Returns: `{ checkoutUrl: string }` to redirect the user.

Possible errors:
- 405 Method not allowed (when method !== POST)
- 400 Price not found (when `priceId` is missing)

### Best Practices & Notes

- Sensitive variables (STRIPE_SECRET_KEY) must never be exposed to the client. Use them only in `lib/stripe.ts` and API routes.
- For production environments, consider configuring Stripe webhooks for post-payment automation (not implemented in this project, but recommended for real apps).
- Home uses SSG with `revalidate` to update the catalog periodically without manual rebuilds (2 hours).
- The `files.stripe.com` domain is allowed for remote images in Next Image.
- The carousel depends on `keen-slider` CSS. Ensure `keen-slider/keen-slider.min.css` is imported in Home.

### License

This project is for study/practice. Adjust the license as needed.