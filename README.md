<div align="center">

# Informe EP2 — Comunicaciones Unificadas

![Badge](https://img.shields.io/badge/Asignatura-CUY5132-0078D4?style=for-the-badge)
![Badge](https://img.shields.io/badge/Evaluaci%C3%B3n-EP2-28a745?style=for-the-badge)
![Badge](https://img.shields.io/badge/Secci%C3%B3n-002v-e63946?style=for-the-badge)
![Badge](https://img.shields.io/badge/DUOC%20UC-Mayo%202026-1F4E79?style=for-the-badge)

---

| Campo | Detalle |
|:---|:---|
| **Institución** | DUOC UC |
| **Asignatura** | CUY5132 - Comunicaciones Unificadas |
| **Evaluación** | Experiencia Práctica 2 (EP2) — Aseguramiento VoIP en Cloud |
| **Sección** | 002v |
| **Integrantes** | Abraham Castro · Bárbara Saavedra |
| **Docente** | andres antonio galvez solorza |
| **Fecha** | Mayo 2026 |

</div>

---

---

## 📋 Tabla de Contenidos

- [Arquitectura Implementada](#-arquitectura-implementada)
- [1. Verificación de Asterisk](#1-verificación-de-asterisk)
- [2. Verificación de Kamailio](#2-verificación-de-kamailio)
- [3. Verificación TLS](#3-verificación-tls)
- [4. Registro de Softphone](#4-registro-de-softphone)
- [5. Prueba de Llamada](#5-prueba-de-llamada)
- [6. Verificación de RTP (Audio)](#6-verificación-de-rtp-audio)
- [7. Verificación de Cifrado TLS](#7-verificación-de-cifrado-tls)
- [Conclusión](#-conclusión)
- [Resumen de IPs y Credenciales](#-resumen-de-ips-y-credenciales)

---

## 🏗 Arquitectura Implementada

```
┌─────────────────────────────────────────┐
│   PC / MicroSIP — Extensión 1001/1002   │
│            (Teletrabajador)             │
└──────────────────┬──────────────────────┘
                   │  🔒 TLS puerto 5061 + SRTP
                   │     (Tráfico CIFRADO)
                   ▼
┌─────────────────────────────────────────┐
│       VM-Kamailio — SBC                 │
│   IP Pública:  3.82.14.233              │
│   IP Privada:  172.31.35.212            │
│   Rol: Perímetro de seguridad           │
└──────────────────┬──────────────────────┘
                   │  UDP puerto 5060
                   │  (Red privada AWS)
                   ▼
┌─────────────────────────────────────────┐
│       VM-Asterisk — PBX                 │
│   IP Pública:  54.205.26.179            │
│   IP Privada:  172.31.37.133            │
│   Rol: Central telefónica               │
└─────────────────────────────────────────┘
```

### Tabla de Infraestructura

| VM | Software | IP Pública | IP Privada | Rol |
|:---|:---:|:---:|:---:|:---|
| VM-Asterisk | Asterisk | `54.205.26.179` | `172.31.37.133` | PBX (central telefónica) |
| VM-Kamailio | Kamailio | `3.82.14.233`   | `172.31.35.212` | SBC (perímetro de seguridad) |

---

## 1. Verificación de Asterisk

> Se valida que el servicio Asterisk esté activo y funcionando correctamente en **VM-Asterisk** (`54.205.26.179`).

### 1.1 Estado del servicio Asterisk

```bash
sudo systemctl status asterisk
```

**Resultado esperado:**
```
● asterisk.service - LSB: Asterisk PBX
   Active: active (running)
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="938" height="333" alt="Captura de pantalla 2026-05-29 204956" src="https://github.com/user-attachments/assets/46aa9837-7ba7-4c51-b8b8-ceddda8ffc59" />


---

### 1.2 Extensiones PJSIP configuradas

```bash
sudo asterisk -rx "pjsip show endpoints"
```

**Resultado esperado:**
```
Endpoint: 1001   Unavailable   0 of inf
Endpoint: 1002   Unavailable   0 of inf
Endpoint: 1003   Unavailable   0 of inf
Objects found: 3
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="1897" height="618" alt="Captura de pantalla 2026-05-29 205131" src="https://github.com/user-attachments/assets/a73b6fb7-0e21-4bbd-bfe4-7c8826f5c6c3" />


---

### 1.3 Consola interactiva de Asterisk

```bash
sudo asterisk -rvvv
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="957" height="222" alt="Captura de pantalla 2026-05-29 205451" src="https://github.com/user-attachments/assets/262448ee-dca9-434d-a3ad-ce20afaabcc6" />



---

## 2. Verificación de Kamailio

> Se valida que Kamailio (SBC) esté corriendo sin errores en **VM-Kamailio** (`3.82.14.233`).

### 2.1 Estado del servicio Kamailio

```bash
sudo systemctl status kamailio
```

**Resultado esperado:**
```
● kamailio.service
   Active: active (running)
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="935" height="243" alt="Captura de pantalla 2026-05-29 205626" src="https://github.com/user-attachments/assets/4940f606-12c1-4217-96a3-7cd26ea1dcea" />


---

### 2.2 Validación de sintaxis de configuración

> ⚠️ **IMPORTANTE:** Siempre verificar la sintaxis **antes** de reiniciar el servicio para evitar que quede caído.

```bash
sudo kamailio -f /etc/kamailio/kamailio.cfg -c 2>&1 | tail -8
```

**Resultado esperado:**
```
config file ok, exiting...
Listening on
  udp: 0.0.0.0:5060
  tls: 0.0.0.0:5061
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="937" height="182" alt="Captura de pantalla 2026-05-29 205733" src="https://github.com/user-attachments/assets/bcd90c9f-c8c6-466a-9876-9336a0ac4f3b" />


---

### 2.3 Verificación de puertos activos

```bash
sudo ss -tlnp | grep 506
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="935" height="60" alt="Captura de pantalla 2026-05-29 205846" src="https://github.com/user-attachments/assets/919a8c78-81ed-4ee0-9adf-4d96a16e3b56" />


---

## 3. Verificación TLS

> Se comprueba que el servicio esté escuchando en el puerto seguro **5061 (TLS)** para la señalización cifrada.

### 3.1 Puerto TLS 5061 activo

```bash
sudo ss -tlnp | grep 5061
```

**Resultado esperado:**
```
LISTEN  0  1024  0.0.0.0:5061  0.0.0.0:*  users:(("kamailio",...))
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="933" height="60" alt="Captura de pantalla 2026-05-29 210003" src="https://github.com/user-attachments/assets/ec27cf9c-0a68-4c8f-a455-3f53e1e0c0c7" />


---

### 3.2 Certificados TLS generados

```bash
ls -la /etc/kamailio/certs/
```

**Resultado esperado:**
```
-rw-r----- 1 root kamailio 1342 May 27 00:18 kamailio.crt
-rw-r----- 1 root kamailio 1704 May 27 00:18 kamailio.key
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="941" height="119" alt="Captura de pantalla 2026-05-29 210132" src="https://github.com/user-attachments/assets/d27c80ea-2711-4586-beb2-65336c95afce" />


---

### 3.3 Verificación alternativa de puertos

```bash
sudo ss -tlnp | grep 5061
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="933" height="57" alt="Captura de pantalla 2026-05-29 210440" src="https://github.com/user-attachments/assets/2472ee6f-488a-4df5-bc1d-7cb3b9c977c3" />


---

## 4. Registro de Softphone

> Se valida que MicroSIP se registre correctamente a través del SBC Kamailio usando **TLS** y **SRTP**.

### 4.1 Configuración de MicroSIP — Extensión 1001

| Campo en MicroSIP | Valor |
|:---|:---|
| Servidor SIP | `3.82.14.233:5061` |
| Nombre de usuario | `1001` |
| Dominio | `3.82.14.2335` |
| Contraseña | `pass1001` |
| Cifrado de medios | `Obligatorio SRTP (RTP/SAVP)` |
| Transporte | `TLS` |

### 4.2 Monitoreo de registro REGISTER en logs

Desde **VM-Kamailio**, monitorear el registro del softphone:

```bash
sudo tail -f /var/log/syslog | grep REGISTER
```

📸 **EVIDENCIA — Captura de pantalla:**

> _[ Insertar captura aquí ]_

---

### 4.3 Contactos registrados en Asterisk

Desde **VM-Asterisk**, verificar que la extensión aparezca registrada:

```bash
sudo asterisk -rx "pjsip show contacts"
```

📸 **EVIDENCIA — Captura de MicroSIP mostrando "En línea" + salida del comando:**

> _[ Insertar captura aquí ]_

---

## 5. Prueba de Llamada

> Se verifica que las llamadas se establezcan correctamente. Se llama al **echo test (extensión 9999)** que reproduce la propia voz del llamante.

### 5.1 Monitoreo de logs durante la llamada

Desde **VM-Asterisk**, monitorear en tiempo real mientras se realiza la llamada:

```bash
sudo tail -f /var/log/syslog
```

> 📞 Marcar **9999** en MicroSIP y hacer clic en **Llamar**. Se debe escuchar la propia voz.

📸 **EVIDENCIA — Captura de pantalla de los logs:**

> _[ Insertar captura aquí ]_

---

### 5.2 Consola Asterisk durante la llamada

```bash
sudo asterisk -rvvv
```

Se verifica el flujo: `INVITE → 100 Trying → 200 OK → ACK`

📸 **EVIDENCIA — Captura de pantalla:**

> _[ Insertar captura aquí ]_

---

## 6. Verificación de RTP (Audio)

> Se valida el flujo de audio en los puertos **RTP/SRTP** (rango `10000–20000 UDP`). El uso de **SRTP** garantiza que el audio viaja cifrado.

### 6.1 Captura de tráfico RTP durante llamada

Ejecutar la captura **antes** de realizar la llamada:

```bash
sudo tcpdump -i any -n portrange 10000-20000
```

> 📞 Realizar llamada al **9999** desde MicroSIP y observar el tráfico.

📸 **EVIDENCIA — Captura con tráfico RTP visible:**

> _[ Insertar captura aquí ]_

---

### 6.2 Señalización SIP en red interna (Kamailio → Asterisk)

Tráfico SIP no cifrado entre el SBC y la PBX en la red privada de AWS:

```bash
sudo tcpdump -i any -n port 5060
```

📸 **EVIDENCIA — Captura de pantalla:**

<img width="932" height="117" alt="image" src="https://github.com/user-attachments/assets/4feb3d37-79d8-4473-8a2b-362bc3ad0cdd" />


---

## 7. Verificación de Cifrado TLS

> Se comprueba que el tráfico SIP desde Internet viaja **completamente cifrado** por TLS en el puerto **5061**. Esta es la **evidencia principal** del cifrado de señalización.

### 7.1 Captura de tráfico cifrado TLS

Desde **VM-Kamailio**, ejecutar mientras el softphone está activo:

```bash
sudo tcpdump -i enX0 -A port 5061
```

**Salida esperada (contenido ilegible = CIFRADO):**
```
IP <tu-IP-ISP> > ip-172-31-35-212.sip-tls
E...s@.t.%S....*......6v.+^..P...[.....
...X...Y.D....4.....0.}.A....4..9.,...
```

Esto demuestra:
- ✅ El tráfico viaja por puerto **5061 (TLS)** — señalización cifrada
- ✅ El contenido es **ilegible** — nadie puede interceptar credenciales ni llamadas
- ✅ La IP origen es la del ISP del teletrabajador — conexión desde Internet

📸 **EVIDENCIA — Captura con tráfico ilegible (cifrado):**

> _[ Insertar captura aquí ]_

---

### 7.2 Análisis con sngrep (señalización interna)

```bash
sudo sngrep -d enX0 port 5060
```

Si sngrep aparece vacío, usar alternativa:

```bash
sudo tcpdump -i enX0 -A host 172.31.37.133 and port 5060
```

📸 **EVIDENCIA — Captura de pantalla:**

> _[ Insertar captura aquí ]_

---

### 7.3 Resumen de capas de seguridad implementadas

| Capa | Protocolo | Puerto | Evidencia |
|:---|:---:|:---:|:---:|
| Señalización (softphone → SBC) | **TLS** | `5061 TCP` | Secciones 3 y 7 |
| Audio cifrado | **SRTP** | `10000-20000 UDP` | Sección 6 |
| Señalización interna (SBC → PBX) | UDP/SIP | `5060 UDP` | Secciones 6.2 y 7.2 |

---

## ✅ Conclusión

La implementación de la plataforma VoIP segura fue completada exitosamente. Se logró:

- ✔️ Despliegue de **Asterisk** como PBX en VM-Asterisk (AWS EC2 t2.micro)
- ✔️ Despliegue de **Kamailio** como SBC con soporte TLS/SRTP en VM-Kamailio
- ✔️ Generación e instalación de **certificados digitales** (autofirmados) para cifrado TLS
- ✔️ Registro del softphone **MicroSIP** con transporte TLS y SRTP obligatorio
- ✔️ Verificación de llamadas y cifrado mediante **tcpdump**, **sngrep** y herramientas de red

El tráfico de señalización SIP desde Internet viaja completamente cifrado, **imposibilitando la interceptación de credenciales o conversaciones** por parte de terceros.

---

## 📋 Resumen de IPs y Credenciales

| Componente | IP Pública | IP Privada | Usuario | Contraseña |
|:---|:---:|:---:|:---:|:---:|
| VM-Asterisk (PBX) | `54.205.26.179` | `172.31.37.133` | — | — |
| VM-Kamailio (SBC) | `3.82.14.233`   | `172.31.35.212` | — | — |
| Extensión 1001 | — | — | `1001` | `pass1001` |
| Extensión 1002 | — | — | `1002` | `pass1002` |
| Extensión 1003 | — | — | `1003` | `pass1003` |
| Echo Test | — | — | `9999` | _(sin contraseña)_ |

---

<div align="center">

**DUOC UC — Comunicaciones Unificadas — Mayo 2026**

</div>
