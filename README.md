# 🔍 Laboratorio de Triaje y Análisis de Phishing

## 🎯 Objetivo del Ejercicio
Simular el flujo de trabajo de un analista SOC (Nivel 1) para investigar una campaña de phishing. El objetivo es aislar la amenaza, extraer Indicadores de Compromiso (IoCs) y documentar la infraestructura del atacante utilizando inteligencia de fuentes abiertas (OSINT).

## 🛠️ Entorno y Herramientas Utilizadas
* **Entorno Seguro:** Máquina virtual aislada con Kali Linux (Red configurada en NAT).
* **Privacidad:** Conexión a través de VPN para evitar exposición de la IP pública.
* **Herramientas OSINT:** VirusTotal, Cisco Talos Intelligence, urlscan.io.

## 🕵️‍♂️ Resumen de la Investigación
Durante el análisis de los artefactos, se identificó una URL sospechosa (`iniciosociedad.aatmaiv.com`). Al investigar la infraestructura subyacente (IP `143.95.108.238`), se descubrió lo siguiente:
1. **Falsos Positivos:** La IP mostraba una baja tasa de detección maliciosa en plataformas de seguridad.
2. **Hosting Compartido:** Mediante consultas Whois, se determinó que la IP pertenece a *Newfold Digital, Inc.*, un proveedor legítimo de hosting masivo. El atacante abusó de este servicio para alojar la página fraudulenta, lo que explica por qué la IP completa no estaba bloqueada por los antivirus.
3. **Estado Actual:** Al intentar capturar una imagen forense con urlscan.io, los servidores DNS devolvieron un error, indicando que la campaña fue dada de baja rápidamente (Takedown).

## 📋 Tabla de Artefactos / IoCs Extraídos

| Tipo | Valor | Comentario / Enriquecimiento |
| :--- | :--- | :--- |
| **IPv4** | `143.95.108.238` | IP del servidor. Pertenece a Newfold Digital (Hosting compartido en EE.UU.). No bloquear directamente para evitar falsos positivos. |
| **URL** | `https://iniciosociedad[.]aatmaiv[.]com/1776189058/imagenes/_personas/home/default[.]asp` | URL del sitio falso principal. **Estado actual: Offline** (DNS Error). |
| **URL** | `https://servicio[.]botafopalmeiras[.]com/activacion/cuenta-hiqi/` | URL de redirección asociada a la campaña. **Estado actual: Offline**. |

---
*Nota: Los enlaces maliciosos han sido neutralizados (Defanging) con corchetes para prevenir clics accidentales.*
