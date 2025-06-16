# Device42 JSON Alan Analizi Raporu

## Genel Yapı
- **limit**: (int) API çağrısında bir seferde dönen maksimum kayıt sayısı.
- **Devices**: (array) Sunucu/cihaz nesnelerinin listesi.
- **total_count**: (int) Toplam cihaz sayısı.
- **offset**: (int) Sayfalama için başlangıç indeksi.

---

## Devices Dizisi İçindeki Alanlar

Her bir cihaz (device) için aşağıdaki alanlar mevcut:

| Alan Adı                | Tipi         | Açıklama / Örnek / Kullanım |
|-------------------------|--------------|-----------------------------|
| last_updated            | string (ISO datetime) | Son güncellenme zamanı. |
| hw_size                 | int/null     | Donanım boyutu (rack unit gibi). |
| uuid                    | string       | Cihaza özel benzersiz kimlik. |
| ip_addresses            | array        | Cihaza ait IP adresleri (her biri obje). |
| notes                   | string       | Açıklama/notlar. |
| ram                     | int/float    | RAM miktarı (MB cinsinden). |
| hw_model_id             | int/null     | Donanım modeli ID'si. |
| cpuspeed                | int/string   | CPU hızı (MHz veya boş). |
| hw_depth                | int/null     | Donanım derinliği (rack için). |
| virtual_host_name       | string/null  | Sanal makine ise host ismi. |
| mac_addresses           | array        | MAC adresleri (her biri obje). |
| hddraid                 | string/null  | RAID tipi (örn: Hardware). |
| type                    | string       | Cihaz tipi (örn: physical, virtual). |
| device_id               | int          | Cihazın benzersiz ID'si. |
| is_it_blade_host        | string       | Blade host olup olmadığı ("yes"/"no"). |
| cpucount                | int          | CPU sayısı. |
| is_it_virtual_host      | string       | Sanal host olup olmadığı ("yes"/"no"). |
| virtual_subtype         | string/null  | Sanal makine alt tipi (örn: VMWare). |
| manufacturer            | string/null  | Üretici firma. |
| hdd_details             | array/null   | Disk detayları (her biri obje). |
| datastores              | array        | Bağlı veri depoları (string). |
| hddraid_type            | string/null  | RAID tipi detayı. |
| preferred_alias         | string/null  | Tercih edilen takma ad. |
| ucs_manager             | string/null  | UCS yöneticisi. |
| hw_model                | string/null  | Donanım modeli adı. |
| virtual_subtype_id      | int/null     | Sanal makine alt tipi ID'si. |
| hddcount                | int/string   | Disk sayısı. |
| nonauthoritativealiases | array        | Diğer takma adlar. |
| is_it_switch            | string       | Switch olup olmadığı ("yes"/"no"). |
| tags                    | array        | Etiketler (string). |
| in_service              | bool         | Serviste olup olmadığı. |
| service_level           | string       | Servis seviyesi (örn: Production). |
| hddsize                 | int/float/string | Toplam disk boyutu (GB). |
| corethread              | int          | Çekirdek başına thread sayısı. |
| aliases                 | array        | Alternatif isimler. |
| os                      | string/null  | İşletim sistemi adı. |
| name                    | string       | Cihaz adı. |
| customer                | string/null  | Müşteri adı. |
| device_external_links   | array        | Harici bağlantılar. |
| asset_no                | string       | Varlık numarası. |
| custom_fields           | array        | Özelleştirilmiş alanlar (her biri obje: key, value, notes). |
| category                | string       | Kategori. |
| id                      | int          | Cihaz ID'si (bazı API'lerde device_id ile aynı olabilir). |
| cpucore                 | int          | CPU çekirdek sayısı. |
| device_purchase_line_items | array     | Satın alma kalemleri. |
| serial_no               | string       | Seri numarası. |
| device_sub_type         | string/null  | Cihaz alt tipi (örn: Generic, Rackable). |
| osverno                 | string/null  | İşletim sistemi versiyon numarası. |
| customer_id             | int/null     | Müşteri ID'si. |
| osver                   | string/null  | İşletim sistemi versiyonu. |
| osarch                  | int/null     | İşletim sistemi mimarisi (örn: 64). |
| start_at                | int/null     | Başlangıç zamanı (bazı cihazlarda var). |
| rack                    | string/null  | Rack adı. |
| rack_id                 | int/null     | Rack ID'si. |
| row                     | string/null  | Rack satırı. |
| room                    | string/null  | Oda adı. |
| orientation             | int/null     | Rack yönü. |
| where                   | int/null     | Konum ID'si. |
| building                | string/null  | Bina adı. |
| datacenter              | string/null  | Veri merkezi adı. |

### Alt Nesneler

- **ip_addresses**:  
  Her biri şu alanlara sahip olabilir:  
  - type, subnet, subnet_id, ip, label, macaddress

- **mac_addresses**:  
  Her biri şu alanlara sahip olabilir:  
  - port_name, port, vlan, mac

- **hdd_details**:  
  Her biri şu alanlara sahip olabilir:  
  - description, hdd (içinde: partno, type, rpm, bytes, notes, manufacturer_id, description, hd_id, size, location), hddcount, raid_group, serial_no, part_id

- **custom_fields**:  
  Her biri şu alanlara sahip:  
  - key, value, notes

---

## Alanların Kullanım Notları

- **Zorunlu Alanlar:**  
  - name, device_id/id, type, ram, cpucount, cpucore, serial_no, os, in_service gibi alanlar genellikle SMAX veya başka sistemlere aktarımda gereklidir.
- **Opsiyonel/Null Olabilen Alanlar:**  
  - manufacturer, hw_model, virtual_subtype, osver, osverno, customer, asset_no, rack, datacenter gibi alanlar her cihazda olmayabilir.
- **Diziler:**  
  - ip_addresses, mac_addresses, hdd_details, custom_fields, tags, aliases gibi alanlar birden fazla değer içerebilir veya boş olabilir.
- **Boş/Null Değerler:**  
  - Bazı alanlar (ör: cpuspeed, manufacturer, hw_model_id) boş string veya null dönebiliyor. Aktarımda dikkat edilmeli.

---

## Örnek Kullanım Senaryosu

- **SMAX'e Aktarım:**  
  - Playbook'ta, yukarıdaki alanlardan bazıları doğrudan SMAX'e aktarılıyor (ör: name, ram, asset_no, cpucount, cpucore, serial_no, os, osver, category, ip_addresses[0].ip).
  - Bazı alanlar (ör: custom_fields) SMAX'te karşılığı varsa eklenebilir, yoksa atlanabilir.
  - Dizi alanlarda (ör: ip_addresses) genellikle ilk eleman kullanılıyor, ama birden fazla IP/MAC varsa iş kurallarına göre işlenmeli.

---

## Sonuç

Bu JSON yapısı, Device42'den gelen cihaz verilerini detaylı şekilde içeriyor.  
Her alanın anlamı ve kullanımı yukarıda özetlenmiştir.  
SMAX veya başka bir sisteme aktarımda, zorunlu ve opsiyonel alanlar ile dizi/boş değer durumlarına dikkat edilmelidir.

Başka bir alan veya özel bir cihaz örneği için detaylı analiz isterseniz belirtmeniz yeterli! 