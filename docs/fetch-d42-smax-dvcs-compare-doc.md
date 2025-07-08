# fetch-d42-smax-dvcs-compare.yaml Çalışma Mantığı ve Akış Dokümantasyonu

Bu doküman, `playbooks/flow_playbooks/fetch-d42-smax-dvcs-compare.yaml` Ansible playbook'unun adım adım nasıl çalıştığını, kullanılan değişkenleri, task'ların işlevlerini ve Device42 ile SMAX sistemleri arasındaki veri karşılaştırma ve senkronizasyon mantığını açıklamaktadır.

---

## Genel Amaç

Bu playbook, Device42 ve SMAX sistemlerindeki cihazları karşılaştırır, eksik veya güncellenmesi gereken cihazları tespit eder ve SMAX'e uygun formatta insert (yeni ekleme) veya update (güncelleme) işlemlerini toplu olarak gerçekleştirir.

---

## Kullanılan Değişkenler

- **device42_api_url, device42_auth_header**: Device42 API erişimi için gerekli URL ve yetkilendirme bilgileri.
- **smax_token_url, smax_user, smax_pass, smax_api_url, smax_layout, smax_tenantid**: SMAX API erişimi ve oturum açma için gerekli bilgiler.

Tüm bu değişkenler genellikle AWX Extra Vars ile dışarıdan playbook'a aktarılır.

# D42 için
device42_api_url: "https://213.153.197.248:8585/api/1.0/devices/all"
device42_auth_header: "Basic YWRtaW46QWExMjM0NTYu."
# SMAX için
smax_token_url: "https://destek.kafein.com.tr/auth/authentication-endpoint/authenticate/token?TENANTID=669866050"  
smax_user: "duosis.admin"
smax_pass: "duosisITSM.25"
smax_api_url: "https://destek.kafein.com.tr/rest/669866050/ems/Device"
smax_layout: "LastUpdateTime,DisplayLabel,Description,Cpus,MemorySize,RunningSoftwares,D42id_c,IpAddresses,OsName,DiskDevices,NetworkCards"
smax_tenantid: "669866050"

---

## Adım Adım Akış

### 1. Device42 ve SMAX'ten Cihazların Çekilmesi
- Device42 API'den cihazlar çekilir ve `d42_devices` değişkenine atanır.
- SMAX'e login olunur, token alınır ve SMAX API'den cihazlar çekilerek `smax_devices` değişkenine atanır.

### 2. SMAX Cihazlarının Device42 Formatına Dönüştürülmesi
- SMAX cihazları, Device42 ile karşılaştırma yapılabilmesi için uygun bir dict formatına (`smax_devices_mapped`) dönüştürülür.
- Burada, SMAX cihazlarının property'lerinden gerekli alanlar (ör. name, ip, memory, os, SMAX_ID vs.) çekilir ve dönüştürülür.

### 3. Cihaz Adlarının Karşılaştırılması
- Device42 ve SMAX cihazlarının adları (`name`) listelenir.
- Sadece Device42'de olup SMAX'te olmayan cihazlar (`yeni_yazdirilacak_cihazlar`) ve her iki tarafta da olan cihazlar belirlenir.

### 4. Epoch (LastUpdate) Karşılaştırması
- Ortak cihazlar için Device42 ve SMAX'teki son güncelleme zamanları epoch formatında karşılaştırılır.
- Device42'deki güncelleme zamanı SMAX'tekinden daha yeni olan cihazlar, güncellenecek cihazlar (`guncellenecek_cihazlar`) olarak işaretlenir.

### 5. Update ve Insert Payloadlarının Hazırlanması
- Güncellenecek cihazlar için, SMAX update endpoint'ine uygun JSON payload'ları (`update_payloads`) oluşturulur.
- Yeni yazdırılacak cihazlar için, SMAX insert endpoint'ine uygun JSON payload'ları (`insert_payloads`) oluşturulur.
- NetworkCards, DiskDevices, Cpus, IpAddresses gibi alanlar JSON string olarak ve SMAX'in beklediği attribute isimleriyle hazırlanır.

### 6. SMAX'e Toplu Update ve Insert İşlemleri
- Hazırlanan update payload'ları, SMAX'e toplu olarak gönderilir.
- Hazırlanan insert payload'ları, SMAX'e toplu olarak gönderilir.
- Her iki işlemde de sonuçlar kaydedilir ve gerekirse debug amaçlı gösterilir.

---

## Önemli Noktalar ve Hatalara Karşı Önlemler
- Jinja2 ile list/dict oluştururken tip ve quote yönetimine dikkat edilmiştir.
- Epoch değerleri int'e çevrilerek karşılaştırma hataları önlenmiştir.
- SMAX'e gönderilen payload formatı, SMAX API'nin beklediği yapıya birebir uyumludur.
- Her adımda debug çıktıları ile veri akışı ve dönüşümler izlenebilir.

---

## Akış Diyagramı (Özet)

1. Device42 cihazlarını çek → 2. SMAX cihazlarını çek → 3. SMAX cihazlarını dönüştür → 4. Ad listelerini karşılaştır → 5. Epoch karşılaştırması → 6. Update/Insert payloadlarını hazırla → 7. SMAX'e gönder

---

## Kullanım Senaryosu
- Bu playbook, iki sistem arasında cihaz envanterinin senkronize tutulmasını sağlar.
- Otomatik olarak eksik cihazları ekler, güncel olmayanları günceller.
- Büyük ölçekli ortamlarda manuel iş yükünü azaltır ve veri tutarlılığını artırır.

---

## Ek Notlar
- Playbook, Ansible 2.9+ ve Jinja2 ile uyumludur.
- API endpoint ve authentication bilgileri dışarıdan parametre olarak verilmelidir.
- Hatalı veya eksik veri durumunda debug task'ları ile kolayca sorun tespiti yapılabilir.

---

Daha fazla detay veya örnek için ilgili playbook dosyasına ve mapping analiz dokümanlarına bakabilirsiniz. 