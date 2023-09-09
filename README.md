# Voi


> ubuntu 22.04 lazım 4 cpu 8 ram kurdum


# Güncellemeler:
sudo apt update && sudo apt-get upgrade -y && echo OK

sudo systemctl start unattended-upgrades && sudo systemctl enable unattended-upgrades

# Hep Cosmos'u yükleyecek değiliz, Algorand'ı yüklüyoruz:

sudo apt install -y jq gnupg2 curl software-properties-common
curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc
sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"
# Komutlar sonlanınca ENTER diyebilirsiniz.

# Tekrar güncelleyelim ve node'un otomatik başlamaması için durduralım:
sudo apt update && sudo apt install -y algorand && echo OK
sudo systemctl stop algorand && sudo systemctl disable algorand && echo OK

# goal setup'ı yapalım
echo -e "\nexport ALGORAND_DATA=/var/lib/algorand/" >> ~/.bashrc && source ~/.bashrc && echo OK
sudo adduser $(whoami) algorand && echo OK

# yapılandırma işlem:

sudo algocfg set -p DNSBootstrapID -v "<network>.voi.network" -d /var/lib/algorand/ &&\
sudo algocfg set -p GossipFanout -v 8 -d /var/lib/algorand/ &&\
sudo algocfg set -p EnableCatchupFromArchiveServers -v true -d /var/lib/algorand/ &&\
sudo chown algorand:algorand /var/lib/algorand/config.json &&\
sudo chmod g+w /var/lib/algorand/config.json &&\
echo OK

# Genesis
sudo curl -s -o /var/lib/algorand/genesis.json https://testnet-api.voi.nodly.io/genesis &&\
sudo chown algorand:algorand /var/lib/algorand/genesis.json &&\
echo OK

# Algorand'ı Voi olarak yapılandıralım:
sudo cp /lib/systemd/system/algorand.service /etc/systemd/system/voi.service &&\
sudo sed -i 's/Algorand daemon/Voi daemon/g' /etc/systemd/system/voi.service &&\
echo OK

# ve node'u çalıştralım:
sudo systemctl start voi && sudo systemctl enable voi && echo OK

# node'u kontrrol edelim status ile:
goal node status

# ==> Genesis ID: voitest-v1
# ==> Genesis hash: IXnoWtviVVJW5LGivNFc0Dq14V3kqaXuK2u5OQrdVZo=
# Çıktının sonu bu şekilde olmalı (hash değişebilir)

# Hızlı sync olalım:
goal node catchup $(curl -s https://testnet-api.voi.nodly.io/v2/status|jq -r '.["last-catchpoint"]') &&\
echo OK

# Yine status yapalım ama bu sefer loglarda Catchpoint göreceğiz:
goal node status

# Bu komutla kontrol ettiğimizde Sync Time'ın sıfırlanmasını ve loglarda Catchpoint'in gitmesini bekleyelim.
goal node status -w 1000
# Yukarda ki şartlar gerçekleince CTRL + C

# Cüzdan oluşturalım:
goal wallet new voi
# Şifre belirledikten sonra enter diyip 24 kelimenizi alıp saklayın.

# Şimdi cüzdanımızı node'umuza import edelim:
goal account import
# Şifre ve 24 kelimeyi girince bize bir Imported adres verecek bunu saklayalım cüzdan adresimiz.

# Şimdi bu kodları girelim ve bizden Imported adresimizi isteyecek.
echo -ne "\nEnter your voi address: " && read addr &&\
echo -ne "\nEnter duration in rounds [press ENTER to accept default (2M)]: " && read duration &&\
start=$(goal node status | grep "Last committed block:" | cut -d\  -f4) &&\
duration=${duration:-2000000} &&\
end=$((start + duration)) &&\
dilution=$(echo "sqrt($end - $start)" | bc) &&\
goal account addpartkey -a $addr --roundFirstValid $start --roundLastValid $end --keyDilution $dilution
# Imported adresinden sonra ki soruda ENTER diyip varsayılanı tercih edebiliriz.
# Import işleminin tamamlanmasını bekleyin ve Participation ID'inizi saklayın.

# Aktifliğimize bakalım, burada çıktı OFFLİNE OLMALI!
checkonline() {
  if [ "$addr" == "" ]; then echo -ne "\nEnter your voi address: " && read addr; else echo ""; fi
  goal account dump -a $addr | jq -r 'if (.onl == 1) then "You are online!" else "You are offline." end'
}
checkonline

> Bu aşamadan sonrasına devam etmek için [Buradan](https://docs.google.com/forms/d/e/1FAIpQLSehNL0nNP0mtIXK5j615vxQtzz6QQpYUKHTVN4irN6YpHjXfg/viewform) Formu doldurarak token alın.

> Imported adresiniz ile [Explorer'dan](https://app.dappflow.org/dashboard/home) kontrol edin token gelince devam edin.











