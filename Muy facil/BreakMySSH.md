# BreakMySSH

Dificultad: Muy fácil

Link: https://dockerlabs.es/

## Summary

- [Reconocimiento](#reconocimiento)
- [Explotacion](#explotacion)
- [Escalada de privilegios](#escalada-de-privilegios)

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



<img width="1916" height="991" alt="image" src="https://github.com/user-attachments/assets/61e202b3-190a-4fcb-9550-d25c4bbb93ee" />


