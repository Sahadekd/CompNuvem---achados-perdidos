# Achados & Perdidos

Aplicação web para gerenciamento de itens achados e perdidos, desenvolvida com Flask e PostgreSQL e executada em containers Docker por meio do Docker Compose.

## 1. Arquitetura

A aplicação é composta por dois containers:

* **web**: aplicação Flask executada com Gunicorn.
* **db**: banco de dados PostgreSQL 16 Alpine.

Os containers se comunicam pela rede Docker `rede_interna`. O serviço web acessa o banco utilizando o hostname `db`.

A aplicação utiliza dois volumes nomeados:

* `dados_banco`: armazena os dados do PostgreSQL.
* `fotos`: armazena as fotos enviadas para a aplicação.

O acesso à aplicação é feito pela porta `8080` do computador, encaminhada para a porta `5000` do container web.

## 2. Estrutura do projeto

```text
achados-perdidos/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
├── db/
│   └── init.sql
├── docs/
│   └── evidencias/
├── docker-compose.yml
├── .env
├── .env.example
├── .gitignore
└── README.md
```

O arquivo `.env` contém as credenciais utilizadas pelos containers e não deve ser versionado.

## 3. Configuração

Copie o arquivo de exemplo:

```powershell
Copy-Item .env.example .env
```

Depois, confira os valores do arquivo `.env` e, se necessário, altere as credenciais.

Exemplo de estrutura:

```text
POSTGRES_DB=achados
POSTGRES_USER=usuario_exemplo
POSTGRES_PASSWORD=senha_exemplo

DB_HOST=db
DB_PORT=5432
DB_NAME=achados
DB_USER=usuario_exemplo
DB_PASSWORD=senha_exemplo
UPLOAD_DIR=/app/uploads
```

O arquivo `.env` está incluído no `.gitignore` para evitar que as credenciais sejam versionadas.

## 4. Executando o projeto

Na pasta raiz do projeto, execute:

```powershell
docker compose up -d --build
```

O comando cria as imagens, inicia os containers e configura a rede e os volumes necessários.

Para verificar o estado dos containers:

```powershell
docker compose ps
```

O esperado é que os serviços `db` e `web` estejam em execução, com o banco apresentando o status `healthy`.

## 5. Testando a aplicação

### Health check

```powershell
curl.exe http://localhost:8080/health
```

Resposta esperada:

```json
{
  "banco": "conectado",
  "container": "ID_DO_CONTAINER",
  "status": "ok"
}
```

### Listar itens

```powershell
curl.exe http://localhost:8080/itens
```

A aplicação retorna os itens cadastrados no banco.

### Filtrar por local

```powershell
curl.exe "http://localhost:8080/itens?local=biblio"
```

O filtro utiliza o local informado para pesquisar os itens cadastrados.

### Cadastrar item

Exemplo utilizando um arquivo de imagem:

```powershell
curl.exe -F "descricao=Oculos de grau" -F "local=Bloco B" -F "foto=@oculos.jpg" http://localhost:8080/itens
```

A aplicação retorna o ID criado e o nome gerado para a foto.

### Estatísticas

```powershell
curl.exe http://localhost:8080/estatisticas
```

A resposta apresenta a quantidade de itens no banco e a quantidade de fotos armazenadas no volume.

### Acessar uma foto

Após cadastrar um item com foto, utilize o nome retornado pela API:

```text
http://localhost:8080/fotos/NOME_DA_FOTO.jpg
```

## 6. Persistência dos dados

Os dados são armazenados em volumes Docker nomeados.

Para verificar os volumes:

```powershell
docker volume ls
```

Devem existir volumes semelhantes a:

```text
achados-perdidos_dados_banco
achados-perdidos_fotos
```

Para testar a persistência:

```powershell
docker compose down
```

Depois:

```powershell
docker compose up -d
```

Os itens cadastrados e as fotos continuam disponíveis, pois os volumes não são removidos pelo comando `docker compose down`.

## 7. Verificação do usuário do container

A aplicação web é executada com um usuário não-root.

Para verificar:

```powershell
docker compose exec web whoami
```

Resultado esperado:

```text
appuser
```

## 8. Logs

Para visualizar os logs da aplicação web:

```powershell
docker compose logs web --tail 20
```

Para visualizar os logs do banco:

```powershell
docker compose logs db --tail 20
```

## 9. Parando o projeto

Para parar e remover os containers e a rede:

```powershell
docker compose down
```

Os volumes não são removidos nesse comando.

Para remover também os volumes, utilize:

```powershell
docker compose down -v
```

**Atenção:** esse último comando remove os volumes e, consequentemente, os dados armazenados neles.

## 10. Tecnologias utilizadas

* Python 3.12
* Flask 3.0.3
* Gunicorn 23.0.0
* PostgreSQL 16 Alpine
* Psycopg 3.2.3
* Docker
* Docker Compose

## 11. Requisitos atendidos

* Inicialização da aplicação com Docker Compose.
* Aplicação Flask executada em container.
* PostgreSQL executado em container.
* Comunicação entre containers por rede Docker interna.
* Persistência do banco de dados utilizando volume nomeado.
* Persistência das fotos utilizando volume nomeado.
* Health check do PostgreSQL.
* Inicialização do serviço web somente após o banco estar saudável.
* Execução da aplicação web com usuário não-root.
* Credenciais configuradas por meio do arquivo `.env`.
* API para consulta, filtro e cadastro de itens.
* Upload e disponibilização de fotos.
* Endpoint de estatísticas.
