# API

## 📌 Visão geral

A API do Voto Vivo é responsável por disponibilizar os dados armazenados no sistema para as aplicações que os consomem, principalmente o Frontend.

A API segue o padrão **REST** e utiliza requisições HTTP para comunicação.

```text
Frontend
   │
   │ HTTP
   ▼
API REST
   │
   ▼
MySQL
```

---

## 🔌 Principais recursos

A API disponibiliza endpoints relacionados principalmente aos parlamentares e suas atividades.

Os principais recursos incluem:

* Parlamentares;
* Perfil parlamentar;
* Proposições;
* Votações.

---

## 👤 Parlamentares

### Listagem

```http
GET /parlamentares
```

Retorna uma lista de parlamentares disponíveis para consulta.

---

### Consulta por ID

```http
GET /parlamentares/{id}
```

Retorna as informações de um parlamentar específico.

---

### Perfil

```http
GET /parlamentares/{id}/perfil
```

Retorna informações relacionadas ao perfil do parlamentar.

---

### Proposições

```http
GET /parlamentares/{id}/proposicoes
```

Retorna proposições relacionadas ao parlamentar.

---

### Votações

```http
GET /parlamentares/{id}/votacoes
```

Retorna informações relacionadas às votações do parlamentar.

---

## 📖 Swagger / OpenAPI

A especificação completa da API está documentada utilizando **Swagger/OpenAPI**.

O projeto possui um arquivo:

```text
swagger.yaml
```

que descreve os endpoints e suas respectivas estruturas.

Durante a execução local do Backend, a documentação pode ser acessada em:

```text
http://localhost:3000/api-docs
```

A porta pode variar de acordo com a configuração do ambiente.

---

## 🔄 Comunicação com o Frontend

O Frontend realiza requisições HTTP para a API.

Um exemplo simplificado:

```text
Usuário
   │
   ▼
Frontend
   │
   │ GET /parlamentares
   ▼
Backend
   │
   ▼
Prisma
   │
   ▼
MySQL
   │
   ▼
Backend
   │
   │ JSON
   ▼
Frontend
```

As respostas da API são utilizadas pelo Frontend para apresentar as informações ao usuário.

---

## ⚠️ Fonte da especificação

A documentação deste arquivo apresenta os principais endpoints para facilitar o entendimento do projeto.

Para informações detalhadas sobre:

* parâmetros;
* tipos de dados;
* códigos HTTP;
* estruturas das respostas;
* campos obrigatórios;
* schemas;

deve-se consultar o arquivo `swagger.yaml` e a documentação Swagger disponibilizada pelo Backend.

---

## 📚 Documentação relacionada

* [Arquitetura do Sistema](./arquitetura.md)
* [Frontend](./frontend.md)
* [Backend](./backend.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
