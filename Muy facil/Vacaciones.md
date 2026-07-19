# Vacaciones

Dificultad: Muy fácil

Link: https://dockerlabs.es/

## Summary

- [Reconocimiento](#reconocimiento)
- [Explotacion](#explotacion)
- [Escalada de privilegios](#escalada-de-privilegios)

## Reconocimiento
Lo primero que hice nada mas desplegar el laboratorio fue empezar con la fase de reconocimiento utilizando la herramienta nmap.

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.88.0.2 -oG escaneo
```

En el primer escaneo vemos que estan abiertos el puerto 22 en el que corre el ssh y tambien un puerto 80 en el que corre un protocolo http. Ademas de que se puede ver que el ttl es de 64 por lo que estamos ante una máquina Linux.

<img width="650" height="298" alt="image" src="https://github.com/user-attachments/assets/0a66e38c-af2b-4a7b-9a84-0961aa4f9233" />

Después hacemos un escaneo mas robusto centrandonos en los dos puertos que nos ha mostrado el escaneo anterior utilizando el siguiente comando.

```bash
sudo nmap -sCV -p22,80 10.88.0.2 -oN allPorts
```
Podemos ver en este escaneo que el puerto 22 utiliza OpenSSH 7.6p1 y que la versión del paquete es 4ubuntu0.7 por lo que esto nos confirma que estamos antes una máquina Ubuntu Linux.
Llendo ahora con el puerto 80 vemos que utiliza Apache httpd 2.4.29.

<img width="620" height="255" alt="image" src="https://github.com/user-attachments/assets/f37a9da6-967b-4e3d-8c37-a027f7013bc6" />

Una vez hecho un poco de reconocimiento con la herramienta nmap voy al navegador para ver que hay en la aplicación web que esta en el puerto 80 y me encuentro una pagina totalmente en blanco.

<img width="1279" height="551" alt="image" src="https://github.com/user-attachments/assets/33537cfc-3a26-4ca7-a682-6788877ba3b4" />

Al ver esto uso la herramienta que tengo como extensión del navegador llamada Wappalyzer para ver si me reporta alguna información interesante de la página pero solo me reporta lo mismo que me reporto la herramienta nmap.

<img width="252" height="275" alt="image" src="https://github.com/user-attachments/assets/61f86757-261c-47da-b7e4-1464bac510aa" />

El siguiente paso que se me ocurre es ver el código fuente de la página web y observamos que hay un comentario HTML

```html
<!-- De: Juan Para: Camilo, te he dejado un correo es importante..-->
```
<img width="463" height="67" alt="image" src="https://github.com/user-attachments/assets/e4817c47-a58a-4e67-ac7a-483fd0e86865" />

Al ver ese comentarior en el código fuente pensé que podían ser nombres de usuarios y realicé con la herramienta hydra un ataque de fuerza bruta para sacar la contraseña y poder conectarme por ssh

```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://10.88.0.2 -t 15
```
<img width="1267" height="178" alt="image" src="https://github.com/user-attachments/assets/f5cf0fd8-30b5-483a-9ecc-edb3b181a452" />

## Explotacion

Para comprobar que las credenciales son correctas nos conectamos por ssh con lo que nos ha reportado la herramienta hydra.

<img width="517" height="149" alt="image" src="https://github.com/user-attachments/assets/f4415310-925a-4a57-913a-05f8c55659b3" />

Después estuve aplicando un poco de reconocimiento y no encontré nada interesante, solo tres carpetas con los nombres Camilo, Juan y Pedro.

<img width="527" height="269" alt="image" src="https://github.com/user-attachments/assets/a4d77ebd-7969-4512-8737-ac6229afae5f" />

Viendo que no encontraba nada revisé nuevamente el comentario y como se hablaba algo de un correo electrónico inspeccioné el directorio de correo.

```bash
ls -la /var/mail
```
Dentro de este directorio logré identificar una carpeta que pertenecía al usuario Camilo. Al inspeccionar su interior, encontré un archivo llamado `correo.txt` que contenía un mensaje muy interesante: explicaba que el usuario Juan se iba de vacaciones y le había dejado su contraseña apuntada por si Camilo necesitaba acceder al sistema en su ausencia.
Gracias a esta fuga de información, obtuve unas credenciales válidas para el usuario Juan, lo que me permitió pivotar y acceder a su cuenta de inmediato.

<img width="838" height="199" alt="image" src="https://github.com/user-attachments/assets/53d1751d-9d8a-470e-b9aa-0798643ac9d5" />

## Escalada de privilegios

Una vez que logré cambiar al usuario Juan, lo primero que hice fue revisar sus privilegios de sudo disponibles en el sistema para ver si existía alguna vía de escalada.

```bash
sudo -l
```
El análisis de los permisos reveló una configuración insegura: el usuario Juan tiene permisos para ejecutar el binario Ruby como root sin necesidad de proporcionar ninguna contraseña.

<img width="787" height="178" alt="image" src="https://github.com/user-attachments/assets/078ccd3d-354c-4f50-9e16-f06242780581" />

Sabiendo esto, realicé una búsqueda rápida en la web de GTFOBins para encontrar un comando adecuado que me permitiera abusar de esta configuración y spawnear una shell interactiva con máximos privilegios:

```bash
sudo ruby -e 'exec "/bin/sh"'
```
Al ejecutar la línea, el sistema me otorgó de forma inmediata una consola como el usuario administrador, logrando el control total de la máquina virtual y completando con éxito el laboratorio.

<img width="203" height="57" alt="image" src="https://github.com/user-attachments/assets/d0ec324a-388d-4c00-8a18-611af667a0e7" />



