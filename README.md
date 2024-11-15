# DNS-Linux
## Configuración dun servidor DNS con cliente Alpine e Ubuntu
A práctica segue como referencia a guía oficial de Ubuntu para configuración de DNS. O obxectivo é crear un sistema cun servidor DNS e un cliente Alpine que cumpran os seguintes requisitos.

- Arquivos necesarios
Achego un ficheiro .yaml xunto a dous directorios, denominados conf e zonas, que conteñen os volumes necesarios e xa están configurados correctamente.

- Volumes separados para configuración
No ficheiro .yaml, definimos os volumes do seguinte xeito:
```
  volumes:
    - ./conf:/etc/bind/
    - ./zonas:/var/lib/bind/
```
Isto indica que as carpetas locais conf e zonas se montarán como volumes no contedor de bind9. É necesario crear estas carpetas no directorio de traballo coas configuracións específicas:

- Carpeta zonas: contén os ficheiros de configuración das zonas do servidor DNS. Por exemplo, un ficheiro chamado db.pepe.int define a zona pepe.int. Ao executar un comando como dig ns.pepe.int, obteremos a IP 192.28.5.1 definida na configuración.

- Carpeta conf: contén o ficheiro principal de configuración, onde se inclúen as zonas e outros axustes, como os forwarders.

- Rede interna para os contedores
No ficheiro .yaml, definimos unha rede interna con IP fixa para o servidor:

```
networks:
  bind9_subnet:
    ipv4_address: 192.28.5.4
```

- Configuración dos forwarders
No ficheiro de configuración principal (named.conf), incluímos os forwarders no apartado de options, que serven para consultar outros servidores DNS se o actual non pode resolver unha consulta:

```
options {
  directory "/var/cache/bind";

  dnssec-validation no; # Soluciona erros de validación

  forwarders {
    8.8.8.8;
    1.1.1.1;
  };
  forward only;
};
```

- Creación dunha zona personalizada
Para engadir unha zona, creamos un ficheiro na carpeta zonas co nome da nova zona, como db.pepe.int. Este inclúe os rexistros necesarios:


NS, A, CNAME, TXT, SOA: Exemplos no ficheiro db.pepe.int:
```
ns       IN A       192.28.5.1
test     IN A       192.28.5.4
www      IN A       192.28.5.7
alias    IN CNAME   test
texto    IN TXT     "mensaxe"
prueba23 IN A       125.44.32.1
```
- Cliente con ferramentas de rede
Para o cliente DNS, decidín usar un sistema Ubuntu en lugar de Alpine para probar diferenzas nos comandos:

- Instalación manual no cliente Ubuntu: Executamos o seguinte comando tras iniciar o contedor co docker-compose:

```
docker exec -it ubuntu /bin/bash
apt-get update && apt-get install -y dnsutils
```
- Instalación automática definida no .yaml: Configuración para instalar automaticamente as ferramentas de rede:

Ubuntu:

```
command: /bin/bash -c "apt-get update && apt-get install -y dnsutils && bash"
```
Alpine:
```
command: /bin/sh -c "apk update && apk add --no-cache bind-tools && sh"
```
- Verificación
Unha vez configurado todo, podemos usar o comando dig para verificar que o servidor resolve correctamente as consultas DNS. Exemplo de saída para dig ns.pepe.int:
```
; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> ns.pepe.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4194
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;ns.pepe.int.      IN      A

;; ANSWER SECTION:
ns.pepe.int.       38400   IN      A       192.28.5.1

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; MSG SIZE  rcvd: 84
```
- Conclusión
Tanto o ficheiro .yaml como os contidos das carpetas conf e zonas están documentados con comentarios para facilitar a comprensión e reprodución do sistema.
