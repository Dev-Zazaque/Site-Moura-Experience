<div align="center">

<img src="Moura.png" alt="Moura Experience" width="160" />

# Moura Experience

**Plataforma web de vendas e assistência técnica Apple, com catálogo dinâmico, visualização 3D e simulador de troca.**

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)](#stack-tecnológica)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)](#stack-tecnológica)
[![JavaScript](https://img.shields.io/badge/JavaScript_ES2022-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](#stack-tecnológica)
[![model-viewer](https://img.shields.io/badge/model--viewer-3.4.0-4285F4?style=for-the-badge&logo=google&logoColor=white)](#visualizador-3d)
[![Zero Build](https://img.shields.io/badge/build-zero_config-00A3FF?style=for-the-badge)](#instalação-e-execução)

[![Ver site ao vivo](https://img.shields.io/badge/🌐_Ver_site_ao_vivo-site--moura--cell.vercel.app-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://site-moura-cell.vercel.app/)

[Visão geral](#visão-geral) · [Funcionalidades](#funcionalidades) · [Arquitetura](#arquitetura) · [Instalação](#instalação-e-execução) · [Customização](#customização) · [Roadmap](#roadmap)

</div>

---

## Visão geral

> 🔗 **Demo online:** [site-moura-cell.vercel.app](https://site-moura-cell.vercel.app/)

O **Moura Experience** é uma landing page *single-page* de alta performance para uma loja de iPhones e produtos Apple localizada em **Itaguaí – RJ**. O projeto une a linguagem visual minimalista da Apple a uma conversão orientada ao **WhatsApp**, canal principal de venda do negócio.

O site foi desenhado com três objetivos:

1. **Converter:** cada produto, simulação e orçamento termina em uma conversa de WhatsApp com a mensagem já preenchida.
2. **Transmitir confiança:** garantia de 1 ano, aparelhos lacrados e seminovos testados, localização física e prova social em destaque.
3. **Impressionar sem custo de infraestrutura:** experiência premium (3D, animações, logo em SVG animado) entregue como site estático, sem *build step* e sem backend.

---

## Funcionalidades

### Catálogo dinâmico
- Renderização client-side a partir de um array tipado (`produtos`), com **filtro por categoria** (seminovo, lacrado, Apple) e **busca em tempo real** insensível a acentos e maiúsculas.
- Cards com *badge*, capacidade, preço e dois CTAs (compra e dúvida), ambos com mensagem contextual por produto.
- Imagens carregadas por convenção de nome (`img/<slug-do-modelo>.png`) com *lazy loading* e **fallback gracioso** quando a imagem não existe.

### Visualizador 3D
- Modelos `.glb` interativos via [`<model-viewer>`](https://modelviewer.dev/), com rotação automática, controle por toque/mouse e troca de modelo por *chips*.
- **Estado de carregamento e de erro** tratados, com indicador visual e mensagem de falha sem quebrar a página.
- **Pré-carregamento oportunista** dos modelos secundários após o `load`, para troca quase instantânea.

### Simulador de troca
Calculadora que estima o valor do iPhone usado do cliente, com regras de negócio explícitas e auditáveis:

| Fator | Regra |
|---|---|
| **Modelo** | Valor base de referência por geração (iPhone 11 → 16 Pro Max) |
| **Armazenamento** | Ajuste por faixa de capacidade (64 GB a 1 TB) relativo ao GB de referência |
| **Estado** | Excelente (100%), Bom (85%) ou Regular (70%) |
| **Defeitos** | Descrição livre analisada por **expressões regulares sem acento**, com desconto por categoria (Face ID, tela, câmera, bateria, Wi-Fi, líquido, etc.) |
| **Piso** | Valor mínimo de 20% do valor base, com aviso de confirmação presencial |
| **Bloqueio** | Detecta menção a iCloud/Conta Apple e alerta sobre liberação prévia |

O resultado exibe um **detalhamento linha a linha** e gera uma mensagem de WhatsApp pronta com todos os dados da simulação.

### Assistência técnica
Formulário de orçamento que valida os campos e abre o WhatsApp com aparelho, defeito e nome do cliente estruturados.

### Identidade e experiência visual
- **Tela de abertura (splash)** com logotipo vetorial animado em SVG (traçado progressivo via `pathLength`).
- **Título do hero animado** letra a letra, mantendo texto acessível para leitores de tela.
- **Fundo do hero** em carrossel contínuo de fotos dos aparelhos, com pré-carregamento para evitar *flicker*.
- **Mockup do perfil do Instagram** renderizado em um iPhone (dispositivos de toque) ou MacBook (dispositivos com mouse), alternado por *media queries* de `hover`/`pointer`.
- Animações de revelação ao rolar via `IntersectionObserver`, barra de progresso de leitura e navegação em *pill* com vidro fosco (`backdrop-filter`).
- Botão flutuante de WhatsApp e seção de localização com Google Maps incorporado.

---

## Arquitetura

O projeto é intencionalmente um **site estático de arquivo único**, priorizando simplicidade de deploy, custo zero de hospedagem e tempo de carregamento mínimo.

```text
┌──────────────────────────────────────────────────────────┐
│                       index.html                         │
│                                                          │
│  <head>   SEO · Open Graph · JSON-LD · Tailwind config   │
│  <style>  Design tokens, componentes, animações          │
│  <body>   Seções semânticas (nav, hero, catálogo, 3D…)   │
│  <script> Estado, renderização, regras de negócio        │
└───────────────┬───────────────────────┬──────────────────┘
                │                       │
        ┌───────▼───────┐       ┌───────▼────────┐
        │  img/*.png    │       │   *.glb (3D)   │
        │  convenção:   │       │  model-viewer  │
        │  slug(modelo) │       │  lazy + cache  │
        └───────────────┘       └────────────────┘
                         │
                 ┌───────▼────────┐
                 │  WhatsApp API  │  wa.me/<número>?text=…
                 │  (conversão)   │
                 └────────────────┘
```

### Decisões técnicas

- **Sem framework e sem *build***: JavaScript nativo com templates literais. Menos *overhead*, deploy imediato em qualquer host estático.
- **Dados separados da apresentação**: `produtos`, `TM` (tabela de troca), `DEF` (defeitos) e `depoimentos` são estruturas de dados no topo do script, editáveis sem tocar na lógica.
- **Normalização de texto** (`NFD` + remoção de diacríticos) compartilhada entre busca e detecção de defeitos, garantindo comportamento consistente em português.
- **Degradação graciosa**: imagem ausente, falha no modelo 3D ou fallback de fundo não interrompem a experiência.
- **Acessibilidade**: `lang="pt-BR"`, `aria-label` em ícones e links, texto `sr-only` no título animado e `:focus-visible` customizado.

---

## Stack tecnológica

| Camada | Tecnologia |
|---|---|
| Marcação | HTML5 semântico |
| Estilo | [Tailwind CSS](https://tailwindcss.com/) (CDN) + CSS customizado com *design tokens* |
| Lógica | JavaScript (ES2022), sem dependências de runtime próprias |
| 3D | [`@google/model-viewer`](https://modelviewer.dev/) 3.4.0 |
| Ícones | [Lucide](https://lucide.dev/) 0.460.0 |
| Tipografia | SF Pro Display (sistema) com *fallback* para [Manrope](https://fonts.google.com/specimen/Manrope) |
| Mapas | Google Maps Embed |
| Conversão | WhatsApp Click-to-Chat |

---

## SEO e dados estruturados

- `<title>` e `meta description` otimizados para busca local ("iPhones e Assistência Apple em Itaguaí").
- Tags **Open Graph** para pré-visualização em redes sociais e mensageiros.
- **JSON-LD** (`schema.org/MobilePhoneStore`) com nome, telefone, endereço postal e horário de funcionamento, favorecendo *rich results* e o Google Meu Negócio.
- `theme-color`, `apple-touch-icon` e favicon embutidos.

---

## Estrutura do repositório

```text
.
├── index.html            # Aplicação completa (markup, estilos e lógica)
├── Moura.png             # Logotipo / imagem Open Graph
├── img/                  # Fotos dos produtos: <slug-do-modelo>.png
│   ├── iphone-11.png
│   ├── iphone-12.png
│   ├── iphone-13-pro.png
│   ├── iphone-14-pro-max.png
│   ├── iphone-15.png
│   ├── iphone-15-pro-max.png
│   ├── iphone-16.png
│   ├── iphone-16-pro-max.png
│   ├── iphone-17-pro.png
│   ├── iphone-17-pro-max.png
│   ├── apple-watch-se-2.png
│   └── macbook-air-m5.png
├── iphone-15.glb         # Modelos 3D
├── iphone-16.glb
├── iphone_17.glb
├── apple_.glb
├── macbook_.glb
└── README.md
```

> **Importante:** o `index.html` busca as fotos em `img/<slug>.png`, onde o *slug* é o nome do modelo em minúsculas, sem acentos e com hifens (ex.: `iPhone 15 Pro Max` → `iphone-15-pro-max.png`). Já os arquivos `.glb` ficam na **raiz** do projeto.

---

## Instalação e execução

Não há dependências para instalar nem etapa de *build*.

```bash
# 1. Clone o repositório
git clone https://github.com/<seu-usuario>/<seu-repositorio>.git
cd <seu-repositorio>

# 2. Sirva os arquivos com qualquer servidor estático
npx serve .
# ou
python3 -m http.server 8080
```

Acesse `http://localhost:8080`.

> Os modelos 3D são carregados via `fetch`/XHR e **não funcionam abrindo o arquivo diretamente** (`file://`). Use sempre um servidor local.

### Deploy

A versão em produção está hospedada na **Vercel**: [site-moura-cell.vercel.app](https://site-moura-cell.vercel.app/).

Compatível com qualquer hospedagem estática, sem configuração adicional:

| Plataforma | Como publicar |
|---|---|
| **GitHub Pages** | *Settings → Pages → Deploy from branch → `main` / root* |
| **Netlify / Vercel** | Importar o repositório (sem *build command*, diretório de saída `.`) |
| **Cloudflare Pages** | Conectar o repositório, sem *framework preset* |

---

## Customização

Todo o conteúdo comercial fica concentrado no topo do bloco `<script>`.

### Produtos

```js
const produtos = [
  {
    modelo: "iPhone 15",
    gb: "128GB",
    preco: "4.199,00",
    tipo: "lacrado",          // "seminovo" | "lacrado" | "apple"
    badge: "Lacrado Apple",
  },
  // ...
];
```

Para adicionar um produto, inclua o objeto no array e salve a foto em `img/<slug-do-modelo>.png`.

### Tabela de troca

```js
// [modelo, valor base (R$), GB de referência, GBs disponíveis]
["iPhone 15 Pro", 4100, 128, [128, 256, 512, 1024]],
```

Os descontos por defeito (`DEF`) e por armazenamento (`GB_D`) também são editáveis e independentes da lógica de cálculo.

### WhatsApp

```js
const WA = "5521972152070"; // DDI + DDD + número, apenas dígitos
```

Altere também o número no JSON-LD e nos links `wa.me` estáticos do HTML.

### Depoimentos

Substitua o array `depoimentos` por avaliações reais do Google Meu Negócio.

### Identidade visual

As cores são definidas em `tailwind.config` (`blue: #00a3ff`, `ink`, `s1`–`s3`, `mute`) e podem ser trocadas em um único ponto.

---

## Desempenho e boas práticas

- Imagens de produto com `loading="lazy"` e `decoding="async"`.
- Pré-carregamento de imagens do hero antes de iniciar a animação.
- *Preconnect* para Google Fonts e `font-display: swap`.
- Animações baseadas em `transform` e `opacity` (compostas pela GPU).
- Observadores (`IntersectionObserver`) desconectados após o primeiro disparo.
- `rel="noopener"` em todos os links externos com `target="_blank"`.

---

## Roadmap

- [ ] Migrar Tailwind de CDN para *build* com *purge* (redução de CSS e remoção do aviso de produção).
- [ ] Extrair CSS e JS para arquivos próprios e separar dados em `products.json`.
- [ ] Converter imagens para WebP/AVIF e comprimir modelos `.glb` (Draco/Meshopt).
- [ ] Painel simples para atualização de preços sem editar código.
- [ ] Testes automatizados da calculadora de troca (regras de negócio).
- [ ] Auditoria Lighthouse (Performance, Acessibilidade, SEO) com metas ≥ 95.

---

## Informações do negócio

| | |
|---|---|
| **Loja** | Moura Experience |
| **Endereço** | Av. Dep. Octávio Cabral, 740, Jardim América, Itaguaí – RJ, 23810-301 |
| **Horário** | Seg–Sex, 10h às 19h · Sáb, 10h às 16h |
| **Instagram** | [@moura.experience](https://instagram.com/moura.experience) |

---

## Licença

© 2026 Moura Experience. Todos os direitos reservados.

O código-fonte, a identidade visual e os materiais deste repositório são de uso exclusivo do proprietário. *iPhone*, *MacBook*, *Apple Watch* e *Apple* são marcas registradas da Apple Inc. Este projeto não é afiliado, patrocinado ou endossado pela Apple.
