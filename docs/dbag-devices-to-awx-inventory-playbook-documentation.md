# Dynamic Custom-Field Devices to AWX Inventory Playbook Documentation

## Genel Bakış

Bu playbook, Device42'dan cihazları çekerek, verilen custom field isimleri (CSV) cihazın custom field anahtarları içinde (case-sensitive) geçiyorsa o cihazın yalnızca IPv4 adreslerini AWX'te aynı isimli envanterlere ekler. Envanterler yoksa ilgili organization altında otomatik oluşturulur.

## Playbook Dosyası
- **Dosya Adı**: `dbag-devices-to-awx-inventory.yaml`
- **Konum**: `playbooks/flow_playbooks/`
- **Hosts**: localhost (API çağrıları için)

## Çalışma Prensibi

### 1. AWX API Bağlantı Testi
Playbook çalışmaya başladığında, önce AWX API'sine bağlantıyı test eder:

```yaml
- name: AWX API bağlantısını test et
  uri:
    url: "{{ awx_api_url }}/ping/"
    method: GET
    headers:
      Authorization: "Bearer {{ awx_token }}"
    validate_certs: no
    timeout: 30
  register: awx_ping_result
```

### 2. Device42'dan Cihaz Verilerini Çekme
Device42 API'sine bağlanarak tüm cihaz bilgilerini çeker:

```yaml
- name: Device42'dan cihazları çek
  uri:
    url: "{{ device42_api_url }}"
    method: GET
    user: "{{ device42_username }}"
    password: "{{ device42_password }}"
    force_basic_auth: yes
    return_content: yes
    validate_certs: no
  register: d42_response
```

### 3. Custom Field Filtreleme ve IPv4 Toplama
`custom_field_names_csv` içinde verilen isimler için (örn: "DBAG,critical,internal") case-sensitive içerme kontrolü uygulanır ve eşleşen cihazların yalnız IPv4'leri toplanır. Her isim, AWX'te aynı isimdeki envantere karşılık gelir.

```yaml
- name: dbag custom field'ı olan cihazları filtrele
  set_fact:
    dbag_devices: "{{ d42_response.json.Devices | selectattr('custom_fields', 'defined') | list }}"
```

### 4. IP Adreslerini Toplama
DBAG custom field'ına sahip cihazların IP adreslerini toplar:

```yaml
- name: dbag custom field'ı olan cihazları ve ip'lerini bul
  set_fact:
    dbag_hosts: "{{ dbag_hosts | default([]) + (item.ip_addresses | map(attribute='ip') | list) }}"
  loop: "{{ dbag_devices }}"
  when: >
    item.custom_fields is defined and
    item.custom_fields | selectattr('key', 'search', '^dbag', ignorecase=True) | list | length > 0
```

### 4. Unique IPv4 Listeleri Oluşturma
Her envanter adı için IP listeleri tekilleştirilir.

```yaml
- name: dbag_hosts listesini unique yap
  set_fact:
    dbag_hosts: "{{ dbag_hosts | unique }}"
```

### 5. Envanter Oluştur/Var Olanı Kullan ve Host Ekle
Her isim için önce AWX'te envanter araması yapılır (`/inventories/?name=<name>&organization=<orgId>`). Yoksa oluşturulur (`POST /inventories`). Ardından ilgili IPv4'ler `POST /inventories/{id}/hosts/` ile eklenir (201/409 kabul edilir).

```yaml
- name: dbag inventory'sine host ekle
  uri:
    url: "{{ awx_api_url }}/inventories/{{ dbag_inventory_id }}/hosts/"
    method: POST
    headers:
      Authorization: "Bearer {{ awx_token }}"
      Content-Type: "application/json"
    body_format: json
    body:
      name: "{{ item }}"
      inventory: "{{ dbag_inventory_id }}"
    status_code: [201, 409]  # 409 = zaten var
    validate_certs: no
    timeout: 30
  loop: "{{ dbag_hosts }}"
  when: dbag_hosts | length > 0
```

## Değişkenler

### Zorunlu Değişkenler
- `device42_api_url`: Device42 API endpoint URL'i
- `device42_username`: Device42 kullanıcı adı
- `device42_password`: Device42 şifresi
- `awx_api_url`: AWX API endpoint URL'i (örn: `https://awx.example.com/api/v2`)
- `awx_token`: AWX API token'ı
- `awx_organization_id`: AWX Organization ID (envanter oluşturma için)
- `custom_field_names_csv`: Virgülle ayrılmış custom field isimleri (örn: "dbag,critical,internal")

## API Entegrasyonları

### Device42 API
- **Endpoint**: `{{ device42_api_url }}`
- **Authentication**: Basic Auth (username/password)
- **Method**: GET
- **Response**: JSON formatında cihaz listesi

### AWX API
- **Ping**: `{{ awx_api_url }}/ping/`
- **Inventory Search**: `{{ awx_api_url }}/inventories/?name=<name>&organization=<orgId>`
- **Inventory Create**: `{{ awx_api_url }}/inventories/`
- **Hosts Create**: `{{ awx_api_url }}/inventories/<id>/hosts/`
- **Auth**: Bearer Token

## Filtreleme Mantığı

### Custom Field Kontrolü
Playbook, Device42'daki cihazların custom field'larını kontrol eder ve şu kriterlere uyanları seçer:

1. **Custom Fields Varlığı**: Cihazın `custom_fields` özelliği tanımlı olmalı.
2. **İçerme (Case-Sensitive)**: `custom_fields[].key` içinde hedef isim tam haliyle geçmeli (ör. `DBAG` → `DBAG - Asset Owner`).
3. **IPv4**: `ip_addresses[].ip` sadece IPv4 regex'ine uyanlar alınır.

### Örnek Filtreleme
```json
{
  "Devices": [
    {
      "name": "Server01",
      "custom_fields": [
        {"key": "dbag_environment", "value": "production"},
        {"key": "dbag_team", "value": "infrastructure"}
      ],
      "ip_addresses": [
        {"ip": "192.168.1.100"},
        {"ip": "10.0.0.100"}
      ]
    },
    {
      "name": "Server02",
      "custom_fields": [
        {"key": "environment", "value": "development"}
      ],
      "ip_addresses": [
        {"ip": "192.168.1.101"}
      ]
    }
  ]
}
```

Bu örnekte sadece "Server01" DBAG custom field'ına sahip olduğu için seçilir.

## Hata Yönetimi

### AWX API Hataları
- **401 Unauthorized**: Token geçersiz veya süresi dolmuş
- **404 Not Found**: Inventory ID bulunamadı
- **500 Internal Server Error**: AWX sunucu hatası

### Device42 API Hataları
- **401 Unauthorized**: Kullanıcı adı/şifre hatalı
- **403 Forbidden**: API erişim yetkisi yok
- **500 Internal Server Error**: Device42 sunucu hatası

### Duplicate Host Yönetimi
- **Status Code 409**: Host zaten inventory'de mevcut (hata olarak değerlendirilmez)
- **Status Code 201**: Host başarıyla eklendi

## Kullanım Senaryoları

### 1. AWX Template ile Çalıştırma
```yaml
device42_api_url: "https://device42.company.com/api/1.0/devices/"
device42_username: "api_user"
device42_password: "api_password"
awx_api_url: "https://awx.company.com/api/v2"
awx_token: "your_awx_token"
awx_organization_id: 42
custom_field_names_csv: "dbag,critical,internal"
```

### 2. Manuel Çalıştırma
```bash
ansible-playbook playbooks/flow_playbooks/dbag-devices-to-awx-inventory.yaml \
  -e "device42_api_url=https://device42.company.com/api/1.0/devices/" \
  -e "device42_username=api_user" \
  -e "device42_password=api_password" \
  -e "awx_api_url=https://awx.company.com/api/v2" \
  -e "awx_token=your_awx_token" \
  -e "awx_organization_id=42" \
  -e "custom_field_names_csv=dbag,critical,internal"
```

### 3. Scheduled Job
AWX'de bu playbook'u düzenli olarak çalıştırmak için:
- **Schedule**: Her saat başı
- **Purpose**: Device42 değişikliklerini AWX'e senkronize etmek

## Çıktı Örnekleri

### Başarılı Çalışma
```
TASK [AWX API test sonucunu göster]
ok: [localhost] => {
    "awx_ping_result": {
        "status": 200,
        "json": {
            "version": "21.8.0",
            "active": true
        }
    }
}

TASK [Bulunan IP'leri debug et]
ok: [localhost] => {
    "msg": "DBAG IP'leri: ['192.168.1.100', '192.168.1.101', '10.0.0.100']"
}

TASK [dbag inventory'sine host ekle]
ok: [localhost] => (item=192.168.1.100) => {
    "status": 201,
    "json": {
        "id": 456,
        "name": "192.168.1.100",
        "inventory": 123
    }
}
```

### Hata Durumu
```
TASK [AWX API bağlantısını test et]
fatal: [localhost]: FAILED! => {
    "status": 401,
    "msg": "Unauthorized"
}
```

## Güvenlik Özellikleri

### API Güvenliği
- **HTTPS**: Tüm API çağrıları HTTPS üzerinden yapılır
- **Token Authentication**: AWX için Bearer token kullanılır
- **Basic Auth**: Device42 için username/password kullanılır
- **Certificate Validation**: `validate_certs: no` (gerekirse değiştirilebilir)

### Veri Güvenliği
- **Sensitive Variables**: Şifreler ve token'lar AWX'de encrypted olarak saklanır
- **Timeout**: API çağrıları için 30 saniye timeout
- **Error Handling**: Hata durumlarında güvenli çıkış

## Monitoring ve Logging

### Debug Çıktıları
- AWX API URL debug edilir
- Bulunan IP'ler listelenir
- API test sonuçları gösterilir

### AWX Job Logs
- Her çalıştırma AWX'de loglanır
- Başarı/hata durumları takip edilebilir
- Eklenen host sayısı görülebilir

## Geliştirme Önerileri

1. **Incremental Sync**: Sadece değişen cihazları senkronize etme
2. **Host Removal**: Device42'den silinen cihazları AWX'den de silme
3. **Custom Fields Mapping**: Device42 custom field'larını AWX host variables'a map etme
4. **Notification**: Senkronizasyon sonucu email/Slack bildirimi
5. **Backup**: Inventory yedekleme mekanizması
6. **Validation**: IP adresi format kontrolü
7. **Rate Limiting**: API çağrıları için rate limiting

## Troubleshooting

### Yaygın Sorunlar

1. **AWX Token Expired**
   - Çözüm: Yeni token oluştur ve güncelle

2. **Device42 API Rate Limit**
   - Çözüm: API çağrıları arasına delay ekle

3. **Duplicate Hosts**
   - Çözüm: Status code 409 normal, host zaten mevcut

4. **Network Connectivity**
   - Çözüm: Firewall ve DNS ayarlarını kontrol et

### Debug Komutları
```bash
# AWX API test
curl -H "Authorization: Bearer YOUR_TOKEN" https://awx.company.com/api/v2/ping/

# Device42 API test
curl -u username:password https://device42.company.com/api/1.0/devices/
```

## İlgili Dosyalar
- `playbooks/flow_playbooks/dbag-devices-to-awx-inventory.yaml`: Ana playbook dosyası
- `playbooks/flow_playbooks/helpers/get_or_create_inventory.yml`: Envanter arama/oluşturma yardımcı görevleri
- AWX Template: AWX'de template konfigürasyonu
- Device42 API Documentation: Device42 API referansı
- AWX API Documentation: AWX API referansı 