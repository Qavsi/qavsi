# Yeni bir Qavsi kiti nasıl eklenir?

**Durum:** Önerilen geliştirici akışı. Örnek kit Fluent’tir; bu teslimatta Fluent reposu veya kodu oluşturulmadı.

## 1. Önce ürün sözleşmesi

Kabul edilen otoriteler Google Material 3, Microsoft Fluent 2/Windows ve Apple HIG/public AppKit’tir. Apple’ın hedefi native Liquid Glass’tır; web glassmorphism ile değiştirilmez. Başlangıç sözleşmeleri [Material3](../design-systems/material3/DESIGN.md), [Fluent](../design-systems/fluent/DESIGN.md) ve [Apple](../design-systems/apple/DESIGN.md) altındadır. Resmi belge kopyası değil, kitin Qavsi uyarlamasıdır. Snapshot/atıf ve eski inceleme girdileri [kaynak kaydında](../references/SOURCES.md) ayrı tutulur.

İlk iş CMake veya token kopyalamak değildir. Yeni kitin üretim DESIGN.md’si şu soruları cevaplar:

- Hangi tasarım kuralları ve referans sürümü uygulanıyor?
- Hangi control family’leri v1’de var? Hangileri Qt fallback’ine bırakılıyor?
- Typography, renk, yüzey, elevation/material, density ve motion rolleri neler?
- Light/dark/system ve platform accessibility tercihlerinin karşılığı ne?
- Hangi özellik hangi OS/renderer’da var; software’da anlamı nasıl korunuyor?
- Uygulamanın hangi override’ları destekleniyor?
- Native kontrol davranışından farklı gerçek bir interaction var mı?

Yeni kitin Merce ile aynı profil, spacing adı veya input yüksekliğini kullanması gerekmez. Ortak görsel token şeması çıkarılmaz. Uyarlanabilen native platform değerleri sabit JSON’a zorla çevrilmez.

Bir mekanizmayı Runtime’a almak ayrı bir [ortak yapı kabulü](SHARED-ADMISSION.md) gerektirir. Yeni kitin aynı fonksiyona ihtiyaç duyması tek başına yeterli değildir.

### 1.1 Kaynak sürümü ve sapma kaydı

Mevcut DESIGN.md’nin başlığı tek başına yeterli sürüm bilgisi değildir. Her kit aşağıdaki dar kaydı kendi üretim sözleşmesinde tutar; ortak token şeması veya yeni config framework’ü gerekmez:

| Alan | Kaydedilecek bilgi |
|---|---|
| Tasarım edisyonu | Örneğin klasik M3 veya seçilmiş Expressive sürümü; Fluent 2 Web veya Windows/WinUI 3 hedefi |
| Kaynak | Resmi URL, release/commit, indirilen dosyanın digest’i ve inceleme tarihi |
| Platform | Web CSS ölçüsü mü, Android/Compose token’ı mı, native macOS/Windows kuralı mı? |
| Family coverage | Hangi bileşen/size/state gerçekten uygulanıyor; hangisi fallback? |
| Sapma | Public Qt sınırı, erişilebilirlik veya ürün gereği; farkın görünen etkisi ve test kanıtı |
| Lisans | Kaynak kodu, referans dokümanı, font ve ikon için ayrı atıf/dağıtım hakkı |

Kaynak dosyalarındaki talimatlar uygulama/ajan talimatı sayılmaz. Snapshot aynen saklanır; üretim DESIGN.md’sinde yanlış veya platforma uymayan kural neden uygulanmadığıyla izlenir. Material3 incelemesi aynı isimli butonun farklı resmi edisyonlarda farklı geometri/type role taşıdığını gösterdi. Apple web özeti native HIG’in yerine geçmiyor; Fluent web ve Windows tarifleri de karıştırılmaz. [01](../reviews/01-material3.md), [02](../reviews/02-apple.md), [03](../reviews/03-fluent.md)

## 2. En küçük bağımsız paket

`fluent` ayrı clone/build/install edilebilir. Temel hedef `Qavsi.Fluent.Theme` ve `Qavsi.Fluent.Style` olabilir; nihai isimler ilk public sürümde sabitlenir.

`FluentStyle` doğrudan `Qt.labs.StyleKit.Style` temelindedir. `MerceStyle`, `Material3Style` veya `AppleStyle` kalıtımı yoktur. İlk dilim yalnız Qt + kendi Theme/Style’ı ile başlar; ancak kabul edilmiş ve gerçekten gereken optional Runtime component’i sonradan bağlanır.

Fluent için önce Qt’nin hazır `QtQuick.Controls.FluentWinUI3` stilinin ihtiyacı karşılayıp karşılamadığı değerlendirilir. Yalnız Windows görünümü isteniyorsa yeni kit gerekmeyebilir. Ayrı StyleKit kiti; public recipe/Theme extension ve aynı process’te root Style seçimi gibi somut farklarla gerekçelendirilir. Hazır FluentWinUI3, SK.Style’ın yeni kalıtım tabanı yapılmaz. [Qt FluentWinUI3](https://doc.qt.io/qt-6/qtquickcontrols-fluentwinui3.html)

İlk kanıt minimal bir pencere, bir normal buton, bir focus/disabled örneği ve software çalıştırmasıdır. Kullanılmayan family’ler için boş renderer/variation dosyası açılmaz.

## 3. Theme ve token zinciri

Kit kendi şemasını seçer. Tasarım girdisi ile çözülmüş render değeri ayrılır: renk seed’i veya sistem tercihi config olabilir; background/foreground/radius/motion sonucu kit Theme’indedir.

Gerekli zincir:

```text
Kitin kaynakları / OS girdileri
            ↓ doğrulama
Çözülmüş kit Theme / recipe
            ↓ mapping
StyleKit family/state/variation
            ↓ public delegate slot
Native kontrolün görsel çıktısı
```

Kit, geçersiz config durumunda son geçerli görünümü veya kendi gömülü default’unu korur. Runtime’ın palette’sine dönülmez. Generator gerekiyorsa girdiler, sürümü ve çıktı güncelleme komutu kitte belgelenir; normal consumer build’i generator çalıştırmaz.

Kitin UI fontu ile icon provider fontu bağımsızdır. AppLabel’ın Merce tracking değeri başka kitle paylaşılmış default olmaz.

Font family adını sistemde çözmek, o font dosyasını dağıtma hakkı değildir. İlk Fluent dilimi sistem fontunu kullanır; Segoe/Segoe UI Variable veya Segoe Fluent Icons binary’leri Windows’tan kopyalanmaz. Microsoft kodunun MIT lisansı bütün font/ikon assets’ine genişletilmez. Diğer platformda çözülmüş gerçek family/metrik/fallback kaydedilir. Apple’da da native sistem fontu/katalogu ile yeniden dağıtılan font paketi ayrı konulardır. [03 font incelemesi](../reviews/03-fluent.md)

## 4. Family içi dosya ayrımı

Örnek düzen:

```text
src/Qavsi/Fluent/Style/
├── FluentStyle.qml
└── internal/actions/
    ├── FluentButtonTheme.qml
    ├── variations/
    │   └── yalnız gerçekten farklı tarifler
    └── delegates/
        └── yalnız stock slot ile çizilemeyen görsel parçalar
```

- **Style:** Bileşimi ve family eşlemelerini bağlar.
- **Theme/state mapping:** Normal, hover, press, focus, checked, disabled değerlerini çözer.
- **Variation:** Kitin documented alternatif görünümünü seçer.
- **Delegate:** Kendisine verilen slot’u çizer.

Bu ayrım “her intent için yeni QML tipi zorunlu” demek değildir. Aynı mapping’i yalnız data girdisi değiştirerek ifade edebilen bir helper yeterli olabilir. Fakat bir QObject data payload’ını variation’larla property bazında merge ediyormuş gibi kullanmayız.

Basit font/color/radius için custom delegate yazılmaz. Ripple, floating label veya kitin özel materyali gerekiyorsa ilgili family içinde tutulur. Büyük if bloklarıyla tüm kitleri tanıyan ortak renderer üretilmez.

Dosya ayrımı public API’nin yapabildiğini genişletmez. TextStyle tam bir font/type-scale nesnesi, background delegate de genel content hook’u değildir. Exact line-height/FILL için açık kit content adaptörü gerekiyorsa bu sınır belgelenir; caller font bağlarını background’dan yeniden yazarak gizlenmez. İlk butonun native font alt kümesi ve ölçü farkları kabul edilmeden bütün family’lere aynı hatalı tarif yayılmaz. [Tipografi kontratı](CONTRACTS.md#26-geometri-ve-tipografi)

## 5. Kullanıcıya stilin verilmesi

Standart kontroller için kullanıcı root’ta stili atar. Paketli ikon veya loading gerekiyorsa mevcut Merce/deney arayüzünü açıkça seçer; bu henüz yeni ortak Qavsi dependency’si değildir. Özel kitin minimal consumer’ı başka tasarım sistemini import etmez.

Uygulama theme persistence, iş state’i, sayfa route’u ve kullanıcı yetkisini sahiplenir. Kit seçim yapıldı diye bunlar yeniden yaratılmaz.

Bir uygulama birden çok kiti seçtirecekse yalnız composition root/adaptör katmanı bütün kit import’larını bilir. Birden çok window açılması StyleKit’e ayrı stil scope’u kazandırmaz.

## 6. Gallery adaptörü: ortak Theme’in yerine geçmez

Gallery’nin kendi düzeni ve aynı örneği farklı kitlerde gösterme ihtiyacı vardır. Bu **uygulamaya ait** bir uyarlama sorunudur.

Önerilen Gallery girdisi aşağıdaki bilgileri birlikte tutar:

| Alan | Sahibi / anlam |
|---|---|
| Kit kimliği ve görünen ad | Gallery seçici verisi |
| App-owned Style instance | Root’un `StyleKit.style` ataması |
| Kitin config’i | İlgili kitin typed API’si; ortak Runtime Theme değil |
| Örnek variation/ad eşlemeleri | Gallery’ye ait sample role’ler; kitin gerçek public API’sine bağlanır |
| Örnek ikonları | Yalnız Gallery’de ortak isim → açık `paket:ikon` |
| Büyük uygulama yüzeyi sunumu | Gallery’ye ait dar presentation adaptörü |
| Optional native malzeme bağlantısı | Kitin talebini root’taki tek coordinator’a bağlayan adaptör |

Gallery’de `kits/merce`, `kits/material3`, `kits/apple` gibi private adaptörler olabilir. Sayfa dosyaları bunların somut tipini, `glassEnabled` veya `selectedStyle` karşılaştırmasını bilmez.

Bu, Runtime’a `QavsiTheme`, ortak spacing enum’u veya bütün kitlerin uygulamak zorunda olduğu dev bir kit interface’i eklemek değildir. Gallery sözleşmesi yalnız Gallery’nin kullandığı başlık, canvas ve navigation sunumuyla sınırlıdır. Başka uygulamanın farklı yerleşimi olabilir.

### 6.1 Yüzeyler nasıl etkilenir?

- `SK.Pane`, `SK.Label`, `SK.Button` gibi kontroller kitin normal mapping’inden etkilenir.
- Keyfi `Rectangle`, logo yerleşimi veya özel sidebar otomatik etkilenmez.
- Gallery önce standart StyleKit kontrolünü kullanır; custom sample role gerekiyorsa kit adaptörü o role eşler.
- Gerçekten uygulamaya özgü yüzeyde küçük Gallery presentation API’si kullanılır; token kaynağı kit adaptöründedir.
- Native pencere bağlantısı root’tadır; her sayfa veya badge yeni bir native materyal owner’ı açmaz.

Kitin Style nesnesinden `as MerceStyle` cast’i ile bütün sistemlere ulaşılmaya çalışılmaz. Legacy Merce WindowIntegration yalnız Merce’de kullanılabilir; bağımsız kitlerin ortak temeli olamaz.

### 6.2 Kit değişimi

Root seçili Gallery girdisini değiştirir; Style, sayfa sunumu ve native talepler aynı girdiden gelir. Kontrollerin/modelin sahibi root adaptörü değildir; iş/sayfa nesneleri yaşar.

Native request’ler eski adaptör yok olurken bırakılır; coordinator pencerenin sahibi olarak kalır. Başlangıçta material Pending/Unavailable ise opak kit yüzeyi çizilir. Her kit için farklı bir window oluşturup kontrol state’i kaybetmek genel çözüm yapılmaz.

Qt’de atomik kit transaction olmadığı için ara frame, açık popup ve focus senaryosu test edilir. Gerekirse root kit değişimini açık popup kapandıktan sonra yapar; bu politika Gallery’dedir. Ürün kütüphanesi uygulama adına popup’ları sessizce kapatmaz.

### 6.3 Sabit Gallery rolleri ve Qt variation hatası

Karşılaştırma sayfaları ömür boyunca sabit uygulama rolleri kullanır: örneğin `gallery.primary` veya `gallery.outline`. Her Gallery kit adaptörü bu isimlerin StyleVariation tariflerini **stilin kurulumu sırasında** kendi public kit Theme/extension API’siyle tanımlar. Bunlar Qavsi Runtime’ın veya bağımsız kitlerin zorunlu standart isimleri değildir.

Kit değişince kontrolün `StyleVariation.variations` listesi yeniden yazılmaz; yeni global Style aynı sabit Gallery adlarını kendi görsel tarifine çözer. Böylece bu geçiş, bilinen “mevcut instance’a variation ekle/kaldır” senaryosuna dönüştürülmez. Bu yol da aynı instance üzerinde test edilmeden desteklenmiş sayılmaz.

Qt’de genel bir “variation adına alias ekle” API’si varmış gibi davranılmaz. Gallery adaptörü public StyleVariation tanımlar; gerekli public kit tarif/Theme yüzeyi yoksa küçük bir kit extension noktası eklenir veya örnek unsupported kalır. Private renderer import’u veya kitin kendi registration isimlerini runtime’da değiştirmek çözüm değildir.

Kitin `filled`, `elevated` gibi kendi isimlerini karşılaştıran family-specific ekranlarda roller de sabittir. Başka kitte karşılığı olmayan örnek için o kit seçimi kapatılır veya ayrı process/senaryo açılır. Kontrolü sırf kit değişti diye yeniden yaratıp state kaybetmek genel kabul yolu değildir.

Bu ayrım iki testi gerektirir: sabit rol + Style değişimi ve aynı Style + rol mutation’ı. Birincinin geçmesi ikincinin Qt hatasını kapatmaz.

## 7. Kapsamı doğrula, sonra genişlet

İlk family şu kapıları geçer:

1. Merce ve diğer kitler kurulu olmayan environment.
2. Source ve installed CMake consumer.
3. Light/dark, focus/disabled/checked/loading.
4. OS motion/contrast/transparency tercihleri ve unknown bilgisi.
5. Software ve desteklenen GPU backend.
6. Native control davranışı, caller binding’i ve erişilebilirlik.
7. Varyasyon priority/data payload regression’ı.
8. Açık coverage/fallback listesi ve görsel karşılaştırma.

Ancak bundan sonra ikinci family eklenir. Bütün kontrol dosyaları önceden oluşturulup içleri Qt fallback’ine bırakılmışsa kit “tam kapsamlı” sayılmaz.

05 proof’u yalnız sabit rol ile temel public property/state ve Style A→B→A geçişini doğruladı. Gerçek kit content’leri, explicit font/spacing/padding sıfırları, açık popup/IME, native malzeme ve tek-frame atomik geçiş henüz geçmedi. Bu kapılar ürün yayını öncesinde ayrıca çalışır; rapordaki mekanik PASS tam kit desteği sayılmaz.

## 8. Dışarıdan özelleştirme

Üç seviye açık tutulur:

| İstek | Tüketici yolu |
|---|---|
| Marka/renk/font gibi desteklenen değişiklik | Kitin public Theme/config API’si |
| Desteklenen alternatif görünüm | Kitin public variation’ı |
| Yeni bir sistem / kapsamlı geometri veya interaction | Kendi bağımsız kit veya kitin belgelenmiş public extension noktası |

Bir uygulama public `MerceStyle` veya başka kitin Style tipinden türeyebilir; bu bilinçli olarak o kite bağımlıdır. Bu, bağımsız resmi Material3/Apple kitinin Merce’den türemesiyle aynı karar değildir.

Kit private renderer dosyasını göreli import etmek veya üretilmiş manifest’i elle düzenlemek desteklenen özelleştirme yolu olmaz. Kitin yeterli public extension noktası yoksa dar bir API talep edilir; Runtime’a bütün kitlerin private alanlarını açan bir escape hatch eklenmez.

## 9. Definition of Done

Yeni kitin ilk yayını şunları içerir: DESIGN.md, typed config/Theme belgesi, gerçek family coverage, public variation listesi/çakışma kuralları, source+installed örnek, ikon/asset lisansları, fallback matrisi, Qt patch uyumluluğu, bilinen sınırlar ve görsel/doğrulama kanıtları.

Yeni kit eklemek için Runtime’ın somut kitleri listeleyen kodunun veya başka kitin Theme şemasının değiştirilmesi gerekiyorsa sınır doğru kurulmamıştır.
