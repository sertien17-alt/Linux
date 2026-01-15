# Sprint 1 – Linux Server + Samba Active Directory Domain Controller

## 📌 Lab Activity
**Linux Server + Samba Active Directory Domain Controller en VirtualBox**

![Configuración de la máquina virtual](Imagenes/0.png)

## 🖥️ 1. Creación de la Máquina Virtual

- **Sistema Operativo:** Ubuntu Server 22.04 / 24.04  
- **RAM:** 4 GB  
- **CPU:** 2  
- **Disco:** 40 GB  
- **Red:** 2 adaptadores  




## 🌐 2. Configuración de Red en VirtualBox

### 🔹 Adaptador 1 – INTERNET
- Estado: Enabled  
- Tipo: Bridged Adapter  
- Propósito: Acceso a Internet y red real  
- IP del servidor: `172.30.20.39`

### 🔹 Adaptador 2 – DOMAIN NETWORK
- Estado: Enabled  
- Tipo: Internal Network  
- Nombre: `intnet`  
- Propósito: Tráfico interno para futuros clientes del dominio  

![Configuración de la máquina virtual](Imagenes/1.png)

## 🐧 Instalación de Ubuntu Server y Configuración de Red

Durante la instalación se configura el hostname y la IP estática.

### 🔐 Configuración del Servidor
- **Server Team Name:** ls04  
- **Usuario:** Sergio  
- **Contraseña:** admin_21  

![Configuración de la máquina virtual](Imagenes/2.png)

## 🌍 Configuración de Red (Netplan)

### 🔹 Adaptador Bridge
- **IP:** `172.30.20.39/16`  
- **Gateway:** `172.30.20.1`  
- **DNS:**  
  - `10.239.3.7`  
  - `10.239.3.8`

### 🔹 Adaptador Red Interna
- **IP:** `192.168.10.37/24`  
- **Gateway:** *(vacío)*  
- **DNS:** `127.0.0.1`

---

## 🛠️ Archivo de Configuración Netplan

Editar el archivo de configuración con el siguiente comando:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
