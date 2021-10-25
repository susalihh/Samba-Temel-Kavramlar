# Samba-Temel-Kavramlar

## CIFS dosya paylaşımları oluşturun ve yapılandırın

### CIFS (Common Internet File System)

CIFS, bilgisayar kullanıcılarının kurumsal intranetler ve İnternet üzerinden dosya paylaşmalarının standart yoludur.  Kullanıcılara Linux'ler ve Windows tabanlı istemciler arasında sorunsuz dosya ve yazıcı servislerinin birlikte çalışabilirliğini sağlar. 

#### CIFS Kullanarak Linux'ta Windows Share Nasıl Oluşturulur

Windows paylaşımlarının Linux sistemlerinde nasıl oluşturulacağını açıklayacağım. 

1. CIFS Yardımcı Program Paketlerini Yükleme

Linux sistemine bir Windows paylaşımı eklemek için önce CIFS yardımcı program paketlerini yüklememiz gerekir.

```
sudo apt update 
sudo apt install cifs-utils
```

2. CIFS Windows Paylaşımını Oluşturma

Windows paylaşımının bağlanması, normal dosya sistemlerinin bağlanmasına benzer.

İlk olarak, Windows paylaşımı için bağlama noktası görevi görecek bir dizin oluşturuyoruz.

```
sudo mkdir /win_paylasim
```

Paylaşımı bağlamak için aşağıdaki komutu root veya sudo ayrıcalıklarına sahip kullanıcı olarak çalıştırın:

```
sudo mount -t cifs -o username=<kullanici>,domain=<sunucu> //<win-share-ip>/<paylasim-klasörü> /win_paylasim/
```

Windows paylaşımının başarıyla bağlandığını doğrulamak için **df -h** komutunu kullanıyoruz.

```
Dosyasistemi          Boy  Dolu   Boş Kull% Bağlanılan yer
udev                 979M     0  979M    0% /dev
tmpfs                200M   15M  186M    8% /run
/dev/sda1             31G  2,5G   27G    9% /
tmpfs                998M   16K  998M    1% /dev/shm
tmpfs                5,0M     0  5,0M    0% /run/lock
tmpfs                998M     0  998M    0% /sys/fs/cgroup
tmpfs                200M     0  200M    0% /run/user/1000
//10.154.127.247/ss   32G   19G   13G   60% /win_paylasim**
```

Paylaşım bağlandıktan sonra, bağlama noktası, bağlanan dosya sisteminin kök dizini olur. Artık Paylaşılan dosyalarla yerel dosyalarmış gibi çalışabiliriz.

3. Otomatik Mount Etme

Paylaşım mount komutuyla manuel olarak bağlandığında, yeniden başlatmanın ardından devam etmez.

Sistem başlangıcında nereye ve hangi dosya sisteminin bağlanacağını tanımlayan bir giriş listesi **/etc/fstab** dosyasında bulunur.

Linux sisteminiz başladığında bir Windows paylaşımını otomatik olarak bağlamak için, **/etc/fstab** dosyasında bağlamayı tanımlayalım.

**/etc/fstab** dosyasını metin düzenleyiciyle açıyoruz:

```
sudo nano /etc/fstab
```

Dosyaya aşağıdaki satırı ekleyin:

```
# <file system>   <dir>   <type> <options>  <dump>  <pass>
//win-share-ip/paylasim-klasörü  /win_paylasim  cifs  credentials=/etc/win-credentials,file_mode=0755,dir_mode=0755   0   0
```

Paylaşımı bağlamak için aşağıdaki komutu çalıştıralım:

```
sudo mount /win_paylasim
```

*mount* komutu, **/etc/fstab** dosyasının içeriğini okuyacak ve paylaşımı bağlayacaktır. Sistemi yeniden başlattığınızda, Windows paylaşımı otomatik olarak yüklenecektir.

4. Windows Paylaşımını Kaldırma

*umount* komutu, bağlı dosya sistemini dizin ağacından bağını kaldırır.

Bağlı bir Windows paylaşımını ayırmak için, *umount* komutunu ve ardından da bağlı olduğu dizini ya da uzak paylaşımını yazıyoruz:

```
sudo umount /win_paylasim
```

CIFS bağlamasının fstab dosyasında bir girişi varsa, bu girişleri kaldırıyoruz.

#### Kerberos

Protokol, adını Yunan mitlerindeki efsanevi üç başlı köpek Kerberos'tan (Cerberus olarak da bilinir), yeraltı dünyasının girişindeki köpek koruyucusundan alır. Kerberos'un üç başı istemciyi, sunucuyu ve Anahtar Dağıtım Merkezi'ni (KDC) temsil eder. Kerberos, client ve server arasında; kdc (key distribution center) dan yardım alarak güvenli bir iletişimin  kurulmasını sağlar.

Windows 2000 ve sonrası sürümler kimlik doğrulama metodu olarak Kerberosu kullanmaktadır. Kerberos uygulamaları, Apple OS, FreeBSD, UNIX ve Linux gibi diğer işletim sistemleri için de mevcuttur.

Kerberos'u kullanan kullanıcılar, makineler ve hizmetler, kimlik doğrulama ve bilet verme işlevlerini sağlayan tek bir işlem olarak çalışan Anahtar Dağıtım Merkezine(KDC) bağlıdır. KDC biletleri, tüm taraflara kimlik doğrulaması sunarak düğümlerin kimliklerini güvenli bir şekilde doğrulamasını sağlar. Kerberos kimlik doğrulama işlemi, ağda dolaşan paketlerin okunmasını veya değiştirilmesini engelleyen ve aynı zamanda mesajları gizlice dinleme, oynatma veya yeniden oynatma saldırılarına karşı koruyan geleneksel bir paylaşılan gizli şifreleme kullanır.

**Kerberos Protokol Akışı**

Kerberos iş akışında yer alan başlıca varlıklar şunlardır:

- **Client**. İstemci, kullanıcı adına hareket eder ve bir hizmet talebi için iletişimi başlatır.
- **Server**. Sunucu, kullanıcının erişmek istediği hizmeti barındırır.
- **Key Distribution Center (KDC)**. Kerberos ortamında, kimlik doğrulama sunucusu mantıksal olarak üç bölüme ayrılmıştır: Bir veritabanı (db), Kimlik Doğrulama Sunucusu (AS) ve Bilet Verme Sunucusu (TGS). Bu üç bölüm, sırayla, Anahtar Dağıtım Merkezi adı verilen tek bir sunucuda bulunur.
- **Authentication Server (AS)**. Kimlik doğrulama hizmeti, istenen istemci kimlik doğrulamasını gerçekleştirir. Kimlik doğrulama başarılı olursa, AS istemciye TGS adlı bir bilet verir. Bu bilet, diğer sunuculara istemcinin kimliğinin doğrulandığını garanti eder.
- **Ticket Granting Server (TGS)**. TGS, hizmet biletlerini hizmet olarak yayınlayan bir uygulama sunucusudur.

Protokol akışı aşağıdaki adımlardan oluşur:

İlk olarak, Kerberos akışında yer alan üç önemli gizli anahtar vardır. İstemci/kullanıcı, TGS ve Kimlik doğrulama sunucu ile paylaşılan sunucu için benzersiz gizli anahtarlar vardır.

1. İstemci kimlik doğrulama ister

Kullanıcı, kimlik doğrulama sunucusundan bir Bilet Verme Bileti (TGT) ister. Bu istek, müşteri kimliğini içerir.

2. Kimlik doğrulama sunucu istemcinin kimlik bilgilerini doğrular

Kimlik doğrulama sunucusu, kullanıcı ve TGS'nin kullanılabilirliği için veritabanını kontrol eder. Her iki değeri de bulursa, kullanıcının parola karmasını kullanarak bir istemci/kullanıcı gizli anahtarı oluşturur.

3. TGT'yi istemciye gönderir

Kimlik doğrulama sunucusu daha sonra TGS gizli anahtarını hesaplar ve istemci/kullanıcı gizli anahtarı tarafından şifrelenmiş bir oturum anahtarı (SK1) oluşturur. Kimlik doğrulama sunucusu daha sonra istemci kimliğini, istemci ağ adresini, zaman damgasını, yaşam süresini ve SK1'i içeren bir TGT oluşturur. TGS gizli anahtarı daha sonra bileti şifreler ve istemciye sunar.

4. İstemci, erişim istemek için TGT'yi kullanır

İstemci mesajın şifresini çözer. Daha sonra çıkarılan TGT'yi ve oluşturulan kimlik doğrulayıcıyı TGS'ye göndererek hizmeti sunan sunucudan bir bilet talep eder.

5. TGS, dosya sunucusu için bir bilet oluşturur

TGS, kimlik doğrulayıcının şifresini çözer ve istemci kimliği ve istemci ağ adresiyle eşleşip eşleşmediğini kontrol eder. TGS, TGT'nin süresinin dolmadığından emin olmak için çıkarılan zaman damgasını da kullanır. İşlem tüm kontrolleri başarılı bir şekilde yürütürse, KDC, istemci ile hedef sunucu arasında paylaşılan bir hizmet oturum anahtarı (SK2) oluşturur.

KDC, istemci kimliği, istemci ağ adresi, zaman damgası ve SK2'yi içeren bir hizmet bileti oluşturur. Bu bilet daha sonra sunucunun db'den alınan gizli anahtarıyla şifrelenir.

6. TGS, SK1 ile şifrelenmiş olan hizmet biletini ve SK2'yi içeren mesajı istemciye iletir.

7. İstemci, kimlik doğrulaması için dosya biletini kullanır

İstemci, SK1'i kullanarak mesajın şifresini çözer ve SK2'yi çıkarır. Bu işlem, istemci ağ adresini, istemci kimliğini ve zaman damgasını içeren, SK2 ile şifrelenmiş yeni bir kimlik doğrulayıcı oluşturur ve hizmet biletini servis sunucuya gönderir.

8. Hedef sunucu şifre çözme ve kimlik doğrulamasını alır

Hedef sunucu, hizmet biletinin şifresini çözmek ve SK2'yi çıkarmak için sunucunun gizli anahtarını kullanır. Sunucu, kimlik doğrulayıcının şifresini çözmek için SK2'yi kullanır ve kimlik doğrulayıcıdan gelen istemci kimliği ve istemci ağ adresinin ve hizmet biletinin eşleştiğinden emin olmak için kontroller gerçekleştirir. Sunucu ayrıca hizmet biletinin süresinin dolup dolmadığını kontrol eder.

Kontroller karşılandığında, hedef sunucu istemciye, istemcinin ve sunucunun birbirini doğruladığını doğrulayan bir mesaj gönderir. Kullanıcı artık güvenli bir oturuma girebilir.




##### Kerberos kimlik doğrulamasıyla bir Windows paylaşımı bağlayın

Öncelikle gerekli paketleri kuruyoruz

```
	apt-get install krb5-user krb5-config cifs-utils keyutils
```

Paketleri kurduktan sonra nano ile /etc/krb5.conf 'un default_realm alanına domainimizi yazıyoruz

```
[libdefaults]
default_realm = DOMAIN.LAB
```

Daha sonra sunucudan bir bilet almak için aşağıdaki komutu çalıştırıyoruz. Kullanıcı adı user@DOMAIN biçiminde olmalıdır. Domain adı her zaman BÜYÜK HARFLER olmalıdır. 

*FQDN(Fully Qualified Domain Name - Tam Nitelikli Alan Adı) denen bu yapıda her alan adı maksimum  63 karakterden oluşabilir ve toplamda da 255 karakteri aşamaz. Alan adı düğümlerin her biri DNS sunucusunda birer dizindir ve en alttan en üste ilerlenecek şekilde birleştirileren okunur.*

```
kinit yourUserName@SUBDOMAIN.DOMAIN.LOCAL
```

Şimdi sunucudan bir kerberos bileti almış olmalıyız, ***klist*** komutu ile kontrol ediyoruz.

```
test@SalihBilgisayar:~$ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: administrator@SALIH.LAB

Valid starting       Expires              Service principal
21-10-2021 07:53:51  21-10-2021 17:53:51  krbtgt/SALIH.LAB@SALIH.LAB
	renew until 22-10-2021 07:53:46
```

Biletimizin oluştuğunu görebiliyoruz. Bu bilet ile paylaşıma iznimiz varsa, paylaşımı aşağıdaki komut ile mount edebiliriz. 

```
smbclient \\\\subdomain.domain.local\\paylasim-klasörü -k
```

Domain adı, aldığımız bilet adı ile aynı olmalıdır. Bileti aldığımız domain adından farklı bir ad girdiğimizde aşağıdaki gibi bir hata verecektir.

```
gensec_spnego_client_negTokenInit_step: gse_krb5: creating NEG_TOKEN_INIT for cifs/salihpc.salih.lab failed (next[(null)]): NT_STATUS_NO_LOGON_SERVERS
```

Domain adımız, aldığımız bilet ile aynı veya herhangi bir hata yoksa SMB istemcisi **(smb: \\>)** çalışır ve kaynaklara erişebiliriz. Sonrasında ***klist*** ile biletimizi tekrar kontrol ettiğimizde cifs bağlantısının biletimize eklendiğini görebiliriz.

```
test@SalihBilgisayar:~$ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: administrator@SALIH.LAB

Valid starting       Expires              Service principal
21-10-2021 08:27:02  21-10-2021 18:27:02  krbtgt/SALIH.LAB@SALIH.LAB
	renew until 22-10-2021 08:26:59
21-10-2021 08:27:03  21-10-2021 18:27:02  cifs/salihpc.salih.lab@SALIH.LAB
```



### SMB

Sunucu İleti Bloğu (SMB), günümüzün en popüler dosya sunucusu protokollerinden biridir. Düzgün bir şekilde uygulandığında SMB, güvenli, verimli ve ölçeklenebilir ağ kaynağı ve dosya paylaşımı sağlar.

SMB, bir istemci-sunucu modeli kullanan bir ağ dosyası ve kaynak paylaşım protokolüdür. Ağdaki PC'ler gibi SMB istemcileri, dosyalar ve dizinler gibi kaynaklara erişmek veya ağ üzerinden yazdırma gibi görevleri gerçekleştirmek için SMB sunucularına bağlanır.

SMB Client

Bir SMB istemcisi, bir SMB sunucusundaki kaynaklara erişen cihazdır. Örneğin, bir şirket ağı içinde, paylaşılan bir sürücüye erişen kullanıcı bilgisayarları SMB istemcileridir.

İstemciyi çalıştırmak için aşağıdaki komutu çalıştırıyoruz.

```
smbclient \\\\<win-share-ip>\\<paylasim-klasörü> -U <kullanıcı>%<şifre> -W <sunucu>
```

İstemci çalıştığında, kullanıcıya **smb: \\>** diye bir bilgi istemi sunulur. *ls* komutuyla paylaşılan klasör içeriğini görüntüleyebiliriz.

```
smb: \> ls 
  .                                   D        0  Fri Oct 22 07:53:42 2021
  ..                                  D        0  Fri Oct 22 07:53:42 2021
  file1                               D        0  Tue Oct 19 11:19:22 2021
  file2                               D        0  Tue Oct 19 10:01:25 2021
  New Text Document.txt               A        0  Fri Oct 22 07:53:42 2021
  newfile                             D        0  Wed Oct 20 16:13:50 2021

		8260095 blocks of size 4096. 3329607 blocks available
```

Sunucuda çalışan tüm komutlar aslında sunucuya bir istek göndererek gerçekleştirilir. 

**Sık Kullanılan Komutlar**

| Parametre                                  | Açıklama                                                     |
| ------------------------------------------ | ------------------------------------------------------------ |
| cd *dizin-adı*                             | Dizin adı belirtilirse, sunucudaki geçerli çalışma dizini belirtilen dizine gider. |
| ls *mask*                                  | Sunucudaki geçerli çalışma dizinindeki maskeyle eşleşen dosyaların bir listesini sunucudan alır ve görüntüler. |
| mkdir *dizin-adı*                          | Belirtilen adla sunucuda yeni bir dizin oluşturur.           |
| more *dosya adı*                           | Sunucudaki dosya alır ve içeriğiyle birlikte görüntüler.     |
| get *sunucu-dosya-adı* *[yerel-dosya-adı]* | Sunucu-dosya-adı adlı dosyayı sunucudan istemciyi çalıştıran makineye kopyalar. Belirtilmişse, yerel kopyayı yerel-dosya-adı olarak adlandırır. |
| put *yerel-dosya-adı* *[sunucu-dosya-adı]* | İstemciyi çalıştıran makineden yerel dosyayı sunucuya kopyalar. |
| lcd *[dizin-adı]*                          | Dizin adı belirtilirse, yerel makinedeki geçerli çalışma dizini belirtilen dizinle değiştirilir. |
| mget *mask*                                | Maskeyle eşleşen tüm dosyaları sunucudan istemciyi çalıştıran makineye kopyalar. |
| mput *mask*                                | Yerel makinedeki geçerli çalışma dizinindeki maskeyle eşleşen tüm dosyaları sunucudaki geçerli çalışma dizinine kopyalar. |
| print *dosya-adı*                          | Belirtilen dosyayı, sunucudaki yazdırılabilir bir hizmet aracılığıyla yerel makineden yazdırır. |
| rm *mask*                                  | Sunucudaki geçerli çalışma dizininden, maskeyle eşleşen tüm dosyaları kaldırır. |
| rmdir *dizin-adı*                          | Belirtilen dizini sunucudan kaldırır.                        |
| del *mask*                                 | İstemci, sunucudaki geçerli çalışma dizininden maskeyle eşleşen tüm dosyaları siler. |
| exit                                       | Sunucu ile bağlantıyı sonlandırır ve programdan çıkar.       |



### SAMBA

Samba, birlikte kullanıldığında bir Linux sunucusunun dosya sunumu, kimlik doğrulama/yetkilendirme, ad çözümleme ve yazdırma hizmetleri gibi ağ eylemlerini gerçekleştirmesine izin veren farklı uygulamalar topluluğudur.

CIFS gibi, Samba da Windows istemcilerinin bir Samba sunucusundaki Linux dizinlerine, yazıcılarına ve dosyalarına şeffaf bir şekilde erişmesine izin veren SMB protokolünü uygular.

En önemlisi, Samba, bir Linux sunucusunun **Etki Alanı Denetleyicisi** olarak hareket etmesine izin verir. Bunu yaparak, Windows etki alanındaki kullanıcı kimlik bilgilerinin yeniden oluşturulması ve ardından Linux sunucusunda manuel olarak eşitlenmesi gerekmeden kullanılabilir. Bu Etki Alanı Denetçisine Windows, Mac ve GNU/Linux sistemler aynı yapılandırma ile erişebilmektedir.

## Samba paylaşım erişimi yapılandırma parametrelerini yönetin

Dosya  bölümlerden ve parametrelerden oluşur. Bir bölüm, köşeli parantez içindeki bölümün adıyla başlar ve bir sonraki bölüm başlayana kadar devam eder. Bölümler, formun parametrelerini içerir:

**[etc/samba/smb.conf]**

```[etc/samba/smb.conf]
[paylasim]
path=/home/kullanici_adiniz/paylasim
comment=paylasim dizini
valid users=@admins
invalid users=@kullanici1 @kullanici2
browsable=yes
writeable=yes
read only=no
guest ok=yes
```

Dosya satır tabanlıdır, yeni satırla sonlandırılan her satır bir yorumu, bölüm adını veya parametreyi temsil eder. Parametrelerde eşittir girişini izleyen değerlerin tümü ya bir dizedir ya da evet/hayır veya doğru/yanlış olarak verilebilen bir booleandır.

| Parametre     | Açıklama                                                     |
| ------------- | ------------------------------------------------------------ |
| path          | Hizmet kullanıcısına erişim izni verilecek bir dizini belirtir. |
| comment       | Yeni paylaşımla ilişkilendirilecek yorum dizesi              |
| valid users   | Bu hizmete giriş yapmasına izin verilen kullanıcıların listesidir. |
| invalid users | Bu hizmete giriş yapmasına izin verilmemesi gereken kullanıcıların listesidir. |
| browsable     | Bu paylaşımın bir ağ görünümündeki kullanılabilir paylaşımlar listesinde ve göz atma listesinde görünüp görünmeyeceğini kontrol eder. |
| writeable     | Yazma yetkisini kontrol eder.                                |
| read only     | Sadece okuma yetkisini kontrol eder.                         |
| guest ok      | Misafir olarak belirlenen kullanıcılara erişim yetkisi verir. |

