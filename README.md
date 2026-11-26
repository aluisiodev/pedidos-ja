<div align="center">

# Pedido Já

Sistema de gerenciamento de pedidos desenvolvido para a disciplina de **Desenvolvimento de Sistemas Distribuídos**
(Entrega 1: API Pedidos, persistência e integração com interface web).

![Python](https://img.shields.io/badge/Python_3.12-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy_2-D71F00?logo=sqlalchemy&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL_16-4169E1?logo=postgresql&logoColor=white)
![React](https://img.shields.io/badge/React_19-20232A?logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_4-06B6D4?logo=tailwindcss&logoColor=white)
![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white)

<img src="docs/screenshots/catalogo.webp" alt="Catálogo do Pedido Já, com combo de hambúrguer, hot dog, batata frita e refrigerante" width="900">

</div>

## Identificação

| | |
|---|---|
| **Disciplina** | Desenvolvimento de Sistemas Distribuídos |
| **Curso** | Ciência da Computação, 8º semestre |
| **Trabalho** | Entrega 1: API Pedidos (APS) |
| **Versão entregue** | tag [`APIPedidos-1-final`](../../tree/APIPedidos-1-final) |

### Integrantes

| Nome | RA |
|---|---|
| Aluísio Pereira Alves | N135891 |
| Enzo Orlandi Gomes | G788EJ5 |
| Erick de Brito Carvalho | G78HED3 |
| Kayky Crespo dos Santos | G839226 |

## Telas

| Visão geral (admin) | Lista de pedidos (admin) |
|---|---|
| ![Visão geral com contagem de pedidos por status](docs/screenshots/admin-visao.webp) | ![Tabela de pedidos com busca, filtro por status e ordenação](docs/screenshots/admin-pedidos.webp) |
| **Detalhe do pedido** | **Arquitetura** |
| ![Detalhe de um pedido com troca de status](docs/screenshots/admin-detalhe.webp) | ![Página que explica a arquitetura física e interna](docs/screenshots/arquitetura.webp) |

## Objetivo

Implementar uma aplicação de gerenciamento de pedidos composta por:

- uma **API HTTP em FastAPI** para criação, consulta, listagem e atualização de status de pedidos;
- uma **camada de persistência real em PostgreSQL**, com schema versionado por Alembic;
- uma **interface web em React** que consome a API para permitir que clientes criem e acompanhem pedidos e que uma área administrativa consulte e atualize seus status.

Cada pedido representa um único produto com determinada quantidade. O escopo da Entrega 1 é propositalmente reduzido: não há múltiplos itens persistidos por pedido, pagamento, estoque, autenticação ou outros serviços adicionais.

## Arquitetura

### Visão física

```
                     HTTP/JSON
Navegador ─────────────────────────────────────►

┌──────────────────────────────────────────────┐
│ container: pedidos                           │
│                                              │
│ FastAPI                                      │
│ API / Service / Repository                   │
│ SPA React compilada                          │
│ porta publicada: 8000                        │
└──────────────────────┬───────────────────────┘
                       │
                       │ protocolo PostgreSQL
                       ▼
┌──────────────────────────────────────────────┐
│ container: postgres                          │
│ PostgreSQL 16                                │
│ volume persistente postgres_data             │
│ porta 5432 somente na rede interna Docker    │
└──────────────────────────────────────────────┘
```

A solução possui somente dois serviços Docker: `pedidos` e `postgres`.

O PostgreSQL não publica a porta 5432 no host. Ele é acessado pela aplicação através da rede interna do Docker Compose, usando o hostname `postgres`.

### Visão interna da aplicação `pedidos`

```
Requisição HTTP
      │
      ▼
┌─────────────┐
│ API/Router  │
└──────┬──────┘
       ▼
┌─────────────┐
│   Service   │
└──────┬──────┘
       ▼
┌─────────────┐
│ Repository  │
└──────┬──────┘
       ▼
┌─────────────┐
│ PostgreSQL  │
└─────────────┘
```

- **API/Router** (`app/api/`): recebe requisições HTTP, valida entrada com schemas Pydantic, delega o fluxo ao Service e traduz o resultado para respostas HTTP.
- **Service** (`app/services/pedido_service.py`): concentra as regras de negócio, incluindo cálculo de `valor_total`, status inicial `CRIADO` e normalização dos campos textuais.
- **Repository** (`app/repositories/pedido_repository.py`): concentra o acesso ao banco por SQLAlchemy e as operações de persistência.
- **PostgreSQL**: fonte de verdade dos dados. Os pedidos não ficam armazenados na memória do processo FastAPI.

API, Service e Repository são camadas lógicas da mesma aplicação, não containers nem microsserviços separados.
O frontend React é compilado pelo Docker e servido pela própria aplicação FastAPI, na mesma origem da API.

## Estrutura do repositório

```
.
├── app/
│   ├── api/                    # Rotas HTTP
│   ├── models/                 # Modelos SQLAlchemy
│   ├── repositories/           # Acesso a dados
│   ├── schemas/                # Schemas Pydantic
│   ├── services/               # Regras de negócio
│   ├── config.py               # Configuração por variáveis de ambiente
│   ├── database.py             # Engine, Session e Base
│   └── main.py                 # Aplicação FastAPI + SPA
├── alembic/
│   └── versions/               # Migrations
├── docker/
│   └── entrypoint.sh           # Migration + inicialização do Uvicorn
├── docs/screenshots/           # Capturas de tela usadas neste README
├── frontend/                   # SPA React + TypeScript + Vite
├── tests/
│   ├── integration/            # API + PostgreSQL
│   └── unit/                   # Regras do PedidoService
├── .dockerignore
├── .env.example
├── .gitattributes
├── .gitignore
├── alembic.ini
├── COMMITS_SUGERIDOS.md
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── README.md
└── requirements.txt
```

## Tecnologias

| Camada | Tecnologias |
|---|---|
| **Backend** | Python 3.12, FastAPI, Uvicorn, SQLAlchemy 2.x, Pydantic v2, Alembic, psycopg 3, pytest, httpx, Ruff |
| **Persistência** | PostgreSQL 16, volume Docker persistente |
| **Frontend** | React 19, TypeScript, Vite, React Router DOM, TanStack Query, Tailwind CSS v4, shadcn/ui (mais detalhes em [`frontend/README.md`](frontend/README.md)) |
| **Infraestrutura** | Docker, Docker Compose, build multi-stage (Node usado apenas durante o build do frontend, runtime final baseado em Python) |

## Modelo de Pedido

| Campo | Tipo | Regra |
|---|---|---|
| `id` | inteiro | gerado pelo banco |
| `cliente` | texto | obrigatório e não vazio |
| `produto` | texto | obrigatório e não vazio |
| `quantidade` | inteiro | mínimo 1 |
| `valor_unitario` | decimal | positivo, até 2 casas decimais |
| `valor_total` | decimal | calculado pela aplicação |
| `status` | texto | `CRIADO`, `CONFIRMADO` ou `CANCELADO` |
| `data_criacao` | timestamp | definido na criação |

### Regras principais

O cliente envia apenas:

```json
{
  "cliente": "Ana Souza",
  "produto": "Combo de Hambúrguer",
  "quantidade": 2,
  "valor_unitario": 32.90
}
```

A aplicação define automaticamente `id`, `valor_total`, `status` e `data_criacao`.

O valor total é calculado no Service:

```
valor_total = quantidade × valor_unitario
2 × 32,90 = 65,80
```

Valores monetários são tratados internamente com `Decimal` e persistidos como `NUMERIC` no PostgreSQL.

O status inicial de todo pedido é `CRIADO`. Valores permitidos: `CRIADO`, `CONFIRMADO` e `CANCELADO`.

## API

### Endpoints de negócio

| Método | Caminho | Descrição | Sucesso | Erros principais |
|---|---|---|---|---|
| GET | `/health` | Health check | 200 | — |
| POST | `/pedidos` | Cria pedido | 201 | 422 |
| GET | `/pedidos` | Lista pedidos | 200 | — |
| GET | `/pedidos/{id}` | Consulta pedido | 200 | 404, 422 |
| PATCH | `/pedidos/{id}/status` | Atualiza somente o status | 200 | 404, 422 |

Nenhum outro endpoint de negócio existe. Não fazem parte da Entrega 1: autenticação, CRUD de produtos, estoque, pagamento, paginação, `DELETE` e `PUT`.

### Endpoints técnicos

- `GET /docs`: Swagger UI
- `GET /openapi.json`: schema OpenAPI

O ReDoc e o redirect OAuth2 do Swagger estão desabilitados.

## Interface web

| Rotas públicas | Rotas administrativas/técnicas |
|---|---|
| `/` | `/admin` |
| `/produto/:produtoId` | `/admin/pedidos` |
| `/carrinho` | `/admin/pedidos/:id` |
| `/revisar` | `/admin/api` |
| `/acompanhar` | `/admin/arquitetura` |
| `/pedido/:id` | |

A área `/admin` é uma separação de navegação e experiência de uso. Não representa autenticação ou autorização, pois isso está fora do escopo desta entrega.

O catálogo é estático no frontend. O carrinho representa o rascunho de um único pedido e não é persistido no servidor.

## Como executar

### Forma oficial

Pré-requisito: **Docker** e **Docker Compose**.

Não é necessário instalar localmente Python, Node, PostgreSQL, dependências npm ou dependências pip.
Também não é necessário copiar `.env.example` para `.env` para executar a configuração padrão da entrega.

Na raiz do repositório:

```bash
docker compose up -d --build
```

O comando:

1. baixa/constrói as imagens necessárias;
2. compila a SPA React com Node;
3. prepara o runtime Python;
4. sobe PostgreSQL 16;
5. aguarda o healthcheck do banco;
6. executa `alembic upgrade head`;
7. inicia o FastAPI/Uvicorn;
8. disponibiliza API e frontend em http://localhost:8000.

### Verificar os containers

```bash
docker compose ps
```

Devem existir somente `pedidos` e `postgres`. O serviço `pedidos` publica a porta `8000`; o serviço `postgres` não publica `5432` no host.

### URLs

| URL | Conteúdo |
|---|---|
| http://localhost:8000/ | Interface do cliente |
| http://localhost:8000/admin | Área administrativa |
| http://localhost:8000/docs | Swagger |
| http://localhost:8000/openapi.json | OpenAPI |
| http://localhost:8000/health | Health check |
| http://localhost:8000/pedidos | API de Pedidos |

### Health check

```bash
curl http://localhost:8000/health
# {"status":"ok"}
```

## Configuração

O Docker Compose fornece defaults de laboratório para execução da entrega.

### Backend

| Variável | Default no Compose | Descrição |
|---|---|---|
| `DATABASE_URL` | `postgresql+psycopg://pedidos:pedidos@postgres:5432/pedidos` | conexão PostgreSQL |
| `CORS_ORIGINS` | `http://localhost:5173` | origem do frontend em desenvolvimento |
| `POSTGRES_DB` | `pedidos` | banco PostgreSQL |
| `POSTGRES_USER` | `pedidos` | usuário PostgreSQL |
| `POSTGRES_PASSWORD` | `pedidos` | senha PostgreSQL |

Ao executar o backend fora do Docker Compose, `DATABASE_URL` é obrigatória. Veja `.env.example` para referência.

### Frontend

O build de produção usa:

```
VITE_API_BASE_URL=
VITE_APP_BASE_PATH=/
VITE_USE_MOCKS=false
```

Assim, frontend e API usam a mesma origem em produção.

## Alembic

O schema do PostgreSQL é versionado com Alembic. O projeto não utiliza `Base.metadata.create_all()` em produção.
A migration inicial está em `alembic/versions/0001_create_pedidos_table.py`.

Na inicialização do container `pedidos`, o `docker/entrypoint.sh` executa automaticamente `alembic upgrade head` e depois o `uvicorn`.

Para execução local fora do Docker:

```bash
export DATABASE_URL=postgresql+psycopg://pedidos:pedidos@localhost:5432/pedidos
alembic upgrade head
```

## Persistência

Os pedidos são armazenados no PostgreSQL e permanecem no volume Docker `postgres_data`.
O container da aplicação pode ser reiniciado sem perda dos pedidos:

```bash
curl -X POST http://localhost:8000/pedidos \
  -H "Content-Type: application/json" \
  -d '{"cliente":"Ana Souza","produto":"Combo de Hambúrguer","quantidade":2,"valor_unitario":32.90}'

docker compose restart pedidos

curl http://localhost:8000/pedidos/<id>   # o pedido continua existindo
```

Também é possível parar apenas a API com `docker compose stop pedidos`. Nesse momento o serviço `postgres` continua ativo.
Depois de `docker compose start pedidos`, os dados continuam disponíveis.

### Validação realizada

Durante a validação final da entrega foram confirmados:

- `docker compose up -d --build` funcionando;
- `pedidos` e `postgres` saudáveis;
- `/health` retornando `{"status":"ok"}`;
- frontend funcionando em `localhost:8000` e Swagger em `/docs`;
- criação real de pedido, com cálculo `2 × 32,90 = 65,80` e status inicial `CRIADO`;
- atualização para `CONFIRMADO`;
- persistência após `docker compose restart pedidos`;
- persistência após `docker compose stop pedidos` e `docker compose start pedidos`;
- PostgreSQL permanecendo ativo enquanto o container `pedidos` está parado.

## Como parar

```bash
docker compose stop    # para os serviços
docker compose down    # para e remove os containers, preservando o volume
```

Evite usar `docker compose down -v`: o parâmetro `-v` remove o volume do PostgreSQL e apaga os pedidos persistidos.

## Testes

### Backend

Os testes unitários não dependem de PostgreSQL. Os testes de integração exigem um PostgreSQL de testes acessível ao processo que executa `pytest`, pela variável `TEST_DATABASE_URL`.

Por segurança:

- `TEST_DATABASE_URL` é obrigatória para integração;
- o nome do banco deve terminar em `_test`;
- a configuração de testes sobrescreve `DATABASE_URL`;
- isso impede que a suíte destrutiva de integração utilize acidentalmente o banco de desenvolvimento.

```bash
export TEST_DATABASE_URL=postgresql+psycopg://pedidos:pedidos@localhost:5432/pedidos_test
pytest

# lint e formatação
ruff check .
ruff format --check .
```

### Frontend

```bash
cd frontend
npm ci
npm run test
npm run typecheck
npm run lint
npm run build
```

Os testes do frontend incluem validação estrita dos IDs de pedido. IDs válidos: `1`, `42`, `100`.
Formatos rejeitados: `0`, `-1`, `+1`, `1.5`, `1e2`, `abc`, `NaN`, `Infinity`.

## Decisões de implementação

- **Camadas explícitas.** A separação API → Service → Repository mantém responsabilidades claras: HTTP na camada API, regras no Service e persistência no Repository.
- **Valor monetário.** `valor_total` nunca é aceito como entrada. O cálculo é feito com `Decimal`, evitando `float` na regra de negócio.
- **Status.** O conjunto permitido é restrito a `CRIADO`, `CONFIRMADO` e `CANCELADO`. O banco também possui restrições para preservar a integridade dos dados.
- **API stateless.** A aplicação FastAPI não guarda pedidos em memória. O PostgreSQL é a fonte de verdade, por isso reiniciar o container `pedidos` não remove pedidos existentes.
- **SPA same-origin.** O frontend compilado é servido pela FastAPI. Em produção, frontend e API ficam em `http://localhost:8000`, o que simplifica a execução e elimina a necessidade de um servidor web adicional.
- **Fallback da SPA.** Rotas de frontend podem ser acessadas diretamente pelo navegador. Namespaces reservados da API e documentação não são convertidos indevidamente em `index.html`. Exemplos de caminhos inválidos que retornam erro HTTP: `/pedidos/foo/bar`, `/health/foo`, `/openapi.json/foo`, `/redoc`, `/docs/oauth2-redirect`.

## Limitações da Entrega 1

Por decisão de escopo, não existem: autenticação ou autorização, login/JWT, usuários persistidos, múltiplos itens por pedido,
carrinho persistido no servidor, estoque, pagamento, frete, CRUD de produtos no backend, paginação, `PUT /pedidos`,
`DELETE /pedidos`, Redis, Kafka, RabbitMQ e serviços adicionais além de `pedidos` e `postgres`.

O catálogo exibido pela interface é estático e existe apenas no frontend.

## Entrega final

A execução oficial deve ser reproduzível a partir do repositório:

```bash
git clone https://github.com/aluisiodev/pedidos-ja.git
cd pedidos-ja
git checkout APIPedidos-1-final
docker compose up -d --build
```

A versão entregue é identificada pela tag `APIPedidos-1-final`.
