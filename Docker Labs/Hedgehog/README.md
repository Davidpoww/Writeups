# Hedgehog — DockerLabs

| Campo        | Detalle                                                        |
|--------------|----------------------------------------------------------------|
| 🏷️ Plataforma | DockerLabs                                                     |
| 💻 SO         | Linux                                                          |
| 📊 Dificultad | Fácil                                                          |
| 🔑 Técnicas   | Nmap, Hydra, SSH, Privilege Escalation, sudo -l                |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Fuerza bruta con Hydra](#2-fuerza-bruta-con-hydra)
3. [Acceso SSH](#3-acceso-ssh)
4. [Escalada de privilegios](#4-escalada-de-privilegios)
5. [Conclusiones](#5-conclusiones)

---

## 1. Reconocimiento

Desplegamos la máquina y realizamos un escaneo de puertos con la IP que nos otrogo Docker:

```bash
sudo nmap -sV -sS -Pn -n -vvv -T5 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020261008095731.png)

El escaneo revela dos puertos abiertos:
- **22/tcp** — SSH
- **80/tcp** — HTTP (Apache)

Accedemos al puerto 80 desde el navegador y encontramos el nombre de usuario `tails` expuesto en la web. 🔍

---

## 2. Fuerza bruta con Hydra

En lugar de usar `rockyou.txt` directamente — que puede tardar horas — lo invertimos con `tac` para probar primero las contraseñas más recientes y reducir el tiempo de ataque:

```bash
tac /home/kali/rockyou.txt > ~/rockyou_reversed.txt
sed -i 's/ //g' ~/rockyou_reversed.txt
```

> 💡 `tac` es el inverso de `cat` — lee el fichero de abajo a arriba. Combinado con `sed` eliminamos espacios para evitar errores en Hydra.

Lanzamos el ataque contra SSH con el usuario descubierto:

```bash
hydra -l tails -P ~/rockyou_reversed.txt ssh://172.17.0.2/ -t 5
```

![Hydra resultado](imgs/Pasted%20image%2020261008104055.png)

✅ Hydra encuentra la contraseña de `tails`.

---

## 3. Acceso SSH

Accedemos a la máquina con las credenciales obtenidas:

```bash
ssh tails@172.17.0.2
```

![Acceso SSH](imgs/Pasted%20image%2020261008104152.png)

---

## 4. Escalada de privilegios

Una vez dentro, aplicamos la metodología estándar de reconocimiento post-explotación:

```bash
whoami                          	# Usuario actual
cat /etc/passwd                 	# Todos los usuarios del sistema
cat /etc/passwd | grep "/bin/bash"  	# Solo usuarios con shell activa
sudo -l                         	# Comandos que podemos ejecutar como otro usuario
```

![Reconocimiento usuarios](imgs/Pasted%20image%2020261008104703.png)

Filtramos los usuarios con `/bin/bash` — son los que nos interesan como objetivo de escalada:

![Usuarios con bash](imgs/Pasted%20image%2020261008104952.png)

`sudo -l` revela que `tails` puede ejecutar comandos como `sonic`. Aprovechamos esto para pivotar:

![sudo -l resultado](imgs/Pasted%20image%2020261008105510.png)

```bash
sudo -u sonic /bin/bash
```

![Acceso como Sonic](imgs/Pasted%20image%2020261008105700.png)

Repetimos el proceso con `sonic` — `sudo -l` muestra que puede ejecutar comandos como `root`:

```bash
sudo -u root /bin/bash
```

![Root obtenido](imgs/Pasted%20image%2020261008110258.png)

✅ Escalada completa: `tails` → `sonic` → `root`.

---

## 5. Conclusiones

Una máquina que ilustra perfectamente la escalada de privilegios en cadena mediante `sudo -l`. Un usuario mal configurado es suficiente para comprometer todo el sistema.

**Lecciones clave:**
- 🔒 No exponer nombres de usuario en servicios web accesibles.
- 🔒 Auditar los permisos `sudo` de cada usuario — aplicar el principio de mínimo privilegio.
- 🔒 Un pivote de usuario a usuario puede ser tan peligroso como un acceso directo a root.

---

*Writeup realizado con fines educativos en un entorno controlado.* 🛡️
