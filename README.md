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

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío, nos situa en una tesitura donde una conocida empresa de soluciones de seguridad parece estar realizando pruebas en su equipo. Para completar este reto, se deberá .

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- 
- Realizar una *reverse shell*.
- Poner en escucha los puertos de la máquina.
- Obtener una *shell* a partir de un *script*.
- Crear un servidor con *python3*.
- 
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *Bash*, *PHP*, *Netcat*, *Python3*, . 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos pues en este tipo de máquina no será necesario.

<code>nmap -Pn -sV -O 10.10.198.179 -sC</code>

![Captura de pantalla 2025-04-12 174419](https://github.com/user-attachments/assets/b26cb007-7731-455d-a0bb-1105b85ab78b)

El comando devuelve 3 puertos TCPs abiertos:  
- En el puerto 21 corre la versión *vsftpd 3.0.3* del servicio *FTP*. Los *scripts* nos informan de que es posible acceder al servicio con el usuario 'Anonymous', el cual no necesita contraseña.
- En el puerto 22 corre la versión *Openssh 7.6p1*, en un sistema *Ubuntu*, servicio *SSH*.  
- En el puerto 80 corre el servidor *Apache 2.4.49*, en un sistema *Ubuntu*, servicio *HTTP*.

Primero nos dirigimos, desde el navegador, al puerto 80 de la IP dada, donde se puede ver una página web de deportes. Después de inspeccionarla y navegar por ella, no se encuentra ninguna pista para la explotación.

Seguidamente, con la ayuda de la herramienta **Dirsearch**, se hace una enumeración de posibles directorios y archivos que se encuentren en la web. Por la parte de directorios, aparecen varios pero llama la atención '/secret/', al cual nos dirigiremos primero. *Dirsearch* es lanzado con **Python3** indicando la IP a atacar (-u) y la lista a utilizar para la enumeración (-w); en este caso la 'directory-list-lowercase-2.3-small.txt'.

<code>sudo python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u 10.10.198.179 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt</code>

![Captura de pantalla 2025-04-12 183016](https://github.com/user-attachments/assets/5d917c94-9d13-4a58-8966-c37cae458572)

En el directorio '/secret' se encuentra un panel de comandos, el cual tiene una *blacklist*, por medio de la cual, se están filtrando e impidiendo utilizar una serie de palabras. Cada vez que salta este filtro, se muestra una imagen de alerta y un mensaje preguntando '¿eres un hacker?'. Después de realizar varias pruebas, se ha comprobado que hay maneras de saltarse el filtro como concatenando un comando válido con otro por medio de ';'. U ofuscando palabras cambiando caracteres por el signo de interrogación, por poner un ejemplo.

### Vulnerabilidades explotadas



**Flag:**
