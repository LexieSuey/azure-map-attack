# Honeypot en Azure con Attack Map en Microsoft Sentinel

Este proyecto consiste en la implementación de un **honeypot en Azure**, la recolección de eventos mediante **Azure Monitor Agent (AMA)** y la **visualización geográfica de ataques** utilizando un Attack Map dentro de **Microsoft Sentinel**.

El objetivo es analizar intentos de intrusión automatizados y demostrar el flujo completo de captura, enriquecimiento y visualización de eventos de seguridad.

---

## 1. Preparación del entorno

### Resource Group
Se creó un Resource Group dedicado al proyecto.

**Recomendación:** utilizar la región **East US 2**, ya que otras regiones pueden presentar limitaciones en la selección del *VM size*.

<img width="479" height="399" alt="image" src="https://github.com/user-attachments/assets/1f6816a4-b19a-4753-b7c8-b877eb231317" />

### Virtual Network
Se aprovisionó una **Virtual Network** aislada, con el objetivo de segmentar el honeypot del resto de recursos y controlar su tráfico.

<img width="561" height="752" alt="image" src="https://github.com/user-attachments/assets/437dcdc7-c875-463f-ac15-88d3649dc2a4" />


---

## 2. Implementación del Honeypot

### Virtual Machine
Se desplegó una **Máquina Virtual Windows Enterprise**, debido a la cantidad y utilidad de los eventos de seguridad que genera para análisis.

### Configuración del NSG
La regla predeterminada de RDP fue reemplazada por una que permite:
Inbound: Any → Any (Any protocol, Any port)

Esta configuración expone deliberadamente la VM para atraer ataques automatizados.  
**Advertencia:** Esta práctica es únicamente válida en entornos aislados, sin datos sensibles y con objetivos de investigación.

<img width="800" height="370" alt="image" src="https://github.com/user-attachments/assets/e4b25079-22f3-4ac9-8369-a898d2712cae" />


### Firewall
Dentro del sistema operativo se **desactivó el Windows Firewall**, aumentando la superficie de ataque del honeypot.

<img width="485" height="228" alt="image" src="https://github.com/user-attachments/assets/192aa513-c615-48fe-8efd-29bd39489159" />


---

## 3. Integración con Microsoft Sentinel

### Log Analytics Workspace
Se creó un Workspace para almacenar todos los eventos.

### Azure Monitor Agent Security Event Connector (AMA)
Se configuró el conector **AMA**, que actúa como puente entre:

- la **Virtual Machine**,  
- el **Log Analytics Workspace**,  
- y **Microsoft Sentinel**.

Este agente envía eventos de seguridad del sistema operativo directamente al Workspace.

### Data Collection Rule
En Sentinel se habilitó el conector **Windows Security Events via AMA** y se asoció a la VM mediante una **Data Collection Rule (DCR)**, definiendo qué eventos se deben recopilar.

---

## 4. Análisis Inicial de Eventos

Se dejó la VM expuesta durante un periodo de tiempo para recibir intentos de conexión no autorizados.

### Validación con KQL
Se verificó la llegada de logs utilizando consultas **KQL**, filtrando el evento:

EventID: 4625 (Failed Logon Attempt)

Los registros comenzaron a aparecer poco después, confirmando actividad maliciosa automatizada.

<img width="800" height="414" alt="image" src="https://github.com/user-attachments/assets/d4e062ad-f63e-4216-bec0-0736db4ec033" />



---

## 5. Watchlist para Geolocalización

Los eventos no contienen ubicación geográfica, solo dirección IP.  
Para enriquecer los datos sin procesar IP por IP:

1. Se creó una **Watchlist** en Sentinel.
2. Se cargó una tabla con rangos de IP (CIDR) y metadatos de ubicación (país, ciudad, región).
3. Se verificó la correlación mediante una consulta KQL que combinaba eventos 4625 + Watchlist.

<img width="800" height="415" alt="image" src="https://github.com/user-attachments/assets/35642cb1-45ab-4e6d-babb-2c0f43fdef42" />


---

## 6. Creación del Attack Map

Se generó un **Workbook** en Sentinel y se cargó un archivo JSON con la configuración del mapa.

El mapa:

- toma los eventos fallidos,  
- los correlaciona con la Watchlist,  
- y los representa como círculos geolocalizados.  
Mientras mayor sea el número de intentos desde una región, más grande es el punto en el mapa.

<img width="800" height="402" alt="image" src="https://github.com/user-attachments/assets/86ffc31b-1295-4b58-b817-0ed12ded3167" />
<img width="800" height="413" alt="image" src="https://github.com/user-attachments/assets/90474a8a-c045-4b84-a6a5-df9fd248ba3f" />



---

## Resultado

Después de exponer el honeypot el tiempo suficiente, el Attack Map visualizó actividad maliciosa proveniente de múltiples países, demostrando la magnitud del tráfico automatizado dirigido a sistemas accesibles en internet.

<img width="800" height="495" alt="image" src="https://github.com/user-attachments/assets/82afd31b-ec18-4350-9e74-b9695f9efa80" />


---

## Tecnologías utilizadas

- Azure Virtual Machines  
- Azure Virtual Network  
- Network Security Groups (NSG)  
- Log Analytics Workspace  
- Azure Monitor Agent (AMA)  
- Microsoft Sentinel  
- Watchlists  
- KQL  
- Workbooks en Sentinel  

---

## Notas

Este entorno está configurado únicamente para fines educativos y de análisis.  
No debe utilizarse en producción ni con datos sensibles.
