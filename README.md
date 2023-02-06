# Apache Superset dengan VirtualBox

Project ini bertujuan untuk mengimplementasikan Apache Superset dalam guest machine menggunakan VirtualBox. Skenarionya adalah agar user dapat mengakses Apache Superset dari host machine ke guest machine yang berisi Ubuntu Server dan Apache Superset. Hasilnya dapat didistribusikan kepada orang lain berupa VirtualBox Image (berupa file OVA).

Langkah-langkah:
1. Install Ubuntu server di VirtualBox. Dalam project ini saya munggunakan Ubuntu-20.04 (Focal Fossa), dapat diunduh dari https://releases.ubuntu.com/focal/. Tutorial cara instalasi VirtualBox dapat dibaca di url https://www.virtualbox.org/manual/.
2. Setelah server terinstall di VirtualBox, jalankan server dan masuk ke mesin server (disarankan mengakses server menggunakan SSH untuk mempermudah pekerjaan)
3. Install Docker Compose di mesin Ubuntu server. Tutorial lengkap dapat dilihat di url https://www.cherryservers.com/blog/how-to-install-and-use-docker-compose-on-ubuntu-20-04.
    * Add Docker's Repository To Your System: \
      $ apt update -y \
      $ apt install ca-certificates curl gnupg lsb-release \
      $ mkdir /etc/apt/demokeyrings
    * Use curl to download Docker's keyring and pipe it into gpg to create a GPG file so apt trusts Docker's repo: \
      $  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/demokeyrings/demodocker.gpg
    * Add the Docker repo to your system with this command: \
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/demokeyrings/demodocker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
    * Install Docker Compose And Related Packages: \
      $ apt update -y \
      $ apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin \
      $ docker --version; docker compose version;ctr version
4. Unduh atau clone Apache Superset dari https://github.com/apache/superset. Apabila mengunduh berupa file zip, ekstrak file tersebut di ~/home/$USER sehingga superset berada di direktorit ~/home/$USER/superset.
5. Run Your App With Docker Compose
    * Run the app \
      $ docker-compose -f superset/docker-compose-non-dev.yml up -d
    * Validate containers are running \
      $ docker-compose -f superset/docker-compose-non-dev.yml ps
    * Pausing the containers \
      $ docker-compose -f superset/docker-compose-non-dev.yml pause
    * Unpause the containers \
      $ docker-compose -f superset/docker-compose-non-dev.yml unpause
    * Stop the containers \
      $ docker-compose -f superset/docker-compose-non-dev.yml stop
    * Stopping containers does not remove the associated Docker networks. To stop the containers and remove the associated networks, use this command: \
      $ docker-compose -f superset/docker-compose-non-dev.yml down
6. Akses Apache Superset dengan web browser (asumsi Ubuntu server mendapat IP Number 192.168.56.102 dari VirtualBox): \
    http://192.168.56.102:8088

Langkah terakhir adalah menyiapkan Apache Superset agar otomatis menyala servicenya setelah proses booting selesai, sehingga user tinggal akses dari web browser tanpa harus login ke server dan menjalankan perintah untuk menghidupkan Apache Superset setiap kali menghidupkan server.
Langkah-langkah menjalankan Apache Superset sebagai Systemd Service:
1. Menyiapkan shell script (simpan di /usr/bin/)
   - $ cd /usr/bin
   - Script untuk start superset
      - $ sudo nano superset_up.sh (ketikkan baris script berikut): \
	      #!/usr/bin/env bash \
	      docker-compose -f /home/masjoe/superset/docker-compose-non-dev.yml up -d
   - Script untuk shutdown superset
      - $ sudo nano superset_down.sh (ketikkan baris script berikut): \
        #!/usr/bin/env bash \
        $ docker-compose -f /home/masjoe/superset/docker-compose-non-dev.yml down
   - Ubah permission
      - $ sudo chmod 755 superset_up.sh superset_down.sh
      - $ sudo chown root:root superset_up.sh superset_down.sh
2. Buat sebuah file untuk systemd service (misal: "superset.service")
   - $ sudo nano /etc/systemd/system/superset.service (ketikkan baris berikut):
        ```
        [Unit]
        Description=Apache Superset
        After=network.target
        
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/superset_up.sh
        ExecStop=/usr/bin/superset_down.sh
        
        [Install]
        WantedBy=multi-user.target
        ```
   - $ sudo chmod 644 /etc/systemd/system/superset.service
   - Catatan: \
   Ekseskusi file dalam shell script harus full path. Apabila tidak full path akan gagal diakses oleh systemd dalam service.
3. Enable-kan service unit: \
   $ sudo systemctl daemon-reload \
   $ sudo systemctl enable superset.service
4. Reboot system.
5. Coba akses Apache Superset dengan web browser (http://192.168.56.102:8088) dari mesin host.

Mesin Ubuntu server yang telah terpasang Apache Superset dapat didistribusikan kepada orang lain dalam bentuk VirtualBox Image, dengan melakukan prosedur export melalui menu dalam VirtualBox:
* Pilih menu **File** - **Export Appliance** (kemudian pilih nama virtual machine), lalu klik tombol **Next**.
* Arahkan ke drive di mana file .OVA akan disimpan, sekaligus definisikan nama file .OVA tersebut (atau biarkan isian sesuai default), lalu klik tombol **Next**.
* Klik tombol **Finish** (dan proses export akan berlangsung, biarkan hingga selesai).

VirtualBox Image (file .OVA) dapat dengan mudah dideploy ke dalam VirtualBox melalui menu **File** - **Import Appliance** (arahkan ke file .OVA). Selanjutnya tinggal ikuti langkahnya sesuai menu visual dari VirtualBox.
