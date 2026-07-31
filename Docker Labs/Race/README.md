# Race — DockerLabs

| Campo        | Detalles                                                              |
|--------------|-----------------------------------------------------------------------|
| 🏷️ Plataforma | DockerLabs                                                           |
| 💻 SO         | Linux                                                                |
| 📊 Dificultad | Media                                                                |
| 🔑 Técnicas   | Race Condition, Burp Suite, SSH, Escalada de Privilegio              |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Explotación Web — Niveles 1, 2 y 3](#2-explotación-web)
3. [Acceso SSH y Escalada de Privilegios](#3-acceso-ssh-y-escalada-de-privilegios)
4. [Conclusiones](#4-conclusiones)

---

## 1. Reconocimiento

Desplegamos la máquina y obtenemos su dirección IP. Realizamos un escaneo de puertos con Nmap:

![Escaneo Nmap](imgs/Pasted%20image%2020260730171349.png)

El escaneo revela dos puertos abiertos:
- **22/tcp** — SSH
- **5000/tcp** — HTTP

![Puertos abiertos](imgs/Pasted%20image%2020260730171517.png)

Accedemos al servicio web en `http://<IP>:5000`:

![Web principal](imgs/Pasted%20image%2020260730171653.png)

---

## 2. Explotación Web

### Nivel 1 — Race Condition en límite de clicks

La aplicación limita las acciones a **3 clicks**. La vulnerabilidad consiste en enviar múltiples peticiones simultáneas antes de que el servidor pueda procesarlas — una técnica conocida como **Race Condition**.

Activamos **FoxyProxy** y configuramos **Burp Suite** añadiendo la IP de la víctima al scope:

![Burp Suite scope](imgs/Pasted%20image%2020260730172526.png)

Interceptamos la petición POST generada al hacer click en "ejecutar acción" y la enviamos al **Repeater**. Creamos un grupo de 15 peticiones y las enviamos en paralelo con `Ctrl+R`:

![Repeater grupo](imgs/Pasted%20image%2020260730174753.png)
![Envío paralelo](imgs/Pasted%20image%2020260730174848.png)
![Resultado nivel 1](imgs/Pasted%20image%2020260730175001.png)

El servidor no puede procesar todas las peticiones a tiempo — saltamos el límite y completamos el nivel 1:

![Nivel 1 completado](imgs/Pasted%20image%2020260730175123.png)

---

### Nivel 2 — Race Condition en cupón de descuento

El sistema impide canjear el mismo cupón dos veces. Aplicamos la misma técnica: interceptamos la petición POST del canje en Burp Suite:

![Petición cupón](imgs/Pasted%20image%2020260730175257.png)
![Burp Suite cupón](imgs/Pasted%20image%2020260730175418.png)

Agrupamos las peticiones y las enviamos en paralelo:

![Envío paralelo cupón](imgs/Pasted%20image%2020260730180252.png)

Al recargar la página obtenemos saldo adicional, completando el nivel 2:

![Nivel 2 completado](imgs/Pasted%20image%2020260730180331.png)
![Saldo obtenido](imgs/Pasted%20image%2020260730180356.png)

---

### Nivel 3 — Race Condition en exchange de criptomonedas

El exchange solo permite una compra por los fondos disponibles. Interceptamos la petición POST de compra:

![Exchange](imgs/Pasted%20image%2020260730180511.png)
![Petición compra](imgs/Pasted%20image%2020260730180728.png)

Enviamos múltiples peticiones en paralelo antes de que el servidor actualice el saldo:

![Paralelo exchange](imgs/Pasted%20image%2020260730181151.png)

Al recargar obtenemos las credenciales SSH:

![Credenciales SSH](imgs/Pasted%20image%2020260730181236.png)

---

## 3. Acceso SSH y Escalada de Privilegios

Con las credenciales obtenidas accedemos por SSH:

![Acceso SSH](imgs/Pasted%20image%2020260730181431.png)

Realizamos reconocimiento básico del sistema:

```bash
whoami        # Ver usuario actual
ls            # Listar contenido del directorio
cat README.txt  # Leer instrucciones
ps aux        # Ver procesos en ejecución
```

Entre los procesos detectamos un script ejecutándose en segundo plano:

```
/bin/bash /usr/local/bin/backup_script.sh
```

Explotamos la vulnerabilidad en dicho script siguiendo las instrucciones del `README.txt`, obteniendo acceso root y la flag final:

![Root obtenido](imgs/Pasted%20image%2020260730182856.png)
![Flag](imgs/Pasted%20image%2020260730182955.png)

---

## 4. Conclusiones

Esta máquina demuestra el impacto real de las **Race Conditions** en aplicaciones web. Un control de límite implementado solo en el servidor sin mecanismos de bloqueo atómico es explotable enviando peticiones concurrentes.

**Lecciones clave:**
- Implementar bloqueos atómicos o transacciones en operaciones críticas.
- Validar el estado del recurso antes y después de cada operación.
- Limitar la tasa de peticiones por usuario con rate limiting estricto.

---

*Writeup realizado con fines educativos en un entorno controlado.*
