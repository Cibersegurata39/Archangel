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
- Realizar una *reverse shell*.
- Poner en escucha los puertos de la máquina.
- Obtener una *shell* a partir de un *script*.
- Crear un servidor con *python3*.
- Utilizar variables de entorno.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Bash*, *PHP*, *Netcat*, *Python3*, . 

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

El arhcivo 'robots.txt' indica a los rastreadores web de motores de búsqueda, a que páginas pueden acceder. En este caso se indica que no se peude acceder al archivo 'test.php' y está instrucción es para todos los motores de búsqueda. Si nos dirigimos a esta pagina se muestra que está en desarrollo y aparece un botón con el que interactuar.

![image](https://github.com/user-attachments/assets/687d71d4-dab8-4b87-b910-480669239780)


### Vulnerabilidades explotadas



**Flag:**
