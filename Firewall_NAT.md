## Continuação de NAT

Cenario:

![image-20220610135754949](/home/kuak/snap/typora/57/.config/Typora/typora-user-images/image-20220610135754949.png)



Nat -> mascaramento, pega ip da rede de origem e mascara com o ip da rede de destino. Fazendo isso atraves do post routing snat(source nat, alterar endereço de origem).

Nat -> redirecionamento, onat(output nat, altera endereço de destino e o de origem).

Criando arquivo firewall.sh:

```bash
mkdir /etc/firewall
vi /etc/firewall/firewall.sh

#!/bin/bash

case "$1" in
  start)
        $0 stop
        sh /etc/firewall/startFirewall.sh
        ;;
  stop)
        sh /etc/firewall/stopFirewall.sh
        ;;
  *)
        echo $"Usage: $0 {start|stop}"
        exit 2
esac
```

Criando arquivo stopFirewall.sh:

```bash
echo "Stop Firewall"

iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

iptables -F
iptables -t nat -F
iptables -t mangle -F
```

Criando arquivo de startFirewall.sh:

```bash
echo "Start firewall"

iptables -t nat -A POSTROUTING -i eth0 -o eth0 -j SNAT --to 192.168.122.254
```

No host3:

```bash
iperf3 -c 10.1.1.3 -p 80

ps ax
kill <processo que roda na porta 80>
apt update
apt install apache2
apachectl start
echo "bem vindo ao host3" > /var/www/html/index.html
lynx 127.0.0.1
```

No startFirewall.sh:

```bash
echo "Start firewall"

echo "mascaramento"
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 192.168.122.254

echo "redirecionamento"
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to 10.1.1.103
```

No host1:

```bash
lynx 192.168.122.254
```



Fazer com que as maquinas da internet sejam redirecionadas a maquina host3:

Tudo que chegar no firewall na porta 8080, vou mandar pra porta 80 do host 4:

```bash
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to 10.1.1.104:80
```

Testando no host1:

```bash
iperf3 -c 192.168.122.254 -p 8080
```



Apenas o host3 pode aceessar o firewall via ssh:

```bash
iptables -A INPUT -p tcp --dport 22 -m mac --mac-source 00:00:00:00:00:03 -i eth1 -s 10.1.1.103 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```



## modulos de estados

```bash
"-m state --state":
new, established, invalid, related
```

* new: conexoes novas
* established: conexoes estabelecidas
* invalid: conexoes invalidas
* related: conexoes relatadas





















































































































































































































































































































































































































































































































































































































































































































































