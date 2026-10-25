# 04 — Ortak modül kabulü ve mimari sınır

**Karar: CONDITIONAL**\
**İnceleme tarihi:** 2026-10-23\
**Taban:** Qt 6.12.0; QtBase `025bdad181de241e81bf853c8a2d7bf3d19261a9`, Qt Declarative `4332fbbb4c340463dc8cee36dda0c85d932122d8`.

Bağımsız kit, ayrı Theme ve Qt kontrol davranışını koruma yaklaşımı uygundur. Ancak mevcut Runtime listesi kabul edilmiş ortak altyapı listesi olamaz. **Bugün yeni bir component için `QUALIFIES QAVSI` kararı verecek bütün kanıtlar yok.** Qt’nin zaten sunduğu işler doğrudan Qt’de; kalan deneyler gerçek ortak sözleşme kanıtlanana kadar mevcut kit/projede kalmalı. İlk bağımsız kitin Runtime bağımlılığı olmaması geçerli sonuçtur; DELIVERY.md:28 da buna izin verir.

Bu rapor ürün kodunu değiştirmez. Yeni Runtime, font/ikon dağıtımı veya native malzeme desteği onayı değildir.

## Kabul ölçütü

Bir mekanizmayı Qavsi’ye almak için aşağıdakilerin **hepsi** gerekir:

1. **Kaynağı belli anlam:** Yayımlanmış biçim/standart veya public Qt/OS API’si; gerekirse bunun üstünde açık, dar ve kabul edilmiş Qavsi sözleşmesi. Benzer sınıf adı, kod benzerliği veya üçüncü taraf estetik özeti yeterli değil.
2. **Gösterilmiş Qt boşluğu:** Aynı ihtiyacın public Qt ile karşılanamadığını gösteren küçük örnek. Qt’deki API’yi yeniden adlandırmak ortak mekanizma yaratmaz.
3. **Gerçek bağımsız tüketiciler:** Aynı anlam, girdi/çıktı, hata, sahiplik ve yaşam döngüsü. Aynı MerceStyle’dan türeyen üç stil veya iki Gallery örneği bağımsız kanıt değil. İki tüketici saymak tek başına yeterli değil.
4. **Kitten bağımsızlık:** Theme, renk/ölçü default’u, variation adı, seçili ikon politikası, iş/route politikası yok.
5. **Tam teknik sözleşme:** Engine/process/window kapsamı, thread, iptal/yıkım, unknown/unavailable, fallback ve optional asset/install davranışı kanıtlı. Paylaşımın dağıtım ve bakım yararı maliyetini karşılamalı.
6. **En küçük ortak parça:** Mevcut Qt mekanizmasından daha dar adaptasyon; genel motor, plugin fabrikası, ortak base style veya gelecekteki tüketici için boş target yok.

`QUALIFIES QAVSI`, bütün kapıların geçtiği anlamına gelir. `DEFER UNTIL PROVEN`, bugün dosya/target/repo açılması veya taşınması anlamına gelmez.

## Öncelikli bulgular

### P1 — R04-01: SF Symbols ve temel font ikonları için Qt boşluğu varsayımı yanlış

**Kanıt:** [CONTRACTS.md:195](../docs/CONTRACTS.md) Qt’nin tekrar yazılmamasını ister; fakat :197 native SF sembolünü standart Qt ad arayüzünün dışındaki gerekçeler arasında sayar. [ARCHITECTURE.md:154](../ARCHITECTURE.md) ayrı SFSymbols sağlayıcısını listeler. Oysa [QIcon public belgesi, Qt 6.12](https://doc.qt.io/qt-6/qicon.html#creating-an-icon-from-a-theme-or-icon-library), native kütüphaneleri 6.7’den, font adlarıyla ikonları 6.9’dan beri destekler. Sabit sürüm [qappleiconengine.mm:282–287](https://github.com/qt/qtbase/blob/025bdad181de241e81bf853c8a2d7bf3d19261a9/src/gui/platform/darwin/qappleiconengine.mm#L282) haritada olmayan adı doğrudan `NSImage` sembol sorgusuna geçirir.

**Gerçek deney:** Yalnız public Qt kullanan penceresiz program, Cocoa’da `QIcon::fromTheme("star.fill")` için 2x DPR’de 48 px çıktı ve **721 alfa piksel** üretti. Yerel Material Symbols Rounded uygulama fontuyla `save` ve `volume_off`, hem Cocoa hem offscreen üzerinde çizildi. [Yerel arşiv: Cocoa sonucu](evidence/README.md#review-04), [Yerel arşiv: offscreen sonucu](evidence/README.md#review-04).

**Etki:** Native sembolü bulma/PNG üretme, temel font glyph çözümleme ve bunların lifecycle/cache işini yeni bir Core’a taşımak gereksiz bakım ve dağıtım yüzeyi oluşturabilir. Bugünkü [Yerel arşiv: IconResources_mac.mm:14](evidence/README.md#local-source-audit) ayrı AppKit→PNG yolunun bulunması, yeni Qavsi API’sinin gerekli olduğunu kanıtlamaz.

**Düzeltme:** Önce Qt yolu. Mevcut `AppIcon` adı/uyumluluk façade’sı korunabilir. Aynı anda farklı koleksiyonlar, ikon başına `FILL/GRAD/opsz/wght`, explicit paket önceliği veya standalone QML çizimi gerekiyorsa yalnız bu **kalan fark** için ayrı örnek çıkarılmalı. Deney bu farkları çözmüş değildir. Global `QIcon::setThemeName()` her ikon çiziminde değiştirilmemeli.

### P1 — R04-02: “İki tüketici” kapısı modül listesini kabul etmeye yetmiyor

**Kapanış notu:** İnceleme sürerken koordinatör [SHARED-ADMISSION.md](../docs/SHARED-ADMISSION.md) ve ADR-009’u ekledi; modül tablolarını aday diye işaretledi. Genel kabul kapısı boşluğu giderildi. Aşağıdaki component kararları bu kapının somut uygulaması olarak kalıyor; henüz bir extraction kabulü yok.

**Kanıt:** [ARCHITECTURE.md:113](../ARCHITECTURE.md) aynı sözleşme ister; [risk tablosu:386](../ARCHITECTURE.md) ve [DECISIONS.md:35](../docs/DECISIONS.md) ağırlığı iki kite verir. İlk incelenen taslakta [Runtime tablosu:149](../ARCHITECTURE.md), [paket tablosu:13](../docs/DELIVERY.md) ve [MIGRATION.md:25](../docs/MIGRATION.md) bazı parçaların hedefini doğrudan Qavsi diye adlandırır. Belgelerin öneri etiketi yerinde; fakat kabul gerekçesi component bazında eksik.

Mevcut Material3 [Yerel arşiv: MerceMaterial3Style.qml:10](evidence/README.md#local-source-audit) ve Glass [Yerel arşiv: MerceGlassStyle.qml:6](evidence/README.md#local-source-audit), MerceStyle/Theme temelli. Bunlar bağımsız kitin aynı ortak mekanizmayı gerektirdiğini ispatlamaz.

**Düzeltme:** Hedef sahiplik satırlarına kabul durumu ve Qt boşluğu eklenmeli. Icons-first geçişi zorunlu kabul edilmemeli. İlk bağımsız kit Qt + kendi Theme/Style ile başlayabilir; boş Runtime dependency’si eklenmemeli. Bütün kapılar geçmeden dosya taşınmamalı.

### P2 — R04-03: ButtonSupport henüz ortak standart değil; mevcut deney olarak kalmalı

**Kanıt:** [CONTRACTS.md:102](../docs/CONTRACTS.md) loadingText, WithText/IndicatorOnly, enabled guard ve içerik sunumunu birlikte sözleşmeleştirir. Bunların tümünün aynı şekilde gerekli olduğu bağımsız tüketici yok. [Yerel arşiv: ButtonSupportContent.qml:24](evidence/README.md#local-source-audit), `minimumIconSize`/`iconFilled` metadata’sını tüketir; :99 Merce AppIcon’a bağlıdır. [Yerel arşiv: LoadingStyles.qml:3](evidence/README.md#local-source-audit) ve :24/:46/:68 aynı Merce Theme ölçeğini kullanır. Bunlar yararlı deneydir; kitten bağımsızlık kanıtı değildir.

Qt zaten [BusyIndicator](https://doc.qt.io/qt-6/qml-qtquick-controls-busyindicator.html) ve [Binding](https://doc.qt.io/qt-6/qml-qtqml-binding.html#when-prop) mekanizmalarını sağlar. App-owned tek `enabled` binding’i ile iş durumu ilişkisi kurulabilmesi de değerlendirilmelidir. Otomatik guard’ın shared Action/backend tekrarını önlemediği :142 doğru belgelenmiş.

**Düzeltme:** Mevcut deney/MButton korunmalı. Loading görseli ve genişlik politikası kitte, duplicate-start uygulamada kalmalı. Public Qt’nin eksik bıraktığı dar lifecycle gereksinimi iki gerçekten bağımsız kitte aynı sözleşmeyle kanıtlanırsa yalnız o parça yeniden incelenmeli. Yeni genel control/renderer/factory ailesi çıkarılmamalı.

### P2 — R04-04: Platform facts, Qt API’lerini yeniden paketlememeli

**Kanıt:** Qt 6.12 public [QAccessibilityHints](https://doc.qt.io/qt-6/qaccessibilityhints.html), contrast ve motion enum’larını sinyalleriyle sunar. İncelenen SDK [Yerel arşiv: qaccessibilityhints.h:19](evidence/README.md#local-source-audit) ile eşleşir. Deneyde `Application.styleHints.accessibility.motionPreference` ve `contrastPreference` QML’den wrappersız okundu; C++ değerleriyle eşleşti. Bu nedenle yalnız QML erişimi sağlamak için yeni singleton gerekmez.

[GraphicsInfo](https://doc.qt.io/qt-6/qml-qtquick-graphicsinfo.html), bağlı item’ın gerçek pencere backend’ini; [QWindow](https://doc.qt.io/qt-6/qwindow.html), pencere ekran/DPR ve sistem taşıma/resize işlemlerini zaten sağlar. Mevcut [Yerel arşiv: MercePlatformCapabilities.cpp:31](evidence/README.md#local-source-audit) primary-screen DPR’sini döndürür; bunu ortak API olarak taşımak per-window sözleşmeyi bozabilir. Mevcut [Yerel arşiv: MerceAccessibility.h:33](evidence/README.md#local-source-audit) bool default’ları ve [Yerel arşiv: macOS dışı no-op:26](evidence/README.md#local-source-audit), unknown bilgisini temsil etmez.

**Düzeltme:** Qt’nin mevcut olguları doğrudan kullanılmalı; enum unknown değeri bool `false`a indirgenmemeli. Qt’de bulunmayan reduced transparency için public [NSWorkspace API’si](https://developer.apple.com/documentation/appkit/nsworkspace/accessibilitydisplayshouldreducetransparency) dar bir adaydır. Önce Apple/Merce projesinde availability, değişim bildirimi, unknown ve observer temizliği kanıtlanmalı; bütün bir Platform façade’sı gerekçe değildir.

### P2 — R04-05: Public OS malzemesi, kitler arasında ortak malzeme semantiği kanıtlamaz

**Kanıt:** [ARCHITECTURE.md:157](../ARCHITECTURE.md) ve [MIGRATION.md:51](../docs/MIGRATION.md), Materials hedefini önerir. [CONTRACTS.md:251](../docs/CONTRACTS.md) alan adlarının spike sonrası kesinleşeceğini doğru söyler. Bugünkü [Yerel arşiv: MerceSurfaceMaterial_mac.mm:142](evidence/README.md#local-source-audit) özellikle `NSGlassEffectViewStyleRegular` seçer; [Yerel arşiv: WindowIntegration.qml:12](evidence/README.md#local-source-audit) MerceStyle cast’i ve Theme mode kullanır.

Apple’ın [public NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview) tipi macOS 26+ mekanizmasıdır. [Apple HIG materials](https://developer.apple.com/design/human-interface-guidelines/materials) kontrol/navigation katmanı ile içerik malzemesini ayırır. Microsoft’un [Mica](https://learn.microsoft.com/en-us/windows/apps/design/style/mica) sözleşmesi ise opak, duvar kâğıdı temelli uzun ömürlü pencere arka planıdır. Bu farklar tek “cam/material” enum’una eşitlenemez.

**Düzeltme:** Kullanılacak malzeme ve fallback renkleri kitte kalmalı. Mevcut native kod kendi projesinde kalmalı. Aynı OS primitive’inin aynı pencere/surface lifecycle’ı gerçek bağımsız tüketicilerde gerekirse dar, platforma özgü bridge adayı incelenmeli. Per-window tek owner iyi invariant’tır; tek başına ortak coordinator veya Windows/Apple effect fabrikası çıkarmayı haklı kılmaz.

## Component bazında kabul tablosu

| Alan | Bugünkü sınıflandırma | Gerekçe ve en küçük sahiplik/fallback sınırı |
|---|---|---|
| İkon dosyası, standart theme lookup, native temel ikon | **USE EXISTING QT (no wrapper)** | `QIcon`, Controls `icon.name/source`, Qt resource sistemi. [Qt Controls icons](https://doc.qt.io/qt-6/qtquickcontrols-icons.html). Boyut/renk kit veya caller; Qt’nin mevcut fallback sırası yeniden yazılmaz. |
| Icons Core: yeni `AppIcon`, `package:icon` parser/registry | **DEFER UNTIL PROVEN** | `package:icon` bir Qavsi/Merce protokolü; [Freedesktop 0.13](https://specifications.freedesktop.org/icon-theme/latest/) ile eşdeğer değil. Birleşik standalone QML çizimi/adlandırma yararlı olabilir; fakat mevcut Qt + küçük app/provider adaptöründen neden fazlası gerektiği ve bağımsız consumer/install kanıtı yok. Merce façade’sı Merce’de kalır. |
| Provider kaydı ve engine sahipliği | **USE EXISTING QT (no wrapper)** | Public [QQmlEngine::addImageProvider](https://doc.qt.io/qt-6/qqmlengine.html#addImageProvider), [QQuickImageProvider](https://doc.qt.io/qt-6/qquickimageprovider.html) ve QML plugin initialization. İki-engine deneyinde registry izolasyonu ve engine yıkımında cleanup geçti. Yeni registry/factory yalnız farklı ve gösterilmiş ihtiyaç varsa. |
| Material provider: temel named glyph | **USE EXISTING QT (no wrapper)** | Uygulama fontu + `QIcon` yolu gerçekten çizildi. Yalnız asset paketlemek yeni renderer gerektirmez. Font zorunluluğu Core’a taşınmaz. |
| Material provider: ikon başına eksen/metadata | **DEFER UNTIL PROVEN** | [Google axes sözleşmesi](https://developers.google.com/fonts/docs/material_symbols) ve Qt [Text.font.variableAxes](https://doc.qt.io/qt-6/qml-qtquick-text.html#font.variableAxes-prop) mevcut. Aynı anda çoklu family/instance ekseni Qt global theme’den farklı ihtiyaçtır. Eksen değerlerinin kontrol state’ine eşlenmesi **KEEP KIT/APP**; genel provider kabulü ayrıca kanıt ister. |
| Font Awesome provider/paket | **DEFER UNTIL PROVEN** | Mevcut [Yerel arşiv: FontAwesomeIcon.qml:35](evidence/README.md#local-source-audit) public Shape/PathSvg kullanır; bunu korumak mümkün. Yeni QtSvg bağımlılığı veya yeni icon engine otomatik gerekmez. Koleksiyon/renk default’u ve gerçek bağımsız consumer/install gerekçesi ayrıştırılmalı. Mevcut optional paket Merce’de kalır. |
| SFSymbols provider: temel lookup/render | **USE EXISTING QT (no wrapper)** | Ham `star.fill` adı, native Qt icon engine ve missing durumu doğrulandı. Yeni AppKit→PNG üreticisi gerekçesiz. SF’ye özgü advanced configuration/catalog API’si ayrıca **DEFER UNTIL PROVEN**; katalog UI’si Gallery’de. |
| ButtonSupport: birleşik loading/content API | **DEFER UNTIL PROVEN** | Mevcut Merce deneyi korunur. BusyIndicator/Binding zaten Qt’de; WithText/IndicatorOnly, genişlik rezervi, `iconFilled` payload’ı ortak standart sayılmaz. İş politikası uygulamada, indicator görünümü kitte. |
| Platform: motion/contrast, QPA, renderer, screen/DPR, move/resize | **USE EXISTING QT (no wrapper)** | Public Qt olguları doğrudan kullanılır; renderer/window scope’u global primary-screen façade’sına çevrilmez. İşlem desteği Qt’nin return value/unknown bilgisiyle izlenir. |
| Platform: Qt’de olmayan reduced transparency | **DEFER UNTIL PROVEN** | AppKit public fact, dar ortak aday. Bugün bağımsız tüketici ve kurulu paket/unknown/observer lifecycle kanıtı yok. Kit/projede kalır; desteklenmeyen OS için `false` varsayılmaz. |
| Platform.Materials: native bridge | **DEFER UNTIL PROVEN** | OS’ye özgü dar handle/view lifecycle adayı; mevcut Qt’nin karşılamadığı ihtiyaç, bağımsız consumer ve cleanup/install kanıtı gerekli. Kitin görünür renk/fallback ve kullanım politikası **KEEP KIT/APP**. |
| Platform.Menus: toolbar/native menü köprüsü | **USE EXISTING QT (no wrapper)** | Qt 6.8+ `Menu.popupType` native/window/item yollarını sunar. [Qt 6.12 Menu](https://doc.qt.io/qt-6/qml-qtquick-controls-menu.html#menu-types). Action/model ve fallback aynı Qt tipinde tutulabilir. Native delegate çizilmez; bazı platformlarda açma çağrısı bloklar. Gösterilmiş Qt açığı olmadan NSMenu adaptörü veya Runtime target’ı yok. |
| Theme, ortak base style, renk/spacing/type-scale, profil | **KEEP KIT/APP** | Kitin görsel semantiği. Material3/Apple/Fluent referanslarının benzer tablo biçimi ortak değer veya lifecycle yaratmaz. Qt StyleKit Style doğrudan kullanılır. |
| Effects: sheen, ripple, motion modeli, shadow/material tarifi | **KEEP KIT/APP** | Qt public çizim primitive’leri kullanılır; görünüm/duration/easing/fallback kitin kararıdır. Saf algoritma ileride bütün kabul kapılarını geçerse ayrıca incelenir. Genel Effects modülü açılmaz. |
| Notifications, toast/dialog orkestrasyonu | **KEEP KIT/APP** | Mevcut QtToastify ve Merce presentation/source-only dağıtımı korunur. Queue, modal karar, timeout, iş callback’i uygulama sözleşmesidir. Public Qt popup/dialog mümkün olan yerde kullanılır; ortak servis standardı kanıtlanmadı. |
| Navigation/route, sidebar yerleşimi, responsive karar | **KEEP KIT/APP** | Route/veri/yetki uygulamada; item görünümü ve özgün interaction kitte. Qt StackView/Drawer/selection primitive’leri kullanılır. Gallery adaptörü library standardı olmaz. |
| Token değişim biçimi/doğrulama aracı | **DEFER UNTIL PROVEN** | [DTCG 2025.10](https://www.designtokens.org/tr/2025.10/format/) veri değişim biçimidir; W3C Standard değildir ve ortak Theme sözlüğü tanımlamaz. Gerçek ortak araç gerekirse build-time kapsamında; Runtime bağımlılığı değil. |
| Genel plugin/module registry, engine veya reload motoru | **USE EXISTING QT (no wrapper)** | Qt QML/CMake metadata ve public Qt tooling yeterli başlangıçtır. Qavsi’ye yeni lifecycle motoru eklenmez. |

**`QUALIFIES QAVSI`: Şu an boş.** Bu, sonsuza kadar ortak modül olmayacağı anlamına gelmez; kabulün dosya taşıma işleminden önce gelmesi gerektiğini gösterir.

## İkon genişletme sınırı

Qt’de iki farklı genişletme noktası zaten vardır:

- Public [QIconEnginePlugin](https://doc.qt.io/qt-6/qiconengineplugin.html), ikon motorunu dosya/resource suffix’i üzerinden genişletir. Header bu Qt-only programda derlendi. Sabit kaynak [qiconloader.cpp:659](https://github.com/qt/qtbase/blob/025bdad181de241e81bf853c8a2d7bf3d19261a9/src/gui/image/qiconloader.cpp#L659) theme-plugin→font→theme-file→platform yollarını gösterir. **Theme-key seçimi kaynak gözlemidir; undocumented ayrıntı public Qavsi contract’ı yapılmamalı.**
- `QQuickImageProvider`, engine’e kayıtlı `image://provider/id` yolunu ve Qt image cache/lifecycle’ını kullanır. Bu mekanizma custom namespace parser’ın gerekli olduğunu kendiliğinden kanıtlamaz; gerekli renderer için yeni registry yazma zorunluluğunu ortadan kaldırabilir.

Bu uzatma noktalarının varlığı, `icon.name: "material:save"` ifadesinin stock bütün kontrollere kendiliğinden yayılacağı anlamına gelmez. [CONTRACTS.md:203](../docs/CONTRACTS.md) bu ayrımı doğru koyuyor. Native/stock kontrol ile özel AppIcon routing’i birbirinin kanıtı sayılamaz.

## Tasarım girdileri

[SOURCES.md](../references/SOURCES.md) ve Material3/Apple/Fluent mevcut DESIGN.md snapshot’ları okundu. Bunlar ayrı kitlerin girdileridir; ortak Runtime semantiği için birincil standart yerine geçmez. Örneğin [Yerel arşiv: Apple snapshot:65](evidence/README.md#historical-inputs) spacing’i, :91 hover tarifi veya :102 ikon önerisi Qavsi standardı olamaz. Public AppKit/HIG sınırı ayrıca korunur. Snapshot içerikleri değiştirilmedi.

## Gerçekte yapılan doğrulama

| Kontrol | Sonuç | Kanıt ve sınır |
|---|---|---|
| İstenen mimari/sözleşme/teslimat/geçiş/ADR/kit belgeleri | **SOURCE CHECK tamam** | README, ARCHITECTURE, CONTRACTS, DELIVERY, MIGRATION, DECISIONS, KIT-AUTHORING; mevcut Merce AppIcon/registry/resources, button experiment, platform, WindowIntegration ve ilgili tüketiciler okundu. |
| Qt public ikon/plugin/native kaynak kontrolü | **SOURCE CHECK tamam** | QtBase sabit commit; yerel SDK SHA listesiyle eşleşiyor. [Yerel arşiv: Provenance](evidence/README.md#review-04), public kaynak URL’leri ve SHA-256. Tam upstream araştırma snapshot’ları yalnız `/private/tmp/qavsi-review-04/research/` cache’inde; teslimat paketinde yok. |
| İzole public API programı configure/build | **PASS** | CMake 4.4.3, Qt 6.12.0, C++20. [Yerel arşiv: Kaynak](evidence/README.md#review-04), [Yerel arşiv: CMake](evidence/README.md#review-04), [Yerel arşiv: configure](evidence/README.md#review-04), [Yerel arşiv: build](evidence/README.md#review-04). |
| Offscreen, penceresiz çalıştırma | **PASS — 10/10 check** | Font lookup/render, missing font icon, QML public hints/C++ eşleşmesi, iki-engine image-provider izolasyonu ve yıkım sahipliği. Native SF offscreen’de unavailable; bu beklenen kapsam farkı. |
| Cocoa, penceresiz çalıştırma | **PASS — 12/12 check** | Yukarıdakilere ham native SF render/missing kontrolü eklenir. Hiç pencere oluşturulmadı/gösterilmedi; foreground transformation kapalı; OS tercihi yazılmadı. |
| Cocoa stderr | **Kısıt kaydedildi** | Sandbox pasteboard/XPC/LaunchServices tanıları var; [Yerel arşiv: stderr](evidence/README.md#review-04). İkon pixmap sonucu geçerli; bunu native UI, notification delivery veya tam platform kabulü saymıyorum. |

Yeniden çalıştırma yalnız geçici klasörde:

```bash
rtk cmake -S /private/tmp/qavsi-review-04 -B /private/tmp/qavsi-review-04/build -G Ninja -DCMAKE_PREFIX_PATH=QT_SDK
rtk cmake --build /private/tmp/qavsi-review-04/build --parallel 2
rtk env QT_QPA_PLATFORM=offscreen QT_MAC_DISABLE_FOREGROUND_APPLICATION_TRANSFORM=1 QML_DISABLE_DISK_CACHE=1 /private/tmp/qavsi-review-04/build/qavsi-boundary-proof
rtk env QT_QPA_PLATFORM=cocoa QT_MAC_DISABLE_FOREGROUND_APPLICATION_TRANSFORM=1 QML_DISABLE_DISK_CACHE=1 /private/tmp/qavsi-review-04/build/qavsi-boundary-proof
```

Program kanıt klasörüne de kopyalandı; geçici klasör yoksa iki kaynak dosyası o konuma alınabilir. Material font girdisinin tam yolu ve SHA-256 değeri provenance kaydında.

## NOT-RUN / kalan kanıtlar

- Merce veya yeni Qavsi ürün build/test/kurulu-paket consumer’ı çalıştırılmadı; yeni Runtime implementasyonu yok.
- Stock SK.Button veya native Menu’nün **görsel** ikon çıktısı bu deneyde çizdirilmedi. Qt-only QIcon pixmap kanıtı bu kontrol/screenshot testinin yerine geçmez.
- Custom QIconEnginePlugin keşfi/install/static linkage çalıştırılmadı; public API/header ve kaynak yolları kontrol edildi.
- Aynı anda çoklu provider/family, ikon başına axes, advanced SF config/catalog, cache invalidation ve asset lisans kabulü test edilmedi.
- Native Menu açılışı, model mutation/reentrancy, erişilebilirlik/klavye/focus, native materials/Metal/clip/DPR/cleanup ve canlı OS tercihi değişim bildirimleri test edilmedi. OS tercihleri değiştirilmedi.
- Windows/Linux/WASM/embedded çalıştırması veya bağımsız Material3/Apple/Fluent installed consumer’ı yok. İki-engine provider testi, StyleKit’in engine scope’unu test etmez.
- İlgisiz ürün dosyası, MButton, kurulu SDK, Git index/branch/remote değiştirilmedi.

## Koordinatör aksiyonları

1. Runtime/target/geçiş tablolarına bu kabul durumlarını ekle; “iki tüketici” kuralını tek başına kabul kapısı olarak kullanma.
2. SF ve temel Material için Qt-first kararı yaz; `AppIcon` compatibility’sini ayrı tut. Sadece kalan multi-collection/instance-axis ihtiyacı için küçük spike aç.
3. İlk bağımsız kiti Qt + kendi Theme/Style ile doğrula; ihtiyaç çıkana kadar Runtime boş kalabilsin.
4. ButtonSupport, eksik platform tercihleri ve Materials mevcut projelerinde kalsın; ortak çıkarma ancak component bazında bütün kapılar geçtiğinde gelsin. Menus için ayrı target yerine önce public Qt Menu’yu tüket.

Atlanan: ürün migrasyonu, görsel/native etkileşim ve dağıtım matrisi. Risk: dar Qt-only başarı, henüz var olmayan Qavsi modüllerinin destek kanıtı diye sunulmamalı.
