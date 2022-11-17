## D2 - CloudTrail Correlation Challenge

A resolução do desafio pode ser vista neste [Collab](https://colab.research.google.com/drive/1pNM7GfzN5XvLsQlJbNn9Ohat_sp-VDW2?usp=sharing), que permite a execução dos scripts de resolução.

---

O desafio disponibiliza um arquivo de logs do CloudTrail. Dentre desse arquivo é possível perceber diversas tentativas de acesso negadas a um bucket S3 e mudanças feitas dentro desse bucket.

A partir desse arquivo a candidata deverá procurar por comportamentos suspeitos.

### Leitura do arquivo
A solução a seguir foi feita usando Python Notebook, e pandas, a leitura do arquivo pode ser feita da maneira que achar melhor.

````python
import json

filename = '/content/d2-cloudtrail-logs.txt'
all_lines = []
with open(filename) as file:
    for line in file:
     all_lines.append(json.loads(line))
````


Dica: Para entender melhor o conteudo do arquivo, tente verificar as colunas que existem!

```python
import pandas as pd
df = pd.json_normalize(all_lines)
df.columns
```

```
Index(['eventVersion', 'eventTime', 'eventSource', 'eventName', 'awsRegion', 'sourceIPAddress', 'userAgent', 'errorCode', 'errorMessage', 'responseElements', 'requestID', 'eventID', 'readOnly', 'resources', 'eventType', 'managementEvent', 'recipientAccountId', 'vpcEndpointId', 'eventCategory', 'userIdentity.type', 'userIdentity.principalId', 'userIdentity.arn', 'userIdentity.accountId', 'userIdentity.accessKeyId', 'userIdentity.sessionContext.sessionIssuer.type', 'userIdentity.sessionContext.sessionIssuer.principalId', 'userIdentity.sessionContext.sessionIssuer.arn', 'userIdentity.sessionContext.sessionIssuer.accountId', 'userIdentity.sessionContext.sessionIssuer.userName', 'userIdentity.sessionContext.attributes.creationDate', 'userIdentity.sessionContext.attributes.mfaAuthenticated', 'requestParameters.bucketName', 'requestParameters.Host', 'requestParameters.policyStatus', 'additionalEventData.SignatureVersion', 'additionalEventData.CipherSuite', 'additionalEventData.bytesTransferredIn', 'additionalEventData.AuthenticationMethod', 'additionalEventData.x-amz-id-2', 'additionalEventData.bytesTransferredOut', 'tlsDetails.tlsVersion', 'tlsDetails.cipherSuite', 'tlsDetails.clientProvidedHostHeader', 'requestParameters.publicAccessBlock', 'requestParameters.acl', 'requestParameters.policy', 'requestParameters.website', 'requestParameters.ownershipControls', 'requestParameters.max-keys', 'requestParameters.encoding-type', 'requestParameters.versions', 'requestParameters.cors', 'requestParameters.versioning', 'requestParameters.tagging', 'requestParameters.accelerate', 'requestParameters.intelligent-tiering', 'requestParameters.logging', 'requestParameters.requestPayment', 'requestParameters.object-lock', 'requestParameters.notification', 'requestParameters.encryption', 'requestParameters.lifecycle', 'requestParameters.list-type', 'requestParameters.fetch-owner', 'requestParameters.prefix', 'requestParameters.delimiter', 'requestParameters.key', 'requestParameters.x-amz-copy-source', 'requestParameters.accessControlList.x-amz-grant-full-control', 'userIdentity.invokedBy', 'sharedEventID', 'requestParameters.bucketPolicy.Version', 'requestParameters.bucketPolicy.Statement', 'requestParameters.X-Amz-Date', 'requestParameters.X-Amz-Algorithm', 'requestParameters.response-content-disposition', 'requestParameters.X-Amz-SignedHeaders', 'requestParameters.X-Amz-Expires', 'requestParameters.bucketPolicy.Id', 'requestParameters.x-amz-acl', 'requestParameters.x-amz-storage-class', 'requestParameters.inventory', 'requestParameters.replication', 'requestParameters.delete'], dtype='object')
```

### Informe o bucket que mais está sofrendo tentativas anonimas de acesso.

Para encontrar qual o bucket que está sofrendo tentativas de acesso o primeiro passo é agrupar pelo `requestParameters.bucketName` e seu `userIdentity.accountId`

````python
df_bucket_name = df.groupby(['requestParameters.bucketName', 'userIdentity.accountId']) \
                                .size() \
                                .reset_index(name='count') \
                                .sort_values(['count'], ascending=False)

df_bucket_name.head(10)
````


|   | requestParameters.bucketName | userIdentity.accountId | count|
|---|----------------------------|------------------------|------|
|0| d2-arwen| 905965240924| 772|
|4|d2-galadriel|905965240924|528|
|7|d2-shelob|905965240924|348|
|2|d2-eowyn|905965240924|321|
|6|d2-rosie|905965240924|284|
|1|d2-arwen|anonymous|9|
|5|d2-galadriel|anonymous|3|
|3|d2-eowyn|anonymous|2|
|8|d2-shelob|anonymous|2|


A partir de uma procura por `userIdentity.accountId` = `anonymous` nos dados agrupados pode-se perceber que o bucket com mais tentativas de acesso é o `d2-arwen`

```python
df_bucket_event_type = df_bucket_name.loc[df_bucket_name['userIdentity.accountId'] == 'anonymous']
df_bucket_event_type.head(10)
```


|   | requestParameters.bucketName | userIdentity.accountId | count|
|---|----------------------------|------------------------|------|
|1|d2-arwen|anonymous|9|
|5|d2-galadriel|anonymous|3|
|3|d2-eowyn|anonymous|2|
|8|d2-shelob|anonymous|2|


### Identifique o id do evento no Cloudtrail onde o atacante adicionou um arquivo.
Basta localizar o `eventID` em que o `anonymous`, não recebeu um `AccessDenied` no bucket `d2-arwen`.

```python
df_anonymous = df.loc[df['userIdentity.accountId'] == 'anonymous']

df_anonymous_not_deny = df_anonymous.loc[df_anonymous['errorCode'] != 'AccessDenied']

df_bucket_anonymous = df_anonymous_not_deny.loc[df_anonymous_not_deny['requestParameters.bucketName'] == 'd2-arwen']

df_id = df_bucket_anonymous.groupby(['eventID', 'eventName', 'eventTime']) \
                                .size() \
                                .reset_index(name='count') \
                                .sort_values(['eventTime'], ascending=False)

df_id.head(10)
```

| | eventID | eventName | eventTime | count |
|-|---------|-----------|-----------|-------|

|   | eventID | eventName | eventTime| count|
|---|---------|-----------|----------|------|
|1|41d7aae0-e51a-4235-9367-8cabc272f622|PutObject|2022-09-05T17:54:42Z|1|
|3|45d6b56a-fb68-4cd2-bf8c-94c42f93f6ae|PreflightRequest|2022-09-05T16:47:43Z|1|
|0|0b584cec-d1fa-4f2d-8f2a-31ee76a5f4ea|PreflightRequest|2022-09-05T16:31:38Z|1|
|2|43748246-54f9-427b-a218-a12831667590|PreflightRequest|2022-09-05T16:31:37Z|1|

### Informe a flag que está dentro do arquivo adicionado.
A partir do id do evento, descubra qual o nome do arquivo modificado.

```python
df_resouce = df.loc[df['eventID'] == '41d7aae0-e51a-4235-9367-8cabc272f622']
x = df_resouce['resources'].to_dict()
print(x)
```

```
{401: [{'type': 'AWS::S3::Object', 'ARN': 'arn:aws:s3:::d2-arwen/bombadil.bin'}, {'accountId': '905965240924', 'type': 'AWS::S3::Bucket', 'ARN': 'arn:aws:s3:::d2-arwen'}]}
```

Sabendo o nome do arquivo e do bucket, é possivel acessar o arquivo na aws

```shell
!pip install awscli
```

```shell
!aws s3 cp s3://d2-arwen/bombadil.bin . --no-sign-request
```

```shell
!cat /content/bombadil.bin
```

`-> YSH-ladies-of-the-rings-52b90c321d474a97c8`