# TI C2000 (F280049C) Rehberi ve Örnek Kodlar

Texas Instruments C2000 serisi kartlara sıfırdan başlayacaklar için internette güzel Türkçe kaynaklar yoktu. Ben de yaklaşık 1 senelik öğrenimim boyunca öğrendiğim bilgileri, hem kendim kullanmak hem de başka kişilerin yararlanması için Türkçe bir rehberde topladım. Bu repoda rehberin kendisi ve rehberde anlatılan konuların çalışan kod örnekleri bulunmaktadır.

Rehber, hazır kütüphanelerin üzerine kod yazmak yerine donanımı **register seviyesinde** ele alır: her yapılan işlemin *neden* ve *nasıl* yapıldığını, işin mutfağında ne döndüğünü anlatmayı hedefler.

## Rehberde Neler Var?

Rehber, hiç ön bilgisi olmayan birini alıp register seviyesinde peripheral konfigüre edebilecek seviyeye getirecek şekilde, aşağıdaki sırayla ilerler:

1. **Temeller** — Mantık kapıları, flip-flop'lar, RAM ve bit/byte mantığı; bir elektronik cihazın "bilgi üretme" sürecinin en alt katmandan anlatımı.
2. **İşlemci mimarisi** — CPU'nun yapısı (ALU, kontrol ünitesi, register'lar), fetch-decode-execute döngüsü ve mikroişlemci (MPU) ile mikrodenetleyici (MCU) arasındaki fark.
3. **Register mantığı** — Register'ların ne olduğu, neden var oldukları ve C'deki struct/union yapılarıyla register adreslerine nasıl erişildiği. Rehberin belkemiği; sonraki her modül bunun üzerine kurulur.
4. **GPIO** — Pinleri giriş/çıkış olarak konfigüre etme (MUX, DIR, PUD register'ları) ve pin fonksiyon seçimi.
5. **Interrupt ve PIE modülü** — Interrupt'ların neden gerekli olduğu, PIE vektör tablosu, önceliklendirme ve IER/IFR/PIEIER/PIEIFR/PIEACK register'larının birlikte nasıl çalıştığı.
6. **Clock sistemi** — Osilatörler ve clock cycle üretimi, PLL ile SYSCLKOUT oluşturma, MCLK (missing-clock) güvenlik mekanizması.
7. **Timer** — CPU zamanlayıcılarının konfigürasyonu, prescaler (TDDR/PSC) mantığı ve periyot/interrupt ayarları.
8. **ePWM** — Modülün tüm alt birimleri: Time-Base, Counter-Compare, Action Qualifier, Dead-Band, Trip-Zone ve Event-Trigger. Taşıyıcı dalga (carrier waveform) mantığı, faz kaydırmalı senkronizasyon ve güç elektroniği kullanım senaryoları.
9. **ADC** — Analog-dijital çevrim, SOC (Start of Conversion) ve EOC (End of Conversion) alt modülleri, round-robin ve öncelikli çevrim yönetimi, ADC-PWM tetikleme zinciri.

Rehber boyunca konular kod örnekleriyle pekiştirilir ve `InitSysCtrl()` gibi TI'ın hazır fonksiyonlarının içine girilerek satır satır ne yaptıkları açıklanır.

## Öne Çıkan Kod Örnekleri

Örneklerin çoğu tek bir modülü izole ederek gösterir (GPIO toggle, tek/çift çekirdek ADC ölçümü, 100 Hz PWM vb.). Aşağıdakiler ise birden fazla modülü bir arada kullanan, gerçek güç elektroniği uygulamalarına daha yakın örneklerdir:

- **ADC eşiğine göre Trip-Zone ile PWM kapatma** — ADC ölçümü belirlenen eşiği (örn. ~3 V) aştığında ePWM'e one-shot trip sinyali gönderilerek çıkışların donanımsal olarak güvenli duruma alınması. Overcurrent/overvoltage korumasının tipik kurgusu; `TZFRC` ile tetikleme, `TZCLR` ile temizleme.

- **Komplemanter PWM + Dead-Band** — İki ePWM modülünün senkronize, birbirinin tümleyeni (complementary) sinyaller üretmesi ve half-bridge tarzı devrelerde kısa devreyi önlemek için dead-band eklenmesi.

- **PI kontrollü Boost Converter** — ADC ile çıkış gerilimini okuyup PI kontrolcü aracılığıyla PWM duty cycle'ını ayarlayarak istenen çıkış gerilimini sabit tutan kapalı çevrim örneği. Integral windup'ı önlemek için integral sınırlama (anti-windup) ve boost topolojisi için duty saturation (%20–80 arası güvenli bölge) içerir.

- **AC sinyal kopyalama (DAC ile düşük gecikmeli üretim)** — Bir sine look-up table ile şebeke frekansında dalga üretimi ve ADC ile okunan sinyalin DAC üzerinden düşük gecikmeyle yeniden üretilmesi. Inverter/full-bridge uygulamaları için sinyal üretimi ve senkronizasyon mantığını gösterir.

> Not: Gelişmiş örneklerin bir kısmı çalışma sürecinde farklı modlar denenerek geliştirilmiştir; yorum satırına alınmış deneysel bloklar içerebilirler.

## Kullanım

1. `examples` içindeki **`project_stradale`** klasörü, kodların çalışması için gerekli dosyaları ve konfigürasyonları barındırır. Bu klasörü CCS'deki Explorer (sol üst) sekmesine sürükleyerek projeye ekleyin.
2. `examples` klasöründeki `.txt` uzantılı kod örneklerinden kullanmak istediğinizi, `project_stradale` içindeki **`main_code.c`** dosyasının içine kopyalayıp çalıştırın.

## Donanım

- **Kart:** LAUNCHXL-F280049C (TI C2000 Real-Time MCU)
- **Geliştirme ortamı:** Code Composer Studio (CCS)

## Lisans

Bu projedeki kodlar MIT lisansı altında paylaşılmaktadır. Ayrıntılar için [LICENSE](LICENSE) dosyasına bakınız.
