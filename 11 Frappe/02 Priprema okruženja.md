
# Frappe Framework tutorijal

[Sadržaj][00]

## 02 Priprema okruženja

- **Preuzimnje ISO slike i instalacija**

  - Sa adrese <https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso> preuzeti
    ISO za `Ubuntu server 24.04`.

  - Izgradnja Ubuntu Server 24.04 VM  
    VM izgradnja na QUEMU/KVM root sesiji sa sledećim parametrima:

    - vCPU 2
    - RAM 8GB
    - SSD 60GB

- **Ažuriranje paketa distibucije**

  ```sh
  sudo apt update && sudo apt upgrade -y
  ```

- **Instalacija curl-a**

  ```sh
  sudo apt install curl
  ```

- **Aktiviranje firewall-a i dozvola ssh pristupa**

  - Aktiviranje firewall-a
  
    ```sh
    sudo ufw enable
    ```
  
  - Dozvola pristupa preko ssh
  
    ```sh
    sudo ufw allow OpenSSH
    ```
  
  - Proba konekcije na VM ( samo sa localhost-a )
    Iz virtuelne mašine pokrenuti:

    ```sh
    ip a
    ```

    Vratiće, za KVM slučaj, nešto kao 192.168.122.X.
  
  - Sa hosta ssh pristup na VM
  
    ```sh
    ssh username_na_VM@127.198.122.X
    ```
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

- **Promena vremenske zone i sync. vremena**

  ```sh
  sudo timedatectl set-timezone Europe/Belgrade
  sudo timedatectl set-ntp on
  ```

- **Promena locale**

  ```sh
  sudo dpkg-reconfigure locales
  ```

  - Dodaj nove locale `sr_RS@latin UTF-8` i postavi ih za default.
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

- **Instalacija osnovnih dev paketa**

  ```sh
  sudo apt python3-dev
  ```

- **Instalacija uv pajton paket i runtime managera**

  ```sh
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
  
  - Osveži sesiju:

    ```sh
    source "$HOME/.local/bin/env"
    ```

  - Proveri instaliranost:

    ```sh
    uv --version
    ```

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
