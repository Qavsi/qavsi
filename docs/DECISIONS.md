# Qavsi mimari karar kayıtları

**Durum:** ADR-001…008 incelemeye sunulan önerilerdir. Standart ortak yapı ilkesi ADR-009, resmi tasarım otoriteleri/native Apple hedefi ADR-010’da kullanıcı kararı olarak accepted kaydedilir. Yeni modül/API implementasyonları ayrıca kanıt ve plan gerektirir.

Bir ADR kabul edildiğinde tarih ve karar sahibi eklenir. Sonradan değişirse eski kayıt silinmez; onu supersede eden yeni kayıt bağlanır. Bir implementasyon pull request’i hangi ADR’yi uyguladığını ve çıkış testini belirtir.

| ID | Konu | Durum |
|---|---|---|
| ADR-001 | Bağımsız tasarım sistemleri ve token sahipliği | Proposed |
| ADR-002 | Küçük, seçilebilir Runtime | Proposed |
| ADR-003 | Qt davranışı / StyleKit / destek katmanı | Proposed |
| ADR-004 | Native materyal ownership ve fallback | Proposed |
| ADR-005 | Global stil ve Gallery süreçleri | Proposed |
| ADR-006 | Labs API ve Qt bug karantinası | Proposed |
| ADR-007 | Çoklu depo, paketleme ve compatibility | Proposed |
| ADR-008 | Config, güven sınırı ve reload | Proposed |
| ADR-009 | Standart ortak yapı kabulü; mevcut Qt işlevini tekrarlamama | Accepted — ilke |
| ADR-010 | Resmi Google/Microsoft kaynakları ve native Apple Liquid Glass | Accepted — kaynak/hedef |

## ADR-001 — Tasarım sistemleri eş düzey ve bağımsızdır

**Bağlam:** Material3, Apple, Merce ve Fluent aynı typography, motion, renk ve geometri modellerini paylaşmaz. Bugünkü Material3/Glass’ın MerceStyle üzerinden yürümesi Merce token bağımlılığını taşır. Kullanıcı Merce’yi başka kitler için zorunlu temel yapmak istemiyor.

**Alternatifler:**

1. Her kiti MerceStyle’dan türetmek: hızlı başlangıç; Merce semantiği ve varsayımları her yere sızar.
2. Evrensel Qavsi Theme/base style: tek adaptör yüzeyi; her kitin ayırt edici rollerini büyüyen ortak şemaya sığdırma zorunluluğu.
3. Bağımsız kit Theme + doğrudan Qt StyleKit Style: daha açık sahiplik; ortak olmayan kodun bir kısmı tekrarlanabilir.

**Önerilen karar:** Üçüncü seçenek. Her kit kendi token, Theme/config, state mapping, variation, delegate, motion ve fallback’ini sahiplenir. Qavsi Runtime renk/spacing/type-scale sözlüğü içermez.

**Sonuç:** Bir kitte yeni rol eklemek diğerlerinin şemasını büyütmez. Gallery, kitlerin farklı public config’lerini uygulama adaptöründe bilir. “Bir stil satırı keyfi bütün uygulama yüzeylerini dönüştürür” vaadi verilmez.

**Doğrulama:** Material3 minimal consumer’ı Merce kaynak/install/import yolu olmadan derlenir. Runtime kaynaklarında Merce/Material3/Apple Theme import’u bulunmaz.

**Yeniden değerlendirme koşulu:** Aynı anlam/lifecycle ve gerçek tüketim yanında standartlık, Qt boşluğu, fallback ve dağıtım kapıları da geçerse yalnız o küçük mekanizma çıkarılabilir. İki kit saymak tek başına yeterli veya yeni ortak Theme gerekçesi değildir.

## ADR-002 — Runtime bir motor değil, seçilebilir modüllerdir

**Bağlam:** İkon, loading sunumu ve platform yeteneği tasarım sisteminden bağımsız olabilir. Fakat “Foundation” adıyla hepsini birleştirmek font, native framework ve Theme bağımlılıklarını birlikte getiriyor.

**Alternatifler:** Tek Runtime target’ı; her küçük class’a ayrı repo; tek Runtime deposunda component bazlı target/QML modülleri.

**Önerilen karar:** Gerektiğinde son seçenek. Icons/provider, ButtonSupport ve Platform listesi adaydır; her parça ADR-009 kapılarını ayrıca geçer. Qt’nin sunduğu işi tekrar eden target açılmaz; bugün yeni bir Runtime component’i kabul edilmedi. Process manager, network/DBus servisleri, genel plugin fabrikası veya evrensel control family’si eklenmez.

**Sonuç:** Aynı Runtime sürümünde modüller birlikte test edilir; tüketim maliyeti ayrı kalır. CMake component export/discovery işi gerekir. “Import etmek kolay olsun” diye bütün modülleri otomatik yükleyen bir umbrella import başlangıçta yoktur.

**Doğrulama:** Bir aday kabul edilirse dar consumer’ı yalnız ilan edilen bağımlılıklarla açılır. Örneğin Icons Core gerçekten gerekirse fontsuz, StyleKit’siz, Merce’siz çalışır. Opsiyonel component kapalıyken fetch, metadata, asset veya link bağımlılığı gelmez. Bugün hiçbir yeni Runtime component’i bu kabulü almış değil.

**Yeniden değerlendirme koşulu:** Bir modülün ayrı bağımsız release ritmi ve gerçek dış tüketicisi oluşursa ayrı repo düşünülür; klasör sayısı büyüdü diye bölünmez.

## ADR-003 — Kontrol davranışı Qt’de, görünüm kitte kalır

**Bağlam:** SK.Button + ButtonSupport deneyi MButton wrapper’ına alternatif sunuyor. StyleKit delegate’lerinin slot kapsamı sınırlıdır; background delegate’i bütün kontrolü veya arbitrary contentItem’ı yönetmez.

**Alternatifler:**

- Yeni Qavsi.Button/ComboBox/Label ailesi: tek API; native Qt davranışını ve sürekli uyum işini tekrar sahiplenir.
- Yalnız StyleKit mapping: az kod; loading guard ve paketli içerik adaptörü için eksik kalır.
- Native Qt kontrol + kit mapping/delegate + gerektiği yerde dar attached support.

**Önerilen karar:** Son seçenek. ButtonSupport yalnız loading, sunum tercihi, enabled guard ve açık içerik adaptörünü sağlar. Native icon/text/display/checked/Action özellikleri korunur. Style/variation/delegate family bazında ayrılır. Genel size/variant/extraVariations API’si Runtime’a eklenmez.

Buradaki support, mevcut proje deneyidir; yeni ortak modül kabulü değildir. İlk bağımsız kit native font/icon alt kümesiyle support olmadan başlayabilir. Exact line-height/FILL için gereken açık content adaptörü kitte kanıtlanır; public StyleKit slot sınırı background’dan caller binding’ine yazarak aşılmaz.

V1 yalnız SK.Button ve ondan türeyen QML uygulama tiplerini destekler. Round/Tool/DelayButton, MenuItem ve MButton’a ayrıca attach etmek kapsam dışıdır.

**Sonuç:** Loading örneğinde contentItem bağlantısı açık olabilir; tek satırlık “bütün native kontroller destekli” büyüsü sunulmaz. Aynı Action’ın başka tetikleyicisini kilitlemek iş/uygulama sahibinin sorumluluğudur.

**Doğrulama:** Buton kimliği korunur; enabled binding loading sonrasında güncel sonucu verir; font/palette override, keyboard/focus ve accessible isim korunur. Spinner kendi background’unu çizmez.

**Yeniden değerlendirme koşulu:** Qt public API’si bu destek kapsamını doğrudan sunarsa helper küçültülür veya deprecate edilir. MButton’ın kaldırılması ayrıca consumer ve Qt regression kararı gerektirir.

## ADR-004 — Native materyalin pencere başına tek sahibi vardır

**Bağlam:** Pencere blur’u, navigation/header Liquid Glass ve QML decoration farklı roller. Birden çok bağımsız view/blur owner’ı silik, çift katmanlı veya okunaksız sonuç üretebilir. Pencere chrome’u da aynı sorun değildir.

**Alternatifler:** Her yüzey kendi native view’ını yönetir; kit doğrudan AppKit çağırır; pencere başına ortak mekanizma native kaynakları yönetir ve kit sadece talep/politika sağlar.

**Önerilen karar:** Son seçenek bir ownership invariant’ıdır; yeni ortak Materials target’ının kabulü değildir. Kit/projedeki dar native bridge pencere başına tek owner ile view/mask/observer cleanup yapar. Kit nerede hangi malzeme gerektiğini ve unavailable durumda hangi yüzeyin çizileceğini belirler. Ortak mekanizma ancak ADR-009 kapısını geçerse çıkar; o zaman kit id’sini sormaz.

**Sonuç:** Root’ta açık bir entegrasyon bağlantısı gerekir. Bu mekanizma ayrı compositor veya genel window manager’a dönüşmez. QWindowKit gerekirse chrome adaptöründe kalır; material/token/surface sahibi olmaz.

**Doğrulama:** Metal root/layer hiyerarşisi korunur, Qt input’u kesilmez, aynı pikselde çift native katman oluşmaz, screen/DPR değişimi ve destruction cleanup geçer. Reduce Transparency/Contrast ve software fallback okunaklıdır.

02’nin compile-only başarısı bu runtime kapılarını geçirmez. Native attachment, ayrı QML foreground’un vibrancy/okunaklılık başarısı değildir; ikinci owner/stale snapshot ve radius/clip ayrıca test edilir. Yeni bridge public winId/PlatformSurface yaşam döngüsünü kullanır; internal handle yolu yeni public sözleşmeye taşınmaz.

**Yeniden değerlendirme koşulu:** Qt desteklenen public API ile aynı ihtiyacı sağlarsa OS adaptörü yerine Qt yolu tercih edilir. Public olmayan native interface gerekiyorsa platform kapsamı daraltılır; sessiz private bağımlılık eklenmez.

## ADR-005 — StyleKit scope’u süreç geneldir; karşılaştırma ayrı süreçtedir

**Bağlam:** Qt 6.12.0’da QQStyleKitAttached static instance’ı QGuiApplication’a parent edilir. Birden çok QQmlEngine veya window bağımsız StyleKit stili sağlamaz.

**Alternatifler:** Her pencereye farklı Style atamak; engine başına farklı style; Qt’yi patch’lemek; Gallery’de kit başına süreç.

**Önerilen karar:** Tek süreçte tek aktif kit; eşzamanlı karşılaştırmada ayrı executable süreçleri. Gallery worker’ı basit argümanlarla senaryoyu açar. Özel RPC altyapısı veya engine manager yazılmaz.

Tek süreçteki karşılaştırma örnekleri sabit Gallery role adları kullanır; adaptörler bunları kendi public StyleVariation tarifleriyle başlangıçta tanımlar. Kit seçimi mevcut kontrolün variation listesini değiştirmez. Kit-specific rol karşılaştırması ayrı senaryodur; Qt mutation bug’ının çözüldüğü iddia edilmez.

**Sonuç:** Process başlangıç maliyeti vardır; fakat hem global StyleKit hem global Qt Quick Controls style/backend tercihleri birbirinden ayrılır. Tek uygulamadaki bütün pencereler ortak aktif stile uyum sağlar.

**Doğrulama:** Qt-only iki-engine örneği scope’u gösterir. Gallery karşılaştırma raporu worker sürümü/config’i kaydeder. Style değişiminde mevcut uygulama kontrolleri gereksiz yeniden oluşturulmaz.

**Yeniden değerlendirme koşulu:** Desteklenen Qt sürümünde public per-engine/per-window stil scope’u gelir ve regression ile kanıtlanırsa süreç ihtiyacı yeniden değerlendirilebilir.

## ADR-006 — Qt Labs riski ve mevcut bug’lar karantinada tutulur

**Bağlam:** Qt 6.12 StyleKit Labs API’si kalıcı kararlılık garantisi taşımaz. Mevcut butonda variation ekleme/kaldırma için raporlanan hata, yeni instance’ın başlangıç rolüyle aynı senaryo değildir. MButton’da geçici workaround vardır.

**Alternatifler:** SDK’yı patch’lemek; public API’yi workaround’a göre kalıcılaştırmak; her butonu yeniden yaratmak; mevcut compatibility alanını koruyup yeni kapsamı dar tutmak.

**Önerilen karar:** Son seçenek. Kurulu SDK değiştirilmez. MButton mevcut tüketiciler için kalır. Yeni SK.Button+support kapsamı sabit variation listesiyle ilan edilir. Qt-only minimal regression ürün testinden ayrı yaşar. Yeni public API private Qt tiplerini açmaz.

**Sonuç:** Bazı runtime variation senaryoları desteklenmemiş kalır. Bu sınırlama belgelenir; theme reset veya content-reader reload ile native background hatası gizlenmez.

**Doğrulama:** Aynı kontrol kimliğinde variation ekleme ve kaldırma ayrı; fresh instance pozitif kontrol ayrı çalışır. Beklenen Qt hatası “pass” diye raporlanmaz.

**Yeniden değerlendirme koşulu:** Qt düzeltmesi desteklenen patch’te minimal test ve kit contract test’lerini geçer. Ancak bundan sonra workaround kaldırma/migration işi açılır.

## ADR-007 — Bağımsız depolar, target bazlı tüketim ve açık compatibility matrisi

**Bağlam:** Kullanıcı her tasarım sistemini ayrı proje olarak geliştirmek istiyor. Monorepo atomik değişim kolaylığı sağlasa da bağımsız yayın ve sahiplik hedefini zayıflatabilir.

**Alternatifler:** Tek monorepo; her module ayrı repo; Runtime ortak repo + kit başına repo + Gallery consumer.

**Önerilen karar:** Son seçenek. GitHub organizasyonu qavsi; dokümantasyon merkezi qavsi/qavsi. Yerel üst workspace bir Git/submodule projesi değildir. Fluent çalışma başlamadan boş ürün reposu olarak açılmaz.

**Sonuç:** Repo arası commit’ler atomik olamaz. Runtime/kit sürümleri, en düşük/yayımlanmış dependency sürümleri ve integration consumer test’i gerekir. Source ve installed yolları aynı public target/URI verir; network fetch default değildir.

**Koşullu Merce geçiş bedeli:** Ancak kabul edilmiş ortak parça gerçekten kullanılırsa yeni dependency oluşur; bugün zorunlu Runtime dependency’si eklenmiyor. Böyle bir durumda façade eski import/target kullanımını koruyabilir, tek-clone edinme deneyimini koşulsuz korumaz. Değişim experimental/sonraki kırıcı sürüm hattında açıklanır; eski yayımlar değişmez ve gizli vendoring eklenmez.

**Doğrulama:** Temiz prefix’ten consumer, kaynak ağacı olmadan çalışır. Taşınmış prefix, static plugin ve kapalı optional dependency test edilir. Kit bağımsızlığı Merce kurulu olmayan ortamda kanıtlanır.

**Yeniden değerlendirme koşulu:** Sürüm koordinasyonu ölçülebilir biçimde geliştirmeyi engellerse depo sınırı değişebilir; bu değişiklik public paket sınırlarını zorunlu olarak birleştirmez.

## ADR-008 — Uygulama config’i sahiplenir; reload bir ürün runtime’ı değildir

**Bağlam:** Quickshell kendi shell executable’ı ve engine-generation protokolüne sahiptir. Qavsi bir kütüphane ekosistemidir. Tema dosyası, QML plugin’i ve development reload aynı güven/lifecycle sınırında değildir.

**Alternatifler:** Qavsi engine’i ve config launcher’ı; import anında otomatik global tercihler; uygulama-owned config + Qt geliştirme araçları.

**Önerilen karar:** Son seçenek. QGuiApplication/QQmlEngine uygulamanındır. Typed kit config veya veri manifest’i önce doğrulanır, sonra uygulanır. Qavsi kendiliğinden QSettings yazmaz veya uzaktan QML/plugin yüklemez. Debug QML Preview ayrı araçtır.

**Sonuç:** Uygulama startup ve persist işini açık bağlar; gizli import side effect’leri azalır. QML reload bütün state’i koruma garantisi vermez. C++/metadata/native değişimi rebuild/restart ister.

**Doğrulama:** Hatalı manifest son geçerli görünümü bozmaz. Eski generation’ın callback/view/observer’ı kalmaz. Release executable’da tooling debug yolu kapalıdır. Native object ve provider engine’ler arasında taşınmaz.

**Yeniden değerlendirme koşulu:** Gerçek bir shell/launcher ürünü istenirse ayrı uygulama projesidir; bu Runtime’ın sessizce genişlemesiyle çözülmez.

## ADR-009 — Ortak yapı standardı kanıtlanmadan Qavsi’ye taşınmaz

**Durum:** Accepted — kullanıcının koyduğu ilke. Belirli bir modülün extract edilmesi bu ilkeyle otomatik onaylanmaz.

**Kayıt tarihi:** 2026-10-24. **Karar sahibi:** Kullanıcı; “ortak yapıları standart bir yaklaşımsa qavsi içine almalıyız aksi takdirde proje içinde kalır” talebi.

**Bağlam:** Önceki aday katalog, ortak görünen her mekanizmanın baştan Runtime’da yayımlanacağı izlenimi verebilir. Kullanıcı yalnız standart ortak yapıları Qavsi’ye almak; tasarım sistemi/uygulamaya ait yapıları kendi projesinde tutmak istiyor.

**Karar:** Önce public Qt/OS’nin doğrudan sunduğu yol araştırılır. Yeni katman gerekiyorsa dar ek değeri, standart ve kitlerden bağımsız sözleşmesi, gerçek tüketimi, ownership/fallback’i ve paketleme sınırı kanıtlanır. Yalnız tekrar veya iki tüketici bulunması yeterli değildir. [Kabul sözleşmesi](SHARED-ADMISSION.md)

**Alternatifler:** Benzer bütün kodu merkeze taşımak başlangıçta tekrar azaltabilir, fakat ortak API’yi kit ayrıntılarıyla büyütür. Her şeyi yerel bırakmak daha az koordinasyon ister, fakat gerçekten aynı standart mekanizmanın gereksiz kopyalarını doğurabilir. Kabul kapısı bu ikisini somut kanıtla ayırır.

**Sonuç:** Runtime modül listeleri aday kalır; bazı target/URI’ler hiç yayımlanmayabilir. Mevcut AppIcon veya ButtonSupport deneyi yalnız adı ortak göründüğü için taşınmaz. Qt’nin hazır sistem sembolü, menü veya pencere işlevi yeniden adlandırılmış bir Qavsi wrapper’ına dönüşmez.

**Doğrulama:** Altı alanlı review, source/API karşılaştırması ile executable proof’u ayrı raporlar. Bir implementation testinin geçmesi, yeni modülün gerekli veya standart olduğunun kanıtı sayılmaz.

**Yeniden değerlendirme koşulu:** Eksik dar özellik için yeni gerçek tüketici ve public-standard kanıtı oluşur; admission kaydı güncellenir. Kit politikası veya Theme ortaklaştırma gerekçesi değildir.


## ADR-010 — Resmi kaynaklar esas, Apple hedefi native Liquid Glass

**Durum:** Accepted — kullanıcı kararı. **Tarih:** 2026-10-25. **Karar sahibi:** Kullanıcı.

**Bağlam:** İlk incelemelerdeki designmd.app dosyaları geçici girdiydi. Kullanıcının belirttiği VoltAgent deposunda Apple web analizi vardı; Material3/Fluent ayrı dosyası yoktu. Web marketing/glassmorphism analizi native malzeme veya kontrol kontratının yerine geçemez.

**Karar:** Material3 için resmi Google Material 3, Fluent için resmi Microsoft Fluent 2/Windows rehberleri esas alınır. Apple native Liquid Glass hedefi HIG ve public AppKit ile korunur. Her kit kendi DESIGN.md’sini, Theme/token modelini ve kaynak edisyonunu sahiplenir. [Material3](../design-systems/material3/DESIGN.md), [Fluent](../design-systems/fluent/DESIGN.md), [Apple](../design-systems/apple/DESIGN.md).

**Alternatifler:** Katalog dosyasını doğrudan üretim standardı saymak eksik/karışık platform varsayımlarını taşır. Bütün kitleri tek Qavsi tasarım sözlüğüne çevirmek bağımsızlık hedefini bozar. Resmi otorite + dar kit uyarlaması bu sorunları kaynak/sapma kaydıyla yönetir.

**Sonuç:** Kaynak seçimi için kullanıcı yanıtı beklenmez. VoltAgent web analizi yardımcı; eski snapshot’lar tarihsel kalır. Yeni DESIGN.md’ler resmi kuruluşların yayımladığı belgeler değil, Qavsi proje sözleşmeleridir. Runtime için ortak görsel Theme, yeni dependency veya ürün scaffold’u çıkmaz. Merce’nin mevcut DESIGN.md’si değişmez.

**Doğrulama:** Resmi URL/sürüm/hash kaydı, üç sözleşmeye manifest bağlantısı, tarihsel snapshot digest’leri ve mimari çapraz bağlantıları denetlenir. Yeni sözleşmenin görsel/native başarısı ancak sonraki implementation/renderer testleriyle doğrulanır; eski proof’lar bu amaçla yeniden etiketlenmez.

**Yeniden değerlendirme:** Seçilen resmi edisyon/API değişirse kitin kaynak/sapma kaydı güncellenir. Web ve Windows/M3 edisyonları sessizce karıştırılmaz; Apple native hedefi kullanıcı yönlendirmesi olmadan web tasarımıyla değiştirilmez.

## Bekleyen ürün kararları

İlk iki reponun görünürlüğü 2026-10-25 tarihinde kullanıcı tarafından **Public**
olarak seçildi. Ardından **MIT** lisansı ve ilk commitlerin push edilmesi
onaylandı. `Qavsi/qavsi` mimari belgelerini, `Qavsi/material3` bağımsız kiti
tutar. Bu onay gelecekteki Runtime/asset kapsamını kendiliğinden genişletmez.
[Aktarım kaydı](REPOSITORIES.md).

| Karar | Gerekli olduğu an | Bu taslakta varsayım |
|---|---|---|
| Üçüncü taraf asset/Runtime kapsamı | İlgili modülün ilk yayını | İlk iki repo MIT; dış kaynakların özgün lisansları korunur, yeni asset dağıtım hakkı varsayılmaz |
| İlk destek OS/cihazları | İlk stable destek beyanı | Qt 6.12.0 + macOS Metal/software ilk kanıt hattı |
| İlk kit Theme public API’si | Material3 dikey dilim | Kendi typed config’i; Merce şemasına çeviri zorunlu değil |
| Material3 family edisyonu ve coverage | Recipe implementasyonu | İlk Small referansı resmi MDC-Android 1.13.0; diğer family/Expressive kapsamı ayrı doğrulanır |
| Fluent hedef profili ve hazır Qt alternatifine farkı | Yeni Fluent implementasyonu | Kaynak otoritesi Microsoft olarak kabul edildi; ilk portable Button karşılaştırması nihai Web/Windows destek beyanı değil |
| Native menü kapsamı | Toolbar consumer spike | Önce public Qt Menu; custom NSMenu otomatik değil |
| Binary dağıtım kanalı | Paket sürümü | Önce source/installed CMake; paket yöneticisi taahhüdü yok |
| MButton’ın geleceği | SK.Button+support eşdeğerlik kanıtı | Korunur |

İlk beş teknik bölümde builder’ın çözebileceği adlandırma/file-level ayrıntılar taslakta önerildi; kullanıcı kararı gibi gösterilmedi. Bu kayıtlar kabul edildiğinde yeni ürün geliştirme planı küçük dikey dilimlere dönüştürülür.
