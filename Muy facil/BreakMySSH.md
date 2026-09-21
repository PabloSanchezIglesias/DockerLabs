# BreakMySSH

Dificultad: Muy fácil

Link: https://dockerlabs.es/

## Summary

- [Reconocimiento](#reconocimiento)
- [Explotacion](#explotacion)

## Reconocimiento
Lo primero que hice nada mas desplegar el laboratorio fue empezar con la fase de reconocimiento utilizando la herramienta nmap.

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG escaneo
```
En el escaneo vemos que el único puerto que está abierto es el **puerto 22**, el cual corresponde al servicio **ssh**. Al no haber ningún puerto web expuesto, nuestro vector de ataque se centrará por completo en este servicio. Además se puede ver que el ttl es de 64 por lo que estamos ante una maquina Linux.

<img width="965" height="412" alt="image" src="https://github.com/user-attachments/assets/057b49cd-f809-4666-9b28-d0a76e72078b" />

Después hacemos un escaneo mas robusto centrándonos en el **puerto 22** donde están corriendo el **ssh** para poder obtener más información.

```bash
sudo nmap -sCV -p22 172.17..0.2 -oN allPorts
```
Gracias a este escaneo, pudimos confirmar la versión exacta del servicio SSH que se encuentra activo:

* **Puerto 22 (SSH):** Está corriendo la versión `OpenSSH 7.7 (protocol 2.0)`. Al no disponer de servidores web u otros servicios adicionales expuestos, la explotación de la máquina deberá enfocarse directamente a través de este puerto.

<img width="1034" height="309" alt="image" src="https://github.com/user-attachments/assets/09eb7197-0f94-4c43-acc1-7367eab2a721" />

Al ver la versión de OpenSSH que corre en el objetivo (`OpenSSH 7.7`), decidí buscar posibles vulnerabilidades públicas conocidas utilizando la herramienta `searchsploit`.

``` bash
searchsploit OpenSSH 7.7
```

Como podemos observar en los resultados, existen exploits públicos para versiones inferiores a la 7.7 que permiten la **enumeración de usuarios válidos** en el sistema (`Username Enumeration`). Esto resulta muy interesante, ya que nos da un vector potencial para descubrir nombres de usuarios reales en la máquina antes de intentar un ataque de fuerza bruta a ciegas.

<img width="1901" height="190" alt="image" src="https://github.com/user-attachments/assets/2981d197-d92d-4d83-9929-9008d5b8d1c2" />

Para analizar en detalle el funcionamiento del exploit de enumeración de usuarios y poder replicarlo, decidí acudir directamente a la base de datos de Exploit Database (`exploit-db.com`) introduciendo el EDB-ID correspondiente (`45233`).

Desde la web pudimos revisar la estructura del código en Python enfocado en el CVE-2018-15473 y procedí a copiar el exploit completo a mi entorno de trabajo local para prepararlo y adaptarlo para la fase de ataque contra el objetivo.

<img width="1906" height="983" alt="image" src="https://github.com/user-attachments/assets/58db5932-cb26-4001-aca8-e90dbce6a59a" />

Una vez que tenía el script listo en mi máquina local, decidí ejecutarlo pasándole los parámetros correspondientes para validar la existencia de usuarios específicos en el servidor SSH. Para probar suerte, decidí testear en primer lugar si el usuario administrador clásico existía en el sistema.

``` bash
python3 exploit.py -p 22 172.17.0.2 root
```

El script funcionó de forma perfecta y nos confirmó de inmediato que el usuario **root** es un usuario completamente válido en el sistema (`[+] root is a valid username`). Ya con un usuario real confirmado en la máquina, tenemos nuestro objetivo claro para intentar ganar acceso.

<img width="262" height="32" alt="image" src="https://github.com/user-attachments/assets/2312202c-c192-42d2-9994-cbf639c73bcf" />

## Explotacion

Al tener confirmado que el usuario `root` existía en el sistema, decidí lanzar un ataque de fuerza bruta contra el servicio SSH utilizando la herramienta `hydra` junto al popular diccionario `rockyou.txt` para intentar dar con la contraseña de acceso.

``` bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh -t 15
```

El ataque de fuerza bruta funcionó correctamente y logré conseguir una credencial válida para ingresar directamente al sistema con los máximos privilegios:

* **Usuario:** `root`
* **Contraseña:** `estrella`

<img width="1274" height="185" alt="image" src="https://github.com/user-attachments/assets/914b03f8-efdf-46a8-be38-7307ffcfb74c" />

Para terminar con el laboratorio, procedí a conectarme al servidor SSH utilizando las credenciales obtenidas (`root:estrella`) para verificar el acceso.

``` bash
ssh root@172.17.0.2
```

Al introducir la contraseña, logré acceder de forma exitosa obteniendo una shell directa. Una vez dentro de la máquina objetivo, ejecuté los comandos `whoami`, `id` y `hostname` para confirmar mi identidad y privilegios, comprobando que tenemos control total sobre el contenedor como el usuario administrador principal.

``` bash
whoami
id
hostname
```

<img width="467" height="216" alt="image" src="https://github.com/user-attachments/assets/2798c971-3398-43b5-87d0-dac75b9d80c4" />




