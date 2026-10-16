# First Hacking — DockerLabs

| Campo        | Detalle                                              |
|--------------|------------------------------------------------------|
| 🏷️ Plataforma | DockerLabs                                           |
| 💻 SO         | Linux                                                |
| 📊 Dificultad | Fácil                                                |
| 🔑 Técnicas   | Nmap, Searchsploit, vsftpd 2.3.4 (CVE), Python2 RCE |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Búsqueda de vulnerabilidades](#2-búsqueda-de-vulnerabilidades)
3. [Explotación](#3-explotación)
4. [Conclusiones](#4-conclusiones)

---

## 1. Reconocimiento

Desplegamos la máquina y realizamos un escaneo completo de puertos:

```bash
sudo nmap -sS -sV -sC -Pn -n -vvv -T5 -p- 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020261016163707.png)

El escaneo revela el **puerto 21 (FTP)** abierto, corriendo el servicio **vsftpd 2.3.4**.

> 💡 Regla de oro: siempre que identifiques una versión concreta de un servicio, búscala en bases de datos de vulnerabilidades — puede tener una CVE conocida y explotable.

---

## 2. Búsqueda de vulnerabilidades

Sin salir de la terminal, usamos **Searchsploit** para buscar exploits disponibles para esta versión:

```bash
searchsploit vsftpd 2.3.4
```

![Searchsploit resultado](imgs/Pasted%20image%2020261016164141.png)

Searchsploit devuelve dos opciones de explotación — un archivo `.rb` (Metasploit) y un `.py` (standalone). Optamos por el script Python para ejecutarlo directamente sin depender de Metasploit.

Descargamos el exploit al directorio de trabajo con la flag `-m`:

```bash
searchsploit -m unix/remote/49757.py
```

![Descarga exploit](imgs/Pasted%20image%2020261016164619.png)

---

## 3. Explotación

Ejecutamos el exploit contra la máquina objetivo:

```bash
python2 49757.py 172.17.0.2
```

> ⚠️ Este script requiere **Python 2** — con `python` o `python3` fallará.

![Root obtenido](imgs/Pasted%20image%2020261016165237.png)

✅ El exploit se ejecuta correctamente y obtenemos acceso directo como **root** — máquina comprometida con privilegios máximos desde el primer acceso.

---

## 4. Conclusiones

Esta máquina demuestra el riesgo real de mantener servicios con versiones antiguas y vulnerables expuestas. vsftpd 2.3.4 tiene una backdoor conocida desde 2011 — su sola presencia en un sistema es suficiente para comprometer la máquina por completo.

**Lecciones clave:**
- 🔒 Mantener todos los servicios actualizados — las versiones antiguas son objetivos directos.
- 🔒 Deshabilitar servicios que no sean estrictamente necesarios, especialmente FTP.
- 🔍 Searchsploit permite investigar vulnerabilidades conocidas sin conexión a internet — imprescindible en pentesting.

---

*Writeup realizado con fines educativos en un entorno controlado.* 🛡️
