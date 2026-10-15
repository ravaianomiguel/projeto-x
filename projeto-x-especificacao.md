# Projeto X — Plataforma de Divulgação de Tutoriais Manuais

> Nome provisório usado pelo time de desenvolvimento até a definição do nome final.

## 1. Visão Geral

Projeto X é uma plataforma onde usuários publicam tutoriais de trabalhos manuais (crochê, pintura, marcenaria, etc.). Qualquer usuário pode seguir o tutorial de outra pessoa e, como comprovação, publicar uma ou mais fotos do resultado dentro do próprio tutorial original. Isso cria um ciclo de inspiração: um tutorial não é só uma postagem estática, mas acumula um histórico de interações de outros usuários que o seguiram.

**Conceitos-chave:**
- **Usuário**: quem publica tutoriais e/ou interações.
- **Tutorial**: uma postagem de instrução, escrita por um usuário.
- **Interação**: o registro de um usuário que seguiu um tutorial (pode ser de um usuário diferente do autor do tutorial), com uma descrição do resultado.
- **Foto**: uma das imagens do resultado, vinculada a uma interação (uma interação pode ter várias fotos).

---

## 2. Modelo de Dados

### 2.1 Entidade `Usuario`

| Campo       | Tipo    | Restrições                                      |
|-------------|---------|--------------------------------------------------|
| id          | int/uuid | chave primária                                   |
| username    | string  | máx. 30 caracteres, apenas alfanuméricos, `_` e `.`, único |
| realname    | string  | máx. 100 caracteres                              |
| photo       | image   | foto de perfil                                   |
| biography   | string  | máx. 1000 caracteres                             |
| senha       | hash    | armazenada como hash (nunca em texto puro)       |

### 2.2 Entidade `Tutorial`

| Campo      | Tipo    | Restrições                          |
|------------|---------|---------------------------------------|
| id         | int/uuid | chave primária                       |
| usuarioID  | FK → Usuario | autor do tutorial                |
| title      | string  | máx. 100 caracteres                  |
| content    | string  | conteúdo/instruções do tutorial      |

### 2.3 Entidade `Interacao`

Representa um usuário que seguiu um tutorial e registrou o resultado.

| Campo       | Tipo    | Restrições                                  |
|-------------|---------|-----------------------------------------------|
| id          | int/uuid | chave primária                               |
| tutorialID  | FK → Tutorial | tutorial que foi seguido                 |
| usuarioID   | FK → Usuario  | autor da interação (quem seguiu o tutorial) |
| desc        | string  | máx. 1000 caracteres                         |

### 2.4 Entidade `Foto`

Uma imagem do resultado, vinculada a uma interação. Uma interação pode ter N fotos.

| Campo        | Tipo    | Restrições                          |
|--------------|---------|---------------------------------------|
| id           | int/uuid | chave primária                       |
| interacaoID  | FK → Interacao | interação à qual a foto pertence |
| picture      | image   | imagem do resultado                  |

### 2.5 Relacionamentos

```
Usuario 1 ──── N Tutorial
Usuario 1 ──── N Interacao
Tutorial 1 ──── N Interacao
Interacao 1 ──── N Foto
```

- Um **Usuario** pode ter vários **Tutoriais**.
- Um **Usuario** pode ter várias **Interações** (mesmo em tutoriais de outros autores).
- Um **Tutorial** pode ter várias **Interações**, feitas por diferentes usuários.
- Uma **Interação** pode ter várias **Fotos** do resultado.

> **Observação:** o autor do tutorial e o autor da interação podem ser a mesma pessoa (ex: registrar o próprio resultado) ou pessoas diferentes (o caso principal de uso da plataforma).

### 2.6 Diagrama

```mermaid
erDiagram
    USUARIO ||--o{ TUTORIAL : publica
    USUARIO ||--o{ INTERACAO : realiza
    TUTORIAL ||--o{ INTERACAO : recebe
    INTERACAO ||--o{ FOTO : contem

    USUARIO {
        int id PK
        string username
        string realname
        image photo
        string biography
        hash senha
    }

    TUTORIAL {
        int id PK
        int usuarioID FK
        string title
        string content
    }

    INTERACAO {
        int id PK
        int tutorialID FK
        int usuarioID FK
        string desc
    }

    FOTO {
        int id PK
        int interacaoID FK
        image picture
    }
```

---

## 3. Rotas da API

Todas as rotas são de API pura (JSON — sem renderização de páginas). `-` indica rota que exige autenticação (usuário logado).

| Método | Rota                                  | Auth | Descrição                                                          |
|--------|-----------------------------------------|:----:|------------------------------------------------------------------------|
| POST   | `/login`                                |      | Autentica o usuário (cria sessão/token)                               |
| POST   | `/registrar`                            |      | Cria uma nova conta de usuário                                        |
| GET    | `/tutoriais`                            |      | Lista os tutoriais (feed da página inicial); aceita `?search=` por título |
| POST   | `/tutoriais`                            |  ✔   | Cria um novo tutorial (usuário logado é o autor)                      |
| GET    | `/tutoriais/:tutorialID`                |      | Retorna um único tutorial (mesmo formato do item da listagem)         |
| DELETE | `/tutoriais/:tutorialID`                |  ✔   | Remove um tutorial (apenas o autor)                                   |
| GET    | `/interacoes/:usuarioID`                |      | Lista as interações feitas por um usuário específico                  |
| GET    | `/interacoes/tutorial/:tutorialID`      |      | Lista as interações (resultados) de um tutorial específico            |
| POST   | `/interacoes/:tutorialID`               |  ✔   | Cria uma interação vinculada a um tutorial, com N fotos do resultado   |
| DELETE | `/interacoes/:interacaoID`              |  ✔   | Remove uma interação (apenas o autor)                                 |
| GET    | `/usuarios`                             |      | Lista/busca usuários; aceita `?search=` por username                  |
| GET    | `/usuarios/:usuarioID`                  |      | Retorna um único usuário (mesmo formato do item da listagem)          |
| PATCH  | `/usuarios`                             |  ✔   | Edita os dados do próprio usuário logado                              |

> **Observação:** como uma interação pode ter várias fotos, `POST /interacoes/:tutorialID` recebe a `desc` e as N imagens em uma única requisição (ex: multipart/form-data), criando a `Interacao` e suas `Foto`s associadas de uma vez.

---

## 4. Fluxo da Aplicação

A aplicação é organizada em **3 abas principais** de navegação:

1. **Página Inicial**
2. **Pesquisar**
3. **Eu**

As páginas de **usuário**, **tutorial** e **interação** não são abas — são alcançadas por navegação a partir dessas 3 abas (ex: ao clicar em um card).

### 4.1 Página Inicial
- Mostra o feed de **tutoriais** (`GET /tutoriais`).
- Ao clicar em um tutorial, o usuário acessa a **página do tutorial**, onde pode:
  - Ler o conteúdo completo do tutorial.
  - Acessar o **perfil do usuário** que postou o tutorial.
  - Ver as **interações** (com suas fotos) de quem já seguiu aquele tutorial.

### 4.2 Página de Usuário
- Acessível a partir de um tutorial (autor) ou de uma interação (autor) ou da busca.
- Mostra:
  - Dados do perfil (nome, foto, biografia).
  - Lista de **tutoriais** publicados por esse usuário.
  - Lista de **interações** feitas por esse usuário (resultados de tutoriais que seguiu).

### 4.3 Pesquisar
- Permite buscar:
  - **Usuários** pelo `username`.
  - **Tutoriais** pelo `title`.
- Resultados levam às respectivas páginas de detalhe.

### 4.4 Eu
- Se **logado**: mostra o perfil do próprio usuário (mesma estrutura da página de usuário, mas com opções de edição — chama `PATCH /usuarios`).
- Se **não logado**: mostra a tela de **login** / **cadastro**, que envia os dados para `POST /login` / `POST /registrar` respectivamente.

### 4.5 Diagrama de navegação simplificado

```
[Página Inicial] --clica em tutorial--> [Tutorial]
                                            |--> [Usuário (autor)]
                                            |--> [Interações do tutorial (com fotos)]

[Pesquisar] --busca usuário--> [Usuário]
            --busca tutorial--> [Tutorial]

[Eu] --logado--> [Meu Perfil] --editar--> (PATCH /usuarios)
     --não logado--> [Login] / [Registrar]
```

---

## 5. Resumo do Escopo

| Funcionalidade                            | Rota                                  |
|--------------------------------------------|------------------------------------------|
| Cadastro de usuário                        | `POST /registrar`                       |
| Login                                      | `POST /login`                           |
| Publicar tutorial                          | `POST /tutoriais`                       |
| Listar tutoriais / buscar por título       | `GET /tutoriais`                        |
| Ler um tutorial específico                 | `GET /tutoriais/:tutorialID`            |
| Remover tutorial                           | `DELETE /tutoriais/:tutorialID`         |
| Registrar interação (com N fotos) em um tutorial | `POST /interacoes/:tutorialID`    |
| Ver interações de um usuário               | `GET /interacoes/:usuarioID`            |
| Ver interações de um tutorial               | `GET /interacoes/tutorial/:tutorialID`  |
| Remover interação                          | `DELETE /interacoes/:interacaoID`       |
| Listar usuários / buscar por username      | `GET /usuarios`                         |
| Ver perfil de um usuário específico        | `GET /usuarios/:usuarioID`              |
| Editar perfil                              | `PATCH /usuarios`                       |

---

## 6. Wireframes

Representação básica das telas da aplicação, sem detalhamento visual — apenas a estrutura geral dos elementos.

### 6.1 Abas principais

Telas acessadas diretamente pela navegação por abas: Página Inicial, Pesquisar e Eu.

![Abas principais](res/abas.png)

### 6.2 Outras telas

Telas acessadas por navegação a partir das abas: Tutorial, Perfil de usuário, Interações do tutorial e Login/Cadastro.

![Outras telas](res/outras.png)
