# Relatório Técnico — Achados & Perdidos

## (a) Dockerfile da aplicação web

O `app/Dockerfile` é responsável por criar a imagem da aplicação Flask.

A ordem das instruções foi escolhida para aproveitar o cache de camadas do Docker. Primeiro são copiadas as dependências e instaladas. Depois o código da aplicação é copiado. Assim, quando apenas o `app.py` é alterado, o Docker pode reutilizar a camada de instalação das dependências.

### Instruções utilizadas

```dockerfile
FROM python:3.12-slim
```

Utiliza a imagem oficial do Python 3.12 na variante `slim`, que possui uma estrutura menor e adequada para a aplicação.

```dockerfile
WORKDIR /app
```

Define `/app` como diretório de trabalho dentro do contêiner.

```dockerfile
COPY requirements.txt .
```

Copia o arquivo de dependências para dentro da imagem antes do código da aplicação.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

Instala Flask, psycopg e Gunicorn. A opção `--no-cache-dir` evita manter o cache do pip na imagem final.

```dockerfile
COPY app.py .
```

Copia o código da aplicação Flask para o diretório `/app`.

```dockerfile
RUN mkdir -p /app/uploads \
    && useradd -m appuser \
    && chown -R appuser:appuser /app
```

Cria o diretório utilizado para armazenar as fotos, cria o usuário `appuser` e transfere a propriedade dos arquivos e diretórios para esse usuário.

```dockerfile
USER appuser
```

Faz a aplicação executar como um usuário sem privilégios de root.

```dockerfile
EXPOSE 5000
```

Documenta que a aplicação utiliza a porta 5000 dentro do contêiner.

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

Inicia a aplicação utilizando Gunicorn, escutando em `0.0.0.0:5000`. O formato em lista permite que o comando seja executado diretamente pelo Docker.

A ordem `COPY requirements.txt → RUN pip install → COPY app.py` também melhora o aproveitamento do cache. Alterações somente no `app.py` não obrigam a reinstalação das dependências.

---

## (b) Docker Compose

O arquivo `docker-compose.yml` organiza os serviços, volumes e a rede do ambiente.

### Serviço `db`

O serviço `db` utiliza a imagem oficial:

```yaml
image: postgres:16-alpine
```

As configurações do PostgreSQL são obtidas pelas variáveis do arquivo `.env`:

```yaml
environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

Os dados do PostgreSQL são armazenados no volume nomeado `dados_banco`:

```yaml
volumes:
  - dados_banco:/var/lib/postgresql/data
```

O arquivo `db/init.sql` é montado como bind mount e somente leitura:

```yaml
- ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
```

Esse arquivo cria a tabela `itens` e insere os três objetos iniciais na primeira inicialização de um banco vazio.

O PostgreSQL possui um healthcheck usando `pg_isready`:

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 5s
  timeout: 5s
  retries: 5
```

O banco não possui nenhuma porta publicada no host. Ele fica acessível somente pela rede interna.

### Serviço `web`

O serviço `web` é construído a partir do diretório `./app`:

```yaml
build:
  context: ./app
```

As variáveis necessárias para a conexão com o PostgreSQL são fornecidas pelo `.env`.

As fotos são armazenadas no volume nomeado `fotos`:

```yaml
volumes:
  - fotos:/app/uploads
```

O serviço depende da condição de saúde do banco:

```yaml
depends_on:
  db:
    condition: service_healthy
```

Dessa forma, o Compose aguarda o banco estar pronto para aceitar conexões antes de iniciar o serviço web.

O `web` e o `db` utilizam a rede definida pelo projeto:

```yaml
networks:
  - rede_interna
```

O serviço `web` não publica diretamente a porta 5000 no host porque o projeto utiliza o Nginx como proxy reverso.

### Serviço `nginx`

O Nginx funciona como proxy reverso na frente da aplicação web.

Ele publica a porta 8080 do host:

```yaml
ports:
  - "8080:80"
```

As requisições recebidas pelo Nginx são encaminhadas para:

```text
http://web:5000
```

O Nginx também participa da `rede_interna`, permitindo localizar o serviço web pelo nome do serviço Docker.

### Serviço `adminer`

O Adminer foi adicionado como recurso opcional de depuração.

Ele utiliza:

```yaml
profiles: ["debug"]
```

Por isso, não é iniciado no modo normal. Para iniciar o painel:

```powershell
docker compose --profile debug up -d
```

O acesso ocorre pela porta `8081` do host.

### Volumes

O Compose define dois volumes nomeados:

```yaml
volumes:
  dados_banco:
  fotos:
```

O volume `dados_banco` é utilizado pelo PostgreSQL em:

```text
/var/lib/postgresql/data
```

O volume `fotos` é utilizado pela aplicação web em:

```text
/app/uploads
```

Os volumes são independentes dos contêineres. Dessa forma, a recriação dos contêineres não remove automaticamente os dados armazenados.

### Rede

A rede privada do projeto é:

```yaml
networks:
  rede_interna:
    driver: bridge
```

Os serviços utilizam os nomes dos serviços para comunicação interna.

Assim, a aplicação consegue acessar o PostgreSQL usando:

```text
db:5432
```

sem precisar utilizar o endereço IP do contêiner.

---

## (c) Caminho de uma requisição POST `/itens` com foto

O cadastro de um objeto com foto pode ser realizado pelo PowerShell utilizando `curl.exe`:

```powershell
curl.exe -F "descricao=Oculos de grau" -F "local=Bloco B" -F "foto=@oculos.jpg" http://localhost:8080/itens
```

### Caminho da requisição

O usuário executa o `curl.exe` no host Windows utilizando:

```text
http://localhost:8080/itens
```

A requisição chega ao Nginx pela porta `8080` publicada no host.

O Nginx recebe a requisição na porta `80` do contêiner e encaminha para `web:5000`.

O Gunicorn recebe a requisição e executa a rota `POST /itens` da aplicação Flask.

A aplicação lê os campos `descricao` e `local` e verifica se existe um arquivo no campo `foto`.

A aplicação gera um nome único para a foto utilizando `uuid`.

O arquivo é salvo no diretório:

```text
/app/uploads
```

Esse diretório está ligado ao volume nomeado:

```text
fotos:/app/uploads
```

Portanto, a foto é armazenada no volume `fotos`, e não somente na camada temporária do contêiner.

Depois de salvar a foto, a aplicação conecta ao PostgreSQL utilizando:

```text
db:5432
```

O registro é inserido na tabela `itens`, armazenando a descrição, o local e o nome do arquivo da foto.

A API retorna o ID criado e o nome da foto.

### Fluxo da requisição

```text
curl.exe
    |
    | HTTP :8080
    v
Nginx :80
    |
    | web:5000
    v
Flask + Gunicorn
    |
    +----------------------+
    |                      |
    v                      v
Volume fotos           PostgreSQL
/app/uploads           db:5432
                            |
                            v
                     volume dados_banco
```

---

## (d) Ciclo de vida dos volumes

Os volumes `dados_banco` e `fotos` são utilizados para manter os dados separados do ciclo de vida dos contêineres.

### `docker compose down` + `docker compose up -d`

Quando os contêineres são removidos com:

```powershell
docker compose down
```

os volumes nomeados não são removidos.

Ao executar novamente:

```powershell
docker compose up -d
```

os mesmos volumes são montados nos novos contêineres.

Com isso, os registros do PostgreSQL e as fotos continuam disponíveis.

A evidência da Tarefa 5 demonstra essa persistência por meio do endpoint `/estatisticas` e da listagem do diretório `/app/uploads`.

### `docker compose down -v`

Quando o comando é executado com `-v`:

```powershell
docker compose down -v
```

os volumes nomeados do projeto também são removidos.

Consequentemente:

- os registros armazenados no volume `dados_banco` são apagados;
- as fotos armazenadas no volume `fotos` são apagadas;
- ao subir o ambiente novamente, o PostgreSQL inicia com um volume vazio.

Como o banco está sendo inicializado novamente, o PostgreSQL executa o `db/init.sql`, criando a tabela `itens` e inserindo novamente os três objetos de exemplo.

Assim, os objetos cadastrados anteriormente e suas fotos são perdidos após `down -v`, enquanto os três registros definidos no `init.sql` aparecem novamente.

---

## Conclusão

O ambiente foi organizado com contêineres separados para a aplicação e o banco de dados, comunicação por uma rede interna e armazenamento persistente por volumes nomeados.

O Nginx foi utilizado como proxy reverso, deixando a aplicação web sem porta publicada diretamente no host. O PostgreSQL permanece isolado da máquina host, sendo acessível apenas pela rede interna.

A utilização dos volumes `dados_banco` e `fotos` permite que os dados sobrevivam à recriação dos contêineres, enquanto `docker compose down -v` remove explicitamente os dados persistidos.