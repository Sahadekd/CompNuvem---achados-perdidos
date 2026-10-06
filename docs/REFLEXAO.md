# Reflexão e Equivalência em Nuvem — Achados & Perdidos

## Q1. Por que o serviço web consegue se conectar ao banco usando o nome `db` em vez de um endereço IP?

O serviço `web` consegue acessar o banco usando `db:5432` porque os dois contêineres estão conectados à rede Docker `rede_interna`.

O Docker fornece resolução de nomes entre os serviços da mesma rede, associando o nome do serviço ao endereço IP interno do contêiner.

Assim, a aplicação não precisa conhecer o IP do PostgreSQL, utilizando apenas `db` como nome do servidor.

Se os contêineres fossem executados na rede `bridge` padrão sem uma rede definida pelo usuário, essa resolução automática por nome de serviço não funcionaria da mesma forma, sendo necessário utilizar outras formas de comunicação.

No projeto, essa configuração está definida no `docker-compose.yml` pela rede `rede_interna`.

---

## Q2. Qual a diferença entre um volume nomeado e um bind mount?

Um volume nomeado é gerenciado pelo Docker e é utilizado no projeto para armazenar dados persistentes, como o banco e as fotos.

Um bind mount liga diretamente um arquivo ou diretório do host a um caminho dentro do contêiner.

O `init.sql` utiliza bind mount porque é um arquivo do próprio projeto que precisa ser disponibilizado ao PostgreSQL.

Já os dados do banco utilizam o volume nomeado `dados_banco`, pois precisam permanecer independentes do ciclo de vida dos contêineres.

O volume `fotos` também é nomeado para manter os arquivos enviados pela aplicação.

---

## Q3. Se o `init.sql` for alterado depois que o ambiente já subiu uma vez, a mudança é aplicada?

Não. O PostgreSQL executa os scripts presentes em `/docker-entrypoint-initdb.d/` durante a inicialização de um banco com diretório de dados vazio.

Depois que o volume `dados_banco` já possui um banco inicializado, executar `docker compose up -d` novamente não executa o `init.sql` outra vez.

Para alterar a estrutura sem perder os dados, deve ser aplicada uma alteração controlada no banco, como um comando SQL de `ALTER TABLE` ou uma migração.

Remover o volume com `docker compose down -v` faria o banco ser criado novamente, mas também apagaria os dados existentes.

---

## Q4. Por que `depends_on` sozinho não garante que a aplicação consiga se conectar ao banco?

O `depends_on` básico controla a ordem de inicialização dos contêineres, mas não significa necessariamente que o PostgreSQL já esteja pronto para aceitar conexões.

O banco pode estar com o contêiner iniciado enquanto ainda realiza sua inicialização interna.

Por isso, o projeto utiliza um `healthcheck` com `pg_isready`.

Além disso, o `depends_on` utiliza `condition: service_healthy`, fazendo o Compose aguardar o banco ser considerado saudável antes de iniciar o serviço `web`.

---

## Q5. Por que a ordem `COPY requirements.txt → RUN pip install → COPY app.py` influencia o tempo de build?

O Docker cria camadas durante o processo de construção da imagem e pode reutilizar camadas que não foram alteradas.

Como o `requirements.txt` é copiado e instalado antes do `app.py`, uma alteração somente no código da aplicação não invalida a camada de instalação das dependências.

Assim, o Docker pode reutilizar o resultado do `pip install` e executar novamente apenas as etapas necessárias.

Isso reduz o tempo de build quando mudanças são feitas somente no `app.py`.

---

## Q6. Por que a porta 5432 do banco não deve ser publicada no host?

A porta 5432 não precisa ser publicada porque somente os serviços da aplicação precisam acessar o PostgreSQL.

No projeto, o `web` acessa o banco internamente usando `db:5432` pela `rede_interna`.

Manter a porta sem publicação reduz a exposição do banco diretamente à máquina host e a outras conexões externas.

Como as fotos podem conter documentos pessoais, como RG ou carteira de estudante, também é importante considerar a proteção desses dados conforme a LGPD.

Como medidas adicionais, podem ser utilizados controle de acesso/autenticação adequada para os dados e restrições de acesso às fotos, além de criptografia dos dados armazenados e em trânsito quando aplicável.

---

## Q7. O que acontece ao executar `docker compose up -d --scale web=3`?

O Compose tenta criar três réplicas do serviço `web`, permitindo que várias instâncias da aplicação atendam às requisições.

Porém, para utilizar três réplicas de forma adequada, seria necessário um mecanismo de distribuição de requisições, como um proxy reverso configurado para balancear as três instâncias.

Também é necessário considerar o armazenamento compartilhado das fotos entre as réplicas.

O volume `fotos` precisa estar disponível de forma adequada para todas as instâncias; em um ambiente distribuído, seria mais apropriado utilizar um armazenamento compartilhado ou serviço de objetos.

Assim, simplesmente aumentar o número de contêineres não resolve sozinho o problema de armazenamento compartilhado.

---

# 5.1 Do ambiente local para a nuvem

O ambiente Docker desenvolvido localmente pode ser representado na nuvem por serviços gerenciados equivalentes. A escolha abaixo representa uma possível correspondência entre os componentes utilizados no projeto e serviços dos três principais provedores.

| Peça do ambiente local | Microsoft Azure | AWS | Google Cloud |
|---|---|---|---|
| Contêiner `web` | Azure Container Apps | Amazon ECS / AWS Fargate | Cloud Run |
| Contêiner `db` + volume `dados_banco` | Azure Database for PostgreSQL | Amazon RDS for PostgreSQL | Cloud SQL for PostgreSQL |
| Volume `fotos` | Azure Blob Storage | Amazon S3 | Google Cloud Storage |
| `rede_interna` | Azure Virtual Network | Amazon VPC | Google VPC |
| Arquivo `.env` (segredos) | Azure Key Vault | AWS Secrets Manager | Secret Manager |
| Imagem construída localmente | Azure Container Registry | Amazon ECR | Artifact Registry |

### Migração para a nuvem

Ao migrar o projeto para a nuvem, a parte que mais muda é o armazenamento e a infraestrutura que atualmente são gerenciados diretamente pelo Docker Compose.

O PostgreSQL pode deixar de ser um contêiner local e passar a utilizar um banco gerenciado pelo provedor.

As fotos também podem deixar o volume Docker e passar para um serviço de armazenamento de objetos, como Blob Storage, S3 ou Cloud Storage.

A aplicação web pode continuar sendo executada como contêiner, mas utilizando um serviço gerenciado de contêineres.

Com isso, o projeto deixa de depender de volumes e serviços executados somente na máquina local e passa a utilizar recursos gerenciados e escaláveis da nuvem.

---

# Conclusão

A atividade permitiu relacionar conceitos de contêineres, redes, persistência e segurança com uma arquitetura que pode ser posteriormente migrada para a nuvem.

A utilização de uma rede interna permite que os serviços se comuniquem sem expor o banco diretamente ao host.

Os volumes nomeados garantem a persistência dos dados enquanto os contêineres são recriados.

Já o uso do Nginx como proxy reverso permite separar o acesso externo da aplicação da comunicação interna entre os serviços.