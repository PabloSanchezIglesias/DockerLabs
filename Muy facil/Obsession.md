# Obsession

Dificultad: Muy fácil

Link: https://dockerlabs.es/

## Summary

- [Reconocimiento](#reconocimiento)
- [Explotacion](#explotacion)

## Reconocimiento
Lo primero que hice nada mas desplegar el laboratorio fue empezar con la fase de reconocimiento utilizando la herramienta nmap.

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.88.0.2 -oG escaneo
```
En el primer escaneo vemos que están abiertos el puerto 21 en el que está corriendo el servicio ftp, el puerto 22 en el que corre el servicio ssh y el puerto 80 en el que corre el servicio http. Además se puede ver que el ttl es de 64 por lo que estamos ante una maquina Linux.

<img width="610" height="308" alt="image" src="https://github.com/user-attachments/assets/c46a7dfe-afa2-44a9-9f44-c16e48a20ac7" />

Después hacemos un escaneo mas robusto centrándonos en el puerto 21, el 22 y el 80 donde están corriendo el ftp, el ssh y el http para poder obtener más información.

```bash
sudo nmap -sCV -p21,22,80 10.88.0.2 -oN allPorts
```
Al ver los resultados del escaneo de nmap, podemos extraer los siguientes detalles de los servicios activos:

* **Puerto 21 (FTP):** Está corriendo la versión `vsftpd 3.0.5`. Lo más crítico aquí es que el script de nmap nos confirma que el login anónimo está activo (`Anonymous FTP login allowed`). Gracias a esto, podemos ver que existen dos archivos de texto en la raíz llamados `chat-gonza.txt` y `pendientes.txt` con tamaños de 667 y 315 bytes respectivamente. Nuestro primer objetivo claro será conectarnos para descargarlos.
  
* **Puerto 22 (SSH):** Corre `OpenSSH 9.6p1` sobre un sistema Ubuntu Linux. De momento no nos reporta vulnerabilidades directas por versión, pero nos servirá más adelante si logramos obtener credenciales válidas.
  
* **Puerto 80 (HTTP):** Tiene un servidor web `Apache 2.4.58`. El script de nmap extrae el título de la página principal, el cual es `Russoski Coaching`. 


<img width="577" height="435" alt="image" src="https://github.com/user-attachments/assets/a9767a91-369f-49f4-98cc-9066ed6bfc06" />

## Explotacion

Como vimos que el acceso anónimo estaba permitido, me conecté al servicio ftp utilizando el usuario `anonymous` y dejando la contraseña en blanco. Una vez dentro, listé el contenido con `ls` para verificar los archivos y los descargué a mi máquina local usando el comando `mget`.

```bash
ftp 10.88.0.2
```

<img width="1268" height="377" alt="image" src="https://github.com/user-attachments/assets/282c9ad0-2696-4797-8f3b-be31a12a08b7" />

Una vez descargados los archivos a mi máquina local, utilicé el comando `cat` para leer el contenido de ambos y buscar posibles vectores de ataque o nombres de usuario.

``` bash
cat chat-gonza.txt
cat pendientes.txt
```
Al revisar `chat-gonza.txt`, vemos una conversación entre dos usuarios llamados **Gonza** y **Russoski**. Russoski menciona que tiene un vídeo subido y que guarda la URL en su ordenador en una ruta segura.

En el archivo `pendientes.txt`, que parece ser una lista de tareas de Russoski, destaca el punto número 4, donde menciona que cree que tiene **ciertos permisos habilitados que no son del todo seguros** en su equipo. Esto nos da una pista directa de que podríamos estar ante una posible vía de escalada de privilegios más adelante.

<img width="947" height="295" alt="image" src="https://github.com/user-attachments/assets/cf9fd1b1-2a9a-4f2e-b9c0-a4f8b474a358" />

Como los archivos de texto del FTP nos dejaron algunas incógnitas, decidí inspeccionar el servidor web que corría en el puerto 80 introduciendo la dirección IP en el navegador. 

Al entrar vemos una página web llamada "The Aesthetic Dream". Estuve mirando un poco por encima de qué iba la página y parecía ser simplemente un sitio web personal donde te vende un curso o asesorías online para cambiar tu físico, coincidiendo con lo que leímos en las tareas pendientes de Russoski.

<img width="1280" height="671" alt="image" src="https://github.com/user-attachments/assets/cc77b615-5ec3-45f1-9a19-b6c928c0762a" />

Recordando lo que leímos en el archivo `chat-gonza.txt`, donde Russoski mencionaba que tenía guardada una URL en una ruta segura dentro de su ordenador, decidí realizar fuzzing web para descubrir directorios ocultos utilizando la herramienta `gobuster` y el diccionario `directory-list-2.3-medium.txt`.

``` bash
gobuster dir -u http://10.88.0.2 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 50
```

El escaneo finalizó con éxito y nos reportó dos directorios muy interesantes con código de estado 301: `/backup` e `/important`. Este último llama bastante la atención debido al mensaje del chat.

<img width="810" height="306" alt="image" src="https://github.com/user-attachments/assets/72c1208a-82a0-4ec0-bd2c-8aefeaa5d9d5" />

Al acceder al directorio `/important` desde el navegador, me encontré con un indexado de directorios abierto (Directory Listing). Dentro de este directorio descubrí un archivo llamado `important.md`.

<img width="261" height="165" alt="image" src="https://github.com/user-attachments/assets/38a72f1d-2548-4d5f-8839-c6379e428596" />

Para revisar el contenido del archivo `important.md`, decidí realizar una petición utilizando la herramienta `curl` directamente desde la terminal.

``` bash
curl http://10.88.0
```

Al abrirlo, me encontré con el texto completo del famoso "Manifiesto Hacker" (La Conciencia de un Hacker). Tras leerlo detalladamente, comprobé que aparentemente no contiene ninguna pista, credencial o información útil para continuar de forma directa, por lo que parece ser texto de relleno o una distracción.

<img width="878" height="467" alt="image" src="https://github.com/user-attachments/assets/4008314c-f390-412b-9f6f-3db242dd6ee1" />

Al ver que en el archivo anterior no había nada de utilidad, decidí probar suerte inspeccionando el otro directorio que habíamos descubierto con gobuster, el cual se llamaba `/backup`. 

Dentro de este directorio encontré un archivo llamado `backup.txt`. Al abrirlo desde el navegador, descubrí un mensaje muy importante donde se revela el nombre del usuario principal para el sistema:

* **Usuario encontrado:** `russoski`
* **Nota del administrador:** Menciona que es el usuario para todos sus servicios y que debe cambiarlo pronto.

<img width="373" height="80" alt="image" src="https://github.com/user-attachments/assets/2461762a-5d33-4949-a0b5-809cea9dc2bf" />

Al tener confirmado el usuario `russoski` para todos los servicios del sistema, decidí lanzar un ataque de fuerza bruta contra el servicio SSH utilizando la herramienta `hydra` junto al diccionario `rockyou.txt` para ver si lograba dar con la contraseña.

``` bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt 10.88.0.2 ssh -t 15
```

El ataque de fuerza bruta fue exitoso y logré obtener una credencial válida para acceder al sistema de forma legítima:

* **Usuario:** `russoski`
* **Contraseña:** `iloveme`

<img width="1256" height="187" alt="image" src="https://github.com/user-attachments/assets/632289aa-07e8-4a89-a199-5c7624b19ea0" />

Una vez obtenidas las credenciales válidas (`russoski:llovene`), procedí a probar la conexión hacia el servidor SSH para intentar autenticarme en el sistema. Al introducir la contraseña, el logueo funcionó correctamente y logré obtener una shell interactiva como el usuario `russoski`.

``` bash
ssh russoski@172.17.0.2
```

<img width="491" height="288" alt="image" src="https://github.com/user-attachments/assets/49e0b0fa-27cf-4c93-8afc-8917d2df3bae" />



