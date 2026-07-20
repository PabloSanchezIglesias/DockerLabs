# FirstHacking

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
En el primer escaneo vemos que estan abiertos el puerto 21 donde corre el servicio ftp, además se puede ver que el ttl es de 64 por lo que estamos ante una maquina Linux

<img width="575" height="297" alt="image" src="https://github.com/user-attachments/assets/3e735b42-4cd8-4d32-99f8-c2ed5f657e78" />

Después hacemos un escaneo mas robusto centrandonos en el puerto 21 donde esta corriendo el ftp para poder obtener más información

```bash
sudo nmap -sCV -p21 10.88.0.2 -oN allPorts
```
Podemos ver que el ftp su version es la vsftpd 2.3.4

<img width="574" height="152" alt="image" src="https://github.com/user-attachments/assets/91e3b594-b694-496e-b69a-51fba61b7d48" />

## Explotacion

Déspues hice una busqueda en google para ver si la version del ftp es antigua y pueda tener alguna vulnerabilidad y vi que la versión ftp actual es la 3.0.5

<img width="810" height="455" alt="image" src="https://github.com/user-attachments/assets/8ed2f3cc-ee89-45e2-b0ef-775a8017f3f7" />

Al ver eso busqué algún repositorio en github que se aprovechase de esa vulnerabilidad para explotarla y encontre este

```url
https://github.com/Hellsender01/vsftpd_2.3.4_Exploit
```
<img width="1279" height="551" alt="image" src="https://github.com/user-attachments/assets/6dafa1ed-33f6-480f-915c-7a0d10d5df60" />

Por último al ejecutar el script en python nos devuelve una shell interactiva con privilegios de root directamente

<img width="452" height="235" alt="image" src="https://github.com/user-attachments/assets/a59d7bd0-2754-4836-bc09-d2c83d5a6362" />





