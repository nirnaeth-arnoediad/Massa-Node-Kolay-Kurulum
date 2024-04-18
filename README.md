# Massa-Node-Kolay-Kurulum

Massa-Node Sistem Servisi Olarak Kurulumu. Kurulum Ubuntu 22.04 amd64 ve arm64 versiyonları için geçerlidir.

Öncelikle `sudo -i` ile root kullanıcıya geçelim `cd` ile başlangıç konumuna dönelim `wget` ile sistemimize uygun en güncel node versiyonunu indirelim ve `tar` arşivden çıkartalım. Ardindan arşiv dosyasını `rm` silelim. `arch` komutunun çıktısı `x86_64` ise sistemimiz amd64, `aarch64` ise sistemimiz arm64 demektir. En güncel massa versiyonunu https://github.com/massalabs/massa/releases adresinden kontrol edebilirsiniz.

amd64 için 
```
sudo -i
cd
wget https://github.com/massalabs/massa/releases/download/MAIN.2.1/massa_MAIN.2.1_release_linux.tar.gz
tar -xvf massa_MAIN.2.1_release_linux.tar.gz
rm massa_MAIN.2.1_release_linux.tar.gz
```
arm64 için
```
sudo -i
cd
wget https://github.com/massalabs/massa/releases/download/MAIN.2.1/massa_MAIN.2.1_release_linux_arm64.tar.gz
tar -xvf massa_MAIN.2.1_release_linux_arm64.tar.gz
rm massa_MAIN.2.1_release_linux_arm64.tar.gz
```
Bir defaya mahsus cüzdanımızı import edelim. Clienti ilk çalıştırma esnasında cüzdan şifremizi oluşturalım ve saklayalım.
```
cd massa/massa-client
./massa-client -p şifreniz
wallet_add_secret_keys secret_keyiniz
wallet_info
exit
```
Daha sonra cd ile başlangıç yoluna dönüp  `nano` komutu ile sunucumuzun public IP sini kayıt edeceğimiz config dosyasını oluşturalım.
```
cd
nano ~/massa/massa-node/config/config.toml
```
Ardından aşağıdaki  kısmı yapıştırıp xx li kısmı IP miz ile değiştirelim. Düzenledikten sonra ctrl + o, enter, ctrl + x ile çıkış yapalım.
```
[protocol]
routable_ip="xx.xx.xx.xx"
```
Sıradaki komutlarla massanın iletişim için ihtiyacı olan portları açalım. Bazı sistemlerde açık gelmektedir yine de girmekten zarar gelmez.
```
ufw allow 31244
ufw allow 31245
```
Sıradaki komut ile massa service dosyasını oluşturalım. 
```
sudo nano /etc/systemd/system/massad.service
```
Aşağıdaki kısmı dosya içerisine yapıştıralım. şifreniz yazan yere node için de bir şifre girelim cüzdan şifresi ile aynı olabilir. Düzenledikten sonra ctrl + o, enter, ctrl + x ile çıkış yapalım.
```
[Unit]
	Description=Massa Node
	After=network-online.target
[Service]
	User=root
	WorkingDirectory=/root/massa/massa-node
	ExecStart=/root/massa/massa-node/massa-node -p şifreniz
	Restart=on-failure
	RestartSec=3
	LimitNOFILE=65535
[Install]
	WantedBy=multi-user.target
```
Ardından aşağıdaki komutlarla service dosyamızı aktifleştirip çalıştıralım.
```
sudo systemctl daemon-reload
sudo systemctl enable massad.service
sudo systemctl restart massad
```
Logları kontrol etmek için:
```
sudo journalctl -u massad -f -o cat
```
bu şekilde çıktılar görene kadar bekleyelim
```
2024-04-18T18:33:40.132318Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 6): 248zmVhDM8nLtFxhVtjTU4yarH8Zi3V9ZN264foKKanRjXtFz5
2024-04-18T18:33:40.332431Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 7): a8eYMd8JQRqMeLivSUsdaBg1JuRnLMD63RRDKGwrjiLY9cSWj
2024-04-18T18:33:41.102727Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 8): bQ3Hpoht2J2ykTBmu6D7JzUnnUA1uKUvV1w9dwoKZxiiqvqEN
2024-04-18T18:33:41.607554Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 9): 2bjuHKAGbjnp1jCpoteoMU4P59RDcPMFnCzvohjj16UToL1PFi
2024-04-18T18:33:42.434448Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 10): 2KtbsER6nxqBpb5oQngLqUVqT39CWuqKBAm2Ma12qUHUL2the5
2024-04-18T18:33:43.112479Z  INFO massa_final_state::final_state: final_state hash at slot (period: 509524, thread: 11): 7j6CnYvzAZzLRPqdpWm4vm3dZQSyhPnvGrfUQou97v8Lzd4JB
```
Artık nodeumuz çalışıyor ctrl + c ile log akışını kesebiliriz bu nodeumuzu durdurmayacaktır service dosyası oluşturduğumuz için arka planda çalışmaya devam edecektir.

Aşağıdaki komut ile client(cüzdan) üzerinden de node durumunu kontrol edebilirsiniz. In-Out bağlantılar artmaya başlamışsa işler yolunda demektir
```
cd ~/massa/massa-client
./massa-client -p şifreniz
get_status
```
Şimdi roll almamız gerekiyor massada roll almak demek diğer ağlardaki stake işlemine denk düşüyor. 1 roll 100 mas değerinde bu işlem anında herhangş birşey harcamıyoruz ve eğer bir sıkıntı olur node durur ise rollerimizin mas karşılığı cüzdanımıza otomatik olarak geliyor herhangi bir kaybımız olmuyor. roll_adedi yazan yere sayı ile miktarı giriniz
```
buy_rolls cüzdan_adtesi roll_adedi 0
```
Bu işlemden sonra son olarak bu komutu girmemiz gerekiyor.
```
node_start_staking cüzdan_adresi
```
kurulum tamamlandı yaklaşık 2 saat içerisinde rollerimiz aktif hale gelecek ve kazanç sağlamaya başlayacak. Faydalı bulduğum @massa_acheta_bot telegram botuna cüzdanınızı adresini kullanarak ekleyerek kolayca takip edebilirsiniz. Bol kazançlar.

 
