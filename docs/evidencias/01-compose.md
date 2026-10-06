# Evidência 01 — Docker Compose

## Comando

```powershell
docker compose ps
```
![alt text](image.png)

**Legenda:** O Compose inicia os serviços do ambiente, com PostgreSQL saudável, Nginx publicado na porta 8080 e aplicação web em execução.

---

# Evidência 02 — Health Check

## Comando

```powershell
curl.exe http://localhost:8080/health
```
![alt text](image-1.png)

**Legenda:** O endpoint `/health` confirma que a aplicação está funcionando e conectada ao PostgreSQL.

---

# Evidência 03 — Consulta de Itens

## Comando

```powershell
curl.exe http://localhost:8080/itens
```
![alt text](image-2.png)

**Legenda:** O endpoint `/itens` lista os objetos cadastrados no banco de dados.

---

# Evidência 04 — Filtro por Local

## Comando

```powershell
curl.exe "http://localhost:8080/itens?local=biblio"
```
![alt text](image-3.png)

**Legenda:** O parâmetro `local=biblio` filtra corretamente os objetos cadastrados pelo local informado.

---

# Evidência 05 — Cadastro de Item com Foto

## Comando

```powershell
curl.exe -F "descricao=Oculos de grau" -F "local=Bloco B" -F "foto=@oculos.jpg" http://localhost:8080/itens
```
![alt text](image-4.png)

**Legenda:** O POST `/itens` cadastra um novo objeto e armazena a foto enviada na aplicação.

---

# Evidência 06 — Estatísticas

## Comando

```powershell
curl.exe http://localhost:8080/estatisticas
```
![alt text](image-5.png)

**Legenda:** O endpoint `/estatisticas` apresenta a quantidade de itens no banco e de fotos armazenadas no volume.

---

# Evidência 07 — Consulta da Foto

## Comando

```powershell
curl.exe -I http://localhost:8080/fotos/<nome-da-foto>
```
![alt text](image-6.png)

**Legenda:** A foto cadastrada é recuperada corretamente pelo endpoint `/fotos/<nome>`.

---

# Evidência 08 — Logs da Aplicação

## Comando

```powershell
docker compose logs web
```
![alt text](image-7.png)

**Legenda:** Os logs do serviço web confirmam a execução da aplicação Flask através do Gunicorn.    

---

# Evidência 09 — Persistência dos Dados

## Comandos

```powershell
docker compose down
docker compose up -d
curl.exe http://localhost:8080/itens
```
![alt text](image-8.png)

**Legenda:** Após remover e recriar os contêineres com `down` e `up`, os dados persistidos permanecem disponíveis nos volumes.

---

# Evidência 10 — Volumes Docker

## Comando

```powershell
docker volume ls
```
![alt text](image-9.png)

**Legenda:** A listagem confirma a existência dos volumes nomeados `dados_banco` e `fotos` utilizados pelo projeto.

---

# Evidência 11 — Rede Docker

## Comando

```powershell
docker network inspect achados-perdidos_rede_interna
```
![alt text](image-10.png)

**Legenda:** A inspeção confirma a rede `rede_interna` utilizada para a comunicação entre os serviços.

---

# Evidência 12 — Imagem Docker

## Comando

```powershell
docker image ls
```
![alt text](image-11.png)

**Legenda:** A listagem das imagens Docker mostra a imagem construída para a aplicação web e as imagens utilizadas pelos serviços.

---

# Evidência 13 — Inspeção dos Volumes

## Comandos

```powershell
docker volume inspect achados-perdidos_dados_banco
docker volume inspect achados-perdidos_fotos
```

**Legenda:** A inspeção confirma os pontos de montagem dos volumes persistentes do banco de dados e das fotos.

# Evidência 14 — Conteúdo do Volume de Fotos

## Comando

```powershell
docker compose exec web ls -l /app/uploads
```
![alt text](image-12.png)

**Legenda:** A listagem de `/app/uploads` confirma a presença do arquivo de foto armazenado no volume `fotos`.

---

# Evidência 15 — Remoção dos Volumes

## Estado antes da remoção

```powershell
curl.exe http://localhost:8080/estatisticas
```
![alt text](image-13.png)

## Remoção e recriação
```powershell
docker compose down -v
docker compose up -d
```
![alt text](image-14.png)

**Legenda:** Após `docker compose down -v`, os volumes são removidos e, ao subir novamente, o banco é recriado a partir dos três registros definidos no `init.sql`.