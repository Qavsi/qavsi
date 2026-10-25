# 02 — Apple tasarım sistemi ve native malzeme sınırı

**2026-10-25 kaynak notu:** Aşağıdaki katalog karşılaştırması ilk designmd.app Liquid Glass girdisine aittir. Kullanıcı native hedefi korudu; [güncel Apple DESIGN.md](../design-systems/apple/DESIGN.md) HIG/public AppKit’e dayanır. VoltAgent web dosyası [ek incelemede](apple-source-addendum.md) yardımcı referans olarak ayrılır. AppKit/Qt compile-only kanıtı kendi dar kapsamında kalır; yeni sözleşme/görünüm test edilmiş sayılmaz. [Güncel kaynak kaydı](../references/SOURCES.md).

**Karar: CONDITIONAL.** Bağımsız Apple kiti ve public AppKit tabanlı dar native entegrasyon uygulanabilir. Bugünkü Merce Glass kodunu ad değiştirerek bağımsız Apple kiti veya Qavsi ortak modülü kabul etmek **FAIL**. Public API/availability derleme kanıtı **PASS**; gerçek cam çıktısı, foreground okunabilirliği ve native lifecycle kabulü henüz **NOT-RUN**.

**Tarih:** 2026-10-23. **İncelenen taban:** Qt 6.12.0, C++20, CMake 3.30+; macOS 26.5.2 / SDK 26.5. Bu rapor ürün kodunu, SDK’yı, MButton’ı veya Git durumunu değiştirmedi. Yazılanlar yalnız bu rapor ve [Yerel arşiv: 02 kanıtları](evidence/README.md#review-02).

## Kararın dayanağı

ARCHITECTURE, CONTRACTS, MIGRATION, KIT-AUTHORING ve SHARED-ADMISSION okundu. Son sözleşmedeki kit politikası/public Qt/yerel native bridge ayrımı doğru. “Tek owner” bir lifecycle invariant’ıdır; yeni bir Qavsi coordinator sınıfının gereklilik kanıtı değildir. [CONTRACTS:263](../docs/CONTRACTS.md)

SF Symbols, motion/contrast’ın public Qt/QML erişimi ve temel per-window Qt olguları [04 incelemesinde](04-shared-boundary.md) doğrulandığı için tekrar deney yapılmadı. Buradaki ek kanıt AppKit availability, native handle sınırı ve mevcut malzeme lifecycle’ının kaynak incelemesidir. **Bu incelemeden `QUALIFIES QAVSI` kararı çıkmıyor.**

## Önem sırasıyla bulgular

### P1 — R02-01: Public header, desteklenen public native API anlamına gelmiyor

Mevcut [Yerel arşiv: SurfaceMaterial:108](evidence/README.md#local-source-audit), [Yerel arşiv: WindowMaterial:150](evidence/README.md#local-source-audit) ve [Yerel arşiv: chrome adaptörü:63](evidence/README.md#local-source-audit), `QWindow::handle()` kullanıyor. Qt 6.12.0 sabit kaynağında bu fonksiyon **`\internal`** olarak belgelenmiş; public header’da derlenebilmesi public destek sözü sağlamıyor. [QtBase qwindow.cpp:2215, commit 025bdad181de241e81bf853c8a2d7bf3d19261a9](https://github.com/qt/qtbase/blob/025bdad181de241e81bf853c8a2d7bf3d19261a9/src/gui/kernel/qwindow.cpp#L2215)

Belgeli başlangıç `QWindow::winId()` ve `QPlatformSurfaceEvent` lifecycle’ıdır. `winId()` henüz yoksa platform penceresi yaratabilir; teardown sırasında handle aramak için çağrılmamalı. Surface oluşturuldu/yıkılıyor bilgisi izlenmeli, GUI thread’de alınan `NSView*` ödünç referans olarak ele alınmalı ve yıkım başlamadan bırakılmalı. Qt’nin güncel platform belgesi macOS `WId` karşılığını `NSView*` diye açıklıyor; içerideki Qt/Metal layer ağacını public contract yapmıyor. [QWindow::winId](https://doc.qt.io/qt-6/qwindow.html#winId), [Qt UI toolkit entegrasyonu](https://doc.qt.io/qt-6/mixing-qt-with-other-ui-toolkits.html)

**En küçük düzeltme yönü:** Yeni Apple entegrasyonu internal `handle()` kontrolünü kopyalamamalı; belgeli surface lifecycle’ından başlamalı. Qt private interface/header veya SDK yaması gerekmez. Bu karar [güncel sözleşmeye:275](../docs/CONTRACTS.md) işlendi; mevcut ürün implementasyonunun düzeltilmesi bu incelemenin dışındadır.

### P1 — R02-02: Paylaşılan container, tek pencere sahibi sağlamıyor

Her `WindowMaterial` kendi `NativeState`/observer’ını tutuyor, `opaque` ve `backgroundColor` snapshot’ı alıyor, ayrılırken koşulsuz geri yüklüyor. [Yerel arşiv: Snapshot:169](evidence/README.md#local-source-audit), [Yerel arşiv: restore:131](evidence/README.md#local-source-audit). Her SurfaceMaterial de ayrı native state ve observer taşıyor; yalnız container’ı pencerenin alt görünümlerinden bulup ortak kullanıyor. [Yerel arşiv: Surface state:32](evidence/README.md#local-source-audit), [Yerel arşiv: container arama:124](evidence/README.md#local-source-audit)

Kaynak üzerinden bozulma senaryosu: aynı pencereye A ve B WindowMaterial bağlanır; A orijinal opaklığı, B A’nın saydam durumunu kaydeder. A ayrılınca B bağlıyken opaklığı geri koyar; B ayrılınca saydam snapshot geri kalabilir. Ayrıca mask güncellemesi ilk WindowEffectView’i bulup çıkar; ikinci backdrop’ın maskelenmesi garanti değildir. [Yerel arşiv: Mask arama:28](evidence/README.md#local-source-audit). Bu senaryo bu turda çalıştırılmadı.

**En küçük düzeltme yönü:** Kitin root entegrasyonu pencere başına tek owner kurmalı; duplicate talep açıkça reddedilmeli veya aynı owner’a yönlenmeli. Owner yalnız kendi değiştirdiği window durumunu, sonradan uygulamanın yaptığı değişikliği ezmeden geri almalı. Tüm mask/view/observer temizliği aynı ömürde olmalı. Native layer’ların sayısı değil, **aynı pikselde tek native malzeme** invariant’ı korunmalı. Genel compositor/registry gerekmiyor; bridge kit/projede kalır.

### P1 — R02-03: Native bağlantı ve QML foreground, tam Liquid Glass okunabilirliği kanıtlamıyor

[Yerel arşiv: SurfaceMaterial:143](evidence/README.md#local-source-audit) boş native `contentView` oluşturuyor; QML yazısı ayrı Qt Metal sahnesinde kalıyor. Kaynaktaki `shortcut:` bu sınırı doğru kaydetmiş. [Yerel arşiv: Apple public SDK header:27](evidence/README.md#local-source-audit), glass içeriğine ilişkin garantiyi `contentView` ile sınırlar. NSVisualEffectView’nin bulunması da özel foreground’u kendiliğinden vibrant yapmaz. [NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview), [NSVisualEffectView](https://developer.apple.com/documentation/appkit/nsvisualeffectview)

Mevcut uygulamanın olumlu tarafı: Qt content view veya Metal layer yeniden parent edilmiyor; yalnız kendi native arka planları alta ekleniyor. [Yerel arşiv: Surface:135](evidence/README.md#local-source-audit), [Yerel arşiv: Window:181](evidence/README.md#local-source-audit). Fakat `.superview`/`.window` eşitliği yalnız bağlantıyı kanıtlar; ekrandaki optik sonucu veya QML içeriğinin cam tarafından örneklendiğini kanıtlamaz. [Yerel arşiv: Dönen sonuç:174](evidence/README.md#local-source-audit)

**En küçük düzeltme yönü:** Native durum `Attached` gibi mekanik adla sunulmalı; otomatik HIG/native kontrol uyumu vaat edilmemeli. Qt’nin kök layer’ını ele geçirerek vibrancy çözülmeye çalışılmamalı. Tema foreground’u için gerçek kompozit ekran üzerinde açık/koyu, parlak/yoğun arka plan, disabled/focus ve metin yoğun dialog test edilmeli. Kontrast sağlanmıyorsa Apple kit daha güçlü okunaklı yüzey veya opak fallback seçmeli. Merce’nin [Yerel arşiv: DESIGN:399](evidence/README.md#local-source-audit) ve [Yerel arşiv: :477](evidence/README.md#local-source-audit) mevcut sınırlamayı zaten dürüstçe belirtiyor.

### P2 — R02-04: Kabul edilen scale, corner radius ile aynı sözleşmede değil

[Yerel arşiv: surfaceRect:138](evidence/README.md#local-source-audit) scene’e dönüştürülmüş axis-aligned dikdörtgeni kabul ediyor; uniform/nonuniform scale bu kontrolden geçebilir. Native radius ise [Yerel arşiv: macOS:155](evidence/README.md#local-source-audit) üzerinden ölçeklenmemiş `m_cornerRadius` kullanıyor. Örnek: geniş bir yüzeyde ancestor scale=1.1 iken 18 birim QML radius görselde 19.8’e çıkabilir; native radius 18 kalır. Nonuniform scale tek `cornerRadius` ile aynı şekli vermez.

Mevcut test [Yerel arşiv: host scale:244](evidence/README.md#local-source-audit) için yalnız frame eşleşmesini kontrol ediyor. Ölçekli radius/clip uyumu kanıtı değildir. Rotation, ancestor clip, kısmi opacity ve pencere dışına taşma için mevcut fallback kapısı iyi bir başlangıç. [Yerel arşiv: Geometri sınırı:131](evidence/README.md#local-source-audit)

**En küçük düzeltme yönü:** İlk Apple dilimi yalnız translation ve item scale=1 destekleyebilir; diğer transform’lar fallback’e gider. Bu, DPR=1 zorunluluğu değildir. Scroll/clip desteklenecekse explicit viewport kesişimi ve gerçek native mask kabulü ayrıca gerekir. Keyfi transform/region motoru yazılmaz. Sözleşme bu sınırı [CONTRACTS:279](../docs/CONTRACTS.md) içine aldı.

### P2 — R02-05: Legacy bool ve `supported` bilgisi yeni capability sözleşmesi olamaz

`nativeSupported()` Cocoa/Metal/scene graph’ı kontrol eder; alpha, accessibility ve hedef geometri daha sonra elenir. [Yerel arşiv: Surface destek:94](evidence/README.md#local-source-audit), [Yerel arşiv: uygulama kapıları:108](evidence/README.md#local-source-audit). Dolayısıyla `supported=true, active=false`, hata/tercih/başlangıç/unsupported geometriyi ayırmaz. Bu legacy API korunabilir; yeni API’nin yeterli durum modeli sayılmamalı.

Motion/contrast için [04’teki Qt-first kararı](04-shared-boundary.md) geçerli. Qt’nin `Unknown` bilgisi bool false’a indirgenmemeli. Qt’de olmayan reduce transparency, public `NSWorkspace.accessibilityDisplayShouldReduceTransparency` ve workspace notification center üzerinden izlenebilir; property macOS 10.10+, motion 10.12+. [Yerel arşiv: Yerel SDK tanımları:93](evidence/README.md#local-source-audit), [Apple transparency API](https://developer.apple.com/documentation/appkit/nsworkspace/accessibilitydisplayshouldreducetransparency)

Mevcut [Yerel arşiv: Accessibility.h:33](evidence/README.md#local-source-audit) ve macOS dışı no-op, bilgisi olmayan ortamı false gibi sunuyor. [Yerel arşiv: Observer cleanup:29](evidence/README.md#local-source-audit) ve GUI’ye queued aktarım yerinde; canlı bildirim teslimi bu turda test edilmedi. Yeni kitte tercih henüz bilinmiyorsa opak başlangıç yüzeyi kullanılmalı; uygulama override’ı OS’nin azaltma talebini iptal etmemeli. Doğrudan Qt olguları için yeni global Platform façade açılmaz.

## Mevcut DESIGN.md ile resmi Apple rehberi

Başlangıç girdisi [Yerel arşiv: Liquid Glass DESIGN.md](evidence/README.md#historical-inputs); `dc843b4538a962caca06e21c1e43c1a264ea72f638b186a850f6f8ca17550b93` hash’i korundu. [SOURCES](../references/SOURCES.md) ve kaynak metadata’sı incelendi. `pre-liquid.DESIGN.md` yalnız tarihsel ek; Liquid Glass yerine kullanılmadı. Bu dosyalar üçüncü taraf başlangıç girdisi, Apple normatif spesifikasyonu değildir.

| Girdi / fark | Apple kit için sonuç |
|---|---|
| [Yerel arşiv: :4](evidence/README.md#historical-inputs), :75 ve :111–116 her yerde morphing/dynamic blur/iridescence öneriyor; functional/content ayrımı yok | Üretim sözleşmesi kullanım rolünü açık seçmeli. Cam navigation/control katmanında sınırlı; content yüzeyi ayrı. Metin yoğun alert/sidebar/popover için regular uygundur; bütün dialog’lara cam yasağı çıkarılamaz. Clear, görsel olarak zengin medya bağlamına özgüdür. [HIG materials](https://developer.apple.com/design/human-interface-guidelines/materials) |
| [Yerel arşiv: :96](evidence/README.md#historical-inputs) circular spinner’ı yasaklıyor | **Doğrudan çelişki:** Apple macOS rehberi arka plan işi veya dar alan için spinner önerir. Determinate/indeterminate kararını işin bilgisi belirler; kit spinner’ı kategorik yasaklamamalı. [HIG progress indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators) |
| [Yerel arşiv: :78](evidence/README.md#historical-inputs), :79 ve :111 sabit giriş, hover scale ve uzun morphing değerleri veriyor | Bunlar native platform standardı değildir. Reduce Motion’a rağmen zorunlu dekoratif motion olamaz; süre/easing ve devam eden animasyonun azaltmaya tepkisi kit politikasıdır. [Apple reduce motion](https://developer.apple.com/documentation/appkit/nsworkspace/accessibilitydisplayshouldreducemotion) |
| [Yerel arşiv: :49](evidence/README.md#historical-inputs) CSS system stack, rem/px ölçeği; :64–70 CSS grid/web breakpoint/z-index | Qt’ye sayı kopyalama sözleşmesi yok. Apple tipografi rolleri, sistem fontu, locale ve okunabilirlik temel alınmalı; kullanıcı yazı boyutuna yerleşim uyumu test edilmeli. HIG macOS body referansı 13 pt; bu tüm Qt label’larını zorunlu 13 yapma kararı değildir. [HIG typography](https://developer.apple.com/design/human-interface-guidelines/typography) |
| [Yerel arşiv: :6](evidence/README.md#historical-inputs) sabit palet; :102 Lucide/Heroicons önerisi | Özel estetik kararı olabilir; adaptive Apple semantic color/font/icon contract’ı sağlamaz. SF temel lookup için Qt zaten var; 04 kararı uygulanır. Apple sistem fontları adla sabitlenip paketlenmez. Kaynak dosyanın CC atfı font/simge varlık lisansı yerine geçmez. [Apple fonts](https://developer.apple.com/documentation/technologyoverviews/fonts) |

Kaynak snapshot değiştirilmez; üretim Apple DESIGN.md’si bu farkları ve desteklenen family/platform’ları kaydeder. Mevcut Merce Glass, [Yerel arşiv: MerceStyle:10](evidence/README.md#local-source-audit) ve Merce Theme/Foundation’a bağlıdır. [Yerel arşiv: WindowIntegration:12](evidence/README.md#local-source-audit) de MerceStyle cast’i kullanır. Bunlar bağımsız Apple kitinin temel sınıfı olamaz; yeni kit doğrudan StyleKit Style ve kendi Theme/config’inden başlar.

## Sahiplik ve mekanizma kararları

| Alan | Bugün | Önerilen sahiplik / kabul |
|---|---|---|
| Typography, semantic renk, material rolü, motion, fallback tint/radius | Merce Theme/GlassRecipe/GlassPolicy | **KEEP KIT/APP.** Apple kendi çözümlemesini yapar; Merce token adları evrenselleştirilmez. |
| Standart kontrol input/focus/Action/IME/accessibility | Qt Quick Controls/StyleKit | **USE EXISTING QT.** Cam görünümü ikinci input kontrolü yaratmaz. |
| Motion/contrast, renderer, window/screen/DPR, system move/resize | Qt + legacy Merce façade | **USE EXISTING QT.** Pencereye bağlı bilgi doğrudan Qt’den; 04 kanıtı tekrar edilir gibi sunulmaz. |
| Qt’de bulunmayan reduced transparency | Merce AppKit adaptörü | **DEFER.** Önce kit/projede unknown/notification/cleanup ve gerçek consumer kanıtı. |
| Native backdrop, glass surface, mask/view lifecycle | Merce Platform | **DEFER.** Önce Apple/proje içinde dar bridge. Aynı OS primitive’inin aynı lifecycle’ını isteyen bağımsız consumer ve distribution kanıtı olmadan Qavsi target yok. |
| Window chrome ve traffic-light geometrisi | PlaygroundWindowChrome + QWindowKit | **KEEP APP / optional adapter.** Glass, Theme veya input stillerine sahip olmaz. QWindowKit yoksa fetch/build yok; native glass ona bağımlı olmaz. |

QWindowKit mevcut [Yerel arşiv: CMake:25](evidence/README.md#local-source-audit) yolunda Qt private target’larını kullanıyor; consumer’ın optional alanında kalması bu riski core’dan ayırır. Sürümü pinli olsa da private Qt uyumluluğu yeni Qt sürümünde tekrar test ister. Public Qt taşıma/resize yeterliyse yeni bağımlılık eklenmez. Chrome’un window appearance’ı ile material’ın kendi effect appearance’ı farklı sahipliklerdir; eski snapshot yeni temayı ezmemeli.

**Per-window geometri:** [Yerel arşiv: Legacy Platform:31](evidence/README.md#local-source-audit) primary-screen DPR döndürür; native yüzey ölçeği için kullanılmamalı. Qt item scene koordinatları ile AppKit view koordinatları aynı hedef pencereye dönüştürülmeli; mantıksal koordinatlara körlemesine bir kez daha DPR çarpılmaz. Piksel cache/mask gerektiğinde o pencerenin DPR/backing dönüşümü izlenir. Qt’nin Cocoa [devicePixelRatio uygulaması:2182](https://github.com/qt/qtbase/blob/025bdad181de241e81bf853c8a2d7bf3d19261a9/src/plugins/platforms/cocoa/qcocoawindow.mm#L2182) da view backing dönüşümünü kullanır. Ekran/DPR değişimi ayrı kabul senaryosudur.

## En küçük ilk dikey dilim

1. Apple kitinde kendi Theme/Style ile minimal consumer: normal/focus/disabled buton, bir navigation/header yüzeyi, bir opak içerik/form alanı ve metin yoğun dialog. Merce import’u yok; kullanılmayan family için boş API/dosya yok.
2. Önce opak, shadersız software fallback geçer. Root alpha clear gerekiyorsa yapısal RGBA sıfır kullanılır; görünür canvas kit yüzeyidir. Tema/kit değişiminde native request bırakılır, iş nesnesi ve Qt kontrol state’i korunur. Portable efekt yalnız açık app-owned kaynağı örnekleyebilir; masaüstü screenshot capture varsayılan fallback yapılmaz.
3. Ardından yalnız bir macOS 26+/Cocoa/Metal regular glass yüzeyi açılır. Kit-owned tek pencere owner’ı, belgeli `winId()`/surface lifecycle, GUI-thread AppKit, item scale=1/translation, tam görünür/opacity=1/clip yok sınırı. Pending/unavailable/failed durumunda opak kit yüzeyi sürer.
4. İkinci native yüzey ve window backdrop ancak ilk dilim görüldükten sonra eklenir; aynı piksel çift malzeme almaz. Root/Metal hiyerarşisi ve hit testing Qt’de kalır. Eski OS’de NSVisualEffectView seçilirse adı ve görünür davranışı standart material olarak tanımlanır; Liquid Glass eşdeğeri sayılmaz.
5. Source ve installed consumer doğrulanır. Ancak bağımsız ikinci kullanım aynı girdi/sonuç/hata/cleanup sözleşmesini gerçekten gerektirirse SHARED-ADMISSION yeniden açılır. İlk dilim Qavsi Runtime bağımlılığı olmadan başlayabilir.

## Yapılan doğrulama ve gerçek kabul matrisi

Public availability probe yalnız OBJECT target derler; uygulama linklemez veya çalıştırmaz. Kaynak [Yerel arşiv: public-api-probe.mm](evidence/README.md#review-02), [Yerel arşiv: CMake](evidence/README.md#review-02), [Yerel arşiv: configure](evidence/README.md#review-02), [Yerel arşiv: build](evidence/README.md#review-02). macOS 12.0 deployment target ve `-Werror=unguarded-availability` ile SDK 26.5 public NSGlassEffectView/ContainerView, NSVisualEffectView, workspace preference/observer ve Qt public window/renderer API’leri derlendi. `QWindow::handle()` probe’ta yok.

Object metadata’da minOS=12.0, SDK=26.5; iki NSGlass sınıfı weak external. [Yerel arşiv: Availability symbols](evidence/README.md#review-02), [Yerel arşiv: deployment](evidence/README.md#review-02). Bu, macOS 12’de Qt uygulama çalıştırma veya cam davranışı kanıtı değildir. Configure, opsiyonel Qt6TaskTree/Qt6QmlAssetDownloaderPrivate paket uyarısı verdi; configure ve derleme başarılı. SDK değiştirilmedi. Online belgelerdeki daha yeni/beta üyeler bu SDK için destek kabul edilmedi.

| Senaryo | Bu tur / gerçek kabul koşulu |
|---|---|
| Tasarım kaynakları, mevcut native/QML/chrome ve ilgili test kaynakları | **SOURCE CHECK PASS.** Input hash’leri eşleşti; tam upstream metinler teslimata kopyalanmadı. URL/commit/SHA-256 [Yerel arşiv: provenance](evidence/README.md#review-02) içinde; araştırma cache’i `/private/tmp/qavsi-review-02/research/`. |
| Public AppKit/Qt availability compile | **PASS.** Yalnız public header ve compile; yeni bir Qavsi bridge’in gerekli olduğunu göstermez. |
| Mevcut Merce product build/ctest/gates | **NOT-RUN.** Ürün değişikliği yok; mevcut testlerin varlığı bu tur başarı sonucu diye sunulmadı. |
| Cocoa/Metal ekranda composition | **NOT-RUN.** Aynı pikselde çift cam yok; desktop/window/QML content örnekleme, alpha/renk uzayı, ilk frame, light/dark ve active/inactive pencere gerçek ekranda görülmeli. `NSVisualEffectStateActive` seçimi mevcut [Yerel arşiv: WindowMaterial:177](evidence/README.md#local-source-audit) yolunda sabit; foreground kaybı davranışı ayrıca kabul edilmeli. |
| Dialog/alert/popover okunabilirliği | **NOT-RUN.** Regular/tinted/opaque seçenekleri; açık/koyu ve hareketli yoğun arka plan, küçük/uzun metin, focus/disabled ayrımı, screenshot karşılaştırması ve erişilebilirlik değerlendirmesi. Offscreen native desktop glass göstermez. |
| Tek owner ve restore | **NOT-RUN.** Duplicate WindowMaterial, iki surface, üst üste bölgeler, style/request/target yıkımı, retarget, native surface destroy/create, minimize/restore ve uygulamanın arada yaptığı window property değişikliği. |
| Geometry/DPR | **NOT-RUN.** İki pencere farklı ekran/DPR’de; ekranlar arası taşıma, resize, fractional koordinat, radius/mask kesişimi, rotation/nonuniform scale/clip/opacity/z-order ve popup ayrı penceresi. Destek dışı durumda okunaklı fallback doğrulanmalı. |
| OS tercihleri | **NOT-RUN.** Canlı Reduce Motion/Transparency/Contrast, başlangıç unknown, animasyon sürerken değişim, observer temizliği. Tercihler bu turda okunabilir API açısından incelendi; OS ayarı yazılmadı/değiştirilmedi. |
| Native input/chrome | **NOT-RUN.** Klavye/focus/VoiceOver/IME, click-through; drag boşluğu ile input ayrımı, double-click, fullscreen, resize ve traffic-light alanı gerçek input ister. |
| Software/old OS/diğer platform ve paket | **NOT-RUN.** Ayrı software çalıştırması, macOS <26 fallback, Linux/Windows/WASM; Merce kurulu değilken Apple source+installed consumer ve optional adaptör OFF build’i. |
| Performans | **NOT-RUN.** Aynı sahnede 1/4/12 yüzey, frame-time/CPU/GPU/bellek ve inactive/occluded pencere maliyeti. Mevcut her surface’in `afterAnimating` kontrolü (:70) maliyetsiz varsayılmamalı. |

Mevcut native testlerde [Yerel arşiv: rotation/opacity/clip:294](evidence/README.md#local-source-audit), [Yerel arşiv: mask lifecycle:324](evidence/README.md#local-source-audit), [Yerel arşiv: window lifecycle:164](evidence/README.md#local-source-audit) ve appearance sahipliği için yararlı senaryolar var. Native yollar `MERCE_REQUIRE_NATIVE_MATERIAL=1` ister ve pencere gösterebilir; verilen penceresiz inceleme sınırı nedeniyle çalıştırılmadı. Mevcut test kaynağı yeni Apple kitinin bağımsız paket kabulünün yerine geçmez.

Atlanan: ürün taşıma/düzeltme, görsel/native etkileşim ve paket matrisi. Risk: compile veya native attachment başarısı görünür Liquid Glass, otomatik QML vibrancy ya da HIG uyumu diye sunulmamalı.
