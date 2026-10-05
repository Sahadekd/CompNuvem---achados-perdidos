# Achados & Perdidos

Aplicação web para cadastro e consulta de objetos perdidos, desenvolvida para a atividade de Computação em Nuvem.

O projeto utiliza Flask, Gunicorn, PostgreSQL, Docker Compose, Nginx, volumes persistentes, rede interna, variáveis de ambiente e contêineres Docker.

---

## 1. Objetivo

O sistema permite:

- cadastrar objetos encontrados;
- informar a descrição e o local onde o objeto foi encontrado;
- cadastrar uma foto opcional;
- consultar os objetos cadastrados;
- filtrar objetos por local;
- consultar estatísticas;
- acessar as fotos armazenadas.

A aplicação é executada em contêineres Docker e utiliza o PostgreSQL para armazenamento dos dados.

---

## 2. Arquitetura

A aplicação possui os seguintes componentes:

- **Nginx:** proxy reverso e única porta publicada no host;
- **Flask + Gunicorn:** aplicação web;
- **PostgreSQL 16:** banco de dados;
- **Adminer:** painel opcional para administração do banco;
- **Volume `dados_banco`:** persistência dos dados do PostgreSQL;
- **Volume `fotos`:** persistência das fotos enviadas;
- **Rede `rede_interna`:** comunicação entre os contêineres;
- **`db/init.sql`:** criação e inicialização da tabela `itens`.

Fluxo principal:

                    HOST / WINDOWS
                         |
                    HTTP :8080
                         |
                         v
              +---------------------+
              |    Nginx :80        |
              |   Proxy Reverso     |
              +----------+----------+
                         |
                    rede_interna
                         |
                         v
              +---------------------+
              | Flask + Gunicorn    |
              |      :5000          |
              +----------+----------+
                         |
                  +------+------+
                  |             |
                  v             v
           PostgreSQL       Volume fotos
             :5432          /app/uploads
                  |
                  v
           Volume dados_banco
        /var/lib/postgresql/data
---

### 3. Pré-requisitos
Para executar o projeto, é necessário possuir:

- **Docker Desktop**;
- **Docker Compose**;
Git, caso o projeto seja clonado do GitHub.

### 4. Configuração
- Após clonar o projeto, crie o arquivo .env a partir do arquivo de exemplo:
- Copy-Item .env.example .env
- Depois, configure no arquivo .env os valores das credenciais do PostgreSQL.

O arquivo .env não deve ser enviado ao GitHub.

### 5. Executar o projeto
Na pasta raiz do projeto, execute:

- **docker compose up -d --build**;

Para verificar os contêineres:

- **docker compose ps**;

O banco de dados deve aparecer com o status healthy.

### 6. Testar a aplicação
Health Check
Para verificar se a aplicação está funcionando e se existe conexão com o banco:

- **curl.exe http://localhost:8080/health**;

Deve retornar informações indicando que a aplicação está funcionando e que o banco está conectado.

Listar itens
- **curl.exe http://localhost:8080/itens**;

Filtrar por local
- **curl.exe "http://localhost:8080/itens?local=biblio"**;

Cadastrar item com foto
- **curl.exe -F "descricao=Oculos de grau" -F "local=Bloco B" -F "foto=@oculos.jpg" http://localhost:8080/itens**;

Consultar estatísticas
- **curl.exe http://localhost:8080/estatisticas**;

Consultar uma foto
- **curl.exe http://localhost:8080/itens http://localhost:8080/fotos/<nome-da-foto>**;

### 7. Rotas da API
- Método	Rota	Função
- GET	/health	Verifica a aplicação e o banco
- GET	/itens	Lista os objetos cadastrados
- GET	/itens?local=biblio	Filtra os objetos por local
- POST	/itens	Cadastra um objeto e, opcionalmente, uma foto
- GET	/fotos/<nome>	Retorna uma foto armazenada
- GET	/estatisticas	Exibe estatísticas da aplicação

### 8. Persistência
Os dados são armazenados em volumes Docker:
- **dados_banco: /var/lib/postgresql/data**;
- **fotos: /app/uploads.**;

Para testar a persistência:
- **docker compose down**;
- **docker compose up -d**;

Os itens e fotos continuam disponíveis após a recriação dos contêineres.

Para remover também os volumes:
- **docker compose down -v**;

Atenção: o comando down -v remove os dados persistidos nos volumes.

### 9. Verificações
Verificar usuário da aplicação
- **docker compose exec web whoami**;

Resultado esperado:
- **appuser**;

Verificar volumes
- **docker volume ls**;

Verificar rede
- **docker network inspect achados-perdidos_rede_interna**;

### 10. Adminer
O Adminer é opcional e utiliza o profile debug.

Para iniciar:
- **docker compose --profile debug up -d**;

Acesse:
- **http://localhost:8081**;

Para conectar ao banco, utilize:
- **Servidor: db**;

As demais credenciais devem ser preenchidas conforme os valores configurados no arquivo .env.

### 11. Backup
Foi realizado um backup do volume de fotos em:
- **docs/backup/fotos-backup.tar.gz**;

Também foi testada a restauração do backup em um volume separado.

### 12. Documentação
Os demais detalhes da atividade estão disponíveis em:

docs/
├── RELATORIO.md
├── REFLEXAO.md
└── evidencias/