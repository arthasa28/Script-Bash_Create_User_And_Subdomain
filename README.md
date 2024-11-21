# Script-Bash_Create_User_And_Subdomain

##Salin Script Ini

```bash
#!/bin/bash

# Mengatur warna
RED="\033[31m"
GREEN="\033[32m"
YELLOW="\033[33m"
CYAN="\033[36m"
RESET="\033[0m"

# Fungsi untuk menampilkan loading dengan tampilan menarik
cool_loading() {
    local message=$1
    echo -e "\n${CYAN}🔄 $message${RESET}"
    for i in {1..5}; do
        echo -n "⏳"
        sleep 0.5
        echo -ne "\r${CYAN}🔄 $message ${RESET} ${YELLOW}Loading${RESET} ${i}"
        sleep 0.5
    done
    echo -e "\n${GREEN}✅ Selesai!${RESET}"
    sleep 1  # Menunggu sejenak sebelum membersihkan layar
}

# Membersihkan terminal
clear

# Header
echo -e "${CYAN}==============================================================${RESET}"
echo -e "${YELLOW}          🌟 Program Pembuatan User dan Server 🌟 ${RESET}"
echo -e "${CYAN}==============================================================${RESET}"

# Meminta input dari pengguna untuk membuat user
echo -e "\n${CYAN}=================== Buat User Baru ===================${RESET}"
echo -e "${YELLOW}          📋 Silakan masukkan informasi berikut: ${RESET}"
echo ""

# Masukkan Username
printf "${YELLOW}📌 Peringatan: Username Jangan lupa, Karena buat Login!${RESET}\n"
read -p "💼 Masukkan Username      : " Username
while [ -z "$Username" ]; do
    echo -e "${RED}🚨 Username tidak boleh kosong! Silakan coba lagi.${RESET}"
    read -p "💼 Masukkan Username      : " Username
done

# Cek apakah user sudah ada
if id "$Username" &>/dev/null; then
    echo -e "${RED}🚨 User $Username sudah ada! Skrip dibatalkan.${RESET}"
    exit 1
fi

echo ""

# Masukkan Password
printf "${YELLOW}📌 Peringatan: Password harus di ingat!${RESET}\n"
read -p "🔑 Masukkan Password      : " Password
while [ -z "$Password" ]; do
    echo -e "${RED}🚨 Password tidak boleh kosong! Silakan coba lagi.${RESET}"
    read -p "🔑 Masukkan Password      : " Password
done

echo ""

# Masukkan Real Name
printf "${YELLOW}📌 Peringatan: Masukkan nama lengkap yang valid!${RESET}\n"
read -p "📝 Masukkan Real Name     : " Real_Name
while [ -z "$Real_Name" ]; do
    echo -e "${RED}🚨 Real Name tidak boleh kosong! Silakan coba lagi.${RESET}"
    read -p "📝 Masukkan Real Name     : " Real_Name
done

echo ""

# Menentukan grup utama
group="client"

# Membuat user baru dengan parameter yang ditentukan
sudo useradd -m -s /bin/bash -c "$Real_Name" -g $group $Username

# Mengatur password untuk user yang baru dibuat
echo "$Username:$Password" | sudo chpasswd

# Mengubah permission folder home user menjadi 775
sudo chmod 775 /home/$Username

# Menampilkan pesan loading
cool_loading "Sedang membuat user $Username"

# Menampilkan pesan sukses
echo -e "${GREEN}✅ User $Username telah berhasil dibuat dengan grup $group${RESET}"

# Membersihkan terminal setelah selesai membuat user
clear

# Meminta input untuk nama server
echo -e "\n${CYAN}=================== Buat Server Baru ===================${RESET}"
echo -e "${YELLOW}          🖥️ Silakan masukkan informasi berikut: ${RESET}"
echo ""

# Masukkan nama server
printf "${YELLOW}📌 Peringatan: Subdomain Harus Jelas ( Contoh arthasa.team2vps.tech ). WAJIB Menggunakan Domain \"team2vps.cloud\" agar tidak terjadi crash!${RESET}\n"
read -p "🌐 Masukkan Subdomain    : " SERVER_NAME
while [ -z "$SERVER_NAME" ]; do
    echo -e "${RED}🚨 Nama server tidak boleh kosong! Silakan coba lagi.${RESET}"
    read -p "🌐 Masukkan Subdomain    : " SERVER_NAME
done

echo ""

# Masukkan custom folder
printf "${YELLOW}📌 Peringatan: Isi nama folder. Nama Folder Harus di samakan dengan Username. Agar Tidak CRASH!${RESET}\n"
read -p "📁 Masukkan nama folder : " CUSTOM_FOLDER
while [ -z "$CUSTOM_FOLDER" ]; do
    echo -e "${RED}🚨 Custom folder tidak boleh kosong! Harus diisi dengan nama yang valid.${RESET}"
    read -p "📁 Masukkan nama folder : " CUSTOM_FOLDER
done

# Menentukan Document Root sebagai /home/<custom_folder>/public_html
DOC_ROOT="/home/$CUSTOM_FOLDER/public_html"

# Memeriksa apakah subdomain sudah ada
if [ -f "/etc/apache2/sites-available/$SERVER_NAME.conf" ]; then
    echo -e "${RED}🚨 Subdomain $SERVER_NAME sudah ada! Menghapus user $Username dan skrip dibatalkan.${RESET}"
    sudo userdel -r "$Username"  # Menghapus user yang dibuat
    exit 1
fi

# Membuat folder untuk Document Root dan memberikan izin 777
mkdir -p "$DOC_ROOT"
chmod 777 "$DOC_ROOT"

# Menampilkan pesan loading
cool_loading "Sedang membuat server $SERVER_NAME"

# Membuat file konfigurasi virtual host
VHOST_CONF="/etc/apache2/sites-available/$SERVER_NAME.conf"
echo "<VirtualHost *:80>
    ServerName $SERVER_NAME
    DocumentRoot \"$DOC_ROOT\"

    <Directory \"$DOC_ROOT\">
        Options None
        Require all granted
    </Directory>
</VirtualHost>" | sudo tee "$VHOST_CONF" > /dev/null

# Mengaktifkan konfigurasi virtual host
sudo a2ensite "$SERVER_NAME.conf"

# Menguji konfigurasi Apache untuk memastikan tidak ada kesalahan
if apache2ctl configtest; then
    # Merestart Apache2 jika konfigurasi valid
    sudo systemctl restart apache2
    echo -e "${GREEN}🔄 Apache2 telah direstart.${RESET}"
else
    echo -e "${RED}🚨 Terjadi kesalahan dalam konfigurasi Apache. Silakan periksa log untuk detail lebih lanjut.${RESET}"
fi

# Membersihkan terminal setelah selesai membuat server
clear

# Menampilkan informasi akun
echo -e "\n${CYAN}=================== Informasi Akun ===================${RESET}"
printf "%-25s : %s\n" "🔗 Link Login" "https://my.team2vps.cloud"
printf "%-25s : %s\n" "👤 Username" "$Username"
printf "%-25s : %s\n" "🔑 Password" "$Password"
printf "%-25s : %s\n" "🌐 Domain" "$SERVER_NAME"
echo -e "${YELLOW}⚠️ Peringatan: Upload file HTML di Menu Usermin -> Tools -> File Manager -> Cari Folder public_html.${RESET}"

# Footer
echo -e "${CYAN}=============================================================${RESET}"
echo -e "${GREEN}🎉 Virtual host $SERVER_NAME telah dibuat dan diaktifkan 🎉${RESET}"
echo -e "${CYAN}=============================================================${RESET}"

```
