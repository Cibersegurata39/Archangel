# Archangel
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, *reverse shell* y la escucha de puertos, XXXXXXXX y la escalada de privilegios.
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío, nos situa en una tesitura donde una conocida empresa de soluciones de seguridad parece estar realizando pruebas en su equipo. Para completar este reto, se deberá trabajar con el *Virtual hosting* de un dominio e interactuar con este, mediante LFI, para acceder a la máquina y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Funcionamiento *Virtual hosting*.
- Explotación LFI (*Local File Inclusion*).
- Ejecución de RCE (*Remote Code Execution*).
- Realización de *Path Traversal*.
- Funcionamiento de *Burp Suite*
- Realizar una *reverse shell*.
- Poner en escucha los puertos de la máquina.
- Obtener una *shell* a partir de un *script*.
- Crear un servidor con *python3*.
- Utilizar variables de entorno.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Burp Suite*, *Bash*, *PHP*, *Netcat*, *Python3*, . 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos pues en este tipo de máquina no será necesario.

<code>nmap -Pn -sV -O -sC 10.10.215.160</code>

![Captura de pantalla 2025-04-22 164433](https://github.com/user-attachments/assets/d0f3a43b-f403-4342-9595-011ed8b1f77f)

El comando devuelve 2 puertos TCPs abiertos:  
- En el puerto 22 corre la versión *Openssh 7.6p1*, en un sistema *Ubuntu*, servicio *SSH*.  
- En el puerto 80 corre el servidor *Apache 2.4.29*, en un sistema *Ubuntu*, servicio *HTTP*.

Primero nos dirigimos, desde el navegador, al puerto 80 de la IP dada, donde se puede ver la página web de la empresa de soluciones de seguridad. El reto de THM pide encontrar el nombre de *host*, el cual se puede deducir al encontrar un correo en la página web con el dominio 'mafialive.thm'. Esto nos lleva a pensar que se está utilizndo un *Virtual hosting* donde distitos dominios comparten la misma dirección IP con el objetivo de ahorrar en costes y usar los recursos de manera más efectiva.

![Captura de pantalla 2025-04-22 164207](https://github.com/user-attachments/assets/64c88ee4-d635-4ef7-95d7-a82ac57c0ff5)

Una vez encontrado el dominio, se asignará este a la dirección IP 10.10.215.160, al final del archivo '/etc/hosts'. Poniendo ahora en el navegador 'mafialive.thm', se encuentra la primera de las *flags*.

**Flag: thm{f0und_th3_r1ght_h0st_n4m3}**

Seguidamente, con la ayuda de la herramienta **Dirsearch**, se hace una enumeración de posibles directorios y archivos que se encuentren en la web. Por la parte de archivos, aparecen dos que son alcanzables 'test.php' y el clásico 'robots.txt'. *Dirsearch* es lanzado con **Python3** indicando el dominio 'mafialive.thm (-u) y la lista a utilizar para la enumeración (-w); en este caso la *seclist* '/raft-small-files.txt'.

<code>python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u mafialive.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-files.txt</code>

![Captura de pantalla 2025-04-22 172710](https://github.com/user-attachments/assets/f7530f66-7e9a-42e0-abde-57d1f5ef9e7b)

El archivo 'robots.txt' indica a los rastreadores web de motores de búsqueda, a que páginas pueden acceder. En este caso se indica que no se peude acceder al archivo 'test.php' y está instrucción es para todos los motores de búsqueda. Si nos dirigimos a esta pagina se muestra que está en desarrollo y aparece un botón con el que interactuar.

![image](https://github.com/user-attachments/assets/687d71d4-dab8-4b87-b910-480669239780)


### Vulnerabilidades explotadas

Al hacer click en el botón, se muestra el mensaje ‘Control is an ilusion’. Si capturamos la petición mediante **Burp Suite**, vemos como mediante el método GET se lee el archivo ‘mrrobot.php’ utilizando el parámetro *view*.

![image](https://github.com/user-attachments/assets/20a78635-aa86-400c-9f95-7d38e64b76a3)

Jugando con este parámetro se puede aprovechar un *Local File Inclusion* (LFI) para vulnerar la máquina. Mediante un *Path traversal* se intenta llegar hasta el archivo ‘/etc/passwd’, pero para conseguirlo se debe saltar un filtro a ‘../..’ con ‘..//..’. El ataque es positivo y se descubre el usuario 1001 llamado ‘archangel’.

<code>GET /test.php?view=/var/www/html/development_testing//..//..//..//..//etc/passwd HTTP/1.1</code>

![image](https://github.com/user-attachments/assets/876549b7-8b52-4f79-83cf-f69b98f78a46)

El siguiente paso es hacer uso de los *wrappers* de *php* que permiten XXXXXX. En este caso se hace uso del ‘*filter*’, en concreto con el convertidor en base64 y el archivo a tratar será ‘test.php’, página que muestra el botón a clicar. Con esto se pretende conseguir el código de esta web en base64, de tal manera que se muestre el código del *backend* (*php*) que se ejecuta en el lado del servidor y no sólo el código *html* que es ejecutado en el lado del cliente.
Este código es sacado de la página [hacktricks]( https://hacktricks.boitatech.com.br/pentesting-web/file-inclusion).

<code>GET /test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php HTTP/1.1</code>

![image](https://github.com/user-attachments/assets/25d343d1-44ba-45ff-a31e-c9c742826131)

Desde el terminal de *Linux* se decodifica el código devuelto con la siguiente instrucción.

<code>echo "CQo8IURPQ1RZUEUgSFRNTD4KPGh0bWw+Cgo8aGVhZD4KICAgIDx0aXRsZT5JTkNMVURFPC90aXRsZT4KICAgIDxoMT5UZXN0IFBhZ2UuIE5vdCB0byBiZSBEZXBsb3llZDwvaDE+CiAKICAgIDwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iL3Rlc3QucGhwP3ZpZXc9L3Zhci93d3cvaHRtbC9kZXZlbG9wbWVudF90ZXN0aW5nL21ycm9ib3QucGhwIj48YnV0dG9uIGlkPSJzZWNyZXQiPkhlcmUgaXMgYSBidXR0b248L2J1dHRvbj48L2E+PGJyPgogICAgICAgIDw/cGhwCgoJICAgIC8vRkxBRzogdGhte2V4cGxvMXQxbmdfbGYxfQoKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICBpZihpc3NldCgkX0dFVFsidmlldyJdKSl7CgkgICAgaWYoIWNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcuLi8uLicpICYmIGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICcvdmFyL3d3dy9odG1sL2RldmVsb3BtZW50X3Rlc3RpbmcnKSkgewogICAgICAgICAgICAJaW5jbHVkZSAkX0dFVFsndmlldyddOwogICAgICAgICAgICB9ZWxzZXsKCgkJZWNobyAnU29ycnksIFRoYXRzIG5vdCBhbGxvd2VkJzsKICAgICAgICAgICAgfQoJfQogICAgICAgID8+CiAgICA8L2Rpdj4KPC9ib2R5PgoKPC9odG1sPgoKCg==" | base64 -d</code>

![Captura de pantalla 2025-04-24 125131](https://github.com/user-attachments/assets/e69da747-f603-4432-8d26-6e2dd070fdf9)

El código *php* devuelto confirma el filtro que se aplicaba para evitar el *path traversal* y además contiene una nueva *flag*.

**Flag: thm{explo1t1ng_lf1}**

Para no ir mirando archivo por archivo de manera manual como se ha hecho con ‘/etc/passwd’, *BurpSuite* tiene una opción llamada *Intruder*, por medio la cual, se puede pasar una lista (.txt) preparada con archivos de sistema interesantes y que sean comprobados automáticamente. Gracias a esto se descubre el archivo ‘/var/log/apache2/acces.log’.

**Flag:**
