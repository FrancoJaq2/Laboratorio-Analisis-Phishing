# Laboratorio de Análisis SOC: Respuesta a Incidente de Phishing

## 📋 Resumen Ejecutivo
**Fecha del Incidente:** 05 de Mayo de 2026
**Tipo de Alerta:** Posible compromiso de credenciales vía Phishing (Nivel de Severidad: Alto).
**Descripción:** Un usuario de la organización reportó haber interactuado con un enlace sospechoso recibido por correo electrónico, el cual solicitaba la "Actualización de Credenciales de Microsoft 365". El usuario indicó que la página resultante quedó en blanco, lo que levantó sospechas de exfiltración de datos o robo de sesión.

---

## 🔍 Indicadores de Compromiso (IoCs) Iniciales
| Tipo | Indicador | Descripción |
| :--- | :--- | :--- |
| **URL (Redirección)** | `https://servicio.botafopalmeiras.com/activacion/cuenta-hiqi/` | Enlace inicial incrustado en el cuerpo del correo. |
| **URL (Payload)** | `https://iniciosociedad.aatmaiv.com/1776189058/[...]` | Destino final tras la redirección (Página falsa de recolección de credenciales). |
| **IPv4** | `143.95.108.238` | IP asociada a la infraestructura web maliciosa. |

---

## 🛠️ Fase 1: Análisis de Cabeceras de Correo

Como primer paso de la investigación, se solicitó al usuario el correo original en formato `.eml` para extraer las cabeceras y realizar el trazado de la ruta del mensaje evadiendo las técnicas de ofuscación visual.

**Hallazgos Técnicos:**
* **IP de Origen Real (X-Originating-IP):** Se identificó que la IP real desde donde se originó el ataque es `85.203.15.22`, saltándose la IP falsificada mostrada en el cliente de correo del usuario.
* **Técnica de Ingeniería Social (Look-alike domain):** Mientras la empresa legítima opera bajo el dominio `lexdata-abogados.cl`, el atacante utilizó el remitente malicioso `soporte@lexdata-admin-update.com` (Typosquatting) para generar falsa confianza.
* **Fallo en Autenticación (SPF/DMARC):** El análisis de seguridad arrojó un resultado `spf=softfail` y `dmarc=fail`. Esto confirma de manera concluyente que la dirección IP emisora no estaba autorizada para enviar correos en nombre de dicho dominio, confirmando la naturaleza fraudulenta del mensaje (Email Spoofing).

---

## 🌍 Fase 2: Triaje de Infraestructura (OSINT y CTI)

Se procedió a investigar la infraestructura del atacante mediante el análisis de la dirección IP sospechosa (`143.95.108.238`) utilizando plataformas de Cyber Threat Intelligence (CTI).

### 2.1 Análisis de Reputación (Cisco Talos)
Se consultó la base de datos de Talos Intelligence para evaluar la reputación e historial de la IP objetivo.

<img width="1305" height="669" alt="Cisco Talos busqueda" src="https://github.com/user-attachments/assets/e0f60659-e463-4eb8-b600-ceb11a1ed35f" />


* **Propietario de la Red (ASN):** NEWFOLD DIGITAL INC.
* **Reputación:** Neutral (No listada en Blocklists al momento del ataque).
* **Conclusión Parcial:** El uso de un proveedor de alojamiento web comercial masivo sugiere el uso de infraestructura temporal o comprometida, logrando evadir los filtros tradicionales basados en listas negras.

### 2.2 Análisis de Superficie de Ataque (Shodan)
Para entender la naturaleza del servidor utilizado, se realizó un escaneo pasivo de puertos y servicios expuestos utilizando Shodan.

<img width="1300" height="655" alt="Shodan 1" src="https://github.com/user-attachments/assets/191d3409-e885-4af8-9a85-0dc98feb0127" />

* **Servicios Expuestos:** Se identificó una alta exposición de puertos, destacando 80/443 (HTTP/S), 3306 (MySQL), múltiples puertos de correo (465, 587, 993) y, críticamente, los puertos **2082/2083 y 2086/2087**, característicos de paneles de administración **cPanel / WHM**.
* **Vulnerabilidades (CVE):** El servicio OpenSSH expuesto presenta múltiples vulnerabilidades críticas sin parchear (ej. CVE-2026-35414, CVE-2026-35388).
* **Diagnóstico de la Infraestructura:** La evidencia técnica confirma que la IP corresponde a un servidor de **Shared Hosting comprometido**. El atacante explotó la falta de actualizaciones de este servidor legítimo para utilizarlo como máquina pivote, enviando los correos maliciosos y alojando el payload (sitio falso), aprovechando así la reputación neutral del proveedor original.

---

## 🛡️ Fase 3: Contención, Mitigación y Recomendaciones

Basado en la confirmación de la infraestructura maliciosa, se activó el protocolo de respuesta a incidentes, estableciendo las siguientes acciones inmediatas y preventivas:

**1. Medidas de Contención Inmediata:**
* **Bloqueo Perimetral:** Incorporación de la IP (`143.95.108.238`) y los dominios identificados (`servicio.botafopalmeiras.com` e `iniciosociedad.aatmaiv.com`) a las listas negras del Firewall corporativo, proxy web y filtro DNS.
* **Purga de Correo (Search and Destroy):** Ejecución de una búsqueda global en el tenant de Microsoft 365 para identificar y eliminar el correo malicioso de todas las bandejas de entrada de la organización.
* **Protección de Identidad:** Rotación forzada de contraseñas para el usuario afectado y revocación inmediata de los tokens de sesión activos para expulsar cualquier posible acceso no autorizado.

**2. Threat Hunting (Cacería Proactiva):**
* Revisión de los logs del SIEM y Firewall de las últimas 48 horas para verificar si otros hosts internos establecieron comunicación con la IP o las URLs maliciosas. 

**3. Retroalimentación e Inteligencia de Amenazas (CTI):**
* Reporte de los Indicadores de Compromiso (IoCs) en plataformas colaborativas (AlienVault OTX y VirusTotal) para contribuir a la comunidad global de ciberseguridad.

**4. Recomendaciones Estratégicas y Preventivas:**
* **Implementación de MFA:** Exigir Autenticación Multifactor (MFA) obligatoria para todos los accesos a Microsoft 365 y VPN, mitigando el impacto en caso de robo de credenciales.
* **Security Awareness (Concientización):** Diseñar una campaña de capacitación para los empleados utilizando este incidente (anonimizado) como caso de estudio práctico, enfocándose en la identificación de dominios Look-alike y la verificación de remitentes.

---
**Analista a cargo:** Franco Aravena
**Estado del Incidente:** Cerrado / Contenido
