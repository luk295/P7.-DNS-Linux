### Configura un sistema cun servidor DNS e un cliente alpine que cumpla os seguintes requisitos:


### Volumen por separado da configuración
### Red propia interna para tódo-los contenedores
### ip fixa no servidor
### Configurar Forwarders
### Crear Zona propia: Rexistros a configurar: NS, A, CNAME, TXT, SOA
### Cliente con ferramientas de rede


Creo o `docker-compose.yml`

```

services:
  bind:
    image: internetsystemsconsortium/bind9:9.18
    container_name: dns_Practica_7

    tty: true
    ports:
      - 55:53/udp
      - 55:53/tcp
    #- 127.0.0.1:953:953/tcp
    volumes:
    #  - ./configuracionOpciones:/etc/bind
      - ./config:/etc/bind/
      - ./zonas:/var/lib/bind/
    
    networks:
      P7:
        ipv4_address: 172.30.10.1 #ip fixa do servidor
      
  cliente:
    image: alpine  
    container_name: p7-alpine
    tty: true
    
    dns:
      - 172.30.10.1
    networks:
      P7:
        ipv4_address: 172.30.10.88
       #caracteristicas de red. rango etc

# creación da rade común dos contenedores
networks:
  P7:  # a rede común das máquinas
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.30.0.0/16
          ip_range: 172.30.10.0/24
          gateway: 172.30.10.254

```

Levanto o docker compose con `docker compose up`

Comprobo que están na misma rede inspeccionando a rede creada no arquivo Compose, con `docker compose inspect`

```
luk@luk-VirtualBox:~/P7-dns_linux/P7.-DNS-Linux$ docker network inspect p7-dns-linux_P7
[
    {
        "Name": "p7-dns-linux_P7",
        "Id": "96fe6d6ba5b1b40a576b391e22575725099da88f4ad414bc00fc6774c35019bf",
        "Created": "2024-11-12T19:08:22.930101988+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.30.0.0/16",
                    "IPRange": "172.30.10.0/24",
                    "Gateway": "172.30.10.254"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "a2aef4188b83f845473d59ca417945695eb64c853ea94993bb0ae759a53dad1c": {
                "Name": "p7-alpine",
                "EndpointID": "063fbe3710fdfc35cb0955c428e4379f2ae3899918ef8b417ef959717c40725f",
                "MacAddress": "02:42:ac:1e:0a:58",
                "IPv4Address": "172.30.10.88/16",
                "IPv6Address": ""
            },
            "dc2743789a5697503452023d9b205af0bac57dd74eab7a3c36b61d38ac452f23": {
                "Name": "dns_Practica_7",
                "EndpointID": "f961e931ca48d1573dd559f395f13978d283f48833ef94ad02774cd6fe175522",
                "MacAddress": "02:42:ac:1e:0a:01",
                "IPv4Address": "172.30.10.1/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "P7",
            "com.docker.compose.project": "p7-dns-linux",
            "com.docker.compose.version": "2.29.2"
        }
    }
]
```


Entro en cada máquina e comprobo que está correcto novamente:

> [!NOTA]
> Para elo debo executar o comando `docker exec -it "nome do contenedor" sh`

Máquina cliente (alpine):
```
luk@luk-VirtualBox:~/P7-dns_linux/P7.-DNS-Linux$ docker exec -it p7-alpine sh
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
17: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:1e:0a:58 brd ff:ff:ff:ff:ff:ff
    inet 172.30.10.88/16 brd 172.30.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 


```

Máquina servidor
```
luk@luk-VirtualBox:~/P7-dns_linux/P7.-DNS-Linux$ docker exec -it dns_Practica_7 sh
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
15: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:1e:0a:01 brd ff:ff:ff:ff:ff:ff
    inet 172.30.10.1/16 brd 172.30.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 


```

Comprobo que vense entre elas facendo ping:

```
servidor a cliente:
/ # ping 172.30.10.88
PING 172.30.10.88 (172.30.10.88): 56 data bytes
64 bytes from 172.30.10.88: seq=0 ttl=64 time=0.213 ms
64 bytes from 172.30.10.88: seq=1 ttl=64 time=0.122 ms
64 bytes from 172.30.10.88: seq=2 ttl=64 time=0.124 ms
^C
--- 172.30.10.88 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.122/0.153/0.213 ms

cliente a servidor:
/ # ping 172.30.10.1
PING 172.30.10.1 (172.30.10.1): 56 data bytes
64 bytes from 172.30.10.1: seq=0 ttl=64 time=0.204 ms
64 bytes from 172.30.10.1: seq=1 ttl=64 time=0.228 ms
64 bytes from 172.30.10.1: seq=2 ttl=64 time=0.383 ms
^C
--- 172.30.10.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.204/0.271/0.383 ms
/ # 

```

### Engado as carpetas de zonas e configuración, cos seus respectivos ficheiros:

Na carpeta de configuración:
```
zone "practica7.org" {
	type master;
	file "/var/lib/bind/practica7.org";
	allow-query {
		any;
		};
	};
	
# zone "172.30.10.1.in-addr.arpa" {
#     type master;
#     file "/etc/bind/db.reverse";
#     allow-query {
# 		any;
# 		};
# 	};
	
options {
	directory "/var/cache/bind";
	dnssec-validation no;
	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};

```
Na carpeta de zonas:
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.practica7.org. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.practica7.org.
ns		IN A		172.30.10.1
test	IN A			172.30.10.2
www		IN A		172.30.10.3
alias	IN CNAME	test
texto	IN TXT		"PROBA DO DNS"


```


### Agrego as ferramentas de rede no cliente:
No cliente executo os seguintes comandos: `apk update && apk add bind-tools`.

Con estas ferramentes xa teño disponible o comando `dig`. Agora pruebo comandos dig con los registros:









### Contido do Readme (usando markdown):
```
    Liñas do docker-compose explicadas
    Procedemento de creación de servicios (contenedor)
    Modificación da configuración, arranque e parada do servicio bind9

    Configuración zoa e cómo comprobar que funciona
```
### Entregar enlace ó repositorio
