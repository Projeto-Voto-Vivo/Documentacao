# Data Aggregator

## 📌 Visão geral

O **Data Aggregator** é o componente responsável pela coleta, processamento e atualização dos dados utilizados pelo Voto Vivo.

Sua principal função é integrar informações provenientes de diferentes fontes oficiais e armazená-las de forma estruturada no banco de dados da aplicação.

O processo pode ser representado como:

```text
Fontes Oficiais
      │
      ▼
Coleta
      │
      ▼
Processamento / ETL
      │
      ▼
MySQL
```

---

## 🎯 Responsabilidades

O Data Aggregator é responsável principalmente por:

* consultar APIs e fontes oficiais;
* coletar informações sobre parlamentares e outras entidades;
* processar e transformar os dados;
* armazenar os dados no MySQL;
* atualizar informações existentes;
* controlar execuções do processo de ETL;
* registrar erros;
* evitar requisições desnecessárias por meio de cache;
* controlar requisições às APIs externas.

---

## 🧰 Tecnologias

As principais tecnologias utilizadas são:

* Python 3.10+
* MySQL
* APIs HTTP
* Processos de ETL

---

## 📂 Estrutura

Entre os principais componentes do projeto estão:

```text
popular/
├── principal.py
└── atualizacao_semanal.py
```

### `principal.py`

Utilizado para a execução do processo principal de carga dos dados.

### `atualizacao_semanal.py`

Utilizado para processos de atualização periódica dos dados.

---

## 🔄 Processo de ETL

O processo segue o conceito de **ETL — Extract, Transform and Load**.

### Extract

Os dados são obtidos a partir das APIs e fontes oficiais.

### Transform

Os dados são tratados e organizados de acordo com a estrutura utilizada pelo sistema.

### Load

Os dados processados são inseridos ou atualizados no banco de dados MySQL.

---

## ⚙️ Controle das execuções

O sistema possui mecanismos para acompanhar a execução do processo de ETL.

Entre as estruturas utilizadas estão registros relacionados a:

* checkpoints;
* erros;
* execuções.

Esses mecanismos permitem acompanhar o processamento e facilitar a recuperação ou análise de problemas durante a coleta.

---

## 🚦 Controle de requisições

Como o processo depende de APIs externas, o Data Aggregator possui mecanismos para controlar as requisições.

Entre eles estão:

* cache de requisições HTTP;
* limitação de requisições;
* tentativas automáticas;
* retry com backoff;
* processamento paralelo quando aplicável.

Esses mecanismos ajudam a reduzir requisições desnecessárias e aumentam a confiabilidade do processo de coleta.

---

## ▶️ Execução

Recomenda-se utilizar um ambiente virtual Python.

Criar o ambiente:

```bash
python -m venv .venv
```

Ativar no Linux/macOS:

```bash
source .venv/bin/activate
```

No Windows:

```bash
.venv\Scripts\activate
```

Instalar as dependências:

```bash
pip install -r requirements.txt
```

Quando necessário, instalar o projeto em modo editável:

```bash
pip install -e .
```

Configure as variáveis de ambiente utilizando o arquivo `.env`.

Exemplo:

```env
DB_HOST=localhost
DB_USER=...
DB_PASSWORD=...
DB_NAME=votovivo
PORTAL_TRANSPARENCIA_API_KEY=...
```

Depois da configuração, o processo principal pode ser executado com:

```bash
python popular/principal.py
```

E a atualização periódica:

```bash
python popular/atualizacao_semanal.py
```

---

## 🔐 Variáveis de ambiente

Credenciais e chaves de acesso às APIs devem ser configuradas por meio de variáveis de ambiente.

Informações sensíveis não devem ser armazenadas diretamente no código ou versionadas no Git.

---

## 📚 Documentação relacionada

* [Arquitetura do Sistema](./arquitetura.md)
* [Banco de Dados](./banco-de-dados.md)
* [Desenvolvimento e Execução](./desenvolvimento.md)

[⬅ Voltar para o README](./README.md)
