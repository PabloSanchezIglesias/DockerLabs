# Tproot

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

En el primer escaneo vemos que están abiertos el puerto 21 donde corre el servicio ftp y el puerto 80 donde corre el servicio http, además se puede ver que el ttl es de 64 por lo que estamos ante una maquina Linux.

<img width="957" height="447" alt="image" src="https://github.com/user-attachments/assets/dcafe9ab-824d-4817-8876-e39c13af139c" />

Después hacemos un escaneo mas robusto centrándonos en el puerto 21 y 80 donde esta corriendo el ftp y el http para poder obtener más información.

```bash
sudo nmap -sCV -p21 10.88.0.2 -oN Puertos
```

Podemos ver que en el puerto 21 corre el servicio ftp que tiene la version vsftpd 2.3.4 y que en el puerto 80 la version es la Apache httpd 2.4.58 ((Ubuntu)).

<img width="956" height="311" alt="image" src="https://github.com/user-attachments/assets/eced20dd-ddc2-49e3-be03-be708bc7887a" />

## Explotacion

Déspues hice una busqueda en google para ver si la version del ftp es antigua y pueda tener alguna vulnerabilidad y vi que la versión ftp actual es la 3.0.5

<img width="1071" height="545" alt="image" src="https://github.com/user-attachments/assets/0ed2d839-0690-444d-8773-0de8e3878a4a" />

Al ver que la versión estaba desactualizada utilice la herramienta "searchsploit" para ver si en exploit-db había alguna vulnerabilidad.

<img width="1889" height="163" alt="image" src="https://github.com/user-attachments/assets/9c287fac-bdb6-4dd1-a10f-cb0c60490885" />

Encontré que existían dos vulnerabilidades las cuales hacían un Backdoor Command Execution y fui a exploit-db para ver el exploit.

<img width="1792" height="835" alt="image" src="https://github.com/user-attachments/assets/aac0a913-8f88-490b-a738-8005a7761366" />

Copie todo el codigo del exploit lo meti en un archivo que le llame exploit.py 

<img width="1082" height="969" alt="image" src="https://github.com/user-attachments/assets/5e030d0c-a7d2-41d4-9a87-5cdebdf393b1" />

Lo ejecuté con el parámetro "-h" para ver que parámetros había que pasarle.

<img width="636" height="167" alt="image" src="https://github.com/user-attachments/assets/46f324d3-d74c-4c14-8d20-4dfee99a32c6" />

Por último lo ejecuté y me dio una consola interactiva con privilegios de usuario root.

<img width="384" height="154" alt="image" src="https://github.com/user-attachments/assets/ee91ad07-1790-45d2-859b-1786f6da8e1d" />




