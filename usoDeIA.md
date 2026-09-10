# 10/09
- Descricao do modelo para que seja feito o documento de entrega com o modelo de api, com o prompt:
```text
Estamos fazendo uma aplicacao que é uma plataforma de divulgacao de tutoriais de trabalhos manuais, necessito de um documento MD explicando a ideia, o modelo de dados, as rotas da api e o fluxo da aplicacao
A ideia: As pessoas vao postar tutoriais sobre coisas manuais, como crochet, pinturas e etc, e elas também vão poder fazer o tutorial de outra pessoa e postar uma foto do resultado no tutorial da outra. a ideia ainda não tem um nome, então o time de desenvolvimento está chamando de projeto x
Modelo de dados: serão três tabelas: Usuario, Tutorial e Foto:
Usuario {
  username: string (maxL: 10, somente alfanumericos _ e .)
  realname: string (maxL: 100)
  photo: image
  biography: string (maxL: 1000)
  senha: hash
}
Tutorial {
  usuarioID
  title: string(maxL: 100)
  content: string
}
Foto {
  tutorialID,
  usuarioID,
  picture: image,
  desc: string (maxL: 1000)
}
Rotas (- para as q necessitam de auth):
GET /login
GET /registrar
GET /tutoriais 
POST /tutoriais -
GET /fotos/:usuarioID
POST /fotos/:tutorialID -
GET /usuarios
PATCH /usuarios - 
Fluxo
vai ter 3 abas: pagina inicial, pesquisar e eu. as paginas de usuarios, tutoriais e fotos serão alcancadas pela navegacao. a pagina inicial vai mostrar tutoriais, ao cliclar no tutorial, voce pode ler ele todo, acessar o usuario que postou ou as fotos de quem seguiu o tutorial. na pagina de usuario pode se ver seus tutoriais e fotos. a pagina de pesquisa permite buscar usuarios pelo username e tutoriais pelo titulo. a pagina eu vai conter o perfil do usario logado ou uma pagina de login/cadastro caso nao haja login.
```