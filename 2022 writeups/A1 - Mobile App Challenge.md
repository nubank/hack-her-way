## A1 - Mobile App Challenge

Esse desafio de mobile requer conhecimentos de engenharia reversa, revisão de código e programação.

Quando você tenta acessar a aplicação uma tela de login é exibida, mas sem mais detalhes.

Passos para resolver:
  - "Decompilar” usando o Apktool 
  - Fazer um grep, ou revisão de código a procura de endpoints (HTTP/HTTPS)
  - Você irá encontrar no arquivo `./smali/com/nubank/purpleskills/data/LoginDataSource.smali`
    - `const-string v0, "https://im5ixvhuid.execute-api.us-east-1.amazonaws.com`
  - Há diversas menções a `AESEncryption` no arquivo 
  - Note que existe uma chamada para o método `getUrl` passando `creds` como parâmetro
  - O método parece simplesmente concatenar a palavra creds à URL mencionada acima 
  - Nós obtemos um usuário e senha
  - Usamos o usuário e senha, aparece uma tela com Welcome Jane Doe e a aplicação fecha
  - Se olharmos no logcat, ou através da revisão do código, notaremos um texto escrito com a palavra FLAG! seguindo de 
    uma string que parece estar criptografada
  - Se chamarmos o método decrypt da classe AESEncryption passando a string acima como parâmetro nós obteremos a flag:  
    YSH{y0u_h00k_m3_up}

