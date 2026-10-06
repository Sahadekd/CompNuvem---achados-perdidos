# Evidência 01 — Docker Compose

## Comando

```powershell
docker compose ps
```
![alt text](image.png)

---

# Evidência 02 — Health Check

## Comando

```powershell
curl.exe http://localhost:8080/health
```
![alt text](image-1.png)

---

# Evidência 03 — Consulta de Itens

## Comando

```powershell
curl.exe http://localhost:8080/itens
```
![alt text](image-2.png)

---

# Evidência 04 — Filtro por Local

## Comando

```powershell
curl.exe "http://localhost:8080/itens?local=biblio"
```
![alt text](image-3.png)

---

# Evidência 05 — Cadastro de Item com Foto

## Comando

```powershell
curl.exe -F "descricao=Oculos de grau" -F "local=Bloco B" -F "foto=@oculos.jpg" http://localhost:8080/itens
```
![alt text](image-4.png)

---

# Evidência 06 — Estatísticas

## Comando

```powershell
curl.exe http://localhost:8080/estatisticas
```
![alt text](image-5.png)

---

# Evidência 07 — Consulta da Foto

## Comando

```powershell
curl.exe -I http://localhost:8080/fotos/<nome-da-foto>
```
![alt text](image-6.png)

---

# Evidência 08 — Logs da Aplicação

## Comando

```powershell
docker compose logs web
```
![alt text](image-7.png)

---

# Evidência 09 — Persistência dos Dados

## Comandos

```powershell
docker compose down
docker compose up -d
curl.exe http://localhost:8080/itens
```
![alt text](image-8.png)

---

# Evidência 10 — Volumes Docker

## Comando

```powershell
docker volume ls
```
![alt text](image-9.png)

---

# Evidência 11 — Rede Docker

## Comando

```powershell
docker network inspect achados-perdidos_rede_interna
```
![alt text](image-10.png)

---

# Evidência 12 — Imagem Docker

## Comando

```powershell
docker image ls
```
![alt text](image-11.png)

---

# Evidência 13 — Inspeção dos Volumes

## Comandos

```powershell
docker volume inspect achados-perdidos_dados_banco
docker volume inspect achados-perdidos_fotos
```
# Evidência 14 — Conteúdo do Volume de Fotos

## Comando

```powershell
docker compose exec web ls -l /app/uploads
```
![alt text](image-12.png)

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