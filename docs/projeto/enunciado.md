!!! info inline end

    PROJETO: **DUPLA**

    DEADLINE: **28.mai.2025**

!!! warning "Enunciado original"

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

    1.  Caso a variável de ambiente `POSTGRES_DB` não exista ou seja nula - não seja definida no arquivo `.env` - o valor padrão será `projeto`. Vide [documentação](https://docs.docker.com/reference/compose-file/interpolation/){target='_blank'}.

    2. Aqui é feito um túnel da porta 5432 do container do banco de dados para a porta 5432 do host (no caso localhost). Em um ambiente de produção, essa porta não deve ser exposta, pois ninguém de fora do compose deveria acessar o banco de dados diretamente.

    ``` { .env title=".env" }
    POSTGRES_DB=meuprojeto
    POSTGRES_USER=meuprojeto
    POSTGRES_PASSWORD=S3cr3t
    ```

    Ao executar, o docker compose irá utilizar as variáveis de ambiente do arquivo `.env`, caso existam, senão, utilizará os valores padrões definidos já dentro do arquivo `compose.yaml`.


## Etapa 2

### AWS

O próximo passo é implantar a aplicação na AWS.

## Entregas

??? success "Entrega Etapa 1"
    A entrega deverá ser um link do projeto no GitHub, contendo o código da API e o Dockerfile.

    Deve haver uma documentação básica do projeto no MkDocs, contendo:

    - explicação do projeto - scrap do que foi feito;
    - explicação de como executar a aplicação;
    - documentação dos endpoints da API;
    - screenshot com os endpoints testados;
    - video de execução da aplicação - de até 1 minuto;
    - link para o docker hub do projeto;
    - referência explícita a localização do arquivo `compose.yaml`;
    - o arquivo `compose.yaml` FINAL (entregue) deve utilizar apenas images do docker hub (inclusive as geradas para a api), ou seja, não deve ter `build` dentro dele.

??? success "Entrega Etapa 2"
    A entrega deverá ser um link do projeto no GitHub, o mesmo do anterior, mas para uma sessão sobre a publicação na AWS, contendo o uma breve explicação e um link para um vídeo, explicando e executando o trabalho entregue.

    O vídeo apresentado deve ter entre 2 e 3 minutos e **DEVE demonstrar TODOS** os seguintes itens:

    - logar na conta e acessar o projeto;
    - explicar o que foi feito e mostrar os componentes do projeto (eks, roles, etc);
    - executar o comando `kubectl get pods` e mostrar os pods rodando;
      ``` shell
      kubectl get pods
      ```
    - mostrar o projeto executando na AWS: chamada da API por um cliente (curl, postman, etc);

    No texto deve haver um link para os arquivos de configuração do Kubernetes (arquivos .yaml: deployment.yaml, service.yaml, etcs), repositório do projeto.

## Rubrica

??? danger "Rubrica"

    | Etapa | Critério | Nota | Observações |
    |:-:|---|:-:|:-:|
    | 1 | API + Dockerização<br> + Docker Hub + Documentação | C |  |
    | 2 | AWS | + 1 conceito | - 2 conceitos se não entregar a etapa do AWS |
    |   | AWS + Documentação | + 2 conceitos |


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