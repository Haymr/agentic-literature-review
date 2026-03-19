# Başlangıç Rehberi: Agentic AI Literatür Taraması

Hoş geldiniz! Bu başlangıç seviyesi rehber, çoklu ajan (multi-agent) destekli yapay zeka literatür taraması sistemini kendi bilgisayarınızda nasıl kurup çalıştıracağınızı adım adım gösterecektir. **Bu aracı kullanmak için hiçbir kodlama bilgisine ihtiyacınız yoktur.**

## 1. Ön Hazırlıklar

Başlamadan önce bilgisayarınızda **n8n** uygulamasının kurulu ve çalışıyor olduğundan emin olun. n8n'in "local files" (yerel dosyalar) klasörünü bildiğinizden emin olun (Örneğin Docker kullanıyorsanız bu klasör genelde `~/.n8n/local-files/` dizinine denk gelir ve n8n içinden `/home/node/.n8n-files/` olarak erişilir).

## 2. Ajanları (Workflow) İçe Aktarma

Sistemi çalıştıran 5 farklı akış dosyası bu deponun içindeki `workflows/` klasöründe bulunur. Bunları sırayla kendi n8n çalışma alanınıza eklemelisiniz:
1. n8n arayüzünü internet tarayıcınızdan açın.
2. Yeni bir Workflow (Akış) oluşturun, sağ üstteki üç noktaya tıklayın ve **Import from File** (Dosyadan Aktar) seçeneğini tıklayın.
3. Dosyaları sırayla içe aktarıp kaydedin:
   - `02-paper-extractor.json`
   - `03-thematic-analyzer.json`
   - `04-review-writer.json`
   - `05-fact-checker.json`
   - `01-main-supervisor.json`
4. **Önemli:** Akışların arka planda birbirini tetikleyebilmesi için hepsini kaydettikten sonra aktif hale (Active) getirmeyi unutmayın.

## 3. Makaleleri Hazırlama

Yapay zekanın okuyacağı makaleleri sisteme vermeliyiz.
1. İncelemek istediğiniz tüm PDF makalelerini bir araya getirin.
2. Bu PDF dosyalarını kopyalayıp yerel dosyalar dizininize (n8n tarafından `/home/node/.n8n-files/` olarak okunan klasöre) yapıştırın. Sürecin karışmaması için klasörün içinde sadece sizin araştırmak istediğiniz PDF'ler bulunduğundan emin olun.

## 4. Sistemi Çalıştırma (Başlamak)

1. n8n üzerinden yetkili ana ajan olan `01-main-supervisor` çalışma sayfasını açın.
2. Ekranın altındaki **Test Workflow** (Bağlantı Bekle) düğmesine tıklayarak sistemi canlı dinleme moduna alın.
3. n8n ekranındaki **Chat (Sohbet)** penceresini açın.
4. Sisteme `Başla` veya `Start` yazıp Enter tuşuna basın.
5. Arkanıza yaslanın! Supervisor Ajanının teker teker PDF çıkarma, analiz etme, yazma ve doğrulama araçlarını (tool) nasıl çalıştırdığını canlı canlı izleyin.

## 5. Çıktıları Alma

Sistem devasa metinleri hafızasında taşımak yerine kurumsal bir mantıkla direkt diskinize (Filesystem as State) yazdığı için yorucu kopyala-yapıştır işlemleriyle uğraşmazsınız.
1. Ajan sisteminin işi bittiğini Chat üzerinden onayladığında, n8n yerel klasörünüzü açın.
2. İçerisinde yapay zekanın kendi ürettiği çok sayıda yepyeni dosya göreceksiniz:
   - `temp_extracted.json`: PDF'lerden çekilen ham akademik veriler.
   - `temp_analysis.json`: Çekilen veriler üstünde kurulan tematik çıkarımlar.
   - `literature_review_output.txt`: Üretilen makalenin kanıt ve doğruluk (Fact Check) süzgecinden geçtiğini kanıtlayan rapor.
   - **`literature_review.html`**: En nihai, tasarımı giydirilmiş, tertemiz üretilmiş IEEE formatındaki makaleniz.

**PDF Nasıl Çıkartılır:** `literature_review.html` dosyasına çift tıklayıp herhangi bir tarayıcıda açın. Klavyeden CMD+P (veya Ctrl+P) tuşlarına basarak yazdır sayfasını açın ve **PDF olarak Kaydet** deyin. Kusursuz, yerel makaleniz saniyeler içinde masanızda!
