# BreakMFA — BunkerLabs

| Campo        | Detalle                                                              |
|--------------|----------------------------------------------------------------------|
| 🏷️ Plataforma | BunkerLabs                                                           |
| 💻 SO         | Linux                                                                |
| 📊 Dificultad | Media                                                                |
| 🔑 Técnicas   | IDOR, MFA Bypass, Rate Limit Bypass, Brute Force, Account Takeover  |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Análisis de la aplicación web](#2-análisis-de-la-aplicación-web)
3. [Bypass del Rate Limit](#3-bypass-del-rate-limit)
4. [Fuerza bruta al PIN MFA con FFUF](#4-fuerza-bruta-al-pin-mfa-con-ffuf)
5. [Account Takeover](#5-account-takeover)
6. [Conclusiones](#6-conclusiones)

---

## 1. Reconocimiento

Desplegamos el laboratorio y realizamos un escaneo de puertos con Nmap sobre la IP proporcionada:

![Escaneo Nmap](imgs/Pasted%20image%2020260801094813.png)

---

## 2. Análisis de la aplicación web

Accedemos al servicio web por medio del puerto "5000" con la siguiente ruta: `http://172.17.0.2:5000`

![Web principal](imgs/Pasted%20image%2020260801094948.png)

La aplicación implementa un sistema MFA que bloquea el acceso tras varios intentos fallidos — una buena medida de seguridad en teoría, pero con una implementación vulnerable.

Interceptamos la petición con **Burp Suite** y la enviamos al Repeater:

![Intercepción Burp](imgs/Pasted%20image%2020260801095051.png)
![Repeater](imgs/Pasted%20image%2020260801095347.png)

---

## 3. Bypass del Rate Limit

Modificando el campo de usuario en la petición POST del MFA — cambiando `usuario@usuario` por `admin@admin` — comprobamos que el servidor genera un código MFA para cualquier usuario indicado, revelando un **IDOR**:

![IDOR MFA](imgs/Pasted%20image%2020260801095436.png)

Para saltar el bloqueo por exceso de peticiones, añadimos la cabecera HTTP:

```
X-Forwarded-For: 10.10.10.10
```

Esto engaña al servidor haciéndole creer que cada petición proviene de una IP diferente, anulando el rate limit.

---

## 4. Fuerza bruta al PIN MFA con FFUF

Con el bypass confirmado, lanzamos un ataque de fuerza bruta sobre el PIN de 4 dígitos usando **FFUF** en modo Pitchfork con dos listas simultáneas:

- `CODEFUZZ` — Códigos PIN del 0000 al 9999
- `IPFUZZ` — IPs rotativas para la cabecera `X-Forwarded-For`

```bash
ffuf -u http://172.17.0.2:5000/mfa \
  -X POST \
  -H "Cookie: <tu_cookie_de_sesion>" \
  -H "X-Forwarded-For: IPFUZZ" \
  -d "email=admin@admin&code=CODEFUZZ" \
  -w <(seq -w 0 9999):CODEFUZZ \
  -w <(for i in $(seq 0 9999); do echo "10.10.$((i/4/256)).$((i/4%256))"; done):IPFUZZ \
  -mode pitchfork \
  -mc 302
  -fc 2554
```

![Comando FFUF](imgs/Pasted%20image%2020260801104210.png)

---

## 5. Account Takeover

Con el código válido obtenido, lo introducimos en Burp Suite. El servidor devuelve un **302 Found** confirmando el acceso.

Copiamos la URL desde **"Request in browser"**, la pegamos en el navegador y accedemos como administrador — completando el **Account Takeover**:

![Acceso admin](imgs/Pasted%20image%2020260801104509.png)
![Flag](imgs/Pasted%20image%2020260801104534.png)

---

## 6. Conclusiones

Esta máquina demuestra cómo una implementación incorrecta del MFA puede convertirse en una puerta de entrada total al sistema. La combinación de **IDOR** y **Rate Limit Bypass** mediante cabeceras HTTP permite comprometer cuentas privilegiadas sin conocer credenciales.

**Lecciones clave:**
- Validar que el código MFA generado pertenece al usuario autenticado en sesión, no al email indicado en la petición.
- Implementar rate limiting basado en sesión y no solo en IP.
- No confiar en cabeceras como `X-Forwarded-For` para identificar al cliente real.

---

*Writeup realizado con fines educativos en un entorno controlado.*
