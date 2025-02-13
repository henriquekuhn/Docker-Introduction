
# Guia Completo: Configuração do Ambiente de Desenvolvimento com Docker, Ionic e PostgreSQL

## Índice
1. [Introdução ao Uso do Docker para Desenvolvimento](#introducao-ao-uso-do-docker-para-desenvolvimento)
2. [Configuração do Docker e Docker Compose](#configuracao-do-docker-e-docker-compose)
3. [Criação do Projeto Ionic dentro do Container](#criacao-do-projeto-ionic-dionic)
4. [Configuração do Backend (Flask + PostgreSQL)](#configuracao-do-backend-flask-postgresql)
5. [Configuração do VS Code](#configuracao-do-vs-code)
6. [Execução do Projeto e Testes](#execucao-do-projeto-e-testes)
7. [Banco de Dados PostgreSQL no Docker](#banco-de-dados-postgresql-no-docker)
8. [Conclusão e Melhores Práticas](#conclusao-e-melhores-praticas)

## 1. Introdução ao Uso do Docker para Desenvolvimento
**Por que usar Docker?**
- **Padronização:** Todos os desenvolvedores usam o mesmo ambiente.
- **Evita conflitos:** As versões das dependências são mantidas consistentes.
- **Facilidade:** Não é necessário instalar tudo no sistema operacional, basta rodar o container.

Utilizando containers para cada serviço (frontend, backend, banco de dados), garantimos um ambiente isolado e eficiente.

## 2. Configuração do Docker e Docker Compose
### Instalação do Docker
Baixe e instale o Docker no site oficial:
[🔗 Docker Desktop](https://www.docker.com/products/docker-desktop)

Para verificar a instalação, execute:
```sh
docker --version
docker-compose --version
```

Se os comandos retornarem versões, a instalação foi bem-sucedida.

### Criar docker-compose.yml
O arquivo `docker-compose.yml` define nossos containers.

```yaml
version: "3.8"

services:
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
    ports:
      - "8100:8100"

  backend:
    build: ./backend
    depends_on:
      - db
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgresql://htmicron:htmicron@db:5432/mkt_database
    volumes:
      - ./backend:/app

  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: htmicron
      POSTGRES_PASSWORD: htmicron
      POSTGRES_DB: mkt_database
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## 3. Criação do Projeto Ionic dentro do Container
Após subir os containers, acesse o terminal do container frontend e crie o projeto:

```sh
docker-compose up -d
docker exec -it <container_id> bash
ionic start frontend-app blank --type=angular --standalone
```

### Standalone vs NgModule no Angular
- **Standalone:** Estrutura mais moderna e modular.
- **NgModule:** Estrutura tradicional do Angular, usada em projetos antigos.

Recomendamos o uso do **Standalone** para novos projetos.

## 4. Configuração do Backend (Flask + PostgreSQL)
### Criar o ambiente do backend
```sh
mkdir backend && cd backend
touch app.py Dockerfile requirements.txt
mkdir models
```

### Criação do Dockerfile do backend
```Dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### Criação do requirements.txt
```plaintext
flask
flask-sqlalchemy
flask-restful
flask-cors
psycopg2-binary
```

### Configuração do Flask e PostgreSQL
Crie o arquivo `app.py` com a API:

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_restful import Api, Resource
import os
from flask_cors import CORS

app = Flask(__name__)
api = Api(app)
CORS(app)

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://htmicron:htmicron@db:5432/mkt_database")
app.config["SQLALCHEMY_DATABASE_URI"] = DATABASE_URL
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

class Compra(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    item = db.Column(db.String(100), nullable=False)
    quantidade = db.Column(db.Integer, nullable=False)
    preco = db.Column(db.Float, nullable=False)

with app.app_context():
    db.create_all()

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0")
```

## 5. Configuração do VS Code
Para editar arquivos dentro do container, use a extensão **Remote - Containers** no VS Code.

## 6. Execução do Projeto e Testes
```sh
docker-compose up --build
```

- Frontend: [http://localhost:8100](http://localhost:8100)
- Backend: [http://localhost:5000/compras](http://localhost:5000/compras)

## 7. Banco de Dados PostgreSQL no Docker
```sh
docker exec -it <db_container_id> psql -U htmicron -d mkt_database
```

## 8. Conclusão e Melhores Práticas
- ✅ Ambiente de desenvolvimento idêntico para todos os desenvolvedores.
- ✅ Facilidade para subir e testar a aplicação rapidamente.
- ✅ Isolamento entre os serviços, evitando conflitos.

### Expansões futuras
- 🚀 Implementar autenticação com JWT.
- 🚀 Criar um sistema de logs para monitoramento.
- 🚀 Configurar CI/CD para deploy automático.

