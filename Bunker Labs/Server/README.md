# Server — BunkerLabs

| Campo        | Detalle                                                        |
|--------------|----------------------------------------------------------------|
| 🏷️ Plataforma | BunkerLabs                                                     |
| 💻 SO         | Linux                                                          |
| 📊 Dificultad | Media                                                          |
| 🔑 Técnicas   | SSRF, Fuzzing Web, Gobuster, Dirb, Bypass de Blacklist         |

---

## 📖 ¿Qué es SSRF?

> El **Server-Side Request Forgery (SSRF)** es una vulnerabilidad que permite a un atacante manipular un servidor para que realice peticiones HTTP no autorizadas — accediendo a recursos internos que deberían ser inaccesibles desde el exterior.

---

## Índice

1. [🔍 Reconocimiento](#1-reconocimiento)
2. [🌐 Análisis de la aplicación](#2-análisis-de-la-aplicación)
3. [🗂️ Fuzzing — Descubrimiento de rutas](#3-fuzzing--descubrimiento-de-rutas)
4. [💥 Explotación — Nivel 1](#4-explotación--nivel-1)
5. [⚡ Explotación — Nivel 2 y Bypass de Blacklist](#5-explotación--nivel-2-y-bypass-de-blacklist)
6. [🏁 Flag](#6-flag)
7. [📌 Conclusiones](#7-conclusiones)

---

## 1. 🔍 Reconocimiento

Comenzamos con un escaneo completo de puertos:

```bash
sudo nmap -T5 -sS -sC -sV -vvvv -Pn -n 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020260804001006.png)

---

## 2. 🌐 Análisis de la aplicación

Accedemos al servicio web descubierto por Nmap. Regla de oro en pentesting web:

> 💡 **Siempre que veas un campo donde introducir una URL, piensa en SSRF.**

![Web principal](imgs/Pasted%20image%2020260804001123.png)

---

## 3. 🗂️ Fuzzing — Descubrimiento de rutas

Lanzamos **Gobuster** para descubrir directorios y ficheros ocultos:

```bash
gobuster dir -u http://172.17.0.2:3000/ \
  -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt \
  -x bak,html,pgp,txt,py,sh
```

![Gobuster resultado](imgs/Pasted%20image%2020260804001922.png)

Gobuster descubre rutas internas. Al intentar acceder desde el navegador, la aplicación las bloquea. Sin embargo, al sustituir la IP por `localhost` — simulando que la petición viene desde el propio servidor — el error cambia a **"directorio interno no accesible"**, confirmando que existe pero está restringido:

![Intento acceso directo](imgs/Pasted%20image%2020260804002149.png)
![Acceso con localhost](imgs/Pasted%20image%2020260804002213.png)
![Cambio de error](imgs/Pasted%20image%2020260804002436.png)

> 🔎 **El cambio de error es clave** — nos indica que el servidor procesa la petición pero la filtra internamente, confirmando la vulnerabilidad SSRF.

---

## 4. 💥 Explotación — Nivel 1

Hacemos fuzzing específico sobre el directorio `/internal`:

```bash
gobuster dir -u http://172.17.0.2:3000/internal \
  -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt \
  -x bak,html,pgp,txt,py,sh
```

![Gobuster /internal](imgs/Pasted%20image%2020260804003022.png)

Encontramos el subdirectorio `/admin`. Lo introducimos en el campo de verificación de URL usando `localhost` para simular la petición interna:

```
http://localhost:3000/internal/admin
```

![Nivel 1 completado](imgs/Pasted%20image%2020260804003311.png)

✅ **Nivel 1 completado.**

---

## 5. ⚡ Explotación — Nivel 2 y Bypass de Blacklist

El nivel 2 presenta un sistema que mide el rendimiento de sitios web. Al intentar reutilizar `localhost`, el servidor lo bloquea — tiene una **blacklist** con términos como `localhost`, `127.0.0.1`, `0.0.0.0`, etc.

> 💡 **Truco:** El valor `0` en una URL actúa como alias de `0.0.0.0` (equivalente a localhost) y no siempre está en la blacklist del servidor.

Introducimos `http://0:3000` para bypassear el filtro:

![Bypass blacklist](imgs/Pasted%20image%2020260804003853.png)

Con acceso interno confirmado, lanzamos **Dirb** para descubrir rutas desde esa URL:

```bash
dirb http://172.17.0.2:3000
```

![Dirb resultado](imgs/Pasted%20image%2020260804004110.png)

Dirb descubre el directorio `/r`. Lo combinamos con el bypass y agregamos `/r/admin`:

```
http://0:3000/r/admin
```

Lo introducimos en el verificador de la aplicación y conseguimos acceso al recurso interno:

![Flag obtenida](imgs/Pasted%20image%2020260804005221.png)

✅ **Nivel 2 completado — máquina vulnerada.**

---

## 7. 📌 Conclusiones

Esta máquina demuestra cómo el SSRF permite acceder a recursos internos de un servidor abusando de su capacidad para realizar peticiones HTTP. La blacklist es una medida insuficiente si no se validan correctamente todas las representaciones posibles de direcciones locales.

**Lecciones clave:**
- 🔒 Usar listas blancas en lugar de listas negras para validar URLs.
- 🔒 Bloquear todas las representaciones de localhost: `127.0.0.1`, `0.0.0.0`, `0`, `[::]`, etc.
- 🔒 No permitir que usuarios externos controlen el destino de peticiones internas del servidor.

---

*Writeup realizado con fines educativos en un entorno controlado.* 🛡️
