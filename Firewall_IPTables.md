

### Funcionalidade (tabela): 

### "Filtragem de pacotes":

iptables -t filter 

Ou:

iptables 



#### Situações:

input: Endereço de destino do pacote é o do Firewall;

output: Pacote é analisado pelas regras de output, quando o pacote é originado pelo Firewall, ou seja, ip de origem é do Firewall;

forward: Quando o endereço de destino nao for o do Firewall, ou endereço de origem nao for do Firewall, e o Firewall tem o papel de roteador (tem que passar pelo firewall);



#### Criar regras:

* Na tabela filter, vou adicionar na ultima posicao, uma regra que impede a conexao de um host com outro host e que passa pelo Firewall. Lembrando que host-a(ip: 192.168.0.2) e host-c(ip: 10.0.0.2):

  | Comando | Significado          | Dados       |
  | :------ | :------------------- | ----------- |
  | -s      | IP Source            | 192.168.0.2 |
  | -d      | IP Destination       | 10.0.0.2    |
  | -p      | Protocol             | tcp         |
  | -sport  | Source Port          | 50000       |
  | -dport  | Destination Port     | 80          |
  | -j      | Jump ou Acao         | DROP        |
  | -i      | Interface de entrada |             |
  | -o      | Interface de saida   |             |
  | !       | Exceto               |             |

  ```bash
  iptables -t filter -A FORWARD -s 192.168.0.2 -d 10.0.0.2 -p tcp --sport 50000 --dport 80 -j DROP
  ```

  Ou:

  ```bash
  iptables -A FORWARD -s 192.168.0.2 -d 10.0.0.2 -p tcp --dport 80 -j DROP
  ```

* Quero impedir que todas as maquinas da LAN1, se conectem com o host-d(ip: 10.0.0.2):

  ```bash
  iptables -A FORWARD -d 10.0.0.2 -p tcp --dport 80 -j DROP
  ```

* Bloqueia tudo:

  ```bash
  iptables -A FORWARD -j DROP
  ```

  

### "Mascaramento e redirecionamento":

iptables -t nat



### "Prioridade de pacotes, tratamentos diferenciados de pacotes":

iptables -t mangle



## Configuracao do cenario

Cenario de rede:

![image-20220603145953780](/home/kuak/snap/typora/57/.config/Typora/typora-user-images/image-20220603145953780.png)

Configuracao do host-1:

```bash
auto eth0
iface eth0 inet static
	hwaddress ether 00:00:00:00:00:01
	address 192.168.0.1
	netmask 255.255.255.0
	gateway 192.168.0.254
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

up /usr/bin/iperf3 -s -p 80 -D
up /usr/bin/iperf3 -s -p 81 -D
up /usr/bin/iperf3 -s -p 22 -D
up /usr/bin/iperf3 -s -p 23 -D
```

Configuracao do host-2:

```bash
auto eth0
iface eth0 inet static
	hwaddress ether 00:00:00:00:00:02
	address 192.168.0.2
	netmask 255.255.255.0
	gateway 192.168.0.254
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

up /usr/bin/iperf3 -s -p 80 -D
up /usr/bin/iperf3 -s -p 81 -D
up /usr/bin/iperf3 -s -p 22 -D
up /usr/bin/iperf3 -s -p 23 -D
```

Configuracao do host-3:

```bash
auto eth0
iface eth0 inet static
	hwaddress ether 00:00:00:00:00:03
	address 10.0.0.3
	netmask 255.255.255.0
	gateway 10.0.0.254
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

up /usr/bin/iperf3 -s -p 80 -D
up /usr/bin/iperf3 -s -p 81 -D
up /usr/bin/iperf3 -s -p 22 -D
up /usr/bin/iperf3 -s -p 23 -D
```

Configuracao do host-4:

```bash
auto eth0
iface eth0 inet static
	hwaddress ether 00:00:00:00:00:04
	address 10.0.0.4
	netmask 255.255.255.0
	gateway 10.0.0.254
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

up /usr/bin/iperf3 -s -p 80 -D
up /usr/bin/iperf3 -s -p 81 -D
up /usr/bin/iperf3 -s -p 22 -D
up /usr/bin/iperf3 -s -p 23 -D
```



No router Firewall:

```bash
auto eth0
iface eth0 inet static
	hwaddress ether 00:00:00:00:01:01
	address 192.168.0.254
	netmask 255.255.255.0
	
auto eth1
iface eth1 inet static
	hwaddress ether 00:00:00:00:01:02
	address 10.0.0.254
	netmask 255.255.255.0

up /usr/bin/iperf3 -s -p 80 -D
up /usr/bin/iperf3 -s -p 81 -D
up /usr/bin/iperf3 -s -p 22 -D
up /usr/bin/iperf3 -s -p 23 -D
```

* Testando:

  ```bash
  iperf3 -c 10.0.0.3 -p 80
  ```

  

FIREWALL:

* Listar as regras:

  ```bash
  iptables -t filter -L
  ```

* Adicionando regra para bloquear o acesso do host1 com o host3:

  ```bash
  iptables -A FORWARD -s 192.168.0.1 -d 10.0.0.3 -p tcp --dport 80 -j DROP
  ```

* Host2 nao pode acessar nada do host3:

  ```bash
  iptables -A FORWARD -s 192.168.0.2 -d 10.0.0.3 -j DROP
  ```

  * Teste:

    ```bash
    # terminal do host 3
    iperf3 -s -p 80
    
    # terminal do host 2
    iperf3 -c 10.0.0.3 -p 80
    ```

* Alterar politica:

  ```bash
  iptables -P FORWARD DROP
  ```

* Apagar as regras:

  ```bash
  iptables -F
  ```

  

### Configurando o Firewall

* Configuracao:

  ```bash
  cd
  
  vi firewall.sh
  
  #!/bin/bash
  case "$1" in
  	start)
  		echo "start firewall"
  		./firewallStart.sh
  		;;
  	stop)
  		echo "stop firewall"
  		./firewallStop.sh
  		;;
  	restart)
  		sh $0 stop
  		sh $0 start
  		;;
  	*)
  esac
  
  chmod a+x firewall.sh
  ./firewall.sh start
  ```

* Arquivo para parar o firewall, "firewallStop.sh":

  ```bash
  vi firewallStop.sh
  
  echo "firewallStop.sh"
  echo "Zerando politicas"
  iptables -P INPUT ACCEPT
  iptables -P OUTPUT ACCEPT
  iptables -P FORWARD ACCEPT
  echo "Zerando regras filter"
  iptables -F
  
  chmod a+x firewalStop.sh
  ```

* Arquivo para criar regras do firewall, "firewallStart.sh":

  ```bash
  vi firewallStart.sh
  
  echo "firewallStart.sh"
  # 1.Host 1 nao pode acessar tcp/81 do host 3
  iptables -A FORWARD -s 192.168.0.1 -d 10.0.0.3 -p tcp --dport 81 -j DROP
  
  # 2.Nao pode haver telnet(tcp/23) entre as redes, LAN1 nao pode acessar LAN2 
  iptables -A FORWARD -p tcp --dport 23 -j DROP
  
  # 3.As redes nao podem acessar o firewall via telnet
  iptables -A INPUT -p tcp --dport 23 -j DROP
  
  # 4.LAN2 nao pode acessar o firewall via ssh(tcp/22)
  iptables -A INPUT -s 10.0.0.0/24 -i eth1 -p tcp --dport 22 -j DROP
  # OU
  iptables -A INPUT -s 192.168.0.0/24 -i eth0 -p tcp --dport 22 -j ACCEPT
  iptables -A INPUT -p tcp --dport 22 -j DROP
  
  # 5.Host1 nao deve acessar http no host2
  # Obs: nao tem como, pois o host1 nao passa pelo firewall, entao cria-se a seguinte regra no host2:
  iptables -A INPUT -s 192.168.0.1 -p tcp --dport 80 -j DROP
  
  ```

  # Reconfigurando
  
  No firewall:
  
  ```bash
  mkdir /etc/firewall
  
  cp /etc/init.d/zebra /etc/firewall/firewall.sh
  vi /etc/firewall/firewall.sh
  # Apagar ate chegar na linha que começa o: case "$1" in
  # Deixando o firewall.sh do jeito abaixo:
  
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
  
  Agora criando o arquivo "stopFirewall.sh":
  
  ```bash
  vi /etc/firewall/stopFirewall.sh
  
  # Zerando as politicas e as regras
  echo "Stop firewall"
  iptables -P INPUT ACCEPT
  iptables -P OUTPUT ACCEPT
  iptables -P FORWARD ACCEPT
  
  iptables -F
  ```
  
  Parando o firewall:
  
  ```bash
  chmod a+x /etc/firewall/firewall.sh
  /etc/firewall/firewall.sh stop
  ```
  
  Criando o arquivo "startFirewall.sh":
  
  ```bash
  vi /etc/firewall/startFirewall.sh
  
  echo "Start firewall"
  # 2.Nao pode haver telnet(tcp/23) entre as redes, LAN1 nao pode acessar LAN2 
  iptables -A FORWARD -p tcp --dport 23 -j DROP
  ```
  
  Testando agora, abrir o terminal do host 1:
  
  ```bash
  iperf3 -c 10.0.0.3 -p 23
  # Saída
  Connecting to host 10.0.0.3, port 23
  [  5] local 192.168.0.1 port 42150 connected to 10.0.0.3 port 23
  [ ID] Interval           Transfer     Bitrate         Retr  Cwnd
  [  5]   0.00-1.00   sec  12.8 MBytes   107 Mbits/sec   16   86.3 KBytes       
  [  5]   1.00-2.00   sec  13.0 MBytes   109 Mbits/sec   11   97.6 KBytes       
  [  5]   2.00-3.00   sec  13.1 MBytes   110 Mbits/sec    9   90.5 KBytes       
  [  5]   3.00-4.00   sec  13.4 MBytes   113 Mbits/sec    9   87.7 KBytes 
  ```
  
  Ao executar o comando "/etc/firewall/firewall.sh start", o iperf deve parar ou nao conseguir mais pingar.
  
  Criando regra de nenhum host pingar o firewall (**obs:** o firewall sempre consegue pingar os hosts, mas os hosts nao conseguem pingar o firewall):
  
  ```bash
  # Esta regra bloqueia o ping de hosts para o firewall e viceversa
  iptables -A INPUT -p icmp -j DROP
  
  # Esta regra bloqueia apenas os pings de outras redes no firewall
  iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
  ```
  
  Como vai ficar o arquivo "startFirewall.sh":
  
  ```bash
  echo "Start firewall"
  
  iptables -A FORWARD -p tcp --dport 23 -j DROP
  iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
  ```
  
  Excluindo todas as regras e adicionando as políticas:
  
  ```bash
  echo "Start firewall"
  
  iptables -P INPUT DROP
  iptables -P OUTPUT DROP
  iptables -P FORWARD DROP
  ```
  
  Testando:
  
  ```bash
  iptables -L
  #Saida
  Chain INPUT (policy DROP)
  target     prot opt source               destination         
  
  Chain FORWARD (policy DROP)
  target     prot opt source               destination         
  
  Chain OUTPUT (policy DROP)
  target     prot opt source               destination 
  ```
  
  Criando regra: "Quero que o firewall pingue as maquinas das redes":
  
  ```bash
  iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
  iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
  ```
  
  Nova regra: "Quero que as maquinas da rede acessem a porta http(p:80)":
  
  ```bash
  # Cria-se a ida e a volta, pois se criarmos só a ida, na volta o firewall pode bloquear
  iptables -A FORWARD -p tcp --dport 80 -j ACCEPT
  iptables -A FORWARD -p tcp --sport 80 -j ACCEPT
  ```
  
  Para testar:
  
  ```bas
  #HOST1: iperf3 -c 10.0.0.3 -p 80
  #HOST3: tcpdump -i eth0 port 80 -n
  ```
  
  
  
  ## Criando outro cenario de rede
  
  Cenario:
  
  ![image-20220609203808846](/home/kuak/snap/typora/57/.config/Typora/typora-user-images/image-20220609203808846.png)
  
  Configurando a rede:
  
  Host 1:
  
  ```bash
  auto eth0
  iface eth0 inet static
  	hwaddress ether 00:00:00:00:00:01
  	address 192.168.122.101
  	netmask 255.255.255.0
  	gateway 192.168.122.1
  	up echo nameserver 8.8.8.8 > /etc/resolv.conf
  
  up /usr/bin/iperf3 -s -p 80 -D
  up /usr/bin/iperf3 -s -p 81 -D
  up /usr/bin/iperf3 -s -p 22 -D
  up /usr/bin/iperf3 -s -p 23 -D
  ```
  
  Host 2:
  
  ```bash
  auto eth0
  iface eth0 inet static
  	hwaddress ether 00:00:00:00:00:02
  	address 192.168.122.102
  	netmask 255.255.255.0
  	gateway 192.168.122.1
  	up echo nameserver 8.8.8.8 > /etc/resolv.conf
  
  up /usr/bin/iperf3 -s -p 80 -D
  up /usr/bin/iperf3 -s -p 81 -D
  up /usr/bin/iperf3 -s -p 22 -D
  up /usr/bin/iperf3 -s -p 23 -D
  ```
  
  Host 3:
  
  ```bash
  auto eth0
  iface eth0 inet static
  	hwaddress ether 00:00:00:00:00:03
  	address 10.1.1.103
  	netmask 255.255.255.0
  	gateway 10.1.1.254
  	up echo nameserver 8.8.8.8 > /etc/resolv.conf
  
  up /usr/bin/iperf3 -s -p 80 -D
  up /usr/bin/iperf3 -s -p 81 -D
  up /usr/bin/iperf3 -s -p 22 -D
  up /usr/bin/iperf3 -s -p 23 -D
  ```
  
  Host 4:
  
  ```bash
  auto eth0
  iface eth0 inet static
  	hwaddress ether 00:00:00:00:00:04
  	address 10.1.1.104
  	netmask 255.255.255.0
  	gateway 10.1.1.254
  	up echo nameserver 8.8.8.8 > /etc/resolv.conf
  
  up /usr/bin/iperf3 -s -p 80 -D
  up /usr/bin/iperf3 -s -p 81 -D
  up /usr/bin/iperf3 -s -p 22 -D
  up /usr/bin/iperf3 -s -p 23 -D
  ```
  
  No Firewall:
  
  ```bash
  auto eth0
  iface eth0 inet static
  	hwaddress ether 00:00:00:00:01:00
  	address 192.168.122.254
  	netmask 255.255.255.0
  	gateway 192.168.122.1
  	up echo nameserver 8.8.8.8 > /etc/resolv.conf
  	
  auto eth1
  iface eth1 inet static
  	hwaddress ether 00:00:00:00:01:01
  	address 10.1.1.254
  	netmask 255.255.255.0
  
  up /usr/bin/iperf3 -s -p 80 -D
  up /usr/bin/iperf3 -s -p 81 -D
  up /usr/bin/iperf3 -s -p 22 -D
  up /usr/bin/iperf3 -s -p 23 -D
  ```
  
  
  
  Como o NAT funciona:
  
  Mascaramento: Tenho uma rede privada e quero que todas as maquinas da rede naveguem com um ip publico.
  
  
  
  Segurança: A rede nao pode ser acesssada como servidor, torna-se transparente.
  
  

​	Quero pre-redirecionamento -> pre routing

​	Se quero mascaramento -> post routing

Exemplo:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 200.0.0.1
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 10.0.0.3
```



Continuacao:	

Criando o arquivo firewall.sh:

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

```bash
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 192.168.122.254
```

























