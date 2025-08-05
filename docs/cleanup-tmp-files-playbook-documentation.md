# Cleanup Temporary Files Playbook Documentation

## Genel Bakış

Bu playbook, webhook ile gelen alert'lere göre belirtilen dizindeki dosyaları temizlemek için tasarlanmıştır. AWX (Ansible Automation Platform) üzerinden çalıştırılır ve uzak sunuculardaki geçici dosyaları güvenli bir şekilde temizler. Ayrıca Zabbix entegrasyonu ile sistem kaynak analizi yapar ve raporlama sağlar.

## Playbook Dosyası
- **Dosya Adı**: `cleanup-tmp-files.yaml`
- **Konum**: `playbooks/flow_playbooks/`
- **Hosts**: localhost (delegate_to ile uzak sunuculara bağlanır)

## Çalışma Prensibi

### 1. Webhook Payload İşleme
Playbook, AWX'den gelen webhook payload'ını işler ve aşağıdaki değişkenleri alır:
- `event_name`: Olay adı
- `event_id`: Olay ID'si
- `host_name`: Hedef sunucu adı
- `host_ip`: Hedef sunucu IP adresi
- `target_directory`: Temizlenecek dizin yolu

### 2. Debug ve Loglama
Playbook çalışmaya başladığında, gelen tüm değişkenleri debug eder:

```yaml
- name: Tüm gelen değişkenleri debug et
  debug:
    msg: 
      - "=== WEBHOOK PAYLOAD DEBUG ==="
      - "Event Name: {{ event_name | default('TANIMSIZ') }}"
      - "Event ID: {{ event_id | default('TANIMSIZ') }}"
      - "Host Name: {{ host_name | default('TANIMSIZ') }}"
      - "Host IP: {{ host_ip | default('TANIMSIZ') }}"
      - "Target Directory: {{ target_directory | default('TANIMSIZ') }}"
      - "=== PAYLOAD SONU ==="
```

### 3. Sunucu Bağlantı Bilgileri
Playbook, hedef sunucuya bağlanmak için aşağıdaki bilgileri kullanır:
- **Kullanıcı Adı**: `duosis`
- **Şifre**: `Qwerasdf1234`
- **SSH Port**: 22 (varsayılan)
- **Host**: Webhook'tan gelen `host_name`
- **IP**: Webhook'tan gelen `host_ip`

### 4. Bağlantı Testi
Hedef sunucuya bağlantı kurulmadan önce ping testi yapılır:

```yaml
- name: Hedef sunucuya bağlantıyı test et
  ping:
  delegate_to: "{{ target_host }}"
  vars:
    ansible_host: "{{ server_ip }}"
    ansible_user: "{{ server_username }}"
    ansible_password: "{{ server_password }}"
    ansible_port: "{{ ssh_port }}"
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no -o ConnectTimeout=30'
```

### 5. Dosya Temizleme İşlemi
Bağlantı başarılı olduktan sonra:

#### 5.1 Dizin Kontrolü
Önce hedef dizinin var olup olmadığı kontrol edilir:

```yaml
- name: Hedef dizinin varlığını kontrol et
  stat:
    path: "{{ target_directory }}"
  register: dir_stat
```

#### 5.2 Dosya Silme
Dizin varsa, içindeki tüm dosya ve klasörler silinir:

```yaml
- name: Hedef dizinin içindeki tüm dosya ve dizinleri sil (dizini koru)
  command: rm -rf "{{ target_directory }}"/*
  when: dir_stat.stat.exists and dir_stat.stat.isdir

- name: Gizli dosyaları da sil (. ile başlayan dosyalar)
  command: rm -rf "{{ target_directory }}"/.[!.]*
  when: dir_stat.stat.exists and dir_stat.stat.isdir
  ignore_errors: yes
```

#### 5.3 Sonuç Kontrolü
Temizlik işleminden sonra dizin tekrar kontrol edilir:

```yaml
- name: Temizlik sonrası hedef dizini kontrol et
  command: ls -la "{{ target_directory }}"
  register: cleanup_result
  when: dir_stat.stat.exists and dir_stat.stat.isdir
```

### 6. Sistem Kaynak Analizi
Temizlik işleminden sonra hedef sunucuda sistem kaynak analizi yapılır:

```yaml
- name: Top 10 CPU kullanım process'lerini al
  shell: ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -11
  register: top_cpu_processes

- name: Top 10 Memory kullanım process'lerini al
  shell: ps -eo pid,ppid,cmd,%mem --sort=-%mem | head -11
  register: top_mem_processes
```

### 7. Zabbix Entegrasyonu
Sistem kaynak analizi sonuçları Zabbix'e gönderilir:

```yaml
- name: Zabbix API token al
  uri:
    url: "{{ zabbix_api_url }}"
    method: POST
    body_format: json
    body:
      jsonrpc: "2.0"
      method: "user.login"
      params:
        username: "{{ zabbix_user }}"
        password: "{{ zabbix_password }}"
      id: 1
    status_code: 200
  register: zabbix_auth_response

- name: Zabbix event'i acknowledge et ve rapor gönder
  uri:
    url: "{{ zabbix_api_url }}"
    method: POST
    body_format: json
    body:
      jsonrpc: "2.0"
      method: "event.acknowledge"
      params:
        eventids: ["{{ event_id | default('1') }}"]
        action: 4
        message: "{{ full_report }}"
      auth: "{{ zabbix_auth_response.json.result }}"
      id: 1
    status_code: 200
  register: zabbix_ack_response
  when: zabbix_auth_response.json.result is defined
```

## Değişkenler

### Zorunlu Değişkenler
- `host_name`: Hedef sunucu adı
- `host_ip`: Hedef sunucu IP adresi

### Opsiyonel Değişkenler
- `target_directory`: Temizlenecek dizin (varsayılan: `/test`)
- `ssh_port`: SSH port numarası (varsayılan: 22)
- `event_name`: Olay adı (debug için)
- `event_id`: Olay ID'si (Zabbix entegrasyonu için)

### Zabbix Değişkenleri
- `zabbix_api_url`: Zabbix API endpoint URL'i
- `zabbix_user`: Zabbix kullanıcı adı
- `zabbix_password`: Zabbix şifresi

## Güvenlik Özellikleri

### SSH Bağlantı Güvenliği
- `StrictHostKeyChecking=no`: Host key kontrolü devre dışı
- `ConnectTimeout=30`: 30 saniye bağlantı zaman aşımı
- Root kullanıcı ile çalışır (`become: yes`)

### Hata Yönetimi
- Dizin varlığı kontrol edilir
- Sadece dizin varsa silme işlemi yapılır
- Her adımda debug bilgileri loglanır
- `ignore_errors: yes` ile gizli dosya silme hataları tolere edilir

## Kullanım Senaryoları

### 1. AWX Webhook ile Otomatik Çalıştırma
```bash
# AWX'de webhook template'i oluşturulur
# Webhook payload'ında aşağıdaki değişkenler gönderilir:
{
  "host_name": "server01",
  "host_ip": "192.168.1.100",
  "target_directory": "/tmp/logs",
  "event_name": "log_cleanup",
  "event_id": "12345"
}
```

### 2. Manuel Çalıştırma
```bash
ansible-playbook cleanup-tmp-files.yaml \
  -e "host_name=server01" \
  -e "host_ip=192.168.1.100" \
  -e "target_directory=/tmp/logs"
```

### 3. Zabbix Alert ile Tetikleme
Zabbix'den gelen alert'ler AWX webhook'una yönlendirilir ve playbook otomatik çalışır.

## Çıktı Örnekleri

### Başarılı Çalışma
```
TASK [Tüm gelen değişkenleri debug et] 
ok: [localhost] => {
    "msg": [
        "=== WEBHOOK PAYLOAD DEBUG ===",
        "Event Name: log_cleanup",
        "Event ID: 12345",
        "Host Name: server01",
        "Host IP: 192.168.1.100",
        "Target Directory: /tmp/logs",
        "=== PAYLOAD SONU ==="
    ]
}

TASK [Temizlik sonucunu raporla]
ok: [server01] => {
    "msg": [
        "Temizlik tamamlandı!",
        "Hedef dizin: /tmp/logs",
        "Dizin içeriği: total 0"
    ]
}

TASK [Zabbix acknowledgment sonucunu raporla]
ok: [localhost] => {
    "msg": [
        "Zabbix event acknowledgment tamamlandı!",
        "Event ID: 12345",
        "Response: {'jsonrpc': '2.0', 'result': {'eventids': [12345]}, 'id': 1}"
    ]
}
```

### Hata Durumu
```
TASK [Hedef sunucuya bağlantıyı test et]
fatal: [localhost]: UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh",
    "unreachable": true
}
```

## Sistem Kaynak Analizi

### CPU Analizi
```bash
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -11
```

### Memory Analizi
```bash
ps -eo pid,ppid,cmd,%mem --sort=-%mem | head -11
```

### Rapor Formatı
```
Sistem kaynak kullanımı raporu:

=== TOP 10 CPU PROCESSES ===
PID  PPID CMD                         %CPU
1234 1    /usr/bin/python3           15.2
5678 1    /usr/sbin/nginx            8.7
...

=== TOP 10 MEMORY PROCESSES ===
PID  PPID CMD                         %MEM
1234 1    /usr/bin/python3           12.5
5678 1    /usr/sbin/nginx            6.8
...
```

## Zabbix Entegrasyonu

### API Endpoint
- **URL**: `http://213.153.197.248:8081/zabbix/api_jsonrpc.php`
- **Method**: POST
- **Authentication**: JSON-RPC with username/password

### Event Acknowledgment
- **Method**: `event.acknowledge`
- **Action**: 4 (add message)
- **Message**: Sistem kaynak analizi raporu

## Dikkat Edilmesi Gerekenler

1. **Güvenlik**: Root yetkisi ile çalışır, dikkatli kullanılmalıdır
2. **Yedekleme**: Silme işleminden önce önemli dosyalar yedeklenmelidir
3. **Test**: Production ortamında kullanmadan önce test edilmelidir
4. **Monitoring**: AWX'de job logları takip edilmelidir
5. **Zabbix Integration**: Zabbix API bilgileri güvenli şekilde saklanmalıdır

## Geliştirme Önerileri

1. **Soft Delete**: Dosyaları silmek yerine arşivleme seçeneği
2. **Age-based Cleanup**: Belirli yaştaki dosyaları silme
3. **Size-based Cleanup**: Belirli boyuttan büyük dosyaları silme
4. **Backup Integration**: Silme öncesi otomatik yedekleme
5. **Notification**: İşlem sonucu email/Slack bildirimi
6. **Disk Space Monitoring**: Disk kullanımı kontrolü
7. **Log Rotation**: Log dosyalarını rotate etme

## Troubleshooting

### Yaygın Sorunlar

1. **SSH Bağlantı Hatası**
   - Çözüm: SSH anahtarları ve kullanıcı bilgilerini kontrol et

2. **Permission Denied**
   - Çözüm: Root yetkisi ile çalıştığından emin ol

3. **Zabbix API Hatası**
   - Çözüm: API bilgilerini ve network bağlantısını kontrol et

4. **Dizin Bulunamadı**
   - Çözüm: Hedef dizin yolunu kontrol et

### Debug Komutları
```bash
# SSH bağlantı testi
ssh -o StrictHostKeyChecking=no -o ConnectTimeout=30 duosis@192.168.1.100

# Dizin içeriği kontrolü
ls -la /tmp/logs

# Sistem kaynak kontrolü
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -5
ps -eo pid,ppid,cmd,%mem --sort=-%mem | head -5
```

## İlgili Dosyalar
- `playbooks/flow_playbooks/cleanup-tmp-files.yaml`: Ana playbook dosyası
- AWX Webhook Template: Webhook entegrasyonu için gerekli
- SSH Key Management: Güvenli bağlantı için SSH key yönetimi
- Zabbix API Documentation: Zabbix entegrasyonu için referans 