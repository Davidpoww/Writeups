# 🛡️ Writeups — Entender al atacante para defender mejor

Resolución documentada de máquinas y laboratorios de hacking ético. Cada writeup sigue el ataque paso a paso y termina con la **visión defensiva**: qué evidencias deja, cómo se detecta y cómo se corrige.

> Perfil: Técnico IT junior orientado a **Blue Team / SOC** · [Portfolio](https://davidpoww.github.io) · [LinkedIn](https://www.linkedin.com/in/davidpow)

---

## 📋 Máquinas resueltas

| Máquina | Plataforma | SO | Dificultad | Técnicas clave |
|---|---|---|---|---|
| [Microchoft](./The%20Hacker%20Labs/Microchoft) | TheHackersLabs | Windows 7 | Principiante | SMB, EternalBlue (MS17-010), Metasploit |
| [First Hacking](./Docker%20Labs/First%20Hacking) | DockerLabs | Linux | Fácil | vsftpd 2.3.4 (CVE), RCE |
| [Hedgehog](./Docker%20Labs/Hedgehog) | DockerLabs | Linux | Fácil | Fuerza bruta SSH, abuso de sudo |
| [ClientBypass](./Bunker%20Labs/ClientBypass) | BunkerLabs | Linux | Fácil | Manipulación de respuesta, bypass OTP/MFA |
| [Asucar](./Docker%20Labs/Asucar) | DockerLabs | Linux | Media | WordPress, LFI, Hydra, privesc con PuTTYgen |
| [Grooti](./Docker%20Labs/Grooti) | DockerLabs | Linux | Media | Gobuster, SQL, Hydra, SUID |
| [Race](./Docker%20Labs/Race) | DockerLabs | Linux | Media | Race condition, Burp Suite |
| [BreakMfa](./Bunker%20Labs/BreakMfa) | BunkerLabs | Linux | Media | IDOR, bypass de MFA y rate limit |
| [Server](./Bunker%20Labs/Server) | BunkerLabs | Linux | Media | SSRF, fuzzing web, bypass de blacklist |

---

## 🧭 Estructura de cada writeup

1. 🔍 Reconocimiento y enumeración
2. 🎯 Explotación
3. ⬆️ Escalada de privilegios
4. 🔵 **Detección y mitigación**: logs generados, cómo lo detectaría un SOC (Wazuh / MITRE ATT&CK) y corrección
5. 📌 Conclusiones

---

## ⚙️ Objetivo

Practicar técnicas ofensivas reales para entender **qué buscar como analista**: patrones en logs, configuraciones débiles y medidas de hardening en Linux, Windows y aplicaciones web.

---

*Todos los writeups se realizan en entornos controlados y con fines educativos.*
