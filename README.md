# sample.node.jwt - API com autenticação por token JWT

* rotas [GET /help], [POST /login], [GET /lists], [GET /list/:id]
* rota [POST /login] usada para autenticar o usuário, retorna token JWT com dados do usuário
* rota [GET /help] retorn a lista das rotas implementadas 
* rota [GET /lists] valida o token e retorna lista completa 
* rota [GET /list/:id] valida o token e retorna item da lista pelo id
* modularizar os serviços para serem reutilizados em outros projetos

## 1. criar novo projeto, configurar e instalar as dependencias 

```
$ mkdir sample.node.jwt
$ cd sample.node.jwt
$ npm -y init
```

## 2. Associar o projeto a um repositorio externo

```
$ git init
```

### Nodemon

O Nodemon sera usado para reiniciar a aplicacao toda vez que o codigo fonte for alterado.
Vamos instalar como dependencia apenas para o desenvolvimento, ele nao sera incluido no pacote final de producao. 
```
$ npm install --save-dev nodemon
```
Vamos indicar no package.json que o nodemon sera executado atraves do comando ```$ npm start```.
```
{
  "scripts": {
    "start": "nodemon index.js"
  },
}
```

Podemos incluir mais arquivos ou pastas que serao ignorados pelo Nodemom relacionando-os no arquivo nodemon.json
```
{
  "ignoreRoot": ["diretorios", “ou”, “arquivos”, “ignorados”]
}
```

### Express

O [Express](https://expressjs.com) tera dupla funcao, sera usado para fazer o roteamento do app e para tratar as requisicoes API.
```
$ npm i --save express
```

### BodyParser

O [body-parser](http://expressjs.com/en/resources/middleware/body-parser.html) é um módulo capaz de converter o body da requisição em vários formatos, por exemplo, json.
```
$ npm i --save express
```

### JsonWebToken

[JsonWebToken](https://github.com/auth0/node-jsonwebtoken)
```
```

### run.bat

Vamos criar 2 arquivos *.bat* para executar o nosso app, um será usando durante o desenvolvimento pois executará o app através do *nodemon*. 
O outro será usado na produção que executará o app através do *node*. 

run-dev.bat
run-prd.bat

## 3. Server 

Implementar o servidor básico contendo as rotas / e /api

```
const express = require("express");
const app = express();
const PORT = 3000;
 
app.get("/", (req, res) => {
    res.send('Hello World!');
});

app.get('/api', (req, res) => {
    res.json({
        path: req.route.path,
        headers: req.headers
    });
});

app.listen(PORT, () => {
    console.log(`Running in http://localhost:${PORT}`);
})
```

## 4. Login

Implementar um serviço que autentica o usuário (login)

Criar um modulo de validação do usuário
services/user.js
```
exports.authenticate = function(username, password) {
  return new Promise((resolve, reject) => {
        if (username !== 'admin' || password !== '123456') {
            reject('Usuário ou senha inválidos!')
        } else {
            let userData = {
                userid: '2fd4e1c67a2d28fc',
                name: 'Administrador3',
                roles: ['admin'],
                features: ['r23', 'f34', 'f44']
            }    
            setTimeout(()=>resolve(userData), 1500)
    }
  })
}
```

Importar este módulo no app. Importar também o body-parser e adicioná-lo como middleware no express.
index.js
```
const user = require("./src/services/user")
const bodyParser = require("body-parser")
app.use(bodyParser.urlencoded({ extended: false }))
```

Criar a rota do login validando o usuario
```
app.post('/login', (req, res) => {
    const { username, password } = req.body
    user.authenticate(username, password)
        .then((result) => {
            console.log(result)
            res.send(result);
        })
        .catch((err) => {
            res.status(400).send({ error: err })
        })
})
```

---

## 5. JWT

### Gerar um certificado

Executar o script para gerar os arquivos com as chaves pública e privada. 
```
$ ./generateKeys.sh
```

A senha solicitada pelo comando pode ficar em branco. Os arquivos **_private.key_** e **_public.key_** serão gerados na pasta **_src_**.

Configurar .gitignore para ignorar a pasta com as chaves
```
# Auth keys files
src/*.pub
src/*.key
```

### Serviço de Autenticação

Implementar um serviço de autenticação que:
* gere um novo token
* valide um token existente


