# Diagramas

Diagramas em [Mermaid](https://mermaid.js.org/) que ilustram o funcionamento do Voto Vivo. Todos os elementos foram extraídos da documentação existente.

## 📚 Índice

1. [Visão geral da arquitetura](#1-visão-geral-da-arquitetura)
2. [Integração entre componentes](#2-integração-entre-componentes)
3. [Fontes de dados](#3-fontes-de-dados)
4. [Pipeline de ETL](#4-pipeline-de-etl)
5. [Controle de requisições externas](#5-controle-de-requisições-externas)
6. [Fluxo de uma requisição (Frontend → Banco)](#6-fluxo-de-uma-requisição-frontend--banco)
7. [Fluxo de navegação do usuário](#7-fluxo-de-navegação-do-usuário)
8. [Endpoints da API](#8-endpoints-da-api)
9. [Camadas do Backend](#9-camadas-do-backend)
10. [Estrutura do Frontend](#10-estrutura-do-frontend)
11. [Dados armazenados](#11-dados-armazenados)
12. [Ambiente de desenvolvimento](#12-ambiente-de-desenvolvimento)
13. [Fluxo de desenvolvimento](#13-fluxo-de-desenvolvimento)
14. [Infraestrutura atual](#14-infraestrutura-atual)

---

## 1. Visão geral da arquitetura

```mermaid
flowchart TD
    FO["Fontes Oficiais<br/>Câmara · Senado · Portal da Transparência"]
    DA["Data Aggregator<br/>Python · ETL"]
    DB[("MySQL<br/>dados estruturados")]
    BE["Backend<br/>Node.js · Express · Prisma"]
    FE["Frontend<br/>Next.js · React"]
    U(["Usuário"])

    FO -->|APIs HTTP| DA
    DA -->|escrita| DB
    DB -->|leitura via Prisma| BE
    BE -->|API REST / JSON| FE
    FE --> U
```

---

## 2. Integração entre componentes

Mostra quem se comunica com quem e por qual protocolo. O Frontend nunca acessa o banco diretamente; o banco é compartilhado entre Data Aggregator (escrita) e Backend (leitura).

```mermaid
flowchart LR
    subgraph Externo["Externo"]
        CD["API Câmara dos Deputados"]
        SF["API Senado Federal"]
        PT["API Portal da Transparência<br/>(requer chave de API)"]
    end

    subgraph Ingestao["Repositório: Data Aggregator"]
        PR["principal.py<br/>carga completa"]
        AS["atualizacao_semanal.py<br/>atualização periódica"]
    end

    subgraph Dados["Persistência"]
        MY[("MySQL 8<br/>votovivo")]
    end

    subgraph Servico["Repositório: Backend"]
        API["API REST<br/>Express"]
        PRI["Prisma ORM"]
        SW["Swagger UI<br/>/api-docs"]
    end

    subgraph Cliente["Repositório: Frontend"]
        WEB["Next.js · React<br/>Tailwind CSS"]
    end

    U(["Usuário"])

    CD -->|HTTP| PR
    SF -->|HTTP| PR
    PT -->|HTTP| PR
    CD -->|HTTP| AS
    SF -->|HTTP| AS
    PT -->|HTTP| AS

    PR -->|"leitura e escrita"| MY
    AS -->|"leitura e escrita"| MY

    API --> PRI
    PRI -->|SQL| MY
    API --- SW

    WEB -->|"HTTP · NEXT_PUBLIC_API_URL"| API
    U -->|navegador| WEB
```

---

## 3. Fontes de dados

Origem dos dados e entidades armazenadas no banco.

```mermaid
flowchart LR
    CD["Câmara dos Deputados"]
    SF["Senado Federal"]
    PT["Portal da Transparência"]

    DA["Data Aggregator"]

    subgraph DB["MySQL"]
        direction TB
        subgraph Negocio["Dados de negócio"]
            E1["Parlamentares"]
            E2["Partidos"]
            E3["Mandatos"]
            E4["Órgãos"]
            E5["Blocos parlamentares"]
            E6["Proposições"]
            E7["Tramitações"]
            E8["Votações"]
            E9["Despesas"]
            E10["Emendas"]
            E11["Eventos"]
            E12["Presenças"]
        end
        subgraph Controle["Controle do ETL"]
            C1["Execuções"]
            C2["Checkpoints"]
            C3["Erros"]
        end
    end

    CD --> DA
    SF --> DA
    PT --> DA
    DA --> Negocio
    DA <--> Controle
```

> A documentação lista os grupos de dados, mas não detalha de qual fonte vem cada entidade. Consulte o repositório do Data Aggregator para esse mapeamento.

---

## 4. Pipeline de ETL

```mermaid
flowchart LR
    subgraph Extract["Extract"]
        X1["Consulta APIs oficiais"]
    end

    subgraph Transform["Transform"]
        T1["Transformação"]
        T2["Padronização"]
        T3["Relacionamento entre entidades"]
        T1 --> T2 --> T3
    end

    subgraph Load["Load"]
        L1["Insere novos registros"]
        L2["Atualiza registros existentes"]
    end

    DB[("MySQL")]
    CTRL[("Execuções · Checkpoints · Erros")]

    X1 --> T1
    T3 --> L1
    T3 --> L2
    L1 --> DB
    L2 --> DB

    X1 -.registra.-> CTRL
    T3 -.registra erros.-> CTRL
    L2 -.checkpoint.-> CTRL
```

---

## 5. Controle de requisições externas

Mecanismos usados pelo Data Aggregator ao chamar as APIs oficiais.

```mermaid
flowchart TD
    A["Precisa de dados da API externa"] --> B{"Resposta<br/>em cache?"}
    B -- Sim --> C["Usa cache HTTP"]
    B -- Não --> D["Rate limiting<br/>(limita requisições)"]
    D --> E["Requisição HTTP"]
    E --> F{"Sucesso?"}
    F -- Sim --> G["Armazena no cache"]
    G --> H["Segue para Transform"]
    C --> H
    F -- Não --> I{"Tentativas<br/>restantes?"}
    I -- Sim --> J["Retry com backoff"]
    J --> D
    I -- Não --> K["Registra erro<br/>na tabela de erros"]
```

> O processamento paralelo (quando aplicável) executa vários desses fluxos simultaneamente.

---

## 6. Fluxo de uma requisição (Frontend → Banco)

Exemplo com `GET /parlamentares`.

```mermaid
sequenceDiagram
    actor U as Usuário
    participant FE as Frontend (Next.js)
    participant BE as Backend (Express)
    participant PR as Prisma
    participant DB as MySQL

    U->>FE: Acessa a página
    FE->>BE: GET /parlamentares
    BE->>PR: consulta
    PR->>DB: SQL
    DB-->>PR: registros
    PR-->>BE: objetos tipados
    BE-->>FE: 200 · JSON
    FE-->>U: Renderiza a lista
```

---

## 7. Fluxo de navegação do usuário

Fluxo de telas e chamadas à API, derivado dos endpoints disponíveis.

> A documentação do Frontend não descreve as páginas. O diagrama abaixo é inferido dos recursos da API; ajuste os nomes das telas conforme o `app/` real.

```mermaid
flowchart TD
    U(["Usuário"]) --> L["Lista de parlamentares"]
    L -. "GET /parlamentares" .-> API[("API")]
    L --> D["Detalhe do parlamentar"]
    D -. "GET /parlamentares/{id}" .-> API
    D --> P["Perfil"]
    D --> PR["Proposições"]
    D --> V["Votações"]
    P -. "GET /parlamentares/{id}/perfil" .-> API
    PR -. "GET /parlamentares/{id}/proposicoes" .-> API
    V -. "GET /parlamentares/{id}/votacoes" .-> API
```

---

## 8. Endpoints da API

```mermaid
flowchart LR
    R["/parlamentares"] --> R1["GET /parlamentares<br/>lista"]
    R --> R2["GET /parlamentares/{id}<br/>consulta por ID"]
    R2 --> R3["GET /parlamentares/{id}/perfil"]
    R2 --> R4["GET /parlamentares/{id}/proposicoes"]
    R2 --> R5["GET /parlamentares/{id}/votacoes"]
```

---

## 9. Camadas do Backend

```mermaid
flowchart LR
    REQ["Requisição HTTP"] --> RT["routes/"]
    RT --> CT["controllers/"]
    CT --> SV["services/"]
    SV --> PR["Prisma Client"]
    PR --> DB[("MySQL")]
    DB --> PR --> SV --> CT --> RES["Resposta JSON"]

    SW["swagger.yaml"] -.documenta.-> RT
    SW --> UI["/api-docs"]
```

> A estrutura exata de pastas pode evoluir (a documentação indica `controllers/`, `routes/`, `services/` e `...`).

---

## 10. Estrutura do Frontend

```mermaid
flowchart TD
    subgraph Next["Next.js · App Router"]
        APP["app/<br/>páginas e rotas"]
        CMP["components/<br/>componentes reutilizáveis"]
        SRV["services/<br/>comunicação com a API"]
        TYP["types/<br/>tipos TypeScript"]
    end

    ENV[".env.local<br/>NEXT_PUBLIC_API_URL"]
    BE["Backend API"]

    APP --> CMP
    APP --> SRV
    CMP --> TYP
    SRV --> TYP
    ENV -.configura.-> SRV
    SRV -->|HTTP| BE
```

---

## 11. Dados armazenados

Visão agrupada por tema, **sem** relacionamentos exatos (a documentação não os detalha; consulte o `schema.prisma` do Backend).

```mermaid
mindmap
  root((MySQL<br/>votovivo))
    Pessoas e organização
      Parlamentares
      Partidos
      Mandatos
      Órgãos
      Blocos parlamentares
    Atividade legislativa
      Proposições
      Tramitações
      Votações
      Emendas
    Atividade parlamentar
      Despesas
      Eventos
      Presenças
    Controle do ETL
      Execuções
      Checkpoints
      Erros
```

---

## 12. Ambiente de desenvolvimento

Portas e variáveis conforme os exemplos da documentação.

```mermaid
flowchart LR
    subgraph Docker["Docker (opcional)"]
        MY[("mysql_db<br/>:3306")]
    end

    subgraph BE["Backend · npm run dev"]
        API["API Express<br/>PORT=3000"]
        SW["/api-docs"]
    end

    subgraph FE["Frontend · npm run dev"]
        WEB["Next.js"]
    end

    subgraph DA["Data Aggregator · venv Python"]
        PY["popular/principal.py<br/>popular/atualizacao_semanal.py"]
    end

    API -->|"DATABASE_URL"| MY
    PY -->|"DB_HOST · DB_USER · DB_PASSWORD · DB_NAME"| MY
    WEB -->|"NEXT_PUBLIC_API_URL"| API
    API --- SW
```

> ⚠️ **Inconsistência na documentação:** o Backend usa `PORT=3000` e o Swagger em `localhost:3000`, mas `NEXT_PUBLIC_API_URL` aponta para `localhost:3001`. Os dois valores precisam coincidir no ambiente real.

### Ordem de subida local

```mermaid
flowchart LR
    A["1. docker-compose up -d mysql_db"] --> B["2. npx prisma generate"]
    B --> C["3. npx prisma migrate dev"]
    C --> D["4. Data Aggregator<br/>popular dados"]
    C --> E["5. Backend<br/>npm run dev"]
    D --> F["6. Frontend<br/>npm run dev"]
    E --> F
```

---

## 13. Fluxo de desenvolvimento

```mermaid
flowchart LR
    A["Alterar o código"] --> B["Executar testes"]
    B --> C["Verificar funcionamento local"]
    C --> D["Revisar alterações"]
    D --> E["Commit"]
    E --> F["Pull Request"]

    B -. "falhou" .-> A
```

---

## 14. Infraestrutura atual

Estado atual da hospedagem: o Cloudflare atua como proxy reverso e cache na frente de todo o tráfego, tanto do Frontend quanto da API. O Frontend roda em PaaS (Vercel) integrado ao repositório. Backend, banco de dados e Data Aggregator ficam na mesma instância gratuita da Oracle Cloud, com Backend e MySQL em containers.

```mermaid
flowchart LR
    U(["Usuário<br/>navegador"])
    GH["Repositório Git<br/>Frontend"]
    FO["Fontes Oficiais<br/>Câmara · Senado · Portal da Transparência"]

    CF["Cloudflare<br/>proxy reverso · cache"]

    subgraph Vercel["Vercel (PaaS)"]
        FE["Frontend<br/>Next.js (SSR)"]
    end

    subgraph Oracle["Oracle Cloud · instância gratuita"]
        subgraph Containers["Containers"]
            BE["Backend<br/>Node.js · Express"]
            DB[("MySQL")]
        end
        DA["Data Aggregator<br/>Python · ETL"]
    end

    GH -->|"deploy automático<br/>a cada push"| FE
    U -->|"acessa o site"| CF
    CF -->|"cache MISS<br/>(páginas)"| FE
    U -.->|"chamadas à API<br/>(navegador)"| CF
    FE -.->|"chamadas à API<br/>(SSR)"| CF
    CF -->|"cache MISS<br/>(API)"| BE
    BE -->|"Prisma"| DB
    FO -->|"APIs HTTP"| DA
    DA -->|"escrita"| DB
```

---

[⬅ Voltar para o README](./README.md)
