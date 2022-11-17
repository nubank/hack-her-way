## A5 - Cloud Privesc Challenge

### Procedimento

#### A5.1 - Identifique a tag.

Neste desafio, foi idealizado que um bucket irá ser acessível dentro da conta e terá uma tag com a FLAG. Para tal será necessário os conhecimentos em: aws cli

O primeiro passo será tentar listar quais buckets a candidata consegue visualizar, através do comando ela deverá encontrar o bucket `ysh-nubank` :

````shell
$ aws s3 ls s3://ysh-nubank/
2022-09-13 20:24:00 ysh-nubank
````

Após ver que possui algum tipo de leitura no bucket, deve utilizar do cli para ver as tags do mesmo, através do comando abaixo:

````shell
$ aws s3api get-bucket-tagging --bucket ysh-nubank
{
  "TagSet": [
    {
      "Key": "YSH",
      "Value": "CLOUDSEC_1x0x1"
    }
  ]
}

````

Flag a ser submetida pela candidata: `YSH = "CLOUDSEC_1x0x1"`

---

#### A5.2 - Descubra o segredo entre os objetos do bucket e escale o privilégio.

A candidata deve utilizar comandos que listem quais objetos estão no bucket descoberto no passo A5.1, desta forma encontrará o arquivo chamado `file.txt`:

```shell
$ aws s3 ls s3://ysh-nubank/
```

Após achar o arquivo suspeito, deve fazer download para a maquina local através do comando:

```shell
$ aws s3 cp s3://ysh-nubank/file.txt ./
```

Próximo passo é ler o que contem neste arquivo

```shell
$ cat file.txt
```

Agora a candidata terá que tentar identificar em qual codificação o conteúdo do arquivo está e, após a descoberta, decodificar.

```shell
$ cat file.txt
ewogICAgIlN0YXRlbWVudCI6IFsKICAgICAgICB7CiAgICAgICAgICAgICJBY3Rpb24iOiBbCiAgICAgICAgICAgICAgICAic3FzOkxpc3RRdWV1ZVRhZ3MiCiAgICAgICAgICAgIF0sCiAgICAgICAgICAgICJFZmZlY3QiOiAiQWxsb3ciLAogICAgICAgICAgICAiUmVzb3VyY2UiOiAiYXJuOmF3czpzcXM6dXMtZWFzdC0xOjkwNTk2NTI0MDkyNDp5c2gtbnViYW5rLXF1ZXVlIgogICAgICAgIH0KICAgIF0sCiAgICAiVmVyc2lvbiI6ICIyMDEyLTEwLTE3Igp9

$ echo ewogICAgIlN0YXRlbWVudCI6IFsKICAgICAgICB7CiAgICAgICAgICAgICJBY3Rpb24iOiBbCiAgICAgICAgICAgICAgICAic3FzOkxpc3RRdWV1ZVRhZ3MiCiAgICAgICAgICAgIF0sCiAgICAgICAgICAgICJFZmZlY3QiOiAiQWxsb3ciLAogICAgICAgICAgICAiUmVzb3VyY2UiOiAiYXJuOmF3czpzcXM6dXMtZWFzdC0xOjkwNTk2NTI0MDkyNDp5c2gtbnViYW5rLXF1ZXVlIgogICAgICAgIH0KICAgIF0sCiAgICAiVmVyc2lvbiI6ICIyMDEyLTEwLTE3Igp9 | base64 --decode
{
  "Statement": [
    {
      "Action": [
        "sqs:ListQueueTags"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:sqs:us-east-1:905965240924:ysh-nubank-queue"
    }
  ],
  "Version": "2012-10-17"
}
```

Neste ponto irá visualizar que uma fila SQS está disponível, sendo assim, começar a testar o privilegio chegando no sqs permitido na policy descrita no arquivo `file.txt`, desta forma terá o conteúdo da flag no body da mensagem:

````shell
$ aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/905965240924/ysh-nubank-queue --attribute-names All --message-attribute-names All --max-number-of-messages 1
````

Flag a ser submetida pela candidata: `YSH  = "CLOUDSEC_2x02"`

 