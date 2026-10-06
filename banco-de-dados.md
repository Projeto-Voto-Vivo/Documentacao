# Banco de Dados

## 📌 Visão geral

O Voto Vivo utiliza **MySQL** como banco de dados principal.

O banco armazena as informações coletadas pelo Data Aggregator e fornece os dados utilizados pelo Backend.

O acesso ao banco pelo Backend é realizado utilizando o **Prisma ORM**.

```text
Data Aggregator ──────┐
                      │
                      ▼
                   MySQL
                      ▲
                      │
Backend ──────────────┘
```

---

## 🧰 Tecnologias

* MySQL 8
* Prisma ORM

O Prisma é utilizado pelo Backend para facilitar o acesso e a manipulação dos dados.

---

## 🗃️ Dados armazenados

A base de dados contém informações relacionadas a diferentes aspectos da atividade parlamentar.

Entre os principais grupos de informações estão:

* Parlamentares;
* Partidos;
* Mandatos;
* Órgãos;
* Blocos parlamentares;
* Proposições;
* Tramitações;
* Votações;
* Despesas;
* Emendas;
* Eventos;
* Presenças.

Além dos dados utilizados diretamente pela aplicação, existem estruturas destinadas ao controle do processo de ETL.

---

## 🔄 Origem dos dados

Os dados armazenados são obtidos principalmente a partir de fontes oficiais.

O Data Aggregator realiza a coleta e o tratamento antes de armazená-los no banco.

```text
Fontes Oficiais
      │
      ▼
Data Aggregator
      │
      ▼
    MySQL
      │
      ▼
   Backend
```

---

## 🛠️ Prisma

O Prisma é utilizado como ORM no Backend.

Ele permite que a aplicação trabalhe com os dados do MySQL por meio de modelos definidos no projeto.

Entre as principais funcionalidades utilizadas estão:

* definição dos modelos;
* geração do Prisma Client;
* consultas ao banco;
* criação e atualização de registros;
* gerenciamento de migrações.

Para gerar o cliente Prisma:

```bash
npx prisma generate
```

Para executar migrações em ambiente de desenvolvimento:

```bash
npx prisma migrate dev
```

---

## ⚙️ Controle do ETL

O banco também possui estruturas utilizadas pelo Data Aggregator para controlar a execução dos processos de coleta.

Essas estruturas permitem registrar informações relacionadas a:

* execuções;
* checkpoints;
* erros.

Isso facilita o acompanhamento do processo e a identificação de problemas durante a atualização dos dados.

---

## 🔐 Acesso ao banco

A conexão do Backend é configurada por meio da variável:

```env
DATABASE_URL="mysql://root@localhost:3306/votovivo"
```

As credenciais devem ser configuradas de acordo com o ambiente.

Informações sensíveis não devem ser versionadas no Git.

---

## 🐳 Ambiente de desenvolvimento

O projeto Backend possui configuração para execução do MySQL utilizando Docker.

Um exemplo de inicialização do banco é:

```bash
docker-compose up -d mysql_db
```

A configuração exata deve ser consultada no arquivo `docker-compose.yml` do projeto.

---

## 📚 Documentação relacionada

* [Arquitetura do Sistema](./arquitetura.md)
* [Backend](./backend.md)
* [Data Aggregator](./data-aggregator.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
