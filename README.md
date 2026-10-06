# Voto Vivo — Documentação

## 📌 Sobre o projeto

O **Voto Vivo** é uma plataforma desenvolvida para facilitar o acesso e a consulta de informações relacionadas à atuação de parlamentares brasileiros.

O projeto integra dados públicos provenientes de diferentes fontes oficiais, organiza essas informações em uma base de dados estruturada e disponibiliza os dados por meio de uma API para consumo pela aplicação web.

A solução é composta por três principais partes:

* **Frontend:** interface web utilizada pelos usuários;
* **Backend:** API responsável pelo acesso e disponibilização dos dados;
* **Data Aggregator:** responsável pela coleta, tratamento e atualização dos dados provenientes das fontes oficiais.

---

## 🎯 Objetivo

O objetivo do Voto Vivo é **centralizar e facilitar o acesso a informações públicas sobre parlamentares**, permitindo que os dados sejam consultados de forma mais organizada e acessível.

Para isso, o sistema realiza a integração de informações provenientes de fontes oficiais, como:

* Câmara dos Deputados;
* Senado Federal;
* Portal da Transparência.

Os dados coletados são processados e armazenados em um banco de dados, sendo posteriormente disponibilizados pela API para a aplicação frontend.

---

## 🏗️ Arquitetura

De forma simplificada, o fluxo do sistema pode ser representado da seguinte maneira:

```text
Fontes Oficiais
      │
      ▼
Data Aggregator
      │
      ▼
Banco de Dados
      │
      ▼
Backend / API
      │
      ▼
Frontend
      │
      ▼
     Usuário
```

O **Data Aggregator** é responsável pela entrada e atualização dos dados no sistema.

O **Backend** fornece uma API para consulta dessas informações.

O **Frontend** utiliza essa API para apresentar os dados aos usuários por meio da interface web.

Para uma descrição detalhada da arquitetura, consulte:

➡️ [Arquitetura do Sistema](./arquitetura.md)

---

## 📂 Repositórios

O projeto é dividido em diferentes repositórios, de acordo com a responsabilidade de cada parte do sistema.

* **Frontend:** aplicação web responsável pela interface com o usuário.
* **Backend:** API responsável pela lógica de acesso aos dados.
* **Data Aggregator:** aplicação responsável pela coleta e processamento dos dados.
* **Documentação:** documentação técnica e informações sobre o funcionamento do projeto.

---

## 📚 Documentação

A documentação foi organizada por assunto para facilitar a consulta.

### 🏗️ Arquitetura

* [Arquitetura do Sistema](./arquitetura.md)

Visão geral dos componentes do Voto Vivo e da comunicação entre eles.

### 💻 Aplicação

* [Frontend](./frontend.md)
* [Backend](./backend.md)

Documentação das aplicações responsáveis pela interface web e pela API.

### 🔌 Integração

* [API](./api.md)

Endpoints, parâmetros e informações sobre a comunicação entre o frontend e o backend.

### 🗄️ Dados

* [Data Aggregator](./data-aggregator.md)
* [Banco de Dados](./banco-de-dados.md)

Descrição do processo de coleta, tratamento e armazenamento dos dados.

### 🛠️ Desenvolvimento

* [Desenvolvimento e Execução](./desenvolvimento.md)

Instruções para configurar o ambiente, executar os componentes do projeto e realizar testes.

---

## 🧰 Tecnologias

As principais tecnologias utilizadas no projeto incluem:

### Frontend

* Next.js
* React
* TypeScript
* Tailwind CSS

### Backend

* Node.js
* TypeScript
* Express
* Prisma
* MySQL
* Swagger / OpenAPI
* Jest

### Data Aggregator

* Python
* MySQL
* APIs HTTP
* Processos de ETL

---

## 🚀 Desenvolvimento

Cada componente do sistema possui seu próprio ambiente e pode ser executado de forma independente.

Para começar a desenvolver no projeto, consulte:

➡️ [Desenvolvimento e Execução](./desenvolvimento.md)

---

## 📖 Organização da documentação

A documentação segue uma abordagem modular. O `README.md` apresenta uma visão geral do projeto, enquanto os demais arquivos aprofundam cada parte específica do sistema.

```text
Documentacao/
│
├── README.md
├── arquitetura.md
├── frontend.md
├── backend.md
├── data-aggregator.md
├── banco-de-dados.md
├── api.md
└── desenvolvimento.md
```

Essa organização permite que novas informações sejam adicionadas a cada área sem tornar o README excessivamente extenso.
