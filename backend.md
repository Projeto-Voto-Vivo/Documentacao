# Backend

## 📌 Visão geral

O Backend do Voto Vivo é responsável por disponibilizar os dados armazenados no banco de dados por meio de uma **API REST**.

A aplicação foi desenvolvida utilizando **Node.js, TypeScript e Express**, com **Prisma** para acesso ao banco de dados MySQL.

A documentação da API é disponibilizada utilizando **Swagger/OpenAPI**.

---

## 🧰 Tecnologias

As principais tecnologias utilizadas são:

* Node.js
* TypeScript
* Express
* Prisma ORM
* MySQL
* Swagger / OpenAPI
* Jest
* Supertest

---

## 🏗️ Estrutura

O Backend é organizado de forma a separar as responsabilidades da aplicação.

De forma geral, a estrutura contém elementos relacionados a:

```text
src/
├── controllers/
├── routes/
├── services/
├── ...
```

A estrutura exata pode evoluir conforme o projeto, mas a separação tem como objetivo manter a lógica da API organizada e facilitar a manutenção.

---

## 🔌 API REST

O Backend disponibiliza endpoints para consulta das informações do sistema.

Entre os principais recursos estão:

* Parlamentares;
* Perfil de parlamentares;
* Proposições;
* Votações.

Os endpoints detalhados estão disponíveis em:

➡️ [Documentação da API](./api.md)

---

## 🗄️ Banco de Dados

O acesso ao banco de dados é realizado utilizando o **Prisma ORM**.

A conexão é configurada por meio da variável de ambiente:

```env
DATABASE_URL="mysql://root@localhost:3306/votovivo"
```

O valor deve ser ajustado de acordo com o ambiente utilizado.

---

## 📖 Swagger / OpenAPI

A API possui documentação baseada em **OpenAPI**, permitindo visualizar os endpoints, parâmetros e respostas disponíveis.

Durante a execução local, a documentação pode ser acessada em:

```text
http://localhost:3000/api-docs
```

A porta pode variar de acordo com a configuração utilizada no ambiente.

O arquivo `swagger.yaml` presente no projeto serve como referência para a especificação da API.

---

## 🧪 Testes

O projeto utiliza:

* **Jest** para testes;
* **Supertest** para testes de endpoints HTTP.

Os testes podem ser executados utilizando:

```bash
npm test
```

---

## ▶️ Execução

Antes de iniciar o Backend, configure as variáveis de ambiente necessárias.

Em seguida, instale as dependências:

```bash
npm install
```

Gere o cliente Prisma:

```bash
npx prisma generate
```

Para ambientes que utilizam migrações:

```bash
npx prisma migrate dev
```

Depois, inicie o servidor de desenvolvimento:

```bash
npm run dev
```

---

## 🔐 Configuração

As informações de conexão e outras configurações específicas do ambiente devem ser mantidas em variáveis de ambiente.

Não devem ser adicionadas ao repositório informações sensíveis, como:

* senhas;
* chaves de API;
* tokens;
* credenciais de banco de dados.

---

## 📚 Documentação relacionada

* [Arquitetura do Sistema](./arquitetura.md)
* [API](./api.md)
* [Banco de Dados](./banco-de-dados.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
