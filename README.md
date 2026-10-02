# Wazuh All-in-One for openSUSE Leap 16

Standalone Bash installer for deploying a **native Wazuh All-in-One stack** on **openSUSE Leap 16.0 x86_64**.

Repository ini **tidak menggunakan Docker, Podman, Docker Compose, atau Ansible** untuk menjalankan Wazuh.

Wazuh dipasang langsung pada host openSUSE dan berjalan sebagai service native `systemd`.

## Repository

```text
https://github.com/Iszz100/Wazuh-all-in-one-yml
```

## Components

Installer memasang seluruh central component Wazuh dalam satu server:

- Wazuh Indexer
- Wazuh Manager
- Filebeat
- Wazuh Dashboard

Service setelah instalasi:

```bash
systemctl status wazuh-indexer
systemctl status wazuh-manager
systemctl status filebeat
systemctl status wazuh-dashboard
```

## Architecture

```text
openSUSE Leap 16
│
├── wazuh-indexer.service
├── wazuh-manager.service
├── filebeat.service
└── wazuh-dashboard.service
```

Tidak ada layer Docker atau container.

## Requirements

Target utama:

- openSUSE Leap 16.0
- x86_64
- akses `root` / `sudo`
- koneksi internet

Rekomendasi resource untuk deployment kecil:

- 4 vCPU
- 8 GiB RAM
- 50 GiB free storage

## Installation

Clone repository:

```bash
git clone https://github.com/Iszz100/Wazuh-all-in-one-yml.git
cd Wazuh-all-in-one-yml
```

Buat installer executable:

```bash
chmod +x install_wazuh_all_in_one_opensuse.sh
```

Jalankan:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh
```

Setelah instalasi selesai, script akan menampilkan URL dashboard serta credential `admin` yang dibuat oleh Wazuh.

Contoh:

```text
============================================================
 WAZUH ALL-IN-ONE INSTALLATION COMPLETED
============================================================
 Dashboard : https://192.168.1.10:443
 Username  : admin
 Password  : <generated-by-wazuh>
 Installer : /root/wazuh-install/wazuh-install.sh
 Log       : /var/log/wazuh-opensuse-all-in-one.log
 Wazuh log : /var/log/wazuh-install.log
============================================================
```

## Options

Lihat semua opsi:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --help
```

### Custom dashboard port

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --port 8443
```

### Open Wazuh API port

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --open-api
```

Port API `55000/tcp` tidak dibuka secara default.

### Skip firewall changes

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --no-firewall
```

### Ignore hardware checks

Untuk lab/testing:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --ignore-hardware
```

### Force reinstall

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --force-reinstall
```

> **Warning**
>
> Opsi force reinstall dapat menghapus konfigurasi/data instalasi Wazuh yang sudah ada.

Secara default installer akan berhenti apabila menemukan instalasi Wazuh existing.

## Firewall Ports

Jika `firewalld` sudah aktif, installer dapat membuka:

| Port | Fungsi |
|---|---|
| `443/tcp` | Wazuh Dashboard |
| `1514/tcp` | Wazuh agent communication |
| `1515/tcp` | Wazuh agent enrollment |
| `55000/tcp` | Wazuh API, hanya dengan `--open-api` |

Jika menggunakan `--port`, port dashboard akan mengikuti nilai tersebut.

Installer tidak memaksa mengaktifkan `firewalld` apabila sebelumnya disabled.

## Installer Flow

Installer melakukan:

1. Memastikan script dijalankan sebagai root.
2. Memeriksa openSUSE Leap 16 dan arsitektur x86_64.
3. Memeriksa CPU, RAM, dan storage.
4. Mendeteksi instalasi Wazuh sebelumnya.
5. Memasang dependency melalui `zypper`.
6. Menyiapkan compatibility package yang diperlukan openSUSE.
7. Mengatur `vm.max_map_count=262144`.
8. Mengatur firewall bila firewalld aktif.
9. Mengunduh Wazuh Installation Assistant resmi.
10. Menjalankan instalasi Wazuh All-in-One.
11. Memverifikasi seluruh service.
12. Memverifikasi Wazuh Indexer.
13. Memverifikasi credential admin.
14. Memverifikasi Wazuh Dashboard HTTPS.
15. Menampilkan URL dan login dashboard.

## Logs

Log installer wrapper:

```text
/var/log/wazuh-opensuse-all-in-one.log
```

Log installer Wazuh:

```text
/var/log/wazuh-install.log
```

Troubleshooting:

```bash
sudo systemctl status wazuh-indexer --no-pager -l
sudo systemctl status wazuh-manager --no-pager -l
sudo systemctl status filebeat --no-pager -l
sudo systemctl status wazuh-dashboard --no-pager -l
```

Journal:

```bash
sudo journalctl -u wazuh-indexer -n 100 --no-pager
sudo journalctl -u wazuh-manager -n 100 --no-pager
sudo journalctl -u filebeat -n 100 --no-pager
sudo journalctl -u wazuh-dashboard -n 100 --no-pager
```

## Security

Installer ini tidak:

- menyimpan password admin secara hardcoded;
- menjalankan Wazuh menggunakan Docker;
- menjalankan Wazuh menggunakan Podman;
- menggunakan Ansible;
- otomatis membuka Wazuh API ke jaringan;
- otomatis menghapus instalasi Wazuh existing.

Credential admin dihasilkan oleh proses instalasi Wazuh.

## Validate Script

Sebelum menjalankan:

```bash
bash -n install_wazuh_all_in_one_opensuse.sh
```

Jika ShellCheck tersedia:

```bash
shellcheck install_wazuh_all_in_one_opensuse.sh
```

## Compatibility Notice

Wazuh tidak secara resmi mencantumkan openSUSE Leap 16 sebagai platform standar untuk central component All-in-One.

Script ini merupakan compatibility installer untuk membantu Wazuh Installation Assistant berjalan secara native pada openSUSE Leap 16.

Disarankan melakukan pengujian pada VM/lab sebelum digunakan pada environment production.

## Author

GitHub: [@Iszz100](https://github.com/Iszz100)
