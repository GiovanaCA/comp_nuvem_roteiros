!!! info "Enunciado original"

    O enunciado completo do projeto se encontra na página da disciplina no link: [https://insper.github.io/computacao-nuvem/projetos_2025-1/projeto/](https://insper.github.io/computacao-nuvem/projetos_2025-1/projeto/){:target="_blank"}

O projeto trata de uma API RESTful que deve ser capaz de cadastrar e autenticar usuários, além de permitir a consulta de dados de terceiros. Após a construção da API, o projeto deve ser dockerizado e, então, implantado na AWS.

## Etapa 1

### Construção da API

A API dever ter no mínimo 3 endpoints:

??? note "Registro de Usuário"
    
    ``` http title="Endpoint"
    POST /registrar
    ```

    Request
    ``` json title="JSON payload"
    {
        "nome": "Disciplina Cloud",
        "email": "cloud@insper.edu.br",
        "senha": "cloud0"
    }
    ```

    Response
    ``` json title="JSON payload"
    {
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkRpc2NpcGxpbmEgQ2xvdWQiLCJpYXQiOjE1MTYyMzkwMjJ9.s76o9X4UIANSI-aTF8UhqnBYyIRWw_WH4ut8Xqmo6i0"
    }
    ```

??? note "Autenticação de Usuário"

    ``` http title="Endpoint"
    POST /login
    ```

    Request
    ``` json title="JSON payload"
    {
        "email": "cloud@insper.edu.br",
        "senha": "cloud0"
    }
    ```

    Response
    ``` json title="JSON payload"
    {
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
                eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkRpc2N
                pcGxpbmEgQ2xvdWQiLCJpYXQiOjE1MTYyMzkwMjJ9.
                s76o9X4UIANSI-aTF8UhqnBYyIRWw_WH4ut8Xqmo6i0"
    }
    ```

??? note "Aquisição dos Dados"

    ``` http title="Endpoint"
    GET /consultar
    
    ### HEADER
    Authorization: Bearer <JWT>
    ```

    - Response: A resposta pode ser qualquer scrap de uma página de terceiros, e o formato também pode ser qualquer um.

    - Os dados devolvidos pela consulta devem ser de uma página de terceiros, e devem ser atualizados frequentemente.

    - Caso o usuário não tenha um token válido, a API deve retornar um erro 403.

### Dockerinzing

Quando o código da API estiver pronto, ele deve ser dockerizado. Para isso, deve-se criar um arquivo `Dockerfile` e um `compose.yaml` para a execução da aplicação.

O docker compose deve conter pelo menos 2 serviços: a aplicação e o banco de dados. A aplicação deve ser capaz de se conectar ao banco de dados e realizar as operações de CRUD.

A aplicação deve ser autocontida, ou seja, deve ser possível executar a aplicação apenas com o comando `docker compose up` - pois isso é parte essencial da entrega.

Para organização:

``` tree title="estrutura de diretório sugerida"
api/
  Dockerfile
  app/
    app.py
  requirements.txt
  ...
compose.yaml
.env
```

### Publicação no Docker Hub

Após a dockerização, o projeto deve ser publicado no Docker Hub. O link do Docker Hub deve ser incluído na documentação do projeto.

A publicação no docker hub deve ser feita via linha de comando. E os comandos utilizados devem ser incluídos na documentação do projeto.

??? note "Variáveis de Ambiente"
    As credenciais do banco de dados e JWT devem ser passadas via variáveis de ambiente, por um arquivo `.env`. Todavia, **PARA FACILITAR A CORREÇÃO**, as credenciais podem ser passadas diretamente no `compose.yaml` por valores padrões, para que não tenha que haver um arquivo de variáveis de ambiente. Exemplo:

    ``` { .yaml title="compose.yaml" }
    name: app

        db:
            image: postgres:17
            environment:
                POSTGRES_DB: ${POSTGRES_DB:-projeto} # (1)!
                POSTGRES_USER: ${POSTGRES_USER:-projeto}
                POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-projeto}
            ports:
                - 5432:5432 #(2)!
    ```

    - Caso a variável de ambiente `POSTGRES_DB` não exista ou seja nula - não seja definida no arquivo `.env` - o valor padrão será `projeto`. Vide [documentação](https://docs.docker.com/reference/compose-file/interpolation/){target='_blank'}.

    - Aqui é feito um túnel da porta 5432 do container do banco de dados para a porta 5432 do host (no caso localhost). Em um ambiente de produção, essa porta não deve ser exposta, pois ninguém de fora do compose deveria acessar o banco de dados diretamente.

    ``` { .env title=".env" }
    POSTGRES_DB=meuprojeto
    POSTGRES_USER=meuprojeto
    POSTGRES_PASSWORD=S3cr3t
    ```

    Ao executar, o docker compose irá utilizar as variáveis de ambiente do arquivo `.env`, caso existam, senão, utilizará os valores padrões definidos já dentro do arquivo `compose.yaml`.


## Etapa 2

### AWS Lightsail

Após a conclusão da etapa 1 do projeto com a aplicação containerizada localmente utilizando Docker Compose.

Antes de iniciar, certifique-se de ter:

- Conta ativa na AWS com acesso ao Lightsail;
- Docker instalado e configurado;
- Código da aplicação FastAPI pronto e funcional localmente.

Os próximos passos são:

- Implantar a aplicação utilizando o AWS Lightsail Container Service;
- Configurar um banco de dados gerenciado no Lightsail;
- Conectar sua aplicação ao banco de dados;
- Gerenciar e monitorar o custo do serviço em produção.

??? info "Dicas"

    Para mais informações sobre o Lightsail visualizar o conteúdo adicional na página [lightsail](https://insper.github.io/computacao-nuvem/adicional/AWSLightsail/){:target:"_blank"} e para algumas dicas de configuração ver [dicas](https://insper.github.io/computacao-nuvem/projetos_2025-1/projeto/#na-aba-conteudo-adicional-do-site-da-disciplina-temos-explicacoes-sobre-o-ligthsail-utilizem-como-referencia){:target="_blank"}.

## Entregas

??? success "Entrega Etapa 1"
    A entrega deverá ser um link do projeto no GitHub, contendo o código da API e o Dockerfile, e deve haver uma documentação básica do projeto no MkDocs, contendo:

    - explicação do projeto - scrap do que foi feito;
    - explicação de como executar a aplicação;
    - documentação dos endpoints da API;
    - screenshot com os endpoints testados;
    - video de execução da aplicação - de até 1 minuto;
    - link para o docker hub do projeto;
    - referência explícita a localização do arquivo ```compose.yaml```;
    - o arquivo ```compose.yaml``` FINAL (entregue) deve utilizar apenas images do docker hub (inclusive as geradas para a api), ou seja, não deve ter build dentro dele.

??? success "Entrega Etapa 2"
    A entrega deverá ser um link do projeto no GitHub, contendo além do entregue antes (o código da API e o Dockerfile), e deve haver uma documentação básica do projeto no MkDocs, contendo:

    - explicação do projeto - scrap do que foi feito;
    - explicação de como executar a aplicação;
    - screenshot com os endpoints AWS testados;
    - screenshot da infraestrutura funcionando na AWS;
    - tela dos custos da conta no mesmo dia da submissão dos documentos;
    - video de execução da aplicação funcionando no Ligthsail - de até 1 minuto mostrando o acesso e a gravação de dados no banco de dados em Cloud;
    - para conceito B, na documentação dos custos deve ser projetado para: 1, 5 e 10 instancias de containers;

## Rubrica

??? danger "Rubrica"

    | Critério | Observações |
    |:----------|:------------|
    | FastAPI funcional com Docker Compose e banco local (PostgreSQL ou MySQL) | App sobe com `docker-compose up`, banco acessível pela aplicação |
    | Imagem publicada no Docker Hub e projeto organizado (`.env`, `.dockerignore`, estrutura clara) | Demonstra conhecimento mínimo em containerização |
    | Documentação mínima local (`README.md`) | Instruções para build e execução, com informações da aplicação |
    | Deploy funcional no AWS Lightsail Containers | App acessível publicamente via URL fornecida pela AWS |
    | Banco de dados gerenciado no Lightsail funcionando e conectado à aplicação  | Uso correto de variáveis de ambiente, sem `localhost` no backend |
    | Documentação da etapa na nuvem com instruções básicas de deploy | Link de acesso + descrição do processo de publicação |
    | Não estourar o **custo mensal** da infraestrutura que deve ser ≤ USD 50 | Custo estimado com base nos planos usados, infração detalhada abaixo. |
    | **Conceito C = Aprovado** se todas as partes funcionarem e forem documentadas| Projeto mínimo completo e compreensível com as entregas 1 e 2 feitas |
    | Entregar o Conceito C | ------------ |
    | Apresentar a **arquitetura final** do projeto em **diagrama** | Indicar os componentes: app, container, banco, rede, domínio |
    | Informar corretamente os **recursos alocados** (plano Lightsail, RAM, CPU, tipo do DB) | Pode ser um parágrafo ou print com descrição dos planos usados |
    | Estimar o **custo mensal** da infraestrutura (≤ USD 50) | Custo estimado com base nos planos usados, pode ser por texto ou print |
    | **Conceito B = Aluno demonstra clareza na arquitetura e planejamento do uso de nuvem** | A entrega vai além da execução, com compreensão de recursos e custos |
    | Entregar o Conceito B | ------------ |
    | Documentação detalhada (ex: imagens do Lightsail, MkDocs, explicação clara do fluxo) | Entrega cuidadosa e bem comunicada |
    | Explicação sobre a integração app ↔ banco (host, porta, segurança, variáveis) | Demonstra domínio técnico da arquitetura e da integração |
    | Banco de Dados instalado em Instancia no Ligthsail e conectado a aplicação | Demonstra domínio adicional sobre a arquitetura e produção |
    | **Conceito A = Entrega clara, comunicada, com domínio da solução em nuvem** | Mostra que o aluno sabe o que fez, como funciona e quanto custa |

## Docker compose

### Material de aula

``` { tree title="estrutura para dois containers num mesmo compose" }
api
  Dockerfile
web
  Dockerfile
  hello.txt
.env
compose.yaml
```

### Comandos

- Executando o docker compose:
```sh
docker compose up -d --build
```

- Parando o docker compose:
```sh
docker compose down
```