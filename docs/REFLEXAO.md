# Reflexão — Achados & Perdidos

## Q1 — Rede interna e resolução de nomes

O contêiner `web` consegue acessar o PostgreSQL usando o nome `db` porque os dois serviços estão conectados à rede Docker `rede_interna`.
O Docker Compose fornece a resolução de nomes entre os serviços da mesma rede.
Assim, `db` funciona como o endereço do contêiner PostgreSQL, sem precisar utilizar um IP fixo.
Se os contêineres utilizassem apenas a rede `bridge` padrão, essa resolução por nome entre os serviços não teria a mesma configuração automática do Compose.

---

## Q2 — Named volume x bind mount

O volume nomeado é gerenciado pelo Docker e é adequado para armazenar dados que precisam persistir, como os arquivos do PostgreSQL e as fotos.
Já o bind mount conecta diretamente um arquivo ou diretório do computador ao contêiner.
Por isso, o `init.sql` utiliza bind mount, pois é um arquivo do projeto que precisa ser fornecido ao PostgreSQL.
O volume `dados_banco` é usado para manter os dados do banco mesmo quando o contêiner é recriado.

---

## Q3 — Alteração do `init.sql`

Alterar o `init.sql` depois que o banco já foi inicializado não faz com que os comandos sejam executados novamente automaticamente.
Os scripts de inicialização do PostgreSQL são executados quando o diretório de dados é inicializado pela primeira vez.
Para aplicar novamente o `init.sql`, seria necessário remover o volume do banco e criar um novo ambiente.
Isso apaga os dados existentes, portanto essa opção deve ser utilizada somente quando essa perda for aceitável.

---

## Q4 — `depends_on` e `service_healthy`

O `depends_on` simples controla a ordem de inicialização dos serviços, mas não garante que o PostgreSQL esteja pronto para aceitar conexões.
O banco pode ter iniciado o contêiner, mas ainda estar realizando sua inicialização.
Neste projeto foi utilizado `condition: service_healthy`, junto com o `healthcheck` do PostgreSQL.
Assim, o serviço `web` aguarda o banco ser considerado saudável antes de iniciar.

---

## Q5 — Cache das camadas do Docker

A ordem do Dockerfile foi organizada para aproveitar o cache das camadas.
Primeiro o `requirements.txt` é copiado e as dependências são instaladas, e somente depois o `app.py` é copiado.
Dessa forma, se apenas o código do `app.py` mudar, o Docker pode reutilizar a camada de instalação das dependências.
Isso reduz o tempo necessário para reconstruir a imagem durante o desenvolvimento.

---

## Q6 — Porta 5432, segurança e LGPD

A porta 5432 não precisa ser publicada no host porque o `web` acessa o banco diretamente pela rede interna do Docker usando o serviço `db`.
Não publicar essa porta reduz a exposição do PostgreSQL para acessos externos ao computador.
Como as fotos podem conter documentos pessoais, essa configuração também reduz uma superfície de exposição dos dados.
Como medidas adicionais, podem ser aplicados controle de acesso/autenticação adequada e proteção/criptografia dos arquivos armazenados.
Também é importante limitar o acesso às fotos somente aos usuários e serviços que realmente precisam delas.

---

## Q7 — Escalabilidade do serviço `web`

Ao executar `docker compose up -d --scale web=3`, o Compose pode criar três réplicas do serviço `web`.
Para isso funcionar corretamente, o serviço não deve publicar diretamente a mesma porta do host para cada réplica.
O Nginx pode atuar como ponto de entrada e distribuir as requisições entre as réplicas.
As réplicas podem utilizar o mesmo volume `fotos`, desde que o armazenamento compartilhado suporte corretamente o acesso simultâneo aos arquivos.
Em um ambiente de nuvem, esse armazenamento normalmente seria substituído por uma solução de armazenamento compartilhado.

---

# 5.1 — Equivalência com serviços de nuvem

| Componente atual | Microsoft Azure | AWS | Google Cloud |
|---|---|---|---|
| Contêiner `web` | Azure Container Apps | Amazon ECS / Fargate | Cloud Run |
| Contêiner `db` + volume `dados_banco` | Azure Database for PostgreSQL | Amazon RDS for PostgreSQL | Cloud SQL for PostgreSQL |
| Volume `fotos` | Azure Blob Storage | Amazon S3 | Google Cloud Storage |
| `rede_interna` | Virtual Network (VNet) | Amazon VPC | Virtual Private Cloud (VPC) |
| Arquivo `.env` (segredos) | Azure Key Vault | AWS Secrets Manager | Secret Manager |
| Imagem construída localmente | Azure Container Registry | Amazon ECR | Artifact Registry |

No ambiente de nuvem, a aplicação poderia deixar de depender diretamente dos recursos locais do Docker. O contêiner `web` poderia ser executado em um serviço gerenciado, enquanto o PostgreSQL seria substituído por um banco gerenciado e as fotos por armazenamento de objetos. A maior mudança seria adaptar a aplicação para utilizar esses serviços gerenciados, principalmente o armazenamento das fotos e o gerenciamento de credenciais e segredos.
