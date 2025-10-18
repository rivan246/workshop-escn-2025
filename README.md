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
- **Region**: `East Asia` (atau region terdekat)

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
- **Region**: `East Asia`

> 🔒 **Jangan tambahkan inbound rule apa pun**. Biarkan konfigurasi default (deny all inbound).

---

### Langkah 4: Buat VM Linux (Ubuntu)

- **Image**: `Ubuntu Server 24.04 LTS`  
- **VM name**: `vm-product-a`  
- **Size**: `B1s`
- **Authentication type**: `password` user: `labazurees` pass: `escnmantap1!@#`
- **Public IP**: Aktifkan  
- **Networking**:  
  - Virtual network: `vnet-product-a`  
  - Subnet: `subnet-vm`  
  - **Network security group**: Pilih *Advanced* → pilih `nsg-product-a`

---

### Langkah 5: Uji Akses — Semua Port DITUTUP

Buka **PowerShell** di laptop Anda dan jalankan:

`Test-NetConnection <PUBLIC_IP_VM> -Port 22`

`ping <PUBLIC_IP_VM>`


### Langkah 6: Buka Port 22 (SSH) dan Uji Koneksi

Edit **nsg-product-a** → *Inbound security rules* → *Add inbound rule*:

| Pengaturan         | Nilai                     |
|--------------------|---------------------------|
| Source             | IP Addresses              |
| Source IP addresses| `<IP_PUBLIK_ANDA>`        |
| Protocol           | TCP                       |
| Port               | 22                        |
| Action             | Allow                     |
| Priority           | 100                       |

dan icmp
> 🌐 Cek IP publik Anda di [whatismyipaddress.com](https://whatismyipaddress.com/)

Uji koneksi dari laptop menggunakan **PowerShell**:

`Test-NetConnection <PUBLIC_IP_VM> -Port 22`

### Langkah 7: Install dan Uji Web Server (Nginx)

Setelah berhasil masuk ke VM, install web server **Nginx**:

`sudo apt update && sudo apt install nginx -y`

`sudo systemctl start nginx`


## 🔄 Bagian 2 (Opsional): VNet Peering — Komunikasi Antar Produk

> ⏱️ Lakukan hanya jika waktu tersisa (>20 menit)

### Langkah 1: Buat Resource Group dan Lingkungan Product B

1. Buat **Resource Group** baru:
   - Nama: `rg-product-b`
   - Region: `East Asia`

2. Buat **Virtual Network**:
   - Nama: `vnet-product-b`
   - Address space: `192.168.2.0/24`
   - Subnet name: `subnet-vm`
   - Subnet address range: `192.168.2.0/24`

3. Buat **Network Security Group**:
   - Nama: `nsg-product-b`
   - Resource group: `rg-product-b`
   - Region: `East Asia`
   - **Jangan tambahkan inbound rule dulu** (default: deny all)

4. Buat **VM Linux**:
   - Image: `Ubuntu Server 22.04 LTS`
   - VM name: `vm-product-b`
   - Size: `B1s`
   - **Authentication type**: `password` user: `labazurees` pass: `escnmantap1!@#`
   - Public IP: Aktifkan
   - Networking:
     - Virtual network: `vnet-product-b`
     - Subnet: `subnet-vm`
     - Network security group: Pilih `nsg-product-b`

---

### Langkah 2: Buat VNet Peering

1. Di **vnet-product-a** (di `rg-product-a`):
   - Buka *Peerings* → *+ Add*
   - Peering name: `to-product-b`
   - Virtual network: pilih `vnet-product-b` (di `rg-product-b`)

2. Di **vnet-product-b** (di `rg-product-b`):
   - Buka *Peerings* → *+ Add*
   - Peering name: `to-product-a`
   - Virtual network: pilih `vnet-product-a` (di `rg-product-a`)

> ✅ Tunggu hingga status peering di kedua sisi menjadi **Connected**.

---

### Langkah 3: Izinkan Akses Internal di NSG

Edit **nsg-product-a** → *Inbound security rules* → *Add*:
- Source: `IP Addresses`
- Source IP addresses: `192.168.2.0/24`
- Protocol: `TCP`
- Port: `22`
- Action: `Allow`
- Priority: `200`

Edit **nsg-product-b** → *Inbound security rules* → *Add*:
- Source: `IP Addresses`
- Source IP addresses: `192.168.1.0/24`
- Protocol: `TCP`
- Port: `22`
- Action: `Allow`
- Priority: `200`

> 💡 Ini mengizinkan VM di Product A mengakses VM di Product B (dan sebaliknya) via **private IP**.

---

### Langkah 4: Uji Komunikasi Internal (Private IP ke Private IP)

1. Dari **Azure Portal**, catat **private IP** dari:
   - `vm-product-a` (misal: `192.168.1.4`)
   - `vm-product-b` (misal: `192.168.2.4`)

2. SSH ke `vm-product-a` dari laptop Anda:
   ```bash
   ssh azureuser@<PUBLIC_IP_VM_A>
