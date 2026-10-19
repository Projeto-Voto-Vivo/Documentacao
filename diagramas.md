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
    FO["Fontes Oficiais<br/>Câmara · Senado · Portal da Transparência · TSE"]
    DA["Data Aggregator<br/>Python · ETL"]
    DB[("MySQL<br/>dados estruturados")]
    BE["Backend<br/>Node.js · Express · Prisma"]
    CF["Cloudflare<br/>proxy · cache"]
    FE["Frontend<br/>Next.js · React"]
    U(["Usuário"])

    FO -->|APIs HTTP · arquivos CSV/ZIP| DA
    DA -->|escrita| DB
    DA -.->|"purga o cache<br/>ao fim da carga"| CF
    DB -->|leitura via Prisma| BE
    BE -->|API REST / JSON| CF
    CF --> FE
    FE --> U
```

---

## 2. Integração entre componentes

Mostra quem se comunica com quem e por qual protocolo. O Frontend nunca acessa o banco diretamente; o banco é compartilhado entre Data Aggregator (escrita) e Backend (leitura). O Data Aggregator também chama a API da Cloudflare para purgar o cache depois de cada carga.

```mermaid
flowchart LR
    subgraph Externo["Externo"]
        CD["API e dumps anuais<br/>Câmara dos Deputados"]
        SF["API Senado Federal"]
        PT["API Portal da Transparência<br/>(requer chave de API)"]
        TSE["TSE<br/>candidaturas e resultados<br/>(arquivos ZIP)"]
    end

    subgraph Ingestao["Repositório: Data Aggregator"]
        PR["principal.py<br/>carga completa · retomável"]
        AS["atualizacao_semanal.py<br/>cron · domingo 03:00"]
        TS["candidaturaTse.py<br/>resultadoEleicao.py<br/>sob demanda"]
        CDN["utils/cache_cdn.py<br/>purga do CDN"]
    end

    subgraph Dados["Persistência"]
        MY[("MySQL 8<br/>votovivo")]
    end

    subgraph Servico["Repositório: Backend"]
        API["API REST<br/>Express"]
        PRI["Prisma ORM"]
        SW["Swagger UI<br/>/api-docs"]
    end

    CF["Cloudflare<br/>cache · proxy"]

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
    TSE -->|HTTP| TS

    PR -->|"leitura e escrita"| MY
    AS -->|"leitura e escrita"| MY
    TS -->|"leitura e escrita"| MY
    PR --> CDN
    AS --> CDN
    CDN -.->|"purge_cache"| CF

    API --> PRI
    PRI -->|SQL| MY
    API --- SW

    CF -->|"cache MISS"| API
    WEB -->|"HTTP · NEXT_PUBLIC_API_URL<br/>BACKEND_INTERNAL_URL (SSR)"| CF
    U -->|navegador| WEB
```

---

## 3. Fontes de dados

Origem dos dados e entidades armazenadas no banco (tabelas do `schema.sql` do Data Aggregator).

```mermaid
flowchart LR
    CD["Câmara dos Deputados"]
    SF["Senado Federal"]
    PT["Portal da Transparência"]
    TSE["TSE"]

    DA["Data Aggregator"]

    subgraph DB["MySQL"]
        direction TB
        subgraph Negocio["Dados de negócio"]
            E1["Parlamentares · redes sociais"]
            E2["Partidos · filiações"]
            E3["Mandatos"]
            E4["Órgãos · membros"]
            E5["Blocos parlamentares"]
            E6["Proposições · autoria · relações"]
            E7["Tramitações"]
            E8["Votações · orientações · votos"]
            E9["Despesas"]
            E10["Emendas · documentos"]
            E11["Eventos"]
            E12["Presenças"]
            E13["Temas"]
        end
        subgraph Eleicao["Eleições 2026"]
            EL1["Candidaturas"]
            EL2["Resultado da eleição"]
            EL3["Resultado por candidato"]
            EL4["Resultado por partido"]
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
    TSE --> DA
    DA --> Negocio
    DA --> Eleicao
    DA <--> Controle
```

| Fonte | Alimenta |
|-------|----------|
| Câmara dos Deputados | Deputados, blocos, órgãos, proposições (dumps anuais), tramitações, eventos, despesas CEAP (CSV anual), votações, orientações de bancada e votos |
| Senado Federal | Senadores, mandatos, órgãos, proposições, tramitações, votações/presenças e despesas CEAPS |
| Portal da Transparência | Emendas parlamentares |
| TSE | Candidaturas (com foto e situação) e resultados da totalização |

---

## 4. Pipeline de ETL

```mermaid
flowchart LR
    subgraph Extract["Extract"]
        X1["Consulta APIs oficiais<br/>e arquivos anuais"]
    end

    subgraph Transform["Transform"]
        T1["Transformação"]
        T2["Padronização"]
        T3["Relacionamento entre entidades"]
        T1 --> T2 --> T3
    end

    subgraph Load["Load"]
        L1["Insere novos registros"]
        L2["Atualiza registros existentes<br/>(upsert idempotente)"]
    end

    DB[("MySQL")]
    CTRL[("etlExecucao · etlCheckpoint · etlErro")]
    CDN["Purga do cache<br/>Cloudflare"]

    X1 --> T1
    T3 --> L1
    T3 --> L2
    L1 --> DB
    L2 --> DB
    L2 --> CDN

    X1 -.registra.-> CTRL
    T3 -.registra erros.-> CTRL
    L2 -.checkpoint.-> CTRL
```

### Orquestração

Os scripts rodam em ordem, respeitando as dependências entre tabelas. A carga completa (`principal.py`) para na primeira falha e retoma do último script concluído; a atualização semanal (`atualizacao_semanal.py`) segue mesmo após falhas, mas sai com código 1. Nos dois casos o cache do CDN é purgado ao final.

```mermaid
flowchart TD
    subgraph Completa["principal.py · carga completa (26 scripts)"]
        direction LR
        A1["Parlamentares · partidos · blocos · mandatos"] --> A2["Tipos · temas · órgãos"]
        A2 --> A3["Proposições Câmara e Senado<br/>+ vínculo entre casas"]
        A3 --> A4["Despesas · emendas"]
        A4 --> A5["Tipos de tramitação · tramitações"]
        A5 --> A6["Eventos · votações · orientações · votos"]
        A6 --> A7["Vínculo emenda ↔ parlamentar"]
    end

    subgraph Semanal["atualizacao_semanal.py · domingo 03:00 (11 scripts)"]
        direction LR
        B1["Proposições + vínculo entre casas"] --> B2["Despesas"]
        B2 --> B3["Emendas"]
        B3 --> B4["Votações · orientações · votos"]
        B4 --> B5["Vínculo emenda ↔ parlamentar"]
    end

    subgraph Demanda["Sob demanda"]
        direction LR
        C1["candidaturaTse.py"] --> C2["resultadoEleicao.py"]
    end

    Completa --> P["purgar_cdn()"]
    Semanal --> P
```

---

## 5. Controle de requisições externas

Mecanismos usados pelo Data Aggregator ao chamar as APIs oficiais (`utils/http_client.py`).

```mermaid
flowchart TD
    A["Precisa de dados da API externa"] --> B{"ETL_HTTP_CACHE=ler<br/>e resposta em disco?"}
    B -- Sim --> C["Usa staging/http_cache"]
    B -- Não --> E["Requisição HTTP"]
    E --> F{"Status"}
    F -- "200" --> G["Grava em disco<br/>(ETL_HTTP_CACHE=gravar)"]
    G --> H["Segue para Transform"]
    C --> H
    F -- "500/502/503/504" --> J["Retry com backoff<br/>(urllib3.Retry)"]
    J --> E
    F -- "429" --> K{"Até 3 tentativas<br/>com pausa de 60s"}
    K -- "ok" --> G
    K -- "persiste" --> L["Pausas de 5, 10 e 20 min"]
    L -- "ok" --> G
    L -- "persiste" --> M["RateLimitAbort<br/>interrompe o script"]
    M -.->|"checkpoint permite retomar"| A
    F -- "erro em um item" --> N["Registra na fila etlErro<br/>reprocessada na próxima execução"]
```

### Fetch paralelo

Os scripts de maior volume buscam as respostas em paralelo e gravam em ordem, para que o checkpoint continue válido.

```mermaid
flowchart LR
    Q["Fila de itens"] --> L["Lote<br/>ETL_TAMANHO_LOTE"]
    L --> W1["Worker 1"]
    L --> W2["Worker 2"]
    L --> W3["Worker N<br/>ETL_WORKERS"]
    W1 --> O["Gravação sequencial<br/>na ordem original"]
    W2 --> O
    W3 --> O
    O --> CK["Checkpoint"]
```

---

## 6. Fluxo de uma requisição (Frontend → Banco)

Exemplo com `GET /parlamentares`, passando pelo cache da Cloudflare.

```mermaid
sequenceDiagram
    actor U as Usuário
    participant FE as Frontend (Next.js)
    participant CF as Cloudflare
    participant BE as Backend (Express)
    participant PR as Prisma
    participant DB as MySQL

    U->>FE: Acessa a página
    FE->>CF: GET /parlamentares
    alt cache HIT
        CF-->>FE: 200 · JSON (do cache)
    else cache MISS
        CF->>BE: GET /parlamentares
        BE->>PR: consulta
        PR->>DB: SQL
        DB-->>PR: registros
        PR-->>BE: objetos tipados
        BE-->>CF: 200 · JSON · s-maxage=86400
        CF-->>FE: 200 · JSON
    end
    FE-->>U: Renderiza a lista
```

---

## 7. Fluxo de navegação do usuário

Rotas do `app/` do Frontend e os endpoints consumidos por cada uma.

```mermaid
flowchart TD
    U(["Usuário"]) --> H["Home /"]
    H -. "/stats/emendas/total<br/>/parlamentares/ranking" .-> API[("API")]

    H --> L["/parlamentares<br/>lista de parlamentares"]
    L -. "/parlamentares" .-> API
    L --> D["/parlamentares/id<br/>perfil"]
    D -. "/parlamentares/id<br/>/presenca · /alinhamento<br/>/comissoes · /filiacoes · /mandatos" .-> API
    D --> DP["Painéis do perfil"]
    DP -. "/proposicoes · /votacoes<br/>/despesas · /emendas" .-> API
    DP --> EM["/parlamentares/id/emendas/idEmenda<br/>detalhe da emenda"]
    EM -. "/emendas/id/detalhes" .-> API

    H --> PL["/proposicoes<br/>busca de proposições"]
    PL -. "/proposicoes · /proposicoes/filtros" .-> API
    PL --> PD["/proposicoes/id<br/>tramitação · votações · documentos"]
    PD -. "/proposicoes/id/tramitacoes<br/>/proposicoes/id/documentos<br/>/votacoes/id" .-> API

    H --> CA["/candidatos<br/>candidatos e resultados da eleição"]
    CA -. "/candidatos · /eleicoes/resultados<br/>/eleicoes/bancadas" .-> API
    CA --> CD["/candidatos/idCandidaturaTse"]
    CD -. "/candidatos/id" .-> API

    H --> ED["/educacao"]
    H --> PJ["/projeto"]
```

---

## 8. Endpoints da API

Endpoints consumidos pelo Frontend (`services/`).

```mermaid
flowchart LR
    subgraph Parlamentares["/parlamentares"]
        P0["GET /parlamentares"]
        P1["GET /parlamentares/ranking<br/>/ranking/filtros"]
        P2["GET /parlamentares/id"]
        P2 --> P3["/presenca"]
        P2 --> P4["/alinhamento<br/>/alinhamento/temas"]
        P2 --> P5["/comissoes · /filiacoes · /mandatos"]
        P2 --> P6["/proposicoes · /votacoes · /temas"]
        P2 --> P7["/despesas · /despesas/resumo"]
        P2 --> P8["/emendas · /emendas/resumo"]
    end

    subgraph Proposicoes["/proposicoes"]
        R0["GET /proposicoes<br/>GET /proposicoes/filtros"]
        R1["GET /proposicoes/id"]
        R1 --> R2["/tramitacoes"]
        R1 --> R3["/documentos"]
    end

    subgraph Outros["Demais recursos"]
        O1["GET /votacoes/id"]
        O2["GET /emendas/id/detalhes"]
        O3["GET /stats/emendas/total<br/>/stats/emendas/ranking<br/>/stats/despesas/ranking"]
    end

    subgraph Eleicoes["Eleições"]
        E1["GET /candidatos<br/>GET /candidatos/id"]
        E2["GET /eleicoes/resultados<br/>/eleicoes/resultados/id<br/>/eleicoes/resultados/id/candidatos"]
        E3["GET /eleicoes/bancadas"]
    end
```

> Lista derivada das chamadas do Frontend. O Backend pode expor rotas adicionais; consulte o `swagger.yaml`.

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
        APP["app/<br/>/ · parlamentares · proposicoes<br/>candidatos · educacao · projeto"]
        CMP["components/<br/>home · layout · parlamentares · proposicoes<br/>candidatos · eleicoes · educacao · projeto"]
        SRV["services/<br/>api.ts · parlamentares · proposicoes<br/>candidatos · eleicoes"]
        TYP["types/<br/>tipos TypeScript"]
        RW["next.config.ts<br/>rewrite /api/proxy/*"]
    end

    ENV["Variáveis de ambiente<br/>NEXT_PUBLIC_API_URL<br/>BACKEND_INTERNAL_URL"]
    BE["Backend API"]

    APP --> CMP
    APP --> SRV
    CMP --> TYP
    SRV --> TYP
    ENV -.configura.-> SRV
    ENV -.configura.-> RW
    SRV -->|"SSR: BACKEND_INTERNAL_URL"| BE
    SRV -->|"navegador: NEXT_PUBLIC_API_URL<br/>ou /api/proxy"| BE
    RW -->|"proxy server-side"| BE
```

### Escolha da URL da API (`services/api.ts`)

```mermaid
flowchart TD
    A["Chamada do axios"] --> B{"Roda no servidor?<br/>(typeof window undefined)"}
    B -- Sim --> C["BACKEND_INTERNAL_URL<br/>padrão http://localhost:3001"]
    B -- Não --> D{"NEXT_PUBLIC_API_URL<br/>definida?"}
    D -- Sim --> E["NEXT_PUBLIC_API_URL"]
    D -- Não --> F["/api/proxy<br/>rewrite do Next.js para o Backend"]
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
    Eleições
      Candidaturas
      Resultado da eleição
      Resultado por candidato
      Resultado por partido
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
        API["API Express<br/>PORT (padrão no código: 3001)"]
        SW["/api-docs"]
    end

    subgraph FE["Frontend · npm run dev"]
        WEB["Next.js"]
    end

    subgraph DA["Data Aggregator · venv Python"]
        PY["popular/principal.py<br/>popular/atualizacao_semanal.py<br/>popular/candidaturaTse.py"]
    end

    API -->|"DATABASE_URL"| MY
    PY -->|"DB_HOST · DB_USER · DB_PASSWORD · DB_NAME"| MY
    WEB -->|"NEXT_PUBLIC_API_URL · BACKEND_INTERNAL_URL"| API
    API --- SW
```

> ⚠️ **Inconsistência na documentação:** os arquivos de documentação do Backend citam `PORT=3000` e o Swagger em `localhost:3000`, mas o código usa `3001` como padrão (`src/server.ts`) e o Frontend aponta para `localhost:3001`. Os valores precisam coincidir no ambiente real.

### Ordem de subida local

```mermaid
flowchart LR
    A["1. docker-compose up -d mysql_db"] --> B["2. npx prisma generate"]
    B --> C["3. npx prisma migrate dev"]
    C --> D["4. Data Aggregator<br/>popular/principal.py"]
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

Estado atual da hospedagem: o Cloudflare atua como proxy reverso e cache na frente de todo o tráfego, tanto do Frontend quanto da API. O Frontend roda em PaaS (Vercel) integrado ao repositório. Backend e MySQL rodam em containers (`docker-compose.oracle.yml`) na instância gratuita da Oracle Cloud, e o Data Aggregator roda na mesma instância, fora dos containers, agendado por cron.

```mermaid
flowchart LR
    U(["Usuário<br/>navegador"])
    GH["Repositório Git<br/>Frontend"]
    FO["Fontes Oficiais<br/>Câmara · Senado · Portal da Transparência · TSE"]

    CF["Cloudflare<br/>proxy reverso · cache<br/>s-maxage=86400"]

    subgraph Vercel["Vercel (PaaS)"]
        FE["Frontend<br/>Next.js (SSR)<br/>rewrite /api/proxy"]
    end

    subgraph Oracle["Oracle Cloud · instância gratuita"]
        subgraph Containers["Containers (docker compose)"]
            BE["votovivo_api<br/>Node.js · Express<br/>porta 80 → 3001"]
            DB[("votovivo_db<br/>MySQL 8<br/>127.0.0.1:3306")]
        end
        subgraph Host["Host (venv Python + cron)"]
            DA["Data Aggregator<br/>domingo 03:00"]
        end
    end

    GH -->|"deploy automático<br/>a cada push"| FE
    U -->|"acessa o site"| CF
    CF -->|"cache MISS<br/>(páginas)"| FE
    U -.->|"chamadas à API<br/>(navegador)"| CF
    FE -.->|"chamadas à API<br/>(SSR)"| CF
    CF -->|"cache MISS<br/>(API)"| BE
    BE -->|"Prisma"| DB
    FO -->|"APIs HTTP · arquivos"| DA
    DA -->|"escrita"| DB
    DA -.->|"purge_cache<br/>ao fim da carga"| CF
```

### Cache e invalidação

A API responde com `Cache-Control: s-maxage=86400`, então a Cloudflare guarda cada resposta por 24 horas e o servidor só é consultado uma vez por URL por dia. Como esse TTL é longo, o Data Aggregator purga o cache inteiro da zona ao fim de cada carga; sem a purga, dados corrigidos ficariam invisíveis por até um dia.

```mermaid
sequenceDiagram
    actor U as Usuário
    participant CF as Cloudflare
    participant BE as Backend (container)
    participant DB as MySQL (container)
    participant DA as Data Aggregator (cron)

    U->>CF: GET /parlamentares/123
    alt cache HIT
        CF-->>U: 200 · JSON (do cache)
    else cache MISS
        CF->>BE: GET /parlamentares/123
        BE->>DB: SQL via Prisma
        DB-->>BE: registros
        BE-->>CF: 200 · Cache-Control s-maxage=86400
        CF-->>U: 200 · JSON
    end

    Note over DA: domingo 03:00
    DA->>DB: upsert dos dados novos
    DA->>CF: POST /zones/id/purge_cache (purge_everything)
    CF-->>DA: success
```

> O Data Aggregator só purga o cache se `CLOUDFLARE_ZONE_ID` e `CLOUDFLARE_API_TOKEN` estiverem configurados; uma falha na purga não reprova a carga.

---

[⬅ Voltar para o README](./README.md)
