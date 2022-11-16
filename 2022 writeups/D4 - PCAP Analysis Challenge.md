## D4 - PCAP Analysis Challenge

O texto comenta sobre vazamento de informações e busca por novos alvos. Basicamente podemos resolver o exercício todo utilizando a ferramenta Wireshark disponível para todas as plataformas.

Começar o exercício pegando estatística de  quantos endereços IP existem nesse PCAP  para entender o tamanho do problema.

`Statistics -> IPv4 Statistics -> All Addresses`

IMG

Visualmente vemos 3 endereços com uma contagem de pacotes maior.

```
192.168.1.113
192.168.1.110
192.168.1.103
```

Agora vamos ver as estatísticas por destino e portas

`Statistics -> IPv4 Statistics -> Destination and Ports`

Para cada um desses 3 endereços:

### 192.168.1.113

IMG

Temos muitas portas TCP com a contagem de 01 (UM)

### 192.168.1.110

IMG

Somente 2 portas, uma UDP e uma TCP.
```
UDP = 785
TCP = 31337
```

### 192.168.1.103

IMG

Várias portas TCP com contagens pequenas mas variáveis…

Voltando ao enunciado …

- D4.1 - Recupere as informações vazadas pelo protocolo TCP.
- D4.2 - Recupere as informações vazadas pelo protocolo UDP.
- D4.3 - Recupere as informações vazadas pelo protocolo ICMP.

No wireshark podemos fazer um filtro somente pelo protocolo ….

Vamos começar pelo ICMP:

IMG

Visualmente podemos identificar a maioria dos pacotes com tamanho (Lenght) de 98 e 3 pacotes com tamanho diferente. O primeiro com 60 que aparentemente não faz parte de nossa investigação e 2 pacotes com 176 de tamanho.

O protocolo ICMP é utilizado nas comunicações TCP/IP dentre outras coisas como sinalizador de erro, indicando quando um host ou porta não estão acessíveis. Não é utilizado para transportar dados como o TCP e o UDP. E é o protocolo do PING  ( Echo Request e Echo Reply ).

Sabendo disso, se o criminoso está vazando informações e através do protocolo ICMP, o endereço IP origem, quem faz o Request (Echo Request) e  o endereço IP destino, quem responde (Echo Reply) 

Temos assim : 
192.168.1.103  ( Origem )
192.168.1.110 ( Destino )


Clicando no pacote número 412 , vemos que o campo DATA contém 134 bytes e não os 48 bytes padrão.

IMG

Olhando no  conteúdo desse campo, logo abaixo na parte HEXA/ASCII, temos ..

IMG

Flag ICMP encontrada: D4.3 - Recupere as informações vazadas pelo protocolo ICMP. `YSH{SecretsINdataPINGs}`

