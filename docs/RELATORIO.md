# Relatório Técnico — Achados & Perdidos

## (a) Dockerfile

O `app/Dockerfile` é responsável por criar a imagem da aplicação web.

A ordem das instruções foi escolhida para aproveitar o cache de camadas do Docker.

### Instruções utilizadas

```dockerfile
FROM python:3.12-slim