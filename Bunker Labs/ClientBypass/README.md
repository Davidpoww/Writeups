# ClientBypass — BunkerLabs

| Campo        | Detalle                                                        |
|--------------|----------------------------------------------------------------|
| 🏷️ Plataforma | BunkerLabs                                                     |
| 💻 SO         | Linux                                                          |
| 📊 Dificultad | Fácil                                                          |
| 🔑 Técnicas   | Response Manipulation, MFA Bypass, Burp Suite, OTP Bypass      |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Análisis de la aplicación](#2-análisis-de-la-aplicación)
3. [Manipulación de respuesta del servidor](#3-manipulación-de-respuesta-del-servidor)
4. [Conclusiones](#4-conclusiones)

---

## 1. Reconocimiento

Escaneamos los puertos de la máquina objetivo:

```bash
sudo nmap -T5 -sS -sC -sV -vvvv -Pn -n 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020260801175142.png)

El escaneo revela el puerto **8000/tcp** abierto con un servicio web.

---

## 2. Análisis de la aplicación

Accedemos a `http://172.17.0.2:8000` — si no carga correctamente, forzamos recarga con `Ctrl+Shift+R`.

La aplicación presenta un panel de login que solicita usuario, contraseña y un **código OTP** como segundo factor de autenticación.

Activamos **Burp Suite** para interceptar el tráfico:

![Panel login](imgs/Pasted%20image%2020260801175902.png)
![Interceptación Burp](imgs/Pasted%20image%2020260801180028.png)

Al analizar la respuesta del servidor observamos un campo con valor `false` — indicando que el OTP no es válido. La clave está en que el servidor delega la validación al cliente, lo que lo hace vulnerable a manipulación.

---

## 3. Manipulación de respuesta del servidor

Configuramos Burp Suite para interceptar la **respuesta del servidor** antes de que llegue al navegador:

1. Click derecho sobre la petición → **Do intercept** → **Response to this request**
2. Introducimos un código OTP cualquiera y damos **Forward**
3. En la respuesta interceptada cambiamos `false` por `true`
4. Damos **Forward** dos veces para enviar la respuesta manipulada al navegador

![Respuesta manipulada](imgs/Pasted%20image%2020260801180338.png)

El servidor acepta la respuesta modificada — el panel OTP queda bypasseado:

![OTP bypasseado](imgs/Pasted%20image%2020260801180739.png)

Accedemos como administrador y obtenemos la flag:

![Flag](imgs/Pasted%20image%2020260801180828.png)

---

## 4. Conclusiones

Esta máquina ilustra un fallo crítico de diseño: **delegar la validación de seguridad al cliente**. Cualquier control que pueda ser modificado por el usuario antes de llegar al servidor es bypasseable.

**Lecciones clave:**
- Validar siempre el OTP en el servidor, nunca en el cliente.
- No basar decisiones de autenticación en valores booleanos enviados desde el frontend.
- Implementar firmas o tokens de integridad en las respuestas del servidor.

---

*Writeup realizado con fines educativos en un entorno controlado.*
