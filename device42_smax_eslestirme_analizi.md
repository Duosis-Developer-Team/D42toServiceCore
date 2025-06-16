# Device42 - SMAX Veri Yapısı Karşılaştırmalı Eşleştirme ve Analiz Raporu

## 1. Genel Yapı Karşılaştırması

| Özellik         | Device42 JSON                                 | SMAX JSON (entities/properties)         |
|-----------------|----------------------------------------------|-----------------------------------------|
| Kök Alan        | Devices (array)                              | entities (array)                        |
| Cihaz Detayı    | Her bir cihaz bir obje                       | Her bir entity bir obje                 |
| Alt Nesneler    | Dizi veya obje olarak (ör: ip_addresses)     | JSON string olarak (ör: IpAddresses)    |
| Meta Bilgi      | limit, total_count, offset                    | meta (completion_status, error, vs.)    |

---

## 2. Temel Alan Eşleştirmeleri

| Device42 Alanı      | SMAX Alanı (properties)      | Açıklama/Eşleştirme Notu |
|---------------------|-----------------------------|--------------------------|
| name                | DisplayLabel                | Cihaz adı                |
| id / device_id      | Id                          | Cihaz benzersiz ID'si    |
| ram                 | MemorySize                  | RAM, MB cinsinden        |
| cpucount, cpucore   | Cpus (JSON string)          | CPU sayısı/çekirdek      |
| serial_no           | (Eklenebilir)               | Seri numarası            |
| os                  | OsName                      | İşletim sistemi adı      |
| osver, osverno      | (Eklenebilir)               | OS versiyonu             |
| ip_addresses        | IpAddresses (JSON string)   | IP adresleri             |
| hdd_details, hddsize| DiskDevices (JSON string)   | Disk bilgileri           |
| last_updated        | LastUpdateTime              | Son güncelleme zamanı    |
| category            | (Eklenebilir)               | Kategori                 |
| tags                | (Eklenebilir)               | Etiketler                |

---

## 3. Alt Nesne ve Dizi Alanlar

### IP Adresleri
- **Device42:**  
  "ip_addresses": [ {"ip": "192.168.1.1", ...} ]
- **SMAX:**  
  "IpAddresses": "{\"IpAddress\":[{\"IpValue\":\"192.168.1.1\", ...}]}"
- **Eşleştirme:**  
  Device42'deki her bir ip_addresses elemanı, SMAX'teki IpAddresses JSON string'inde bir IpAddress objesine dönüştürülmeli.

### CPU Bilgisi
- **Device42:**  
  cpucount, cpucore, cpuspeed gibi alanlar ayrı ayrı.
- **SMAX:**  
  "Cpus": "{\"Cpu\":[{\"CpuClockSpeed\":\"2400\",\"CpuCoreNumber\":\"2\", ...}]}"
- **Eşleştirme:**  
  Device42'deki CPU ile ilgili alanlar, SMAX'in beklediği JSON string formatına dönüştürülmeli.

### Disk Bilgisi
- **Device42:**  
  hdd_details (array), hddsize, hddcount gibi alanlar.
- **SMAX:**  
  "DiskDevices": "{\"DiskDevice\":[{\"DiskSize\":\"100\",\"DiskModelName\":\"scsi\", ...}]}"
- **Eşleştirme:**  
  Device42'deki disk detayları, SMAX'in DiskDevices JSON string'ine uygun şekilde dönüştürülmeli.

---

## 4. Dönüşüm ve Dikkat Edilmesi Gerekenler

- **JSON String Formatı:**  
  SMAX, bazı alanları (IpAddresses, Cpus, DiskDevices) JSON string olarak bekliyor. Device42'den gelen dizi/obje yapıları, SMAX'e gönderilmeden önce string'e çevrilmeli.
- **Boş/Null Değerler:**  
  Device42'de bazı alanlar null veya boş string olabiliyor. SMAX'e gönderirken bu alanlar ya boş bırakılmalı ya da default değer atanmalı.
- **Alan Adı Farklılıkları:**  
  Alan isimleri birebir aynı değil; eşleştirme sırasında mapping yapılmalı.
- **Ek Alanlar:**  
  Device42'de olup SMAX'te olmayan (ör: custom_fields, tags) veya SMAX'te olup Device42'de olmayan (ör: DisplayLabel) alanlar olabilir. Gerekirse eklenmeli veya atlanmalı.
- **Zaman Formatı:**  
  Device42'de ISO datetime, SMAX'te epoch (milisaniye) kullanılıyor. Dönüşüm yapılmalı.

---

## 5. Örnek Eşleştirme Tablosu

| Device42 Alanı         | SMAX Alanı (properties)      | Dönüşüm Notu                |
|------------------------|-----------------------------|-----------------------------|
| name                   | DisplayLabel                | Direkt                      |
| id / device_id         | Id                          | Direkt                      |
| ram                    | MemorySize                  | MB cinsinden aktarılmalı    |
| cpucount, cpucore      | Cpus                        | JSON string'e dönüştür      |
| serial_no              | (Eklenebilir)               | SMAX'te custom alan olabilir|
| os                     | OsName                      | Direkt                      |
| osver, osverno         | (Eklenebilir)               | SMAX'te custom alan olabilir|
| ip_addresses           | IpAddresses                 | JSON string'e dönüştür      |
| hdd_details, hddsize   | DiskDevices                 | JSON string'e dönüştür      |
| last_updated           | LastUpdateTime              | ISO'dan epoch'a çevir       |
| category, tags         | (Eklenebilir)               | SMAX'te custom alan olabilir|

---

## 6. Sonuç

- **Mapping ve Dönüşüm:**  
  Device42'den gelen veriler, SMAX'in beklediği yapıya uygun şekilde dönüştürülmeli. Özellikle dizi/obje alanlar JSON string'e çevrilmeli ve zaman formatı dönüştürülmeli.
- **Eksik/Boş Alanlar:**  
  Boş veya eksik alanlar için iş kurallarına göre default değer atanabilir veya alan atlanabilir.
- **Alan Eşleştirme:**  
  Temel alanlar kolayca eşleştirilebilir, ancak bazı özel alanlar için ek mapping veya custom alan tanımı gerekebilir.

---

### Not: RAM için Eşleştirme
- Device42'deki `ram` alanı, SMAX'te `MemorySize` olarak properties altında aktarılmalıdır.
- Örnek:  
  Device42:  
  ```json
  "ram": 4096
  ```
  SMAX:  
  ```json
  "MemorySize": 4096
  ```

Bu eşleştirme ile RAM bilgisi SMAX'te doğru şekilde gösterilecektir.

Başka bir alan için de eşleştirme veya örnek isterseniz belirtmeniz yeterli! 