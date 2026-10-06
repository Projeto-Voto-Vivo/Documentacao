# Arquitetura do Sistema

## 📌 Visão geral

O Voto Vivo é organizado em componentes independentes que trabalham em conjunto para coletar, armazenar, processar e disponibilizar informações públicas sobre parlamentares.

A arquitetura é composta principalmente por:

* **Fontes de dados:** APIs e serviços oficiais;
* **Data Aggregator:** coleta e processamento dos dados;
* **Banco de Dados:** armazenamento centralizado das informações;
* **Backend:** API responsável pelo acesso aos dados;
* **Frontend:** interface utilizada pelos usuários.

O fluxo geral pode ser representado da seguinte forma:

```text
┌──────────────────────┐
│    Fontes Oficiais   │
│ Câmara / Senado /    │
│ Portal Transparência │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Data Aggregator   │
│   Coleta e ETL       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│     MySQL Database   │
│ Dados estruturados   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      Backend API     │
│ Node.js / Express    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       Frontend       │
│ Next.js / React      │
└──────────┬───────────┘
           │
           ▼
        Usuário
```

---

## 🔄 Fluxo de dados

O funcionamento do sistema pode ser dividido em quatro etapas principais.

### 1. Coleta

O Data Aggregator acessa as fontes oficiais e obtém os dados disponibilizados por elas.

Entre as principais fontes utilizadas estão:

* Câmara dos Deputados;
* Senado Federal;
* Portal da Transparência.

### 2. Processamento

Os dados coletados passam por processos de tratamento e organização antes de serem armazenados.

Essa etapa inclui atividades como:

* transformação dos dados;
* padronização de informações;
* relacionamento entre entidades;
* controle das execuções;
* tratamento de erros;
* atualização de registros existentes.

### 3. Armazenamento

Após o processamento, os dados são armazenados em um banco de dados MySQL.

O banco centraliza informações relacionadas a parlamentares, partidos, proposições, votações, despesas e outras entidades utilizadas pela aplicação.

### 4. Disponibilização

O Backend acessa o banco de dados e disponibiliza as informações por meio de uma API REST.

O Frontend consome essa API para apresentar os dados ao usuário.

---

## 🧩 Componentes

### Data Aggregator

Responsável pela integração com as fontes externas e pela atualização da base de dados.

Mais informações em:

➡️ [Data Aggregator](./data-aggregator.md)

### Banco de Dados

Responsável pelo armazenamento estruturado das informações utilizadas pelo sistema.

Mais informações em:

➡️ [Banco de Dados](./banco-de-dados.md)

### Backend

Responsável por disponibilizar os dados por meio de uma API REST.

Mais informações em:

➡️ [Backend](./backend.md)

### Frontend

Responsável pela interface web e pela interação com o usuário.

Mais informações em:

➡️ [Frontend](./frontend.md)

---

## 🔗 Comunicação entre os componentes

A comunicação entre os componentes ocorre principalmente por meio de dois fluxos:

### Data Aggregator → Banco de Dados

O Data Aggregator realiza operações de leitura e escrita diretamente no banco de dados para manter as informações atualizadas.

### Frontend → Backend → Banco de Dados

O Frontend realiza requisições HTTP para o Backend.

O Backend processa as requisições, consulta o banco de dados e retorna os dados ao Frontend.

```text
Frontend
   │
   │ HTTP
   ▼
Backend
   │
   │ Prisma
   ▼
MySQL
```

---

## 🛡️ Separação de responsabilidades

A divisão dos componentes permite que cada parte do sistema tenha uma responsabilidade bem definida:

| Componente      | Responsabilidade             |
| --------------- | ---------------------------- |
| Fontes oficiais | Fornecer os dados públicos   |
| Data Aggregator | Coletar e processar os dados |
| Banco de Dados  | Armazenar os dados           |
| Backend         | Disponibilizar os dados      |
| Frontend        | Apresentar os dados          |

Essa separação facilita a manutenção, evolução e desenvolvimento independente dos componentes.

---

## 📚 Documentação relacionada

* [Frontend](./frontend.md)
* [Backend](./backend.md)
* [Data Aggregator](./data-aggregator.md)
* [Banco de Dados](./banco-de-dados.md)
* [API](./api.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
