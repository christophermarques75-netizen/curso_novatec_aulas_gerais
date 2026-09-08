# Guia passo a passo --- Login com Google OAuth + JWT

## Objetivo

Este guia mostra como criar uma aplicação React com login usando Google
OAuth e um back-end Node.js/Express responsável por validar o token do
Google e gerar um JWT próprio.

> **Observação importante:** o roteiro original mistura alguns fluxos
> diferentes de OAuth. Este guia usa `@react-oauth/google` no React e
> `google-auth-library` no back-end, que combina melhor com a validação
> por `verifyIdToken()`.

------------------------------------------------------------------------

# 1. Pré-requisitos

Tenha instalado:

-   Node.js
-   npm
-   Visual Studio Code
-   Uma conta Google

Para verificar o Node.js:

``` powershell
node -v
```

Para verificar o npm:

``` powershell
npm -v
```

------------------------------------------------------------------------

# 2. Criar o projeto React

Abra o PowerShell e execute:

``` powershell
cd C:\Users\SEU_USUARIO\Desktop\aulanpx
```

Crie o projeto:

``` powershell
npx create-react-app oauth-google-login
```

Entre na pasta:

``` powershell
cd oauth-google-login
```

------------------------------------------------------------------------

# 3. Instalar as bibliotecas do React

Dentro da pasta `oauth-google-login`, execute:

``` powershell
npm install @react-oauth/google axios jwt-decode
```

Se aparecer algo como:

``` text
up to date
```

ou

``` text
added packages
```

a instalação foi concluída.

Avisos de `npm audit` não significam necessariamente que a instalação
falhou.

------------------------------------------------------------------------

# 4. Criar o projeto no Google Cloud

Acesse:

https://console.cloud.google.com/

1.  Faça login com sua conta Google.
2.  Abra o seletor de projetos.
3.  Clique em **Novo projeto**.
4.  Escolha um nome, por exemplo: `Projeto Autenticacao`
5.  Clique em **Criar**.
6.  Selecione o projeto criado.

------------------------------------------------------------------------

# 5. Configurar a tela de consentimento OAuth

No Google Cloud Console:

1.  Procure a área de autenticação/OAuth.
2.  Configure a tela de consentimento.
3.  Informe o nome do aplicativo.
4.  Informe um e-mail de contato.
5.  Se for solicitado, adicione os dados necessários.
6.  Salve a configuração.

A interface do Google Cloud pode mudar de nome ou posição ao longo do
tempo. Se uma opção não estiver exatamente com o mesmo nome, procure por
**OAuth**, **Google Auth Platform** ou **Credenciais**.

------------------------------------------------------------------------

# 6. Criar o Client ID

Procure por:

**APIs e serviços → Credenciais**

ou pela área equivalente de autenticação do Google Cloud.

Crie uma credencial do tipo:

**ID do cliente OAuth**

Escolha:

**Aplicativo da Web**

Configure a origem autorizada para desenvolvimento:

``` text
http://localhost:3000
```

Se o console solicitar um URI de redirecionamento autorizado, utilize:

``` text
http://localhost:3000
```

Finalize a criação.

------------------------------------------------------------------------

# 7. Copiar o Client ID

O Google fornecerá algo parecido com:

``` text
123456789012-abc123xyz.apps.googleusercontent.com
```

Esse é o **Client ID**.

Você poderá utilizá-lo no front-end.

## NÃO faça isso

Não coloque o **Client Secret** no React.

O Client Secret deve permanecer protegido no servidor.

------------------------------------------------------------------------

# 8. Abrir o projeto no VS Code

Dentro da pasta do projeto:

``` powershell
code .
```

Se o comando `code` não funcionar, abra o VS Code manualmente e escolha:

**File → Open Folder**

Depois selecione:

``` text
oauth-google-login
```

------------------------------------------------------------------------

# 9. Configurar o App.js

No Explorer do VS Code, abra:

``` text
src
└── App.js
```

Apague o conteúdo de `App.js`.

Cole:

``` javascript
import React, { useState } from "react";
import { GoogleOAuthProvider, GoogleLogin } from "@react-oauth/google";
import { jwtDecode } from "jwt-decode";
import axios from "axios";

function App() {
  const [user, setUser] = useState(null);

  const handleGoogleLogin = async (credentialResponse) => {
    try {
      const googleToken = credentialResponse.credential;

      // Decodifica o token para visualizar os dados do usuário.
      // A validação de segurança é feita pelo back-end.
      const decoded = jwtDecode(googleToken);

      console.log("Usuário Google:", decoded);

      const response = await axios.post(
        "http://localhost:5000/api/auth/google",
        {
          token: googleToken,
        }
      );

      console.log("Resposta do servidor:", response.data);

      localStorage.setItem("jwtToken", response.data.jwtToken);

      setUser(decoded);
    } catch (error) {
      console.error("Erro no login:", error);
    }
  };

  const handleLogout = () => {
    localStorage.removeItem("jwtToken");
    setUser(null);
  };

  return (
    <div style={{ textAlign: "center", marginTop: "100px" }}>
      <h1>Login com Google</h1>

      {user ? (
        <div>
          <h2>Bem-vindo, {user.name}!</h2>
          <p>Email: {user.email}</p>

          <button onClick={handleLogout}>
            Logout
          </button>
        </div>
      ) : (
        <GoogleLogin
          onSuccess={handleGoogleLogin}
          onError={() => {
            console.log("Falha no login");
          }}
        />
      )}
    </div>
  );
}

export default function AppWithGoogle() {
  return (
    <GoogleOAuthProvider clientId="SEU_CLIENT_ID_AQUI">
      <App />
    </GoogleOAuthProvider>
  );
}
```

------------------------------------------------------------------------

# 10. Colocar o Client ID no App.js

Procure:

``` javascript
<GoogleOAuthProvider clientId="SEU_CLIENT_ID_AQUI">
```

Substitua:

``` text
SEU_CLIENT_ID_AQUI
```

pelo Client ID real.

Exemplo:

``` javascript
<GoogleOAuthProvider clientId="123456789012-abc123xyz.apps.googleusercontent.com">
```

Salve:

``` text
Ctrl + S
```

------------------------------------------------------------------------

# 11. Criar o back-end

Não coloque o `server.js` dentro de `src`.

A estrutura correta será:

``` text
oauth-google-login/
│
├── node_modules/
├── public/
├── src/
│   └── App.js
│
├── backend/
│   └── server.js
│
├── package.json
└── package-lock.json
```

No VS Code:

1.  Clique com o botão direito em `oauth-google-login`.
2.  Escolha **New Folder**.
3.  Nomeie como:

``` text
backend
```

4.  Dentro de `backend`, crie:

``` text
server.js
```

------------------------------------------------------------------------

# 12. Configurar o back-end

Abra:

``` text
backend/server.js
```

Cole:

``` javascript
const express = require("express");
const cors = require("cors");
const jwt = require("jsonwebtoken");
const { OAuth2Client } = require("google-auth-library");

const app = express();

app.use(cors());
app.use(express.json());

// Use o mesmo Client ID configurado no React.
const clientId = "SEU_CLIENT_ID_AQUI";

const googleClient = new OAuth2Client(clientId);

app.post("/api/auth/google", async (req, res) => {
  const { token } = req.body;

  try {
    const ticket = await googleClient.verifyIdToken({
      idToken: token,
      audience: clientId,
    });

    const payload = ticket.getPayload();

    const userId = payload.sub;
    const email = payload.email;
    const name = payload.name;

    const jwtToken = jwt.sign(
      {
        userId,
        email,
        name,
      },
      "chave_secreta_do_servidor",
      {
        expiresIn: "1h",
      }
    );

    res.status(200).json({
      jwtToken,
    });
  } catch (error) {
    console.error("Erro ao validar token:", error);

    res.status(400).json({
      error: "Erro na autenticação",
    });
  }
});

app.listen(5000, () => {
  console.log("Servidor rodando em http://localhost:5000");
});
```

------------------------------------------------------------------------

# 13. Colocar o Client ID no server.js

Procure:

``` javascript
const clientId = "SEU_CLIENT_ID_AQUI";
```

Substitua pelo mesmo Client ID usado no React.

Exemplo:

``` javascript
const clientId =
  "123456789012-abc123xyz.apps.googleusercontent.com";
```

O Client ID deve ser o mesmo nos dois lados.

------------------------------------------------------------------------

# 14. Instalar as bibliotecas do back-end

Abra um segundo terminal.

Entre na pasta do projeto:

``` powershell
cd C:\Users\SEU_USUARIO\Desktop\aulanpx\oauth-google-login
```

Entre no back-end:

``` powershell
cd backend
```

Inicialize o projeto Node:

``` powershell
npm init -y
```

Instale as dependências:

``` powershell
npm install express cors jsonwebtoken google-auth-library
```

------------------------------------------------------------------------

# 15. Iniciar o back-end

Ainda dentro de:

``` text
oauth-google-login\backend
```

execute:

``` powershell
node server.js
```

O resultado esperado:

``` text
Servidor rodando em http://localhost:5000
```

Não feche esse terminal.

------------------------------------------------------------------------

# 16. Iniciar o React

Abra outro terminal.

Entre na pasta principal:

``` powershell
cd C:\Users\SEU_USUARIO\Desktop\aulanpx\oauth-google-login
```

Execute:

``` powershell
npm start
```

O React abrirá em:

``` text
http://localhost:3000
```

------------------------------------------------------------------------

# 17. Os dois terminais

Você terá dois processos funcionando.

## Terminal 1 --- React

``` powershell
cd C:\Users\SEU_USUARIO\Desktop\aulanpx\oauth-google-login
npm start
```

Endereço:

``` text
http://localhost:3000
```

## Terminal 2 --- Node/Express

``` powershell
cd C:\Users\SEU_USUARIO\Desktop\aulanpx\oauth-google-login\backend
node server.js
```

Endereço:

``` text
http://localhost:5000
```

------------------------------------------------------------------------

# 18. Testar o login

Abra:

``` text
http://localhost:3000
```

Você deverá ver:

``` text
Login com Google
```

e o botão de login do Google.

Clique no botão.

Escolha uma conta Google.

Depois da autenticação, o React deverá receber o ID Token e enviá-lo
para:

``` text
http://localhost:5000/api/auth/google
```

O back-end valida o token.

Depois cria um JWT próprio.

O JWT é devolvido ao React.

------------------------------------------------------------------------

# 19. Fluxo completo

O funcionamento será:

``` text
┌─────────────────┐
│      Google     │
└────────┬────────┘
         │
         │ login
         ▼
┌─────────────────┐
│   React :3000   │
└────────┬────────┘
         │
         │ ID Token
         ▼
┌─────────────────┐
│ Node/Express    │
│     :5000       │
└────────┬────────┘
         │
         │ valida token
         ▼
┌─────────────────┐
│      Google     │
└────────┬────────┘
         │
         │ token válido
         ▼
┌─────────────────┐
│ Node cria JWT   │
└────────┬────────┘
         │
         │ JWT
         ▼
┌─────────────────┐
│   React :3000   │
└─────────────────┘
```

------------------------------------------------------------------------

# 20. Problemas comuns

## Erro: Cannot find module 'server.js'

Você provavelmente está na pasta errada.

Se o terminal estiver em:

``` text
oauth-google-login>
```

use:

``` powershell
cd backend
node server.js
```

Se estiver em:

``` text
oauth-google-login\backend>
```

use somente:

``` powershell
node server.js
```

Não use:

``` powershell
cd oauth-google-login\backend
```

quando você já estiver dentro de `oauth-google-login`.

------------------------------------------------------------------------

## Erro: Cannot find module 'express'

Entre na pasta `backend` e execute:

``` powershell
npm install express cors jsonwebtoken google-auth-library
```

Depois:

``` powershell
node server.js
```

------------------------------------------------------------------------

## Erro no login do Google

Confira:

1.  O Client ID está correto.
2.  O Client ID do React e do back-end é o mesmo.
3.  `http://localhost:3000` está configurado no Google Cloud.
4.  O React está rodando na porta 3000.
5.  O back-end está rodando na porta 5000.

------------------------------------------------------------------------

# 21. Sobre o aviso de vulnerabilidades do npm

Se aparecer:

``` text
30 vulnerabilities
```

isso não significa automaticamente que o projeto não funciona.

Você pode consultar:

``` powershell
npm audit
```

Evite usar imediatamente:

``` powershell
npm audit fix --force
```

durante uma atividade de aula, porque o `--force` pode atualizar pacotes
de forma incompatível e quebrar o projeto.

------------------------------------------------------------------------

# 22. Sobre o aviso `fs.F_OK`

Se aparecer:

``` text
[DEP0176] DeprecationWarning: fs.F_OK is deprecated
```

isso é um aviso do Node.js/dependência.

Se o React continuar iniciando e mostrar algo como:

``` text
Compiled successfully!
```

o aviso não significa que o projeto falhou.

------------------------------------------------------------------------

# 23. Checklist final

Antes de testar, confira:

-   [ ] Node.js instalado
-   [ ] Projeto React criado
-   [ ] `@react-oauth/google` instalado
-   [ ] `axios` instalado
-   [ ] `jwt-decode` instalado
-   [ ] Projeto criado no Google Cloud
-   [ ] Client ID criado
-   [ ] `http://localhost:3000` configurado
-   [ ] Client ID colocado no `App.js`
-   [ ] Pasta `backend` criada
-   [ ] `server.js` criado dentro de `backend`
-   [ ] Client ID colocado no `server.js`
-   [ ] Express instalado
-   [ ] CORS instalado
-   [ ] jsonwebtoken instalado
-   [ ] google-auth-library instalado
-   [ ] Back-end iniciado com `node server.js`
-   [ ] React iniciado com `npm start`
-   [ ] Página aberta em `http://localhost:3000`

------------------------------------------------------------------------

# 24. Credenciais --- cuidado

O **Client ID** pode ser utilizado no código do front-end.

O **Client Secret** não deve ser colocado no React nem enviado para
outras pessoas.

Também não publique a chave usada para assinar JWT em um repositório
público.

Em um projeto real, a chave secreta deve ser armazenada em variável de
ambiente, por exemplo:

``` text
JWT_SECRET=uma_chave_secreta
```

Para uma atividade introdutória, o exemplo usa uma string diretamente no
código apenas para facilitar o aprendizado.

------------------------------------------------------------------------

# 25. Resultado esperado

Ao final, a aplicação terá:

``` text
Login com Google
        ↓
Autenticação Google
        ↓
Token Google
        ↓
Back-end Node.js
        ↓
Validação
        ↓
JWT próprio
        ↓
Usuário autenticado
```

O objetivo da atividade é compreender a integração entre:

-   React
-   OAuth 2.0 / Google Identity
-   ID Token
-   Node.js
-   Express
-   JWT
-   autenticação de usuários
