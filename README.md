# 🛸 YTÜ Maçka GCS

**Yıldız Teknik Üniversitesi Maçka Aerospace** takımı tarafından geliştirilen, Python tabanlı yer kontrol istasyonu (Ground Control Station) yazılımı.

---

## 📸 Ekran Görüntüsü

> _Uygulama açıldığında sol panelde telemetri verileri, 300m²'den 1km²'ye kadar ölçeklenebilen bir map ve canlı kamera akışı, sağ panelde komut butonları görünür._

---

## ✨ Özellikler

| Özellik | Açıklama |
|---|---|
| 📡 SiK Telemetri İletişim | JSON tabanlı çift yönlü haberleşme, otomatik yeniden bağlanma |
| 🗺️ Canlı Harita | GPS iz takibi, waypoint ekleme/silme, 300–1000m akıllı zoom |
| 📷 Canlı Kamera | Raspberry Pi MJPEG stream (HTTP) |
| 🧭 IMU Göstergesi | Pitch / Roll görsel widget'ı (gerçek telemetri veya mouse simülasyonu) |
| 🎯 Task 1 — Figure-8 | 2 WP'den otomatik sekiz rakamı rota üretimi + harita önizleme |
| 🔍 Task 2 — Tarama | 2 WP'den zigzag alan tarama rotası, ayarlanabilir şerit aralığı |
| 🟢 Payload HUD | Yük sensörü durumu (yeşil/kırmızı gösterge) |
| 🔗 Bağlantı İzleme | Telemetri paket akışına göre otomatik BAĞLI / ZAYIF / KOPUK tespiti |
| 🚨 Acil Durdurma | Tek tuş motor kill komutu (DISARM) |

---

## 🗂️ Proje Yapısı

```
gcs_app/
├── main.py            # Uygulama koordinatörü (DroneApp)
├── config.py          # Tüm ayarlar (port, URL, mod isimleri vb.)
├── communication.py   # SiK serial haberleşme, ping, reconnect, health
├── mission_logic.py   # GPS matematiği, Figure-8 ve tarama WP üretimi
└── ui_components.py   # IMU, Payload, Kamera, Harita widget'ları
```

### Modüller

**`config.py`** — Bütün sabitler tek yerde. Port, baud rate, kamera URL'si ve uçuş modu etiketleri buradan ayarlanır.

**`communication.py`** — `SiKLink` sınıfı. Arka planda dört thread çalıştırır: alım döngüsü, ping, yeniden bağlanma ve bağlantı sağlığı göstergesi. UI'ya sadece callback aracılığıyla mesaj iletir.

**`mission_logic.py`** — Tkinter bağımlılığı olmayan saf Python modülü. `generate_task1_figure8_waypoints()` ve `generate_task2_scan_waypoints()` fonksiyonları bağımsız olarak test edilebilir.

**`ui_components.py`** — Dört ayrı `ctk.CTkFrame` subclass'ı: `IMUWidget`, `PayloadWidget`, `CameraWidget`, `MapWidget`. `MapWidget`, yükleme kararını `on_upload_request` callback'iyle dışarı taşır.

**`main.py`** — Tüm bileşenleri bağlar. Gelen telemetri mesajlarını ilgili UI etiketlerine yönlendirir, görev kararlarını verir ve SiK'e komut gönderir.

---

## 🚀 Kurulum

### Gereksinimler

- Python 3.10+
- Fiziksel veya sanal SiK radyo (COM port)
- Raspberry Pi kamera stream (opsiyonel)

### Kütüphaneler

```bash
pip install customtkinter pillow pyserial
```

## ⚙️ Yapılandırma

`config.py` dosyasını kendi donanımınıza göre düzenleyin:

```python
SIK_PORT  = "COM6"                        # Windows: "COM6" / Linux: "/dev/ttyUSB0"
SIK_BAUD  = 57600                         # SiK radyo baud rate
RPI_STREAM_URL = "http://192.168.1.232:5005"  # Raspberry Pi kamera adresi

MOUSE_IMU_ENABLED = False  # Gerçek donanımda False, test için True
DEBUG_SIK_RX      = False  # Ham paket loglaması
DEBUG_JSON_FAIL   = True   # JSON ayrıştırma hatalarını göster
```

---

## 🗺️ Harita Kullanımı

### Waypoint Ekleme
Drone GPS konumuna sahipken **`+ WP Ekle`** butonuna basın. Her basışta drone'un anlık olduğu konum ekstra olarak waypoint listesine eklenir.

### Task 1 — Figure-8
1. Harita modunu **`TASK1`** olarak ayarlayın (sağ üst toggle)
2. 2 waypoint ekleyin (dairelerin merkezleri)
3. **`📤 Pixhawk'a Gönder`** veya **`GÖREV 1`** butonuna basın
4. İki merkez arasındaki mesafenin yarısı yarıçap olarak hesaplanır, 18 noktalı sekiz rakamı rotası üretilir

### Task 2 — Alan Tarama
1. Harita modunu **`TASK2`** olarak ayarlayın
2. 2 waypoint ekleyin (tarama alanının köşeleri)
3. Şerit aralığını (metre) girin
4. **`📤 Pixhawk'a Gönder`** veya **`GÖREV 2`** butonuna basın
5. Dikdörtgen alan otomatik zigzag rotasına dönüştürülür, vision sistemi başlatılır

---

## 📡 Telemetri Protokolü

GCS, SiK radyo üzerinden JSON satırları alır ve gönderir. Beklenen mesaj tipleri:

| `type` | Açıklama |
|---|---|
| `battery` | `rem`, `voltage_v`, `current_a` |
| `mode` | `mode` (ArduPilot modu), `armed` |
| `alt` | `rel_m` (bağıl irtifa) |
| `speed` | `mps` (hız) |
| `gps` | `fix`, `fix_type`, `sats`, `lat`, `lon` |
| `att` | `pitch`, `roll`, `yaw` |
| `pos` | `lat`, `lon` |
| `payload` | `p1_raw`, `p2_raw` (PWM değerleri) |
| `timer` | `sec` (görev süresi) |
| `status` | `msg` — `wp_upload_ok`, `wp_clear_ok` vb. |

GCS'ten gönderilen komutlar:

```json
{"type": "cmd",      "name": "hold|rtl|land|kill"}
{"type": "wp_upload","waypoints": [...], "mission": "task1|task2"}
{"type": "wp_clear"}
{"type": "mission",  "name": "task1|task2"}
{"type": "ping",     "t": <timestamp>}
```

---

## 🛠️ Geliştirme Notları

- `mission_logic.py` UI'dan tamamen bağımsızdır; birim testleri doğrudan bu modüle yazılabilir.
- `SiKLink.send()` thread-safe'tir, herhangi bir thread'den çağrılabilir.
- `MapWidget`, upload kararını `main.py`'ye devreder — widget seviyesinde SiK bağımlılığı yoktur.
- Jitter filtresi: irtifada < 0.3 m, hızda < 0.2 m/s değişimler UI'ı güncellemez.

---

## 📁 Gerekli Dosyalar

```
gcs_app/
├── imu.png    # IMU widget arka plan görseli
└── logo.png   # Takım logosu (sağ panel)
```

Her iki dosya da eksikse uygulama yine çalışır; görseller yerine düz renk kullanılır.

---

## 🏫 Hakkında

**YTÜ Maçka Aerospace** — Yıldız Teknik Üniversitesi Maçka Mesleki Teknik ve Anadolu Lisesi insansız hava aracı takımı.

Bu yazılım Türkiye'deki insansız hava aracı yarışmaları için geliştirilmiştir.

---

<p align="center">
  <i>Made with ❤️ by YTÜ Maçka Aerospace</i>
</p>
