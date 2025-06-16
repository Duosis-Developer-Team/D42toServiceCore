# SMAX Response JSON Alan Analizi Raporu

## Genel Yapı
- **entities**: (array) SMAX'e gönderilen veya SMAX'ten dönen varlıkların (ör: Device) listesi. Her biri bir cihazı temsil eder.
- **meta**: (object) İşlemin durumu, hata detayları ve sorgu süresi gibi meta bilgileri içerir.

---

## entities Dizisi İçindeki Alanlar

Her bir entity (örnekte bir adet Device var):

### entity_type
- **entity_type**: "Device"  
  Varlık tipi. Burada bir sunucu/cihaz.

### properties
- **properties**: (object)  
  Cihaza ait tüm özellikler burada tutulur.  
  Önemli alanlar:

| Alan Adı         | Tipi      | Açıklama / Örnek / Kullanım |
|------------------|-----------|-----------------------------|
| IpAddresses      | string (JSON) | IP adresleri, JSON string olarak tutulmuş. İçinde bir veya birden fazla IP olabilir. |
| LastUpdateTime   | int       | Son güncelleme zamanı (epoch timestamp, milisaniye cinsinden). |
| OsName           | string    | İşletim sistemi adı. |
| Id               | string    | Cihazın SMAX'teki benzersiz ID'si. |
| Cpus             | string (JSON) | CPU bilgileri, JSON string olarak tutulmuş. Birden fazla CPU olabilir. |
| DiskDevices      | string (JSON) | Disk bilgileri, JSON string olarak tutulmuş. Birden fazla disk olabilir. |
| DisplayLabel     | string    | Cihazın görünen adı/etiketi. |

#### JSON String Olarak Tutulan Alanlar

- **IpAddresses**:  
  Birden fazla IP adresi desteklenir. Her IP için tip, DNS adı, routing domain gibi ek alanlar mevcut.
- **Cpus**:  
  Birden fazla CPU desteği. Her CPU için tip, vendor, hız, çekirdek sayısı gibi bilgiler.
- **DiskDevices**:  
  Birden fazla disk desteği. Her disk için isim, boyut, model, tip gibi bilgiler.

### related_properties
- **related_properties**: (object)  
  İlişkili başka varlıkların özellikleri için kullanılır. Bu örnekte boş.

---

## meta Alanı

| Alan Adı               | Tipi      | Açıklama |
|------------------------|-----------|----------|
| completion_status      | string    | İşlemin başarılı olup olmadığını gösterir (örn: "OK"). |
| total_count            | int       | Dönen toplam kayıt sayısı. |
| errorDetailsList       | array     | Hata detayları (boş ise hata yok). |
| errorDetailsMetaList   | array     | Hata meta verileri (boş ise hata yok). |
| query_time             | int       | Sorgunun tamamlanma zamanı (timestamp). |

---

## Değerlendirme ve Notlar

- **Veri Modeli:**  
  SMAX, ilişkili alanları (IP, CPU, Disk gibi) JSON string olarak saklıyor. Bu alanlar, SMAX API'sine gönderilirken veya alınırken string olarak encode/decode edilmeli.
- **Zorunlu Alanlar:**  
  DisplayLabel, Id, OsName, Cpus, IpAddresses, DiskDevices gibi alanlar genellikle zorunlu.
- **Boş Alanlar:**  
  Bazı alt alanlar (ör: IpType, AuthoritativeDNSName, DiskName) boş bırakılmış. Gerekirse doldurulabilir.
- **ID Alanları:**  
  Her alt nesneye (IP, CPU, Disk) ait benzersiz bir Id mevcut.
- **Zaman Alanı:**  
  LastUpdateTime epoch formatında, milisaniye cinsinden tutuluyor.

---

## Sonuç

SMAX response yapısı, gönderilen cihazın tüm detaylarını ve ilişkili alt nesnelerini (IP, CPU, Disk) JSON string olarak içeriyor.  
İşlem başarılı ise `completion_status` "OK" olarak dönüyor ve hata listeleri boş oluyor.  
Bu yapı, Device42'den gelen verilerin SMAX'e aktarımı ve SMAX'ten alınan yanıtların işlenmesi için uygundur.

Başka bir alan veya örnek için detaylı analiz isterseniz belirtmeniz yeterli! 