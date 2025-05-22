# Projeto DimDimApp - API Flask + MySQL + Docker

## Equipe
- Nome da equipe: [Seu Nome da Equipe]
- Integrantes:
  - Caio Marques - RM555997
  - Caio Amarante - RM558640
  - Felipe Camargo - RM556325

## Tecnologias
- Python 3.11 + Flask
- MySQL 8 (Docker oficial)
- Docker

## Código fonte da aplicação — `app.py`

```python
from flask import Flask, request, jsonify
import mysql.connector
import os

app = Flask(__name__)

def get_db_connection():
    conn = mysql.connector.connect(
        host=os.getenv('MYSQL_HOST', 'mysql-container'),
        user=os.getenv('MYSQL_USER', 'dimdimuser'),
        password=os.getenv('MYSQL_PASSWORD', 'dimdimpass'),
        database=os.getenv('MYSQL_DATABASE', 'dimdimdb')
    )
    return conn

@app.route('/items', methods=['GET'])
def get_items():
    conn = get_db_connection()
    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT * FROM items")
    items = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify(items)

@app.route('/items', methods=['POST'])
def add_item():
    new_item = request.json
    name = new_item.get('name')
    if not name:
        return jsonify({"error": "O campo 'name' é obrigatório"}), 400

    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO items (name) VALUES (%s)", (name,))
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({'message': 'Item adicionado com sucesso'}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

---

## Dependências — `requirements.txt`

```
flask
mysql-connector-python
```

---

## Dockerfile da aplicação — `Dockerfile`

```dockerfile
FROM python:3.11-slim

RUN useradd -ms /bin/bash appuser

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER appuser

ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

EXPOSE 5000

CMD ["flask", "run"]
```

---

## Passo a passo para executar o projeto

### 1. Clonar o repositório

```bash
git clone [URL_DO_REPOSITORIO]
cd DimDimApp
```

### 2. Criar rede Docker

```bash
docker network create dimdim-network
```

### 3. Criar volume para persistência do banco

```bash
docker volume create mysql_data
```

### 4. Subir container MySQL

```bash
docker run -d \
  --name mysql-container \
  --network dimdim-network \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=dimdimdb \
  -e MYSQL_USER=dimdimuser \
  -e MYSQL_PASSWORD=dimdimpass \
  -v mysql_data:/var/lib/mysql \
  mysql:latest
```

### 5. Criar tabela `items` no banco

```bash
docker exec -it mysql-container mysql -u root -p
```

Digite a senha: `rootpassword`

No prompt MySQL, execute:

```sql
USE dimdimdb;

CREATE TABLE items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);

EXIT;
```

### 6. Construir a imagem da aplicação

```bash
docker build -t dimdimapp .
```

### 7. Rodar container da aplicação

```bash
docker run -d \
  --name python-app \
  --network dimdim-network \
  -e MYSQL_HOST=mysql-container \
  -e MYSQL_USER=dimdimuser \
  -e MYSQL_PASSWORD=dimdimpass \
  -e MYSQL_DATABASE=dimdimdb \
  -p 5000:5000 \
  dimdimapp
```

### 8. Testar a API

- GET `http://[IP_DA_VM]:5000/items` — para listar os itens (retorna JSON)  
- POST `http://[IP_DA_VM]:5000/items` — para adicionar um item  
  JSON do body:  
  ```json
  {
    "name": "Nome do Item"
  }
  ```

---

## Comandos para verificar os containers

### Container da aplicação

```bash
docker exec -it python-app bash
whoami
ls -l /app
exit
```

### Container do MySQL

```bash
docker exec -it mysql-container bash
whoami
exit
```

---

## Evidências para entregar

- Print do terminal mostrando os containers rodando (`docker ps`)  
- Print do comando `docker exec python-app whoami` (deve mostrar usuário não root, ex: appuser)  
- Print do comando `docker exec mysql-container whoami`  
- Print da estrutura de arquivos dentro do container da aplicação (`ls -l /app`)  
- Print da resposta da API via Postman ou curl mostrando os dados do banco  
- Print mostrando persistência dos dados no banco (antes e depois de reiniciar o container MySQL)  

---

## Considerações finais

- Não esqueça de versionar todo esse código no GitHub.  
- Envie o link do repositório junto com as evidências.  
- Faça o upload do PDF com a documentação da equipe e link do repo conforme o requisito.  
