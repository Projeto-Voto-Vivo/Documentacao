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

## 🏗️ Estrutura

A aplicação utiliza a estrutura do Next.js baseada no **App Router**.

De forma geral, os principais diretórios são:

```text
app/
├── ...
    
components/
├── ...

services/
├── ...

types/
├── ...
```

### `app/`

Contém as páginas e rotas da aplicação.

### `components/`

Contém componentes reutilizáveis da interface.

### `services/`

Centraliza funcionalidades relacionadas à comunicação com serviços externos, principalmente a API do Backend.

### `types/`

Contém definições de tipos utilizadas pela aplicação TypeScript.

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
