# Documentação da Implementação de Estrutura com Docker Compose

<p align="justify">

## Desafio: Implementação de Estrutura com Docker Compose
Esta documentação descreve o processo de implementação de uma estrutura utilizando Docker Compose para o jogo de adivinhação baseado em Flask, com PostgreSQL como banco de dados e NGINX como proxy reverso. O objetivo é criar uma arquitetura resiliente e escalável, mantendo os dados persistentes e facilitando atualizações de componentes.

## Objetivo
O objetivo principal desta atividade é criar um ambiente Docker Compose para o jogo de adivinhação disponível no repositório guess_game, que atenda os seguintes requisitos:

Um container para o backend em Python (Flask).
Um container para o banco de dados PostgreSQL.
Um container NGINX atuando como proxy reverso e servindo o frontend React.

# Componentes
## Containers Backend e Frontend
Backend (Flask): O container do backend executará a aplicação Flask que gerencia a lógica do jogo de adivinhação.
Frontend (React): O container do frontend exibirá a interface React para o usuário.
NGINX: Atuará como proxy reverso para balancear a carga entre múltiplas instâncias do backend e também servirá o frontend.
Banco de Dados PostgreSQL
O container PostgreSQL armazenará os dados do jogo, como as senhas e os resultados das tentativas de adivinhação.
O banco será armazenado em um volume persistente para garantir que os dados não sejam perdidos em reinicializações ou atualizações.
Requisitos
Containers
Backend Python: Deve rodar a aplicação Flask do jogo de adivinhação.
NGINX: Servirá como proxy reverso, balanceando a carga entre instâncias do backend e servindo o frontend React.
Banco de Dados PostgreSQL
O container do PostgreSQL deve armazenar as informações do jogo e ser configurado com um volume separado para garantir a persistência dos dados.
Resiliência e Manutenção
Os containers devem ser configurados para reiniciar automaticamente em caso de falha.
O NGINX deve balancear a carga entre múltiplas instâncias do backend, garantindo escalabilidade e alta disponibilidade.
O banco de dados PostgreSQL deve ser mantido em um volume persistente.
A estrutura deve permitir a atualização de componentes (backend, frontend, banco de dados) sem alterações complexas no código.

# Estrutura de diretório
1. Estrutura dO Repositório

.
├── README.md
├── frontend
│   ├── README.md
│   ├── default.conf
│   ├── jest.config.ts
│   ├── package-lock.json
│   ├── package.json
│   ├── public
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── logo192.png
│   │   ├── logo512.png
│   │   ├── manifest.json
│   │   └── robots.txt
│   ├── src
│   │   ├── App.css
│   │   ├── App.test.tsx
│   │   ├── App.tsx
│   │   ├── components
│   │   │   ├── Breaker.test.tsx
│   │   │   ├── Breaker.tsx
│   │   │   ├── Home.tsx
│   │   │   └── Maker.tsx
│   │   ├── index.css
│   │   ├── index.js
│   │   ├── logo.svg
│   │   ├── mocks
│   │   │   └── handlers.ts
│   │   ├── reportWebVitals.js
│   │   └── setupTests.ts
│   └── tsconfig.json
├── guess
│   ├── __init__.py
│   ├── discover.py
│   └── game_routes.py
├── repository
│   ├── __init__.py
│   ├── dynamodb.py
│   ├── entities.py
│   ├── hash.py
│   ├── postgres.py
│   └── sqlite.py
├── requirements-dev.txt
├── requirements.txt
├── run.py
├── start-backend.sh
└── tests
    └── test_app.py

2. Dockerfile do Backend Python (Flask)
No diretório backend/, o arquivo Dockerfile para configurar o container que executa o backend Flask:

```Dockerfile

FROM python:3.8-slim

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python", "run.py"]
```

3. Dockerfile do Frontend React
No diretório frontend/, o arquivo Dockerfile para configurar o container que serve o frontend via NGINX:

```Dockerfile

FROM node:18.17.0 as build

WORKDIR /app

COPY . /app

RUN npm install
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

4. Configuração do NGINX
No diretório nginx/, o arquivo default.conf para configurar o proxy reverso e balanceamento de carga:

```conf

upstream flask_backend {
    server backend1:5000;
    server backend2:5000;
}

server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://flask_backend;
    }
}
```

5. Arquivo Docker Compose (docker-compose.yml)
No diretório raiz, crie o arquivo docker-compose.yml para orquestrar os serviços:

```yaml

version: '3'
services:
  backend1:
    build: ./backend
    container_name: backend1
    environment:
      - FLASK_ENV=development
    volumes:
      - ./backend:/app
    networks:
      - backend-network
    restart: always
    ports:
      - "5001:5000"

  backend2:
    build: ./backend
    container_name: backend2
    environment:
      - FLASK_ENV=development
    volumes:
      - ./backend:/app
    networks:
      - backend-network
    restart: always
    ports:
      - "5002:5000"

  frontend:
    build: ./frontend
    container_name: frontend
    volumes:
      - ./frontend:/app
    networks:
      - backend-network
    depends_on:
      - backend1
      - backend2
    ports:
      - "3000:80"
    restart: always

  nginx:
    image: nginx
    container_name: nginx
    volumes:
      - ./nginx:/etc/nginx/conf.d
    ports:
      - "80:80"
    networks:
      - backend-network
    depends_on:
      - backend1
      - backend2
      - frontend
    restart: always

  db:
    image: postgres
    container_name: postgres_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secretpass
      POSTGRES_DB: guess_game_db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend-network
    restart: always
    ports:
      - "5432:5432"

networks:
  backend-network:
    driver: bridge

volumes:
  db-data:

```


# Instruções para Rodar o Projeto

1. Clonar o Repositório

**Sintaxe:**
```bash
git clone https://github.com/fams/guess_game.git
cd guess_game
```
2. Configurar e Executar o Docker Compose

**Sintaxe:**
```bash
docker-compose up --build
```
3. Acessar o Jogo

> Backend disponível em http://localhost:5001 e http://localhost:5002
> Frontend disponível em http://localhost:3000

# Atualização de Componentes
Para atualizar um dos componentes (backend, frontend ou banco de dados), basta alterar a versão da imagem Docker no docker-compose.yml e recriar os containers:

**Sintaxe:**
```bash
docker-compose up --build
```
# Melhorias Futuras
> Otimizar tamanho e segunraça das imagens utilizadno por exemplo Chainguard.
> Implementar autenticação de usuário.
> Adicionar limite de tentativas no jogo.
> Adicionar suporte para mais bancos de dados.

</p>