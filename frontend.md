# Frontend

## 📌 Visão geral

O Frontend do Voto Vivo é responsável pela interface web utilizada pelos usuários para consultar as informações disponibilizadas pelo sistema.

A aplicação foi desenvolvida utilizando **Next.js, React e TypeScript**, com estilização baseada em **Tailwind CSS**.

---

## 🧰 Tecnologias

As principais tecnologias utilizadas são:

* **Next.js**
* **React**
* **TypeScript**
* **Tailwind CSS**
* **ESLint**

A aplicação também possui configuração para execução utilizando Docker.

---

## 🔌 Comunicação com o Backend

O Frontend não acessa diretamente o banco de dados.

As informações são obtidas por meio da API disponibilizada pelo Backend.

O fluxo é:

```text
Usuário
   │
   ▼
Frontend
   │
   │ HTTP
   ▼
Backend API
   │
   ▼
Banco de Dados
```

A URL base da API pode ser configurada por meio da variável de ambiente:

```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

O valor deve ser ajustado de acordo com a configuração utilizada no ambiente de desenvolvimento.

---

## ▶️ Execução

Para executar o Frontend localmente:

```bash
npm install
```

Em seguida:

```bash
npm run dev
```

A aplicação será disponibilizada pelo servidor de desenvolvimento do Next.js.

Para gerar uma versão de produção:

```bash
npm run build
```

E para iniciar a aplicação:

```bash
npm start
```
---

## 📂 Estrutura do Projeto

O frontend do **Voto Vivo** está organizado em pastas que separam as rotas da aplicação, os componentes visuais, os serviços de comunicação com a API, os tipos de dados e os arquivos estáticos. Essa organização facilita a manutenção, reutilização de componentes e evolução das funcionalidades.

### Estrutura principal

```text
VotoVivoFrontEnd/
├── app/
├── components/
├── docs/
├── public/
├── services/
├── types/
├── Dockerfile
├── package.json
├── next.config.ts
├── tsconfig.json
└── README.md
```

## 📂 O que cada pasta contém

### `app/`

Responsável pelas rotas e páginas da aplicação, utilizando o **App Router do Next.js**. Cada diretório representa uma rota, enquanto os arquivos `page.tsx` definem o conteúdo exibido em cada página.

- `page.tsx`: página inicial da aplicação.
- `layout.tsx`: estrutura global compartilhada entre as páginas.
- `globals.css`: estilos globais da aplicação.
- `loading.tsx`: interface de carregamento.
- `candidatos/`: listagem de candidaturas e páginas de detalhes de cada candidatura.
- `educacao/`: página de conteúdo educativo sobre o sistema político e eleitoral brasileiro.
- `parlamentares/`: listagem de parlamentares, perfis individuais e detalhes de emendas.
- `proposicoes/`: listagem e detalhes de proposições legislativas.
- `projeto/`: página institucional com informações sobre o projeto Voto Vivo.

### `components/`

Armazena os componentes reutilizáveis da interface, organizados por funcionalidade. Cada subpasta reúne os elementos visuais relacionados a uma área específica da aplicação.

| Subpasta | Responsabilidade |
|---|---|
| `candidatos/` | Cards, informações, resultados e indicadores das candidaturas. |
| `educacao/` | Componentes da área educativa, incluindo seções de conteúdo, navegação, dados e elementos compartilhados. |
| `eleicoes/` | Componentes para apresentação de resultados eleitorais, candidaturas e representação gráfica do Congresso. |
| `home/` | Componentes da página inicial, como mapas, destaques, buscas, estatísticas e cards de parlamentares. |
| `layout/` | Elementos estruturais compartilhados, como cabeçalho, navegação, rodapé e botões de retorno. |
| `parlamentares/` | Componentes de busca, rankings, perfis parlamentares, painéis de informações e detalhes de emendas. |
| `projeto/` | Componentes da página institucional do projeto. |
| `proposicoes/` | Componentes para apresentação de autores, documentos, tramitações, votações e informações das proposições. |

**Organização interna de `components/parlamentares/`:**

- `busca/`: componentes de busca avançada e apresentação de resultados de rankings.
- `emendas/`: componentes para visualização detalhada de emendas parlamentares.
- `profile/`: estrutura do perfil parlamentar, indicadores, informações principais e painéis.
- `profile/panels/`: painéis específicos para visão geral, despesas, emendas, proposições, votações e dashboards.
- `profile/shared/`: componentes e funções auxiliares compartilhadas entre os painéis, como formatação, paginação e controle de layout.

**Organização interna de `components/educacao/`:**

- `data/`: dados e informações utilizados nas seções educativas.
- `layout/`: elementos de estrutura e navegação da página.
- `sections/`: seções de conteúdo educativo sobre o sistema eleitoral, Congresso, parlamentares, emendas e outros temas.
- `shared/`: componentes visuais reutilizados entre as seções.

### `services/`

Contém os serviços responsáveis pela comunicação entre o frontend e a API do Voto Vivo. Centraliza as requisições HTTP, a obtenção de dados e as operações relacionadas a cada funcionalidade.

- `api.ts`: configuração e funcionalidades compartilhadas para comunicação com a API.
- `candidatos.ts`: requisições relacionadas às candidaturas.
- `eleicoes.ts`: serviços de obtenção de dados e resultados eleitorais.
- `parlamentares.ts`: requisições relacionadas aos parlamentares e suas informações.
- `proposicoes.ts`: serviços de consulta de proposições legislativas.

### `types/`

Armazena as definições de tipos e interfaces do TypeScript, padronizando a estrutura dos dados utilizados pela aplicação.

- `index.ts`: centraliza os principais tipos e interfaces compartilhados entre os módulos.

### `public/`

Armazena arquivos estáticos utilizados na aplicação, como imagens, ícones, ilustrações e outros recursos visuais.

- `imagens/`: imagens utilizadas nas páginas e componentes da plataforma.

### `docs/`

Reúne documentos técnicos e arquivos de referência que auxiliam no desenvolvimento e na manutenção das funcionalidades.

- `EMENDAS_REQUISITOS.md`: documentação dos requisitos relacionados às emendas parlamentares.
- `TRAMITACAO_REQUISITOS.md`: documentação dos requisitos relacionados à tramitação de proposições.
- `VOTACOES_REQUISITOS.md`: documentação dos requisitos relacionados às votações legislativas.

---
## ⚙️ Arquivos de configuração

| Arquivo | Descrição |
|---|---|
| `package.json` | Define as dependências, informações e scripts do projeto. |
| `package-lock.json` | Registra as versões exatas das dependências instaladas. |
| `Dockerfile` | Define as instruções para construção da imagem Docker do frontend. |
| `next.config.ts` | Configurações do framework Next.js. |
| `tsconfig.json` | Configurações do compilador TypeScript. |
| `eslint.config.mjs` | Configurações das regras de análise e padronização do código. |
| `postcss.config.mjs` | Configurações do processamento de CSS com PostCSS. |
| `next-env.d.ts` | Declarações de tipos utilizadas pelo Next.js. |

---

Essa estrutura permite separar as responsabilidades da aplicação, mantendo as páginas, os componentes visuais e a comunicação com a API organizados de maneira independente, facilitando a manutenção e a implementação de novas funcionalidades no Voto Vivo.



---

## 🧪 Qualidade do código

O projeto utiliza **ESLint** para auxiliar na identificação de problemas e na padronização do código.

A execução das ferramentas de desenvolvimento deve seguir os scripts definidos no `package.json`.

---

## 🐳 Docker

O projeto possui configuração para utilização de Docker.

Quando utilizado, o ambiente pode ser executado de forma isolada, reduzindo diferenças entre ambientes de desenvolvimento.

As configurações específicas devem ser consultadas nos arquivos Docker presentes no repositório.

---

## 📚 Documentação relacionada

* [Arquitetura do Sistema](./arquitetura.md)
* [Backend](./backend.md)
* [API](./api.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
