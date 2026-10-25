# Merce’den Qavsi ekosistemine kontrollü geçiş

**Durum:** Önerilen geçiş planı. Bu teslimatta aşağıdaki ürün dosyaları taşınmadı, API değiştirilmedi ve SDK’ya dokunulmadı.

## 1. Başlangıç durumu

İncelenen çalışma ağacı `feat/stylekit-sheen-effects` dalındadır. Merce kök projesi Qt 6.12, C++20 ve CMake 3.30 kullanır. Mevcut Glass/Material3 stilleri Merce Style/Theme/Foundation bağımlılıkları taşır. Bu, bugünkü tek proje için çalışabilir; bağımsız tasarım sistemleri hedefinin son mimarisi değildir.

`Merce/experiments/button-support` ve `Merce/experiments/stylekit-dynamic-variation` devam eden deneylerdir. Bunların worktree değişiklikleri bu mimari belgeleme işiyle commit edilmez veya topluca temizlenmez.

Yeni depoya gitmek, ilk günde bütün dosyaları yeniden adlandırmak değildir. En küçük güvenli dilim seçilir; mevcut tüketici aynı public API’yle çalışırken ayrılan modül ayrı tüketiciyle doğrulanır.

## 2. Dosya ve sorumluluk haritası

Aşağıdaki yollar mevcut **Merce Git köküne** göredir; mimari doküman deposuna göre göreli dosya link’i değildir.

| Mevcut alan / dosya | Hedef sahiplik | İşlem / dikkat |
|---|---|---|
| `Core/Utils/MerceResult.h` | Önce Merce | Sırf “common” göründüğü için Runtime’a taşınmaz; gerçek ortak ihtiyaç yoksa kalır |
| `Core/BuildInfo/*` | Paket bazlı metadata | Her paket kendi sürüm bilgisini verir; merkezde Merce’yi bilmek zorunlu olmaz |
| `Core/Resources/MerceModuleRegistry.*` | Önce Merce | Genel plugin framework’üne dönüştürülmez; QML/CMake metadata yeterliyse kullanılmaz |
| `Theme/*` | Merce | Colors, spacing, radius, motion, typography, size, profiles Merce’nin tasarım sistemidir |
| `tools/design-tokens/tokens/*` | İlgili kit | Merce token kaynakları korunur; Material3/Apple kaynakları ayrı sahiplik kazanır |
| `generated/*` | Üreten kit | Kaynak/generator sürümü kayıtlı olur; üretilmiş dosyalar elle temizlenmez |
| `Foundation/AppIcon.qml` | Şimdilik Merce | Ad ve compatibility korunur; Qt’nin doğrudan çizdiği temel ikon işi yeni Core gerekçesi değildir |
| `Foundation/IconRegistry.qml` | Şimdilik Merce; ortak aday DEFER | Public Qt provider yolu ve gerçek kalan routing ihtiyacı kanıtlanmadan taşınmaz |
| `Foundation/MaterialIcon.qml` | Material provider iç renderer’ı | Material olmayan font/ikon çizimi bu isim altında toplanmaz |
| `Foundation/FoundationFonts.qml` | İlgili kit/provider | UI fontu ile Material icon fontu ayrı yüklenir |
| `Foundation/IconResources.*`, `IconResources_mac.mm` | Önce Merce / Qt-first değerlendirme | Native temel SF render Qt’de mevcut; ileri configuration/catalog ayrı kanıt ister |
| `Foundation/fonts/MaterialSymbolsRounded/*` | Material provider | Font, codepoints, lisans ve provenance birlikte taşınır |
| `Icons/FontAwesome/*` | Mevcut Merce optional provider; extract DEFER | SVG ve paket compatibility’si korunur; yeni genel provider gerekçesi ayrıca sınanır |
| `Foundation/AppLabel.qml` | Merce | Merce typography/tracking wrapper’ı evrensel Runtime Label yapılmaz |
| `Foundation/Surface.qml` | Merce / ilgili kit | Görsel yüzey politikası ve token’lar kittedir |
| `Controls/MButton.qml`, `ButtonStyleContext.qml` | Merce compatibility alanı | Mevcut API ve Qt workaround korunur; ButtonSupport’a private bağımlılık taşınmaz |
| `Controls/MBadge.qml` | Merce | Tek SK.Label + delegate hedefi korunur; Runtime kontrol ailesi doğmaz |
| `Controls/LoadingIndicator.qml` | Merce veya kit indicator | Tip/görünüm kitin sorumluluğu; ortak iş servisi üretilmez |
| `Style/MerceStyle.qml` | Merce | Merce’nin asıl StyleKit eşlemesi olarak kalır |
| `Style/ButtonContent.qml` | Önce Merce | Merce Theme bağı ve sunum politikası kitte; ortak destek ancak bütün kabul kapıları geçerse |
| `Style/Badge*Variation.qml`, `BadgeDotBackground.qml` | Merce | Intent/size ve badge renderer’ı kitin tarifidir |
| `Style/ComboBoxDelegate.qml`, `ComboBoxPopupState.h` | Önce Merce | Hover/highlight davranışı Qt’deki açığı tamamlıyorsa dar public contract’ı ayrıca kanıtlanır |
| `Style/Toolbar*` | Kit/uygulama + public Qt Menu | Toolbar sunumu ile form ComboBox ayrılır; Qt açığı yokken ortak menu target’ı açılmaz |
| `Style/SurfaceBackground.qml` | Merce | Native mekanizma değil, kit yüzeyi |
| `Style/WindowIntegration.qml` | Kit sunum adaptörü + root bağlantısı | Platform gerçeği ve kitin malzeme talebi ayrılır; sayfalara style kontrolü yayılmaz |
| `Style/Material3/*` | Bağımsız material3 | MerceStyle kalıtımı kaldırılmadan “bağımsız kit” sayılmaz |
| `Style/Material3/internal/actions/*` | Material3 | Family theme/variation/delegate ayrımı korunur; source alias ve metadata test edilir |
| `Style/Glass/*` | Bağımsız apple | GlassPolicy, decoration ve native entegrasyon kit/projede; ortak bridge henüz DEFER |
| `Style/Effects/*` | Önce Merce | Sheen/effect tercihleri otomatik olarak Runtime ortak standardı olmaz |
| `Effects/MGlass.qml`, `MGloss.qml`, `MSheen.qml`, shader’lar | Önce Merce | Apple native glass yerine aynı şeymiş gibi sunulmaz; shader/fallback ayrı değerlendirilir |
| `Platform/MerceAccessibility.*` | Qt public hints + mevcut proje | Qt’de olmayan dar tercih adaptörü ancak ortaklık/unknown/lifecycle kanıtıyla çıkar |
| `Platform/MercePlatformCapabilities.*` | Önce doğrudan Qt window/GraphicsInfo | Yeni global façade yok; per-window screen/DPR çözümü |
| `Platform/MerceWindowMaterial*`, `MerceSurfaceMaterial*` | Mevcut proje / sonraki Apple kit entegrasyonu | Ortak Materials DEFER; native ownership ve cleanup korunur; API varlığı ortak semantik kanıtı değildir |
| `playground/PlaygroundWindowChrome*` | Uygulama veya opsiyonel chrome adaptörü | QWindowKit yalnız chrome işlemleri; tasarım/malzeme sahibi olmaz |
| `Notifications/*` | Önce Merce | QtToastify-backed davranış ve Merce sunumu kalır; source-only dağıtım gerçeği açık korunur |
| `playground/component/PlaygroundDesign.qml` | Gallery uygulama sunumu | Merce profili veya style ismi sayfalara dağılmaz |
| `playground/component/PlaygroundIcons.qml` | Gallery | Ortak örnek ikon adlarının kit/paket seçimi yalnız burada |
| `playground/view/*` | Gallery’ye kontrollü aktarım | İşlevsel örnek, etkileşim ve içerik korunur; stil ayrıştırma bilgi içeriğini değiştirmez |
| `cmake/InstallMerce.cmake` | Merce + her yeni paket | Aynı source/installed target sözleşmesi ayrı paketlerde tekrarlanır |
| `tests/source-consumer`, `tests/package-consumer` | Merce + her paket | Yeni pakette bağımsız consumer testi; eski testler geçiş boyunca kalır |

Bu harita yeniden kullanılabilir olanı **otomatik kabul etmez**. Örneğin hover yardımcı nesnesinin yalnız Merce popup geometri kuralını çözmesi, bütün kitler için ortak Runtime modülü yapılmasını gerektirmez.

## 3. Önce neyi sabit tutuyoruz?

- Mevcut `Merce.*` import’ları ve `Merce::*` CMake target’ları.
- `AppIcon` adı; paket adları ve documented missing davranışı.
- MButton API’si, eski tüketiciler ve Qt bug workaround’u.
- Merce token kaynak/generation zinciri ve DESIGN.md.
- Light/dark, OS azaltma tercihleri ve shadersız software fallback.
- Pencere input, native Qt/Metal root sahipliği ve cleanup.
- Mevcut kurulu SDK; hiçbir adım SDK yaması gerektirmez.

Public davranış farkı çıkarsa değişiklik ayrıca açıklanır; “refactor” adı altında gizlenmez.

### Yeni Runtime bağımlılığı için compatibility sınırı

Public QML URI ve CMake target korunması, build bağımlılıklarının hiç değişmediği anlamına gelmez. **Koşullu tercih:** ancak bir ortak mekanizma kabul edilip Merce gerçekten onu kullanırsa ilgili QavsiRuntime component’i açık dependency olur. Bugün yeni Runtime kabulü veya Merce’ye yeni dependency ekleme kararı yok. Böyle bir extract yapılırsa tek Merce clone’u her şeyi kendiliğinden indirmez; Runtime sessiz vendored kopya olarak gömülmez.

Mevcut yayımlanmış Merce sürümleri bu yüzden geriye dönük değiştirilmez. Çıkarma önce experimental/sonraki kırıcı sürüm hattında doğrulanır. Yeni dependency edinme gereksinimi release notunda kırıcı build/packaging değişikliği olarak açıklanır; façade yalnız public kullanım biçimini korur.

Kaynak workspace’inde parent Runtime’ı önce `add_subdirectory` ile sağlayabilir. Kurulu Merce config’i component bazlı `find_dependency` kullanır; dağıtım paketinin dependency metadata’sı gereken Runtime paketini bildirir. CMake kullanıcı adına internetten paket indirmez.

Bu tercih iki implementasyonu uzun süre bakımda tutmamak içindir. Mevcut tek-clone deneyimini birebir sürdürmek zorunlu ürün koşulu haline gelirse source bundle/dağıtım paketi ayrıca tasarlanır; mimari taslak bunu ücretsiz veya yapılmış saymaz.

## 4. Aşamalar ve çıkış ölçütleri

### Aşama 0 — mimari ve baseline

Bu belgenin kabulü, repo görünürlüğü, lisans ve ilk destek hedefleri netleşir. Mevcut Merce snapshot’ında build/test/UI sonuçları alınır; bilinen Qt variation hatası ayrı kaydedilir.

Kaynak otoritesi kararı kullanıcı tarafından kapandı: Google Material 3, Microsoft Fluent 2/Windows ve native Liquid Glass için Apple HIG/public AppKit. Yeni kitin görsel girdisi `design-systems/<kit>/DESIGN.md` sözleşmesidir; tarihsel katalog dosyası veya VoltAgent Apple web analizi native recipe yerine kopyalanmaz. Merce’nin mevcut DESIGN.md/token zinciri değişmez.

**Çıkış:** Çalıştığı bilinen commit/toolchain, mevcut kontrol/API listesi, ilgili ekran görüntüleri ve bilinen hatalar.

**Geri dönüş:** Ürün kodu değişmediğinden gerekmez. Taslak kararlar revize edilir.

### Aşama 1 — Qt-first sınır ve kalan ihtiyaç

Önce mevcut public Qt ile nelerin çözüldüğü kanıtlanır. [04 incelemesi](../reviews/04-shared-boundary.md) temel SF/font çizimi, QML accessibility hints ve engine image-provider ownership için bu ilk proof’u çalıştırdı.

- Temel ikon lookup/render, native menü ve public platform olguları doğrudan Qt’de kalır.
- Mevcut Merce AppIcon adı ve paketli ikon compatibility’si korunur; henüz başka projeye taşınmaz.
- Aynı anda çoklu koleksiyon, ikon başına eksen veya özel routing gerekiyorsa yalnız bu kalan fark için ayrı küçük örnek hazırlanır.
- Registry/factory ihtiyacı public Qt image-provider/plugin yolu karşılaştırılmadan kabul edilmez.
- Yeni Runtime component’i bütün admission kapılarını geçmediyse dosya, target veya repo açılmaz.

**Çıkış:** Her aday için USE EXISTING QT / KEEP KIT/APP / DEFER / QUALIFIES QAVSI kararı. Hiçbir yeni ortak modül çıkmaması geçerli sonuçtur; bağımsız Material3 dilimi Runtime beklemeden başlayabilir.

**Geri dönüş:** Ürün kodu taşınmadığından mevcut tüketici korunur. Sonradan kabul edilen bir extract ayrı source+installed consumer ve compatibility planı gerektirir.


### Aşama 2 — Material3 ilk bağımsız kit

Başlangıç family’si butonlar olur. Mevcut actions dosya ayrımı yeniden kullanılır; bağımsız Theme modeli Material3’ün kendi kaynaklarıyla tanımlanır. Merce recipe’leri içerik/format açısından tek tek değerlendirilir; isim değiştirip aynı bağı korumak yeterli değildir.

Önce klasik M3 / Expressive edisyonu ve referans platform/release sabitlenir. Mevcut recipe’nin kendi generated çıktısıyla eşleşmesi resmi tasarıma uyum kanıtı değildir. [01 incelemesinde](../reviews/01-material3.md) seçilen Compose Expressive snapshot’ına göre beş size içinde üç yükseklik ve dört icon ölçüsü farklı çıktı; bu sonuç bütün M3 edisyonları için evrensel ölçü hükmü değildir. Aynı sürümün geometri, icon, type role ve state verileri birlikte karşılaştırılmadan değerler otomatik kopyalanmaz.

- İlk minimal örnek Small normal butonla doğrudan Qt + Qavsi.Material3 kullanır; Runtime, loading veya özel ikon sağlayıcısı zorunlu değildir.
- Merce’nin source/install/import yolları environment’tan çıkarılır.
- Filled, outlined, tonal, elevated, text, icon/FAB gibi gerçekten uygulanan roller açık coverage tablosunda görünür.
- Ripple, focus, disabled, loading, checkable ve icon FILL davranışı test edilir.
- Native control’e açık font/padding/geometry override’ı korunur.
- `fonts.button`/native font alt kümesi ve line-height sapması belgelenir. Exact typography veya Material FILL için gerekiyorsa yalnız kit içi açık content adaptörü sınanır; background’dan caller font’una yazılmaz.
- Explicit sıfır spacing/padding/tracking ve canlı font binding’i için theme/hover/checked/A→B→A regression’ı geçer; önceki temel StyleKit proof’u bu testi kapsamıyor.
- Başlangıçta sabit variation listesi kullanılır; runtime mutation desteklendi diye yanlış beyan verilmez.

**Çıkış:** Önce MerceStyle kalıtımı ve Merce.Theme import’u olmayan Small buton; sabitlenmiş resmi recipe, public font sınırları, software ve source/installed consumer ile geçer. Diğer görünüm/size’lar ancak aynı kapıyla eklenir; bütün buton family’sinin tamamlandığı ilk örnekten çıkarılmaz.

**Geri dönüş:** Merce içindeki eski Material3 adaptörü kullanıcıları kırmadan kalır. Yeni kit experimental olarak yayımlanır; eski adaptör daha sonra deprecate edilebilir.

### Aşama 3 — ButtonSupport ortaklık kapısı; deney korunur

Mevcut deney API’si loading ve paketli ikon kapsamıyla kendi projesinde değerlendirilir. Binding restoration proof’u tek başına ortak kabul değildir. Public Qt’nin yetmediği aynı dar lifecycle ihtiyacı bağımsız kitlerde kanıtlanırsa extract düşünülebilir.

- Mevcut SK.Button instance’ı korunur.
- Loading true ile yaratılma dahil keyboard/mouse aktivasyonu test edilir.
- Native background ve public state çözümlemesi bozulmaz.
- Content helper, Merce Theme ve somut kitten bağımsızdır.
- İzin binding’i loading sırasında değişse de sonunda güncel değer geri gelir.
- Static indicator ve spinner-only sabit geometri reduced motion/software’da görünür kalır.

**Çıkış:** Aynı standart sözleşme, Qt boşluğu, bağımsız tüketim ve source/installed/fallback kapıları birlikte geçerse yalnız gerekli parça çıkar. Aksi halde deney/kitte kalır; mevcut MButton kullanıcıları zorunlu migrate edilmez.

**Geri dönüş:** Deney tekrar experimental kalır; MButton kaldırılmadığı için üretim tüketicisi etkilenmez.

### Aşama 4 — Platform ve Apple ayrımı

Önce public Qt’nin verdiği olgular doğrudan kullanılır. Qt’de olmayan native mekanizma kit/projede kalır; ortak target kendiliğinden açılmaz. Tek pencere owner’ı, Qt/Metal root yapısı ve restore davranışı korunur; bağımsız ortak bridge ayrıca kabul kapısı gerektirir.

Apple kit kendi Theme/policy’siyle navigation/header için malzeme talep eder. İçerik, dialog ve input yüzeyleri HIG’e uygun okunaklı malzemeleri seçer. Her kontrolü saydam yapmak başarı ölçütü değildir.

İlk kabul software opak yüzey + okunaklı dialog’dur. Sonra macOS 26/Cocoa/Metal üzerinde tek regular native yüzey, public winId/PlatformSurface lifecycle ve tek owner ile sınanır. Aynı pencerede ikinci owner’ın snapshot açması, boş glass contentView ile ayrı QML foreground’un okunaklılığı ve scale/radius uyumu ayrı regression kapılarıdır. İlk native yüzey geçmeden ikinci bölge veya window-backdrop kapsamı eklenmez. Bu turdaki compile-only probe bu kapıları geçirmiş değildir. [02 raporu](../reviews/02-apple.md)

**Çıkış:** Metal ve software çalıştırmaları, gerçek masaüstü, Reduce Motion/Transparency/Contrast ve native view cleanup doğrulanır. Window chrome ile material adaptörleri birbirine stil bilgisi taşımaz.

**Geri dönüş:** Native materyal kapatılır; kitin opak fallback’i eksiksiz çalışır. Eski pencere state’i eski snapshot ile başka owner’ın değişikliğini ezmeden geri alınır.

### Aşama 5 — Gallery ve kalan family’ler

Gallery ayrı consumer olur. Ortak örnek verisi ile kitin rol/config adaptörleri ayrılır. Kit seçimi root’tadır; sayfalarda Merce/Apple/Material3 if blokları yoktur.

Aynı anda karşılaştırma ayrı süreçlerde yapılır. Tek süreçte kit değişimi process-globaldir; diğer window’lar için ayrıca bağımsız stil beklentisi doğurmaz.

Sonraki family sırası: input/selection → popup/dialog/menu → navigation/surface → kalan değer/ilerleme/scroll kontrolleri. Her dilim aynı aile için mapping, variation, delegate, fallback ve acceptance örneğini birlikte tamamlar.

**Çıkış:** Her kitin kapsanan/fallback/eksik family listesi açık; kurulu paket consumer’ı ve karşılaştırma raporu var.

### Aşama 6 — compatibility temizliği

Eski namespace/target ancak yayımlanmış deprecation politikası ve consumer taraması sonrası kaldırılır. Aynı implementasyonun iki yerde uzun süre bakımı yapılmaz; mümkünse façade tek implementasyona yönlenir.

MButton’ın kaldırılması bu planın otomatik sonucu değildir. Şu koşullar ayrıca değerlendirilir:

- Native SK.Button + support bütün kullanılan davranışları karşılıyor.
- Qt’nin dinamik variation regression’ı desteklenen SDK’da geçiyor veya uygulamada bu gereksinim yok.
- Mevcut uygulamalar için açık migration örneği var.
- Public API değişikliği ayrıca onaylanmış.

**Çıkış:** Compatibility alanı küçülür; private Qt/StyleKit workaround yeni bağımsız kitlerin public sözleşmesine dönüşmez.

## 5. Qt 6.12 bug stratejisi

QTBUG-148461’e eklenen minimal örnek mevcut butonda role ekleme/kaldırma senaryosunu ayırır. Yeni instance başlangıç rolünün çalışması, mutation’ın düzeldiği anlamına gelmez.

SDK patch’lemek, her click’te kontrolü recreate etmek veya global theme reset’i tetiklemek migration varsayımı yapılmaz. Mevcut MButton workaround’u legacy alanda tutulur; yeni kitte sabit rollerle ilerlenir. Qt fix geldiğinde minimal test önce yeni patch üzerinde çalıştırılır, sonra kit matrisi genişletilir. [Qt raporuna eklenen yorum](https://qt-project.atlassian.net/browse/QTBUG-148461?focusedCommentId=2662762)

Qt’ye bağlı bir sınırlama tasarım sistemi API’sini gereksiz enum/extraVariations katmanıyla kalıcılaştırmak için gerekçe değildir.

## 6. Git ve repo taşıma

Mevcut Merce deposu bu çalışmada taşınmaz, force-push edilmez veya yeniden init edilmez. Organizasyona gerçek repo transfer’i yapılacaksa repository identity, issue/PR geçmişi, Pages URL, Actions secret/environment, release link ve consumer remote’ları ayrıca gözden geçirilir.

Yeni Runtime’a kod çıkarırken mevcut MIT copyright bildirimi korunur; başka referans projeden alınan kodun lisansı ayrıca incelenir. İlk iki bağımsız repo için kullanıcı MIT lisansını seçti; üçüncü taraf kaynakların özgün lisansları korunur.

Her implementasyon commit’i tek sorumluluk taşır: örneğin Core parser ayrımı, Material provider ayrımı, Merce facade bağlantısı veya installed consumer testi. Büyük “rename all + architecture rewrite + visual redesign” commit’i üretilmez.

Geçiş sonunda kullanıcı görünümü değişiyorsa önce çalışan UI sol ekranda gösterilir. Bu mimari belgeleme teslimatı, ürün commit’ine veya onaysız görsel değişime izin sayılmaz.

## 7. Başarı tanımı

Geçiş, klasör adları değiştiğinde tamamlanmış olmaz. Başarı:

1. Tek kit başka kit olmadan build/deploy ediliyor.
2. Runtime hiçbir kit Theme’ini bilmiyor.
3. Uygulama sayfaları stil bağımlılığı taşımıyor; root/adaptör bağı açık.
4. Native Qt davranışları ve caller binding’leri korunuyor.
5. Yazılım renderer’ında anlamlı UI var.
6. Kurulu consumer gerçek dağıtım çıktısıyla açılıyor.
7. Eski Merce tüketicisi ilan edilen compatibility süresinde çalışıyor.
8. Her iddia test sonucu, kaynak kanıtı veya açık taslak hedefi olarak ayrılmış.
