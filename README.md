# 🛡️ Azure Cloud Security Lab: Network Security Group (NSG) & VNet Peering

> **Durasi**: 45–60 menit  
> **Prasyarat**: Akun [Azure for Students](https://azure.microsoft.com/free/students/)  
> **Estimasi biaya**: ~$1–2 (jika resource dihapus setelah selesai)

---

## ⚠️ Peringatan Penting

1. **HAPUS SEMUA RESOURCE** setelah selesai praktikum.  
2. Jangan lupa **stop/deallocate VM** jika tidak langsung dihapus.  
3. **Jangan gunakan port scanning (seperti Nmap)** — melanggar kebijakan Azure.

---

## 🎯 Tujuan Pembelajaran

- Memahami bahwa **VM di cloud terbuka secara default** jika tidak diamankan.  
- Mampu mengamankan akses ke VM menggunakan **Network Security Group (NSG)**.  
- (Opsional) Menghubungkan dua lingkungan produk berbeda melalui **VNet Peering**.

---

## 🧪 Bagian 1: Dasar Keamanan Jaringan dengan NSG

### Langkah 1: Buat Resource Group

- **Nama**: `rg-product-a`  
- **Region**: `Southeast Asia` (atau region terdekat)

> 💡 Lakukan via Azure Portal → *Create a resource group*.

---

### Langkah 2: Buat Virtual Network (VNet)

- **Nama**: `vnet-product-a`  
- **Address space**: `192.168.1.0/24`  
- **Subnet name**: `subnet-vm`  
- **Subnet address range**: `192.168.1.0/24`

---

### Langkah 3: Buat Network Security Group (NSG)

- **Nama**: `nsg-product-a`  
- **Resource group**: `rg-product-a`  
- **Region**: `Southeast Asia`

> 🔒 **Jangan tambahkan inbound rule apa pun**. Biarkan konfigurasi default (deny all inbound).

---

### Langkah 4: Buat VM Linux (Ubuntu)

- **Image**: `Ubuntu Server 24.04 LTS`  
- **VM name**: `vm-product-a`  
- **Size**: `B1s`
- **Authentication type**: `username and password` user: labazurees pass: escnmantap!@#
- **Public IP**: Aktifkan  
- **Networking**:  
  - Virtual network: `vnet-product-a`  
  - Subnet: `subnet-vm`  
  - **Network security group**: Pilih *Advanced* → pilih `nsg-product-a`

---

### Langkah 5: Uji Akses — Semua Port DITUTUP

Buka **PowerShell** di laptop Anda dan jalankan:

```powershell
Test-NetConnection <PUBLIC_IP_VM> -Port 22