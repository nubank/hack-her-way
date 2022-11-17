## D1 - Bucket Challenge

### Procedimento

#### Flag D1.1
A candidata deverá inspeccionar o código fonte do [site](http://ysh-www.s3-website-sa-east-1.amazonaws.com/) comprometido. Aqui a candidata vai achar a primeira 
flag, dentro do arquivo credenciais.txt linkado no código fonte. Além disso, estará exposto o link do bucket que será usado na seguinte etapa do desafio.

Flag a ser submetida pela candidata: `YSH{WEBSITE_SOURCECODE_SECRETS}`

---

#### D1.2
No arquivo credenciais.txt, da etapa anterior, a candidata terá acesso para o arquivo XML com as informações 
para se conectar à primeira instância EC2:

- XML possui: endereço do [bucket](https://ysh-secrets.s3.sa-east-1.amazonaws.com/), uma chave pública 
  (ysh-instance-01.pem) e o script `conecta.sh`

Se a candidata explora o bucket, vai achar o arquivo `README.md`, onde estará mais uma flag

Flag a ser submetida pela candidata: `YSH{SSH_KEY_AND_SCRIPT} `

---

#### D1.3
A candidata poderá copiar e lançar no terminal o arquivo conecta.sh renomeando a chave pública da etapa 
anterior, para se conectar na instância EC2 (YSH_INSTANCE_01)

Uma vez conectada, se listar os arquivos, poderá achar o arquivo CONGRATULATIONS.md e dentro dela estará a flag desta etapa.

Comando a executar pela candidata:
````shell
ssh -i <nome-da-chave-publica.pem> ubuntu@ec2-52-67-160-131.sa-east-1.compute.amazonaws.com
````
Flag a ser submetida pela candidata: `YSH{SUCCESSFUL_EC2_CONNECTION}`

---

#### D1.4
A candidata poderá copiar as informações do arquivo CONGRATULATIONS.md, obtida na etapa anterior, para se 
conectar apenas desde esta EC2 numa outra instância (YSH_INSTANCE_02).

Comando a executar pela candidata:

````shell
ssh -i <nome-da-chave-publica.pem> ubuntu@18.228.204.226
````
Dentro da segunda instância EC2 estará o arquivo CONGRATULATIONS.md com outra flag.

Flag a ser submetida pela candidata: `YSH{VULNERABLE_NGINX_SERVER}` 

 