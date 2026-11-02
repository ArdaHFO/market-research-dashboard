Market Research Dashboard, şirket haberlerini, fiyat grafiklerini ve kayıtlı tahminleri kolay karşılaştırman için bir araya getirir.

<h1 align="center">Market Research Dashboard</h1>

<p align="center"><strong>Haberi oku. Grafiği incele. Kanıtları karşılaştır.</strong></p>
<p align="center">Finans haberlerini, teknik göstergeleri ve kayıtlı yön tahminlerini birleştiren Python tabanlı bir piyasa araştırma paneli.</p>

<p align="center">
  <a href="README.md">English</a> · <strong>Türkçe</strong><br />
  <a href="#yerel-çalıştırma"><strong>Yerel çalıştırma</strong></a> ·
  <a href="#panelin-içinde">Kullanım akışı</a> ·
  <a href="https://github.com/ArdaHFO/market-research-dashboard/issues">Sorun bildir</a>
</p>

---

Market Research Dashboard sembol aramasını, haber analizini, etkileşimli grafikleri, favori listelerini ve tahmin geçmişini tek bir FastAPI uygulamasında toplar. Haber ve teknik ağırlıklı görüşleri karşılaştırabilir, farklı zaman dilimlerini inceleyebilir ve tek bir skoru kararın tamamı olarak görmek yerine kayıtlı sonuçları değerlendirebilirsin.

**Bu, arayüzü Türkçe olan bir araştırma uygulamasıdır.** Analiz çıktıları üretir ve tahminleri kaydeder. Aracı kuruma veya borsaya emir ileten bir entegrasyon içermez. İngilizce ve Türkçe dokümantasyon sunulur.

## Yerel çalıştırma

| Gereksinim | Mevcut repo |
| --- | --- |
| Python | Sağlanan Dockerfile Python 3.12 kullanır |
| Bağımlılıklar | [`requirements.txt`](requirements.txt) dosyasını yalıtılmış bir ortama kur |
| Depolama | Varsayılan SQLite, `DATABASE_URL` tanımlanırsa PostgreSQL |
| Piyasa verisi | `yfinance` üzerinden Yahoo Finance verileri için internet |
| Haber analizi | Dil modeliyle duygu analizi için Groq anahtarı, ek haber kaynakları için isteğe bağlı sağlayıcı anahtarları |
| Tarayıcı | Grafik kütüphanelerinin CDN adreslerine erişebilen güncel bir tarayıcı |

Yeni bir sanal ortam oluştur. Repoda izlenen `.venv` ağacı taşınabilir bir geliştirme ortamı değildir.

```bash
git clone https://github.com/ArdaHFO/market-research-dashboard.git
cd market-research-dashboard
python -m venv .venv-local
```

PowerShell'de `.venv-local\Scripts\Activate.ps1`, POSIX kabukta `source .venv-local/bin/activate` ile etkinleştir. Ardından bağımlılıkları kur.

```bash
python -m pip install -r requirements.txt
```

[`.env.example`](.env.example) dosyasını `.env` olarak kopyala. Yerel HTTP oturumu için `COOKIE_SECURE=false` ayarla. Haber duygu analizini açmak için kendi `GROQ_API_KEY` değerini ekle veya teknik analiz yolunu incelemek için boş bırak. Özel bir yerel geliştirme oturumunda `REGISTRATION_CODE` boş olabilir.

```bash
python main.py serve
```

`http://127.0.0.1:8000` adresini aç, **Kaydol** ile yerel hesap oluştur ve **Giriş Yap** ile oturum aç. CLI sunucusu `0.0.0.0` adresine bağlandığı için bu geliştirme yapılandırmasını herkese açık hizmet olarak sunma.

Barındırılan kullanımda güvenli çerezleri açık tut, HTTPS sağla ve kayıt kodu belirle. Mevcut tarayıcı kayıt formu bu kodu göndermiyor. Korumalı kayıt için `POST /api/register` isteğinde `username`, `password` ve `invite_code` alanları kullanılmalı. Sonrasında mevcut tarayıcı giriş ekranı kullanılabilir. API şeması `/docs` adresindedir.

## Panelin içinde

- **Ara ve seç.** Sembol veya şirket adıyla arama yap. BIST, ABD ve Avrupa için hazırlanmış listelerden seçim yapıp kişisel favorilerine ekle.
- **Kanıtı oku.** Google News ve Yahoo Finance başlıklarını, isteğe bağlı Finnhub ve NewsAPI kaynaklarıyla birleştir. Groq yapılandırıldığında düzenli bir haber değerlendirmesi üretir.
- **Grafikleri incele.** Fiyat mumlarıyla birlikte hareketli ortalamalar, RSI, MACD, Bollinger Bantları, Supertrend, VWAP, pivot ve Fibonacci seviyelerini gör.
- **Profilleri karşılaştır.** Dengeli, haber ağırlıklı, teknik ağırlıklı, yalnız haber, yalnız teknik veya öğrenen model seçeneklerini kullan. Karşılaştırma işlemi üç karma profili yan yana çalıştırır.
- **Zaman dilimlerini araştır.** `30m`, `1h`, `1d`, `1wk` veya `1mo` seç ve birden fazla aralığı karşılaştır.
- **Tara ve izle.** Teknik tarayıcı, haber dil modelini çağırmadan önceden tanımlı listeleri sıralar. Geçmiş, sonuç özetleri ve CSV dışa aktarma sonraki incelemeyi destekler.
- **Kişisel ayarlarını sakla.** Hesapların kendi favori listeleri, ayarları ve kayıtlı analiz durumu vardır.

Temel kullanım akışı **Analiz** → sembol ve ayar seçimi → **Analiz Et** → sonuç kartını aç şeklindedir. Profil kıyaslaması için **Profilleri Karşılaştır**, aralık kıyaslaması için **Çoklu Zaman Dilimi**, tarama için **Tarayıcı**, geçmiş için **Panom** kullanılır.

## Yapılandırma

| Değişken | Amaç |
| --- | --- |
| `GROQ_API_KEY` / `GROQ_MODEL` | Haber analizi kimlik bilgisi ve model seçimi |
| `FINNHUB_API_KEY` / `NEWSAPI_KEY` | İsteğe bağlı haber sağlayıcılarını açar |
| `DATABASE_URL` | Sunucu veritabanı kullanılacaksa PostgreSQL bağlantı adresi |
| `FORECASTER_DB_PATH` | `DATABASE_URL` yoksa SQLite yolu, varsayılan `predictions.db` |
| `WATCHLIST_PATH` | CLI izleme listesi, varsayılan `watchlist.json` |
| `MODEL_PATH` | Öğrenen model JSON dosyası, varsayılan `model.json` |
| `REGISTRATION_CODE` | Kayıt API'sinin isteyebileceği kod |
| `COOKIE_SECURE` | Varsayılan güvenli çerez, yalnız yerel HTTP için `false` |
| `PORT` | Web sunucusu portu, varsayılan `8000` |

Haber aralığı, makale sınırı, birleştirme ağırlıkları ve sembol sınırları gibi ek ayarlar [`config.py`](src/forecaster/config.py) dosyasındadır. Yerel kimlik bilgilerini ve veritabanlarını commit'lere ekleme. Haber analizi etkinse seçilen haber içeriği Groq'a gönderilir.

## Komut satırı akışı

Komutları sanal ortam etkinken repo kökünden çalıştır.

| Komut | Etki |
| --- | --- |
| `python main.py serve` | Web arayüzünü ve API'yi başlatır |
| `python main.py run` | JSON izleme listesini bir kez analiz eder ve tahminleri kaydeder |
| `python main.py backfill` | Fiyatları getirip uygun bekleyen tahminleri sonuçlandırır |
| `python main.py report --days 7` | Veritabanındaki yakın dönem sonuçlarını okur |
| `python main.py train --universe us --timeframe 1d --horizon 60` | Geçmiş veriyi getirip aday modeli eğitir ve değerlendirir |

Analiz, tarama, sonuçlandırma ve eğitim dış servislere istek gönderebilir. `run` tek seferlik bir komuttur, kurulu bir zamanlayıcı değildir. Örnek izleme listesi AAPL, MSFT, NVDA, TSLA ve GOOGL içerir. Kendi araştırma listenle değiştirebilirsin.

## Modeller ve yorumlama

Standart profiller haber ve teknik skorları birleştirir. Haber kullanılamıyorsa karma profiller haber ağırlığını teknik analize aktarır. Kullanılabilir haber olmayan yalnız haber profili nötr kalır. Kullanılabilir model yoksa öğrenen profil dengeli birleştirmeye döner.

İsteğe bağlı öğrenen model bir lojistik regresyon uygulamasıdır. Eğitim, sembol başına kronolojik ayrım, bölümler arasında tahmin ufku kadar boşluk ve doğrulama verisine göre düzenlileştirme seçimi kullanır. Aday model, kodun trend taban çizgisi doğruluğu ve AUC koşullarını sağladığında kaydedilir. Tarihsel eğitim örneklerinde teknik ve piyasa özellikleri bulunur, haber özelliği sıfırdır.

Sağlanan [`model.json`](model.json), tarihsel eğitim özeti ve 60 mumluk tahmin ufku içerir. İçindeki değerler yeni bir doğrulama sonucu değildir. Uygulamanın sonuç kaydedicisi tahminleri sonraki fiyat mumuna göre değerlendirir. Bu nedenle panelin isabet oranlarıyla modelin 60 mumluk değerlendirmesi farklı ufukları anlatır ve eşdeğer sonuçlar olarak karşılaştırılmamalıdır. Yön skorları ve geçmiş doğruluk, gelecekteki getiriyi kanıtlamaz.

## Geliştirme ve doğrulama

FastAPI, JSON uç noktalarıyla birlikte [`webapp.py`](src/forecaster/webapp.py) içine yerleştirilmiş HTML/CSS/JavaScript arayüzünü sunar. Tarayıcı Chart.js ve Lightweight Charts kütüphanelerini CDN üzerinden yükler. Ayrı bir frontend derleme adımı yoktur.

Repodaki pytest dosyaları göstergeleri, skor birleştirmeyi, haber yanıtı ayrıştırmayı, tekrarları ayıklamayı, pipeline önbelleğini ve zaman dilimlerini, model araçlarını, grafik seviyelerini, depolamayı ve favori listesini kapsar.

```bash
python -m pytest -q
```

2 Kasım 2026 kaynak incelemesinde istek durumu yönetimi, model kullanılamadığındaki profil etiketi, çok dilli haberlerin tekrar kontrolü, veritabanı uyumluluğu, tahmin kaynaklarının kaydı, yalnız sonuçlanmış tahminlerden hesaplanan performans özetleri ve kimlik doğrulama/arayüz sorunları düzeltildi. İnceleme sırasında uygulama ve testler çalıştırılmadı; değişikliklerin çalışma zamanı doğrulaması hâlâ gerekli. Yukarıdaki mevcut test kaynakları bunun için sunuluyor. Bağımlılıklar sürüme sabitlenmediği için yeni kurulum özgün ortamdan farklı olabilir. Veri kapsamı, istek sınırları ve sağlayıcı erişimi sonuçları etkiler.

Konteyner kullanımı için sağlanan [`Dockerfile`](Dockerfile) aynı gereksinimleri kurar ve `python main.py serve` çalıştırır. SQLite için kalıcılığı ayrıca yapılandır veya PostgreSQL kullan. Konteyner tanımı tek başına doğrulanmış bir açık dağıtım anlamına gelmez.

## Repo haritası

| Yol | Amaç |
| --- | --- |
| [`main.py`](main.py) | CLI giriş noktaları |
| [`src/forecaster/webapp.py`](src/forecaster/webapp.py) | API, kimlik doğrulama ve gömülü panel |
| [`src/forecaster/pipeline.py`](src/forecaster/pipeline.py) | Haber/teknik analiz akışı ve kayıt |
| [`src/forecaster/news/`](src/forecaster/news/) | Haber toplama ve Groq analizi |
| [`src/forecaster/technical/`](src/forecaster/technical/) | Fiyat verisi, göstergeler ve skorlama |
| [`src/forecaster/fusion/`](src/forecaster/fusion/) | Profile göre skor birleştirme |
| [`src/forecaster/learning/`](src/forecaster/learning/) | Özellikler, lojistik model, eğitim ve değerlendirme |
| [`src/forecaster/storage/`](src/forecaster/storage/) | SQLite/PostgreSQL kayıtları ve sonuçlandırma |
| [`tests/`](tests/) | Mevcut otomatik kontroller |

## Katkıda bulunma

Sorun bildirimine ilgili sembolü, zaman dilimini, profili, beklenen davranışı ve hassas bilgilerden arındırılmış tekrar adımlarını ekle. Hesaplama değişikliklerinde küçük, deterministik bir örnek ve değerlendirme ufku belirt. Kimlik bilgilerini, kişisel kayıtları ve yerel ortamları katkılara ekleme. Doğrulama raporlarında çevrimdışı testleri, taklit sağlayıcı yanıtlarını ve canlı servis kontrollerini ayrı belirt.

## Lisans ve katkılar

Proje düzeyinde lisans dosyası bulunmuyor. Bu dokümantasyon yeni bir lisans atamaz. Python paketleri ve tarayıcı grafik kütüphaneleri kendi lisanslarına, veri sağlayıcıları kendi hizmet koşullarına tabidir. Mevcut uygulama arayüzünde TradeView adı kullanılır. Projenin bu sağlayıcıların resmî ürünü olduğu iddia edilmez.
