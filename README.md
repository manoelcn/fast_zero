# Fast Zero

O **Fast Zero** é um backend moderno de alta performance que serve como API RESTful para um **Gerenciador de Tarefas (To-Do List)**. Desenvolvido com [FastAPI](https://fastapi.tiangolo.com/), o projeto foi estruturado seguindo as melhores práticas da engenharia de software: uso de concorrência assíncrona, injeção de dependências robusta, validação estrita de dados e testes automatizados abrangentes.

A aplicação gerencia usuários e tarefas com isolamento completo, fornecendo autenticação baseada em JWT e persistência de dados relacional.

> [!NOTE]
> Este projeto é parte de um curso prático de desenvolvimento web e boas práticas com Python, que pode ser encontrado em: [FastAPI do Zero](https://fastapidozero.dunossauro.com/estavel/).

---

## 🛠️ Tecnologias Utilizadas

- **Core & Framework:** [Python 3.14+](https://www.python.org/) e [FastAPI](https://fastapi.tiangolo.com/) (executado sob servidor ASGI [Uvicorn](https://uvicorn.dev/)).
- **ORM & Banco de Dados:** [SQLAlchemy 2.0](https://www.sqlalchemy.org/) (Async engine), utilizando [Alembic](https://alembic.sqlalchemy.org/) para o gerenciamento de migrações estruturais do banco.
- **Drivers de Conexão:** [aiosqlite](https://aiosqlite.omnilib.dev/en/stable/) (para SQLite assíncrono em desenvolvimento) e [psycopg 3](https://www.psycopg.org/psycopg3/) (para integração assíncrona ao PostgreSQL).
- **Segurança & Autenticação:** [PyJWT](https://pyjwt.readthedocs.io/) para criação de tokens JWT assinados, e [pwdlib](https://frankie567.github.io/pwdlib/) (com algoritmo criptográfico **Argon2**) para hashing seguro de senhas.
- **Validação de Dados:** [Pydantic v2](https://docs.pydantic.dev/) (para validação de payloads de request/response) e [Pydantic Settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) (para carregamento tipado de variáveis de ambiente).
- **Gerenciamento de Dependências:** [Poetry](https://python-poetry.org/).
- **Garantia de Qualidade & CI:**
  - **Linter & Formatação:** [Ruff](https://astral.sh/ruff) (regras estritas de linting e autoformatação).
  - **Verificação Ortográfica:** [Typos](https://github.com/crate-ci/typos).
  - **Testes Automatizados:** [Pytest](https://docs.pytest.org/) com suporte assíncrono (`pytest-asyncio`), cálculo de cobertura (`pytest-cov`), geração de massa de dados (`factory-boy`), manipulação de tempo (`freezegun`) e testes com contêineres reais (`testcontainers`).
  - **Integração Contínua:** GitHub Actions configurado para execução do pipeline de testes a cada push/pull request.
- **Automação de CLI:** [Taskipy](https://github.com/taskipy/taskipy) para criação de atalhos rápidos de desenvolvimento.

---

## ✨ Funcionalidades Principais (Features)

- **Gestão Completa de Usuários (CRUD):** 
  - Cadastro de usuários com validação de dados (e-mail único via `EmailStr` e `username` único).
  - Consulta de perfil individual e listagem paginada (`offset` e `limit`).
  - Atualização cadastral e exclusão lógica/física da conta.
- **Autenticação & Autorização:**
  - Fluxo baseado no padrão **OAuth2 com Bearer Tokens (JWT)**.
  - Endpoint dedicado para geração de token de acesso (`/auth/token`).
  - Endpoint para atualização (refresh) de sessões ativas (`/auth/refresh_token`).
- **Gestão de Tarefas (To-Do List) Isolada:**
  - Criação de tarefas com título, descrição e estado (`draft`, `todo`, `doing`, `done`, `trash`).
  - Cada tarefa é vinculada a um usuário específico. Usuários autenticados têm acesso exclusivo e isolado apenas aos seus próprios registros.
  - Atualização parcial de tarefas via método HTTP `PATCH`.
  - Remoção de tarefas por ID com validação de propriedade.
- **Filtros e Paginação Avançada:**
  - Listagem de tarefas com filtros por título, descrição e estado.
  - Paginação robusta para otimização de banda de rede e performance de banco.

---

## ⚙️ Pré-requisitos

Para rodar o projeto localmente, certifique-se de ter instalado:
- **Python >= 3.14**
- **Poetry** (Gerenciador de dependências do Python)
- **Docker & Docker Compose** (Necessário se optar por executar o PostgreSQL no ambiente conteinerizado)

---

## 📂 Estrutura de Pastas

Abaixo está descrita a organização estrutural do projeto:

```text
fast_zero/
├── .github/
│   └── workflows/
│       └── pipeline.yaml      # Pipeline de Integração Contínua (GitHub Actions)
├── fast_zero/                 # Módulo principal da aplicação
│   ├── routers/               # Rotas separadas por responsabilidade (users, todos, auth)
│   │   ├── auth.py
│   │   ├── todos.py
│   │   └── users.py
│   ├── app.py                 # Ponto de inicialização do FastAPI e configurações de CORS
│   ├── database.py            # Instanciação do engine assíncrono do ORM
│   ├── models.py              # Definições de entidades de banco de dados (SQLAlchemy)
│   ├── schemas.py             # Modelos de dados e validações estruturais (Pydantic)
│   ├── security.py            # Hashing de senhas, validações e geração de JWT
│   └── settings.py            # Configuração tipada de variáveis de ambiente (.env)
├── migrations/                # Histórico de alterações e migrações do banco de dados (Alembic)
├── tests/                     # Suite de testes unitários e de integração
├── Dockerfile                 # Configuração para empacotamento da aplicação em contêiner
├── compose.yaml               # Orquestração do banco PostgreSQL e do container da API
├── entrypoint.sh              # Script executado ao iniciar o contêiner (migrações + uvicorn)
├── pyproject.toml             # Declarações de dependências, linter (Ruff), testes e tasks
└── alembic.ini                # Arquivo de configuração do Alembic
```

---

## 🔐 Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto com base no arquivo [.env.example](.env.example). As seguintes chaves são suportadas:

| Variável | Descrição | Exemplo / Sugestão |
| :--- | :--- | :--- |
| `DATABASE_URL` | String de conexão assíncrona com a base de dados. | *Ver opções de execução abaixo.* |
| `SECRET_KEY` | Chave criptográfica usada para assinar digitalmente os JWTs. | `80e38143fc20cf10f2...` *(Chave longa e aleatória)* |
| `ALGORITHM` | Algoritmo de hash para assinar os tokens JWT. | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Tempo de vida útil do token de acesso emitido. | `30` |
| `POSTGRES_USER` | Usuário do banco PostgreSQL *(Necessário apenas para Docker Compose)*. | `app_user` |
| `POSTGRES_DB` | Nome da base de dados no PostgreSQL *(Necessário apenas para Docker Compose)*. | `app_db` |
| `POSTGRES_PASSWORD` | Senha da base de dados no PostgreSQL *(Necessário apenas para Docker Compose)*. | `app_password` |

---

## 🚀 Como Executar o Projeto

Escolha um dos dois caminhos abaixo dependendo da sua preferência de infraestrutura local:

### Opção A: Execução 100% Local (Com SQLite)

Esta modalidade é recomendada para testes rápidos de desenvolvimento e elimina a necessidade de inicializar contêineres Docker.

1. **Clonar o Repositório:**
   ```bash
   git clone <URL_DO_REPOSITORIO>
   cd fast_zero
   ```

2. **Instalar Dependências:**
   ```bash
   poetry install
   ```

3. **Configurar as Variáveis de Ambiente:**
   Copie o arquivo de exemplo:
   ```bash
   cp .env.example .env
   ```
   Abra o arquivo `.env` gerado e configure o `DATABASE_URL` para apontar para o SQLite assíncrono:
   ```env
   DATABASE_URL="sqlite+aiosqlite:///database.db"
   SECRET_KEY="sua_chave_secreta_longa_aqui"
   ALGORITHM="HS256"
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   ```

4. **Executar Migrações:**
   Crie as tabelas da aplicação executando as migrações do Alembic:
   ```bash
   poetry run alembic upgrade head
   ```

5. **Iniciar a API:**
   ```bash
   poetry run task run
   ```
   * O servidor local estará ativo no endereço: **[http://127.0.0.1:8000](http://127.0.0.1:8000)**.
   * Acesse a documentação interativa das rotas (Swagger UI): **[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)**.

---

### Opção B: Execução Completa (Com Docker & PostgreSQL)

Esta modalidade simula um ambiente de produção ao subir o banco de dados PostgreSQL integrado à aplicação FastAPI através de contêineres.

1. **Configurar as Variáveis de Ambiente:**
   Copie o arquivo de exemplo:
   ```bash
   cp .env.example .env
   ```
   Ajuste o `.env` para carregar as configurações do Postgres:
   ```env
   POSTGRES_USER=app_user
   POSTGRES_DB=app_db
   POSTGRES_PASSWORD=app_password

   DATABASE_URL="postgresql+psycopg://app_user:app_password@localhost:5432/app_db"
   SECRET_KEY="sua_chave_secreta_longa_aqui"
   ALGORITHM="HS256"
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   ```

2. **Subir os Serviços com Docker Compose:**
   Rode o comando de inicialização com compilação da imagem:
   ```bash
   docker compose up -d --build
   ```
   *Nota: O Docker Compose aguardará o banco estar saudável (Healthcheck do Postgres) e executará o script [entrypoint.sh](file:///home/mcandido/Desenvolvimento/fast_zero/entrypoint.sh), aplicando automaticamente as migrações antes de inicializar o servidor.*

3. **Acessar os Serviços:**
   * A API estará disponível na porta mapeada externa: **[http://localhost:8000](http://localhost:8000)**.
   * Documentação Swagger UI ativa em: **[http://localhost:8000/docs](http://localhost:8000/docs)**.

---

## 🧪 Comandos Úteis (Tasks)

O projeto disponibiliza atalhos via `taskipy` para simplificar o fluxo diário de desenvolvimento. Você pode chamá-los a partir da CLI usando o Poetry:

| Comando | Descrição |
| :--- | :--- |
| `poetry run task run` | Inicia o servidor FastAPI local em modo reload (desenvolvimento). |
| `poetry run task lint` | Executa o validador do [Ruff](https://astral.sh/ruff) sobre o código do projeto. |
| `poetry run task format` | Executa o autoformatador do [Ruff](https://astral.sh/ruff) para alinhar padrões visuais do código. |
| `poetry run task test` | Executa a suite completa de testes automatizados via [Pytest](https://docs.pytest.org/). |
| `poetry run task post_test` | Gera um relatório em formato HTML da cobertura de testes na pasta `htmlcov/`. |

---

## 🧑‍💻 Autores & Contribuição

Desenvolvido por:

* **Manoel Cândido** - [manoelcandidodev@gmail.com](mailto:manoelcandidodev@gmail.com)
  - GitHub: [@manoelcn](https://github.com/manoelcn)
  - LinkedIn: [Manoel Cândido](https://www.linkedin.com/in/manoelcandido/)

---
Para contribuir, abra uma issue ou envie um Pull Request apontando suas sugestões de melhorias.