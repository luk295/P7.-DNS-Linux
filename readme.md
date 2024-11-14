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
  # creo a maquina chamada bind
  bind:
    image: internetsystemsconsortium/bind9:9.18  # imaxe de bind9
    container_name: dns_Practica_7 # nome do contenedor

    tty: true 
    ports: # puertos da maquina local á virtual
      - 55:53/udp
      - 55:53/tcp
    #- 127.0.0.1:953:953/tcp
    volumes: # carpetas que se comparten, é dicir, o contenido que se crea nas máquinas virtuais a partir da máquina local
    #  - ./configuracionOpciones:/etc/bind
      - ./config:/etc/bind/
      - ./zonas:/var/lib/bind/
    
    networks: 
      P7: # aquí indícolle que utilice a rede p7, creada nun nivel superior na xerarquía do yml.
        ipv4_address: 172.30.10.1 # ip de dirección fixa do servidor
    
    # os mesmos campos que á de enrriba, con excepción do dns. 
  cliente:
    image: alpine   
    container_name: p7-alpine
    tty: true
    
    dns:
      - 172.30.10.1 # dirección do dns ó que vai facer as peticións.
    networks:
      P7:
        ipv4_address: 172.30.10.88 # ip da máquina
       #caracteristicas de red. rango etc

# creación da rade común dos contenedores
networks:
 # A rede común das máquinas
  P7:  # Nome da rede que creo
    driver: bridge # adaptador puente
    ipam:
      driver: default
      config:
        - subnet: 172.30.0.0/16 # rede
          ip_range: 172.30.10.0/24 # rango 
          gateway: 172.30.10.254 # dirección ó router

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

`dig ns.practica7.org`

```
/ # dig ns.practica7.org

; <<>> DiG 9.18.27 <<>> ns.practica7.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30015
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 5105d3081d9002990100000067364006856e5c13d9b33b20 (good)
;; QUESTION SECTION:
;ns.practica7.org.		IN	A

;; ANSWER SECTION:
ns.practica7.org.	38400	IN	A	172.30.10.1

;; Query time: 11 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 14 18:23:02 UTC 2024
;; MSG SIZE  rcvd: 89

/ # 

```

`dig test.practica.org` 
```
; <<>> DiG 9.18.27 <<>> test.practica7.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38274
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 376912873d0b1b54010000006736407e2c8d225976d89a55 (good)
;; QUESTION SECTION:
;test.practica7.org.		IN	A

;; ANSWER SECTION:
test.practica7.org.	38400	IN	A	172.30.10.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 14 18:25:02 UTC 2024
;; MSG SIZE  rcvd: 91

/ # 

```

`dig www.practica7.org`
```
; <<>> DiG 9.18.27 <<>> www.practica7.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1972
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 202dbfad484fb83201000000673640bc6770c58262c9574c (good)
;; QUESTION SECTION:
;www.practica7.org.		IN	A

;; ANSWER SECTION:
www.practica7.org.	38400	IN	A	172.30.10.3

;; Query time: 1 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 14 18:26:04 UTC 2024
;; MSG SIZE  rcvd: 90

/ # ^C

```

`dig alias.practica7.org`
```
/ # dig alias.practica7.org

; <<>> DiG 9.18.27 <<>> alias.practica7.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 24268
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 10c9d73ab0b9de6c010000006736413e0f09d0eca9ccc892 (good)
;; QUESTION SECTION:
;alias.practica7.org.		IN	A

;; ANSWER SECTION:
alias.practica7.org.	38400	IN	CNAME	practicadeasirnumero7.com.practica7.org.

;; AUTHORITY SECTION:
practica7.org.		38400	IN	SOA	ns.practica7.org. some.email.address. 10000002 10800 3600 604800 38400

;; Query time: 2 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 14 18:28:14 UTC 2024
;; MSG SIZE  rcvd: 173

/ # 


```
### Contido do Readme (usando markdown):
```
    Liñas do docker-compose explicadas
    Procedemento de creación de servicios (contenedor)
    Modificación da configuración, arranque e parada do servicio bind9

    Configuración zoa e cómo comprobar que funciona
```
### Entregar enlace ó repositorio
