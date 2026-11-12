# AI ile Yazılan Uygulamalar için Yayın Öncesi Güvenlik Kontrol Listesi

> A Turkish pre-launch security checklist for business apps built with AI coding tools (Claude Code, Cursor, Lovable, Replit, ChatGPT).

Claude Code, Cursor, Lovable, Replit veya ChatGPT ile bir iş uygulaması yazdınız ve çalışıyor. Bu liste, o uygulamayı gerçek kullanıcılara ve gerçek verilere açmadan önce neleri kontrol etmeniz gerektiğini anlatır.

Yapay zeka araçları çalışan kod üretmekte çok iyi, güvenli kod üretmekte ise o kadar değil. [Veracode'un 2026 raporuna göre](https://sdtimes.com/?p=60753) yapay zekanın ürettiği kodun yaklaşık yarısı güvenlik testlerinden geçemiyor. Uygulamanın çalışıyor olması güvenli olduğu anlamına gelmiyor.

Bu liste yazılımcı olmayanlar için de yazıldı. Her maddenin yanında nasıl kontrol edileceği ve kullanılabilecek ücretsiz, açık kaynak araçlar var.

## Nasıl kullanılır

1. Listeyi baştan sona okuyun ve uygulamanıza uymayan maddeleri atlayın.
2. Her bölümdeki kontrolleri yapın ya da yaptırın.
3. **[Engel]** işaretli maddelerden biri bile açıksa uygulamayı yayına almayın.
4. Yayından sonra "Bakım" bölümünü takviminize ekleyin.

**[Engel]** yayına engel. **[Önemli]** ilk ay içinde çözülmeli. **[İyi]** iyi uygulama.

---

## 1. Şifreler ve gizli anahtarlar

Yapay zeka araçlarında en sık görülen hata budur. Veritabanı şifresi, API anahtarı veya ödeme anahtarı kodun içine yazılır ve kodla birlikte her yere gider.

- [ ] **[Engel]** Kodun içinde şifre, API anahtarı veya token yok. Hepsi ortam değişkenlerinden (`.env`) okunuyor.
- [ ] **[Engel]** `.env` dosyası `.gitignore` içinde ve Git geçmişinde hiç yer almamış.
- [ ] **[Engel]** Tarayıcıya giden JavaScript kodunda gizli anahtar yok. Sadece herkese açık olması tasarlanmış anahtarlar olabilir.
- [ ] **[Önemli]** Daha önce bir anahtar sızdıysa değiştirildi. Git geçmişinden silmek yetmez, anahtarın kendisi yenilenmeli.
- [ ] **[İyi]** Geliştirme ve canlı ortam farklı anahtarlar kullanıyor.

**Nasıl kontrol edilir.** [gitleaks](https://github.com/gitleaks/gitleaks) ile depoyu ve tüm Git geçmişini tarayın.

```bash
gitleaks detect -s ./uygulama -v
```

## 2. Giriş ve yetki kontrolü

Yapay zeka genellikle giriş ekranını doğru yapar ama giriş yapan kişinin neye erişebileceğini kontrol etmeyi unutur. Sonuçta bir kullanıcı adres çubuğundaki numarayı değiştirerek başka bir müşterinin kaydını görebilir.

- [ ] **[Engel]** Her sayfa ve her API ucu, giriş yapılmadan açılmıyor. Herkese açık olması gerekenler bilinçli olarak seçildi.
- [ ] **[Engel]** Bir kullanıcı sadece kendi verisini görebiliyor. `/siparis/42` adresini `/siparis/43` yapınca başkasının siparişi açılmıyor.
- [ ] **[Engel]** Yönetici sayfaları sadece yönetici rolüyle açılıyor. Kontrol sadece menüyü gizleyerek değil, sunucu tarafında yapılıyor.
- [ ] **[Engel]** Şifreler veritabanında açık metin olarak değil, bcrypt veya Argon2 ile saklanıyor.
- [ ] **[Önemli]** Hatalı giriş denemeleri sınırlı. Aynı hesaba dakikada yüzlerce deneme yapılamıyor.
- [ ] **[Önemli]** Şifre sıfırlama bağlantıları tek kullanımlık ve kısa ömürlü.
- [ ] **[İyi]** Yönetici hesaplarında iki adımlı doğrulama var.

**Nasıl kontrol edilir.** İki ayrı test kullanıcısı açın. Birinin kaydının adresini diğerinin oturumunda açmayı deneyin. Supabase kullanıyorsanız her tabloda Row Level Security'nin açık olduğunu kontrol edin.

## 3. Kullanıcıdan gelen veri

- [ ] **[Engel]** Veritabanı sorguları parametreli. Kullanıcının yazdığı metin sorguya doğrudan eklenmiyor (SQL injection).
- [ ] **[Engel]** Kullanıcının girdiği metin sayfada gösterilirken kaçış karakterleriyle işleniyor (XSS).
- [ ] **[Önemli]** Dosya yükleme varsa dosya türü ve boyutu sınırlı. Yüklenen dosyalar çalıştırılabilir bir klasöre gitmiyor.
- [ ] **[Önemli]** Uygulama, kullanıcının verdiği bir adrese sunucu tarafından istek atmıyor. Atması gerekiyorsa adres listesi sınırlı (SSRF).
- [ ] **[İyi]** Formlarda CSRF koruması var.

**Nasıl kontrol edilir.** [Semgrep](https://github.com/semgrep/semgrep) kodu bu tür hatalar için tarar. Canlıya benzer bir ortamda [OWASP ZAP](https://www.zaproxy.org/) ile otomatik tarama yapın.

```bash
semgrep scan --config auto ./uygulama
```

## 4. Kullanılan kütüphaneler

Yapay zeka bazen eski, açığı bilinen bir kütüphane sürümünü önerir. Bazen de hiç var olmayan bir paket adı uydurur ve saldırganlar bu adlarla zararlı paketler yayınlar.

- [ ] **[Engel]** Kritik seviyede bilinen açığı olan bağımlılık yok.
- [ ] **[Engel]** Projedeki her paket gerçekten var ve doğru yayıncıya ait. Adı tanıdık gelmeyen paketler tek tek kontrol edildi.
- [ ] **[Önemli]** Sürümler kilit dosyasıyla sabitlendi (`package-lock.json`, `poetry.lock`, `requirements.txt` içinde sürüm numaraları).
- [ ] **[İyi]** Ayda bir bağımlılık güncellemesi takvimde.

**Nasıl kontrol edilir.** [Trivy](https://github.com/aquasecurity/trivy) hem bağımlılıkları hem yapılandırma dosyalarını tarar.

```bash
trivy fs --severity CRITICAL,HIGH ./uygulama
```

## 5. Sunucu ve ağ

- [ ] **[Engel]** Veritabanı portu (5432, 3306, 1433, 27017) internete açık değil.
- [ ] **[Engel]** Sunucuya uzaktan erişim (SSH 22, RDP 3389) internete açık değil. VPN veya onaylı erişimle yapılıyor.
- [ ] **[Engel]** Uygulama HTTPS ile yayında. HTTP istekleri HTTPS'e yönleniyor.
- [ ] **[Engel]** Hata ayıklama modu kapalı. Hata olduğunda kullanıcıya kod veya veritabanı ayrıntısı gösterilmiyor.
- [ ] **[Önemli]** Sunucu bir güvenlik duvarının arkasında. Sadece uygulamanın ihtiyaç duyduğu port dışarıya açık.
- [ ] **[Önemli]** Yönetim panelleri (phpMyAdmin, Adminer, `/admin`) internetten erişilemiyor.
- [ ] **[İyi]** İşletim sistemi güncellemeleri otomatik ya da planlı yapılıyor.

**Nasıl kontrol edilir.** Sunucunuzun dışarıdan hangi portlarının açık göründüğüne başka bir ağdan bakın. Daha kapsamlı tarama için [Greenbone OpenVAS](https://github.com/greenbone) kullanılabilir.

```bash
nmap -Pn -p- sunucu-adresiniz
```

## 6. Yedekleme

Yedeği alınmamış bir uygulama, tek bir hatalı komutla bütün verisini kaybedebilir.

- [ ] **[Engel]** Veritabanı en az günde bir kez yedekleniyor.
- [ ] **[Engel]** Yedekler uygulamanın çalıştığı sunucudan başka bir yerde duruyor.
- [ ] **[Önemli]** Yedekten geri dönüş en az bir kez denendi. Hiç geri yüklenmemiş yedek, yedek sayılmaz.
- [ ] **[Önemli]** Fidye yazılımına karşı en az bir yedek kopya silinemez ya da çevrim dışı.
- [ ] **[İyi]** 3-2-1 kuralı uygulanıyor. Üç kopya, iki farklı ortam, bir kopya başka bir yerde.

## 7. Kayıt ve izleme

- [ ] **[Önemli]** Girişler, başarısız giriş denemeleri ve yönetici işlemleri kayıt altında.
- [ ] **[Önemli]** Kayıtlarda şifre, kart numarası veya kimlik numarası gibi hassas veri yok.
- [ ] **[Önemli]** Uygulama çöktüğünde ya da yanıt vermediğinde birine haber gidiyor.
- [ ] **[İyi]** Disk dolmadan önce uyarı geliyor.

## 8. KVKK

Bu bölüm hukuki danışmanlık yerine geçmez. Uygulamanız kişisel veri işliyorsa bir hukukçuyla da görüşün.

- [ ] **[Engel]** Uygulamanın hangi kişisel verileri topladığı ve nerede sakladığı biliniyor.
- [ ] **[Engel]** Kullanıcılara aydınlatma metni gösteriliyor. Açık rıza gereken işlemler için ayrıca rıza alınıyor.
- [ ] **[Önemli]** Veriler yurt dışındaki bir sunucuda duruyorsa bu aktarımın KVKK'nın 9. maddesindeki şartlara uygun olduğu kontrol edildi. Vercel, Supabase ve benzeri platformların sunucularının hangi ülkede olduğunu öğrenin.
- [ ] **[Önemli]** Bir veri ihlali olduğunda kimin, ne yapacağı ve Kurul'a nasıl bildirileceği önceden belli.
- [ ] **[İyi]** Gereğinden fazla veri toplanmıyor ve saklama süresi dolan veriler siliniyor.

## 9. Yayından sonra bakım

Uygulama yayına alındığı gün bitmez. Bakımsız kalan uygulama birkaç ay içinde güvenlik açığına dönüşür.

- [ ] Ayda bir bağımlılık taraması (`trivy`) ve güncelleme
- [ ] Ayda bir yedekten geri dönüş testi
- [ ] Ayda bir dışarıdan port taraması
- [ ] Her büyük değişiklikten sonra `gitleaks` ve `semgrep` taraması
- [ ] Uygulamayı yazan kişi ayrılırsa ne olacağı yazılı. Şifreler, sunucu erişimi ve dağıtım adımları belgelenmiş.

---

## Yapay zekaya düzelttirirken

Kodu yazan araca güvenliği de sorabilirsiniz ama doğru sormak gerekir. Aşağıdaki gibi açık talimatlar daha iyi sonuç verir.

```text
Bu projedeki tüm API uçlarını listele. Her biri için giriş kontrolü ve
kaynak sahipliği kontrolü yapılıp yapılmadığını tablo halinde göster.
Eksik olanları düzelt ama başka hiçbir davranışı değiştirme.
```

```text
Koddaki tüm şifre, anahtar ve token kullanımlarını bul. Hepsini ortam
değişkenlerine taşı ve bir .env.example dosyası oluştur. Gerçek değerleri
hiçbir dosyaya yazma.
```

```text
Veritabanına giden tüm sorguları bul. Kullanıcı girdisini doğrudan
birleştiren sorgu varsa parametreli sorguya çevir.
```

Yapay zekanın "düzelttim" demesi düzeldiği anlamına gelmez. Düzeltmeden sonra yukarıdaki taramaları tekrar çalıştırın.

## Katkı

Eksik gördüğünüz bir madde, yanlış bir bilgi ya da daha iyi bir araç önerisi varsa issue açın veya pull request gönderin.

## Hazırlayan

**Oktay Alver.** 2005'ten beri kurumsal altyapı, ağ ve siber güvenlik alanında çalışıyor. ADEA Sistem'in kurucusu.

- ORCID [0009-0001-4055-1603](https://orcid.org/0009-0001-4055-1603)
- LinkedIn [oktayalver](https://www.linkedin.com/in/oktayalver/)
- [ADEA Sistem](https://www.adeasis.com/ai-uygulama-barindirma-ve-isletim/), AI ile yazılan uygulamalar için barındırma, güvenlik ve işletim

## Lisans

Bu içerik [Creative Commons Atıf 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/deed.tr) lisansıyla paylaşılmıştır. Kaynak göstermek şartıyla kopyalayabilir, uyarlayabilir ve ticari amaçla kullanabilirsiniz.
