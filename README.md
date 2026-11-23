# Cadastro de Interesse em Intercâmbio

Aplicação da AIESEC Brasil para cadastro e qualificação de pessoas interessadas em programas de intercâmbio. O projeto reúne a interface web, uma API de integração e configurações para desenvolvimento e deploy.

## Visão geral

O repositório é organizado em três partes:

- `frontend/`: aplicação Next.js com os formulários e páginas públicas.
- `backend/`: API Flask que valida os dados, consulta metadados e integra com serviços externos.
- `docker-compose.yml` e `proxy/`: ambiente local de teste que inicia os serviços juntos.

Em produção, o frontend é publicado no Cloudflare Workers e o backend no PythonAnywhere. O Docker Compose da raiz é destinado ao desenvolvimento e teste local.

## Tecnologias

- Frontend: Next.js, React, TypeScript, Axios e Tailwind CSS
- Backend: Python, Flask, Flask-OpenAPI3 e Pydantic
- Integrações: Google Apps Script e EXPA
- Infraestrutura local: Docker Compose e Nginx
- Deploy: Cloudflare Workers e PythonAnywhere

## Funcionalidades

- Exibição de formulários de pré-cadastro e qualificação de leads.
- Carregamento de campos e opções a partir dos metadados disponibilizados pela API.
- Importação de cadastros B2C por planilha.
- Validação de dados no frontend e no backend.
- Envio de cadastros para os serviços de integração configurados.
- Consulta de universidades, escritórios e demais opções usadas nos formulários.

## Arquitetura

```text
Navegador
   │
   ▼
Frontend Next.js ── /api/<rota> ──► Proxy da aplicação Next.js
                                      │ EXTERNAL_API_URL
                                      ▼
                                  API Flask
                                    ├── Google Apps Script
                                    └── EXPA
```

O frontend usa uma rota proxy server-side para encaminhar chamadas à API. O endereço do backend é configurado por `EXTERNAL_API_URL` e não precisa ser exposto no código executado pelo navegador. A API consulta e mantém metadados em cache para validar e montar os dados dos formulários.

## Estrutura do projeto

```text
.
├── backend/
│   ├── app/                 # API, controllers, modelos, integrações e cache
│   ├── .github/             # workflow e scripts de deploy
│   ├── tests/               # testes automatizados
│   ├── .env.example         # modelo das variáveis do backend
│   ├── Dockerfile
│   └── README.md
├── frontend/
│   ├── src/app/             # páginas, layouts e rota proxy
│   ├── src/components/      # formulários e componentes de interface
│   ├── src/hook/            # estado e validação dos formulários
│   ├── src/service/         # clientes HTTP
│   ├── .github/             # workflow de deploy
│   ├── Dockerfile
│   └── README.md
├── proxy/                   # configuração do Nginx para Docker local
├── .github/workflows/       # sincronização dos diretórios dos serviços
├── docker-compose.yml
└── README.md
```

## Requisitos

- Node.js e npm compatíveis com as versões definidas no workflow do frontend.
- Python compatível com a versão definida em `backend/requirements.txt` e no deploy.
- Docker Engine e Docker Compose para executar a stack integrada local.

## Execução local

### Frontend

```bash
cd frontend
npm ci
npm run dev
```

O servidor inicia em `http://localhost:3000`. Configure `EXTERNAL_API_URL` no ambiente do processo Next.js para apontar para a API desejada. Consulte [frontend/README.md](frontend/README.md) para os detalhes do proxy e dos comandos.

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Preencha as variáveis necessárias no `.env` e inicie a aplicação:

```bash
python server.py
```

Consulte [backend/README.md](backend/README.md) para as integrações e rotas da API.

### Docker Compose

Na raiz do projeto, crie o `.env` com os valores necessários à stack local e execute:

```bash
docker compose up --build
```

A interface fica disponível em `http://localhost`, servida pelo Nginx. Para encerrar:

```bash
docker compose down
```

Esse ambiente é para teste local; os deploys de produção usam os workflows próprios do frontend e do backend.

## Configuração e segredos

Use `backend/.env.example` como referência para as variáveis da API, incluindo ambiente, cache, domínios autorizados, credenciais do EXPA e endpoints do Google Apps Script. O backend carrega o `.env` do diretório `backend/` ao iniciar.

O frontend usa `EXTERNAL_API_URL` no servidor para definir a API de destino e `NEXT_PUBLIC_URL_DIRECIONADA` para construir links direcionados. Nos deploys, os valores e credenciais são fornecidos pelos secrets dos workflows; não versione `.env` nem tokens.

## Verificações

Backend:

```bash
cd backend
python -m pytest
```

Frontend:

```bash
cd frontend
npm run lint
npm run build:cloudflare
```

## Deploy

O deploy do frontend usa OpenNext para compilar e publicar no Cloudflare Workers. O deploy do backend envia os arquivos e a configuração de ambiente ao PythonAnywhere e recarrega a aplicação. Consulte `frontend/.github/workflows/deploy.yaml` e `backend/.github/workflows/deploy.yml` para os gatilhos e secrets necessários.
