# Firewall NAT Revisao

Criando arquivo firewall:

```bash
mkdir /etc/firewall
vi /etc/firewall/firewall.sh

echo "start firewall"

# Habilitar a maquina como roteador, e tem acesso a rede
echo "Mascaramento"
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

```bash
chmod a+x /etc/firewall/firewall.sh 
```

Para testar:

```bash
iperf3 -c 192.168.122.101
```

Mudar politica de drop para tudo:

```bash
vi /etc/firewall/firewall.sh

echo "politica de negar tudo"
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
```

Rodando o firewall:

```bash
/etc/firewall/firewall.sh start
```

* Quero que os hosts da LAN 1, consigam acessar a internet:

  ```bash
  iptables -A FORWARD -i eth1 -p tcp --dport 80 -j ACCEPT
  iptables -A FORWARD -p tcp --sport 80 -j ACCEPT
  ```

* Quero que os hosts da LAN 1, acessem internet por icmp:

  ```bash
  iptables -A FORWARD -i eth1 -p udp --dport 53 -j ACCEPT
  iptables -A FORWARD -p udp --sport 53 -j ACCEPT
  ```

* Quero que os hosts da LAN 1, acessem internet por https:

  ```bash
  iptables -A FORWARD -i eth1 -p tcp --dport 443 -j ACCEPT
  iptables -A FORWARD -p tcp --sport 443 -j ACCEPT
  ```

* Apaga as duas linhas de http e https. Aplicando estados para todos os protocolos:

  ```bash
  # (IDA) Conexoes novas, estabelecidas ou relatadas
  iptables _A FORWARD -i eth1 -p all -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  
  # (VOLTA) Conexoes estabelecidas ou relatadas
  iptables _A FORWARD -p all -m state --state ESTABLISHED,RELATED -j ACCEPT
  ```

* Teste:

  ```bash
  # Selecionar a opcao "A" duas vezes
  lynx www.google.com.br 
  ping www.google.com.br
  ```

* O host 3, pode acessar o firewall via SSH, controlado por estados:

  ```bash
  echo "Host 3 pode acessar o firewall via ssh"
  iptables -A INPUT -s 10.1.1.103 -p tcp --dport 22 -i eth1 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED,RELATED -j ACCEPT
  ```

* Redirecionar HTTP que chega da Internet para o firewall para o TCP/8080 do host 4:

  * host 4:

    ```bash
    apt update
    apt install apache2
    vi /etc/apache2/ports.conf
    
    Listen 8080
    
    /etc/
    ```

  * Firewall:

    ```bash
    echo "Redirecionar HTTP que chega da Internet para o firewall para o TCP/8080 do host 4"
    iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 10.1.1.104:8080
    iptables -A FORWARD -p tcp --dport 8080 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
    
    ```

    

    

  

  

  



















































