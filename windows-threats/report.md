# Windows Threat Detection & DFIR Analysis

Este directorio forma parte del repositorio Cybersecurity-Hub y contiene el analisis tecnico detallado, comandos de reconocimiento, matriz de deteccion y artefactos de telemetria recopilados durante investigaciones de DFIR en entornos Windows.

---

## Estructura del Directorio

Cybersecurity-Hub/
└── Windows-threats/
    ├── README.md
    └── img/
        ├── rdp-bruteforce.png
        ├── logon-type3.png
        ├── 4624-logon-type10.png
        ├── download-event.png
        ├── extraction-event.png
        ├── process-create.png
        ├── lnk-shortcut.png
        ├── Double-extension.png
        ├── ms-defender.png
        ├── dns-2.png
        ├── ps-get-clipboard.png
        ├── xcopy-pdf.png
        ├── xcopy-xlsx.png
        ├── make-dir.png
        └── dns-stealer.png

---

## Resumen Ejecutivo del Incidente

Se analizo una cadena de ataque multivectorial en infraestructura Windows, mapeada a las tacticas del marco MITRE ATT&CK:

* Acceso Inicial (Initial Access - TA0001):
  * Intrusion perimetral via Fuerza Bruta a RDP (Event ID 4625 / 4624).
  * Vectores de Ingenieria Social / Phishing: Descarga de archivos ZIP con accesos directos maliciosos (.LNK) y binarios con doble extension (.jpg.exe).
  * Simulacion/Riesgo de BadUSB (ejecucion automatica de scripts hid/payloads de teclado en hardware).
* Evasion de Defensas (Defense Evasion - TA0005):
  * Ocultacion de extensiones de archivo (Masquerading).
  * Ejecucion en memoria sin tocar disco usando LOLBins (powershell -WindowStyle hidden).
  * Verificacion y reconocimiento del agente EDR / Microsoft Defender.
* Descubrimiento (Discovery - TA0007):
  * Enumeracion del sistema local, grupos, usuarios y portapapeles.
* Recoleccion y Staging (Collection - TA0009):
  * Extraccion de documentos (.pdf, .xlsx) y empaquetado comprimido (staging_58f1.zip).
* Exfiltracion (Exfiltration - TA0010):
  * Canal encubierto de exfiltracion de datos hacia infraestructura Cloud (AWS S3) mediante peticiones DNS (Event ID 22).

---

## 1. Vectores de Acceso Inicial y Evasion

### A. Fuerza Bruta RDP y Autenticacion de Red (RDP Brute Force)

#### Explicacion Tecnica y Deteccion
El protocolo Remote Desktop Protocol (RDP - Puerto 3389) es uno de los vectores de entrada mas explotados. El ataque consiste en el envio masivo de combinaciones de credenciales contra cuentas administrativas.

* Fuerza Bruta (Event ID 4625 - Failed Logon):
  * Fecha/Hora: 5/20/2025 entre 9:59:46 AM y 9:59:54 AM.
  * Eventos Registrados: 1,567 intentos fallidos en ventana corta de tiempo.
  * TargetUserName: ADMINISTRATOR
  * Status Code: 0xC000006D (Nombre de usuario o contrasena incorrecta).

<a href="img/rdp-bruteforce.png" target="_blank">
  <img src="img/rdp-bruteforce.png" alt="RDP Brute Force 4625" width="350"/>
</a>
