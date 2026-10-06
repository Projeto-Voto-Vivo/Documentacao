# Desenvolvimento e Execução

## 📌 Visão geral

O Voto Vivo é dividido em diferentes repositórios, permitindo que cada componente seja desenvolvido e executado de forma independente.

Os principais componentes são:

* Frontend;
* Backend;
* Data Aggregator.

O banco de dados MySQL é utilizado pelo Backend e pelo Data Aggregator.

---

## 📋 Pré-requisitos

Antes de iniciar o desenvolvimento, recomenda-se possuir:

* Git;
* Node.js;
* npm;
* Python 3.10 ou superior;
* MySQL 8 ou Docker;
* acesso às APIs externas utilizadas pelo projeto.

As versões exatas podem variar de acordo com as configurações atuais de cada repositório.

---

# 💻 Backend

## Instalação

Clone ou acesse o repositório do Backend e instale as dependências:

```bash
npm install
```

Configure as variáveis de ambiente necessárias.

Exemplo:

```env
DATABASE_URL="mysql://root@localhost:3306/votovivo"
PORT=3000
```

---

## Banco de dados

Caso o MySQL seja executado por Docker:

```bash
docker-compose up -d mysql_db
```

Depois, gere o cliente Prisma:

```bash
npx prisma generate
```

Execute as migrações:

```bash
npx prisma migrate dev
```

---

## Execução

Inicie o servidor:

```bash
npm run dev
```

A API estará disponível de acordo com a porta configurada no projeto.

A documentação Swagger pode ser acessada em:

```text
http://localhost:3000/api-docs
```

---

## Testes

Execute os testes com:

```bash
npm test
```

---

# 🖥️ Frontend

## Instalação

Instale as dependências:

```bash
npm install
```

Configure a URL da API no arquivo `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

O endereço deve ser ajustado de acordo com a porta utilizada pelo Backend.

---

## Execução

Inicie o servidor de desenvolvimento:

```bash
npm run dev
```

Para gerar a versão de produção:

```bash
npm run build
```

Depois:

```bash
npm start
```

---

# 🐍 Data Aggregator

## Ambiente virtual

Crie um ambiente virtual:

```bash
python -m venv .venv
```

Ative o ambiente.

### Linux/macOS

```bash
source .venv/bin/activate
```

### Windows

```bash
.venv\Scripts\activate
```

---

## Dependências

Instale as dependências:

```bash
pip install -r requirements.txt
```

Quando necessário:

```bash
pip install -e .
```

---

## Configuração

Configure o arquivo `.env`.

Exemplo:

```env
DB_HOST=localhost
DB_USER=...
DB_PASSWORD=...
DB_NAME=votovivo
PORTAL_TRANSPARENCIA_API_KEY=...
```

As credenciais e chaves utilizadas devem ser mantidas fora do controle de versão.

---

## Execução

Para executar o processo principal:

```bash
python popular/principal.py
```

Para executar a atualização periódica:

```bash
python popular/atualizacao_semanal.py
```

---

# 🔐 Boas práticas

Durante o desenvolvimento:

* não versionar arquivos `.env`;
* não expor senhas ou chaves de API;
* manter as dependências atualizadas;
* executar os testes antes de enviar alterações;
* manter as alterações relacionadas a cada componente organizadas;
* documentar alterações relevantes.

---

# 🔄 Fluxo recomendado

Uma alteração no sistema normalmente deve seguir o fluxo:

```text
1. Alterar o código
       │
       ▼
2. Executar testes
       │
       ▼
3. Verificar funcionamento local
       │
       ▼
4. Revisar alterações
       │
       ▼
5. Commit
       │
       ▼
6. Pull Request
```

---

## 📚 Documentação relacionada

* [README](./README.md)
* [Arquitetura do Sistema](./arquitetura.md)
* [Frontend](./frontend.md)
* [Backend](./backend.md)
* [Data Aggregator](./data-aggregator.md)
* [Banco de Dados](./banco-de-dados.md)
* [API](./api.md)

[⬅ Voltar para o README](./README.md)
