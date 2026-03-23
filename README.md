# 📊 Rakip Kampanya Takip & Analiz Sistemi

> TikTok Ad Library'den rakip banka reklamlarını otomatik olarak toplayan, AI ile analiz eden ve ekibe raporlayan n8n otomasyon projesi.

---

## 🎯 Proje Amacı

Türkiye bankacılık sektöründeki rakip kurumların (Garanti BBVA, Akbank, İş Bankası vb.) TikTok reklam stratejilerini günlük olarak takip etmek, AI destekli içgörüler üretmek ve pazarlama ekibine otomatik raporlar iletmek.

---

## 🏗️ Sistem Mimarisi

```
Cron Trigger (09:00)
        │
        ▼
TikTok Ad Library Scraper  ◄── Custom Python Web Scraper
(API yerine direkt web scraping — TR'de API erişimi kısıtlı)
        │
        ▼
   Switch Node
  ┌─────┴─────┐
  ▼           ▼
Video       Image
Reklamlar   Reklamlar
  │           │
  ▼           ▼
Gemini     GPT-4o
(Video     (Görsel
Analizi)   Analizi)
  │           │
  └─────┬─────┘
        ▼
      ChatGPT
  (Rakip Strateji Özeti
   + Alternatif Reklam
   Metni Üretimi)
        │
        ▼
   Excel Raporu
        │
        ▼
  Outlook ile
  Ekibe İletim
```

---

## ⚙️ Workflow Detayı

### Adım 1 — Tetikleyici
- Her sabah saat **09:00**'da cron job ile otomatik başlar.

### Adım 2 — Veri Toplama
- **TikTok Ad Library** üzerinden Türkiye'deki banka reklamları kazınır.
- TikTok'un Türkiye'deki API kısıtlamaları nedeniyle **custom Python web scraper** kullanılır (Selenium tabanlı).
- Bankacılık anahtar kelimeleri ile filtreleme: `banka`, `kredi`, `kart`, `finans`, `garanti`, `akbank`, `isbank`, `yapikredi`
- Her reklam için toplanan veriler: başlık, metin, marka bilgisi, medya URL'leri, format (video/görsel)

### Adım 3 — Medya Ayrıştırma
Switch node ile reklamlar iki kola ayrılır:
- **Video reklamlar** → Gemini pipeline'ına yönlendirilir
- **Görsel reklamlar** → GPT-4o pipeline'ına yönlendirilir

### Adım 4 — AI Analizi

#### Video Pipeline (Gemini + ChatGPT)
1. Gemini'ye direkt link verilemediği için önce bir **upload oturumu** açılır
2. Video dosyası yüklenir, Gemini'den bir kimlik alınır
3. Gemini video sahnelerini, görselleri ve yazıları **detaylıca betimler**
4. Bu betimleme + reklamın JSON verisi birleştirilerek **ChatGPT'ye** gönderilir
5. ChatGPT rakibin stratejisini, mesajını ve hedef kitlesini özetler → Alternatif reklam metni üretir

#### Görsel Pipeline (GPT-4o + ChatGPT)
1. **GPT-4o** görseli analiz eder, içeriği betimler
2. Betimleme + reklam verisi **ChatGPT'ye** aktarılır
3. ChatGPT rekabet analizi yapar → Alternatif reklam metni üretir

### Adım 5 — Raporlama
- Tüm analizler **Excel tablosu** olarak derlenir
- Rapor **Outlook** üzerinden pazarlama ekibine iletilir

---

## 🛠️ Teknoloji Stack

| Katman | Teknoloji |
|--------|-----------|
| Otomasyon | n8n |
| Web Scraping | Python, Selenium |
| Video Analizi | Google Gemini |
| Görsel & Metin Analizi | OpenAI GPT-4o, ChatGPT |
| Raporlama | Excel (Spreadsheet Node), Microsoft Outlook |
| Çalışma Ortamı | Yerel sunucu (self-hosted n8n) |

---

## 📁 Proje Yapısı

```
tiktok-ad-scraper/
├── src/
│   ├── scraper/
│   │   ├── tiktok_selenium_scraper.py   # Ana Selenium scraper
│   │   └── network_video_extractor.py   # Video URL çıkarma
│   ├── config/
│   │   └── settings.py                  # Ayarlar ve keyword listesi
│   └── models/
│       └── ad_model.py                  # Reklam veri modeli
├── n8n_tiktok_scraper.py                # n8n için wrapper script
├── main.py                              # Bağımsız test çalıştırıcı
└── README.md
```

---

## 🚀 Kurulum

### Gereksinimler
```bash
pip install selenium playwright python-dotenv
```

### Ortam Değişkenleri
```env
OPENAI_API_KEY=your_openai_key
GEMINI_API_KEY=your_gemini_key
N8N_WEBHOOK_URL=http://localhost:5678
```

### n8n Wrapper'ı Çalıştırma
```bash
# Test çalıştırma
python n8n_tiktok_scraper.py --keywords "garanti,akbank,yapikredi" --max-results 50

# n8n formatında output
python n8n_tiktok_scraper.py --keywords "banka,kredi,kart" --max-results 100 --output-format n8n
```

---

## 📤 Çıktı Formatı

Her reklam için üretilen veri yapısı:

```json
{
  "brand": "Garanti BBVA",
  "title": "Reklam başlığı",
  "text": "Reklam metni",
  "media_urls": ["https://..."],
  "n8n_meta": {
    "has_video": true,
    "has_image": false,
    "is_banking": true,
    "processing_priority": "high"
  },
  "ai_analysis": {
    "competitor_strategy": "...",
    "target_audience": "...",
    "suggested_ad_copy": "..."
  }
}
```

---

## 📊 Hesaplanan Metrikler (Proje 2)

Reklam platformu verilerinin birleştirildiği akışta aşağıdaki metrikler hesaplanır:

| Metrik | Formül | Açıklama |
|--------|--------|----------|
| CPC | `spend / clicks` | Tıklama başına maliyet |
| CPM | `(spend / impressions) × 1000` | 1000 gösterim başına maliyet |
| CVR | `installs / clicks` | Dönüşüm oranı |
| CPA | `spend / installs` | Edinim başına maliyet |

---

## ⚠️ Notlar

- TikTok API'si Türkiye'den erişilemediğinden scraper **Selenium ile web scraping** yapar.
- Gemini'ye video **direkt URL ile gönderilemez**, önce upload işlemi gerekir.
- Scraper `headless=True` modunda çalışacak şekilde n8n entegrasyonuna optimize edilmiştir.
- Bankacılık sektörüne özel keyword filtresi ile alakasız reklamlar ayıklanır.

---

## 👤 Geliştirici

**Ege Gürel**

