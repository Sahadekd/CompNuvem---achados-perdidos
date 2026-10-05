# Achados & Perdidos

Aplicação web para cadastro e consulta de objetos perdidos, utilizando Flask, PostgreSQL e Docker Compose.

O projeto foi desenvolvido para a atividade de Computação em Nuvem, utilizando contêineres, rede interna, volumes persistentes, variáveis de ambiente e proxy reverso.

## 1. Arquitetura

O ambiente possui:

- Flask + Gunicorn: aplicação web
- PostgreSQL 16: banco de dados
- Nginx: proxy reverso
- Adminer: painel opcional para administração do banco
- Volume `dados_banco`: persistência do PostgreSQL
- Volume `fotos`: persistência das fotos
- Rede `rede_interna`: comunicação entre os contêineres

Fluxo principal:

```text
Navegador / curl
       |
       | HTTP :8080
       v
+-----------------------+
|       Nginx :80       |
|    Proxy Reverso      |
+----------+------------+
           |
           | HTTP
           v
+-----------------------+
|    Web Flask :5000    |
|       Gunicorn        |
+----------+------------+
           |
      +----+----+
      |         |
      v         v
 PostgreSQL   Volume fotos
    :5432     /app/uploads
      |
      v
Volume dados_banco
/var/lib/postgresql/data