# 03 — Fluent 2 tasarımı ve Qt 6.12 fizibilitesi

**2026-10-25 kaynak notu:** Aşağıdaki katalog/renk diagnostikleri ilk incelemenin geçici designmd.app girdisine aittir. Kullanıcı resmi Microsoft rehberlerini esas aldı; [güncel Fluent sözleşmesi](../design-systems/fluent/DESIGN.md) bu kararı izler. Microsoft/Qt kaynak ayrımı kendi pinlenen sürümüyle geçerlidir; yeni sözleşmenin render testi sayılmaz. [Güncel kaynak kaydı](../references/SOURCES.md).

**Tarih:** 2026-10-24 TRT / araştırma 2026-10-23 UTC. **Karar: CONDITIONAL.** Qt + kendi Theme/Style modülleriyle bağımsız, opak Fluent 2 dikey dilimi uygulanabilir. Mevcut katalog DESIGN.md'si üretim spesifikasyonu olarak **FAIL — eksik ve platformları karıştırıyor**. Gerçek Fluent kiti bulunmadığından ürün/runtime kabulü **NOT-RUN**; bu rapor kod kapsamı veya Windows native başarı iddiası üretmez.

| Değerlendirme | Sonuç | Sınır |
|---|---|---|
| Bağımsız kit mimarisi | **CONDITIONAL** | Aşağıdaki edition/platform/state sözleşmesi önce sabitlenmeli; Merce veya boş Runtime bağı gerekmiyor |
| Mevcut DESIGN.md'nin üretim yeterliliği | **FAIL** | Web/Windows/WinUI/Reveal ayrımı, dark/contrast/state/material/font sözleşmesi eksik |
| Tanımlı dört light renk çifti | **PASS** | Yalnız kaynak literal'leri; hover/pressed/checked/dark/focus ölçümü değil |
| Katalog alpha tariflerinin sağlamlığı | **Karşı örnek bulundu** | Varsayımsal sRGB bileşim hesabı; native Mica/Acrylic ölçümü veya simülasyonu değil |
| Qt'nin mevcut FluentWinUI3 alternatifi | **USE EXISTING QT değerlendirmesi** | Palette + yerleşik görünüm yeterliyse yeni kit gerekmeyebilir; aşağıda sınırı var |
| Fluent ürün, Windows native materyal ve gerçek UI | **NOT-RUN** | Ürün yok; Windows makinesi/toolchain, render/input/a11y testi çalıştırılmadı |
| Yeni koşulsuz Qavsi mekanizması | **Yok** | [04 kabul kararı](04-shared-boundary.md) ile aynı; bütün Runtime adları aday |

## İncelenen kapsam ve kaynakların sabitlenmesi

[ARCHITECTURE](../ARCHITECTURE.md), [CONTRACTS](../docs/CONTRACTS.md), [DELIVERY](../docs/DELIVERY.md), [KIT-AUTHORING](../docs/KIT-AUTHORING.md), [SHARED-ADMISSION](../docs/SHARED-ADMISSION.md) ve [04](04-shared-boundary.md)/[05](05-stylekit-runtime.md)/[06](06-delivery.md) raporları okundu. Gelecek değişikliğin erişmesi gereken yerler: kitin DESIGN.md'si, kendi token/Theme'i, StyleKit family/state mapping'i, source/installed consumer'ı, CMake export'ları, kontrat/görsel testleri ve varsa Gallery adaptörü. Bu incelemenin yazma kapsamı yalnız bu rapor ile `evidence/03/`; ürün, SDK, MButton, Git ve OS ayarı değiştirilmedi.

Mevcut [Yerel arşiv: Fluent DESIGN.md](evidence/README.md#historical-inputs), designmd.app üçüncü taraf özetidir; Microsoft normatif spesifikasyonu değildir. [Kaynak kaydı](../references/sources.json) içindeki SHA-256 `dfd665f72baecaf49c37052cd3f84474ecce2c4bc52c74ae9ce104cd74a4ec9b` diagnostikte doğrulandı. Snapshot değiştirilmedi; içindeki framework/dil/prompt ifadeleri talimat sayılmadı. Katalog atfı/lisansı [SOURCES.md](../references/SOURCES.md) kapsamındadır; Microsoft font/ikon haklarına genişletilemez.

| Birincil kaynak | İncelenen edition/sürüm | Tekrar üretilebilirlik |
|---|---|---|
| [Fluent 2 tasarım rehberi](https://fluent2.microsoft.design/design-tokens) | 2026-10-23 erişimi; sayfalar SemVer edition yayımlamıyor | URL + erişim tarihi + indirilen byte SHA-256 |
| [Microsoft Fluent UI](https://github.com/microsoft/fluentui/commit/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676) | `6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676`; React Button `9.11.3`, React Theme `9.2.2`, onun token bağı `1.0.0-alpha.24` | Sabit commit yolları ve dosya hash'leri; npm kurulum/çalıştırması yok |
| [Windows tasarım belgeleri](https://github.com/MicrosoftDocs/windows-dev-docs/commit/a3812316d9d6678be4e526ffb4b1c4f3958d956d) | `a3812316d9d6678be4e526ffb4b1c4f3958d956d`; Mica/Acrylic/material/typography/system-backdrops kaynakları | Microsoft Learn sayfası + sabit Markdown kaynak hash'i |
| [Qt public belgeleri](https://doc.qt.io/qt-6/qtquickcontrols-fluentwinui3.html) | Qt 6.12.0; StyleKit mekanik kanıtı 05, paketleme kanıtı 06 | Burada yeniden build edilmedi; eski proof'un kapsamı büyütülmedi |

Tam indirilen upstream sayfa/kodlar yalnız `/private/tmp/qavsi-review-03/research/` araştırma cache'indedir. Teslimatta [Yerel arşiv: URL/commit/hash envanteri](evidence/README.md#review-03), küçük kendi diagnostik kodumuz ve sonuçları var; tam Microsoft belgeleri/fontları/assets yok. `master` erişim anında sabitlendi; bunun belirli bir npm dağıtım tarball'ıyla byte-identical olduğu doğrulanmadı.

## Önem sıralı bulgular ve mevcut DESIGN.md düzeltmeleri

### P1 — F03-01: “Fluent 2” adı hedef profili tanımlamıyor

**Yer:** snapshot satır 4, 64–73 ve 101; KIT-AUTHORING §1. Web Fluent 2, Windows 11/WinUI 3 ve WinUI 2/UWP Reveal aynı görünüm ailesinin tek reçetesi değildir. [Fluent tipografi rehberi](https://fluent2.microsoft.design/typography) platformlara ayrı ramp verir. [RevealBrush](https://learn.microsoft.com/en-us/windows/winui/api/microsoft.ui.xaml.media.revealbrush?view=winui-2.8) belgesi açıkça WinUI 2/UWP kapsamındadır ve kullanımı artık önerilmez. Snapshot'taki “Reveal highlight” yeni Fluent 2 kitinin zorunlu efekti sayılmamalı.

**En küçük giderim:** İlk kit sözleşmesine **“Fluent 2 Web referanslı, taşınabilir Qt Quick görünümü; WinUI 3 native eşdeğerlik iddiası yok”** yaz. Yukarıdaki commit'i, light/dark modelini, desteklenen family'leri ve istisnaları sabitle. Windows-native görünüm istenirse ayrı Windows profili/işi olarak kendi WinUI/Windows App SDK/OS sürümüyle değerlendir; web `brandWeb`/geometry ile Windows kaynaklarını sessizce karıştırma.

Önce mevcut Qt alternatifi değerlendirilmelidir: [Qt FluentWinUI3](https://doc.qt.io/qt-6/qtquickcontrols-fluentwinui3.html), Qt 6.8'den beri var; native değildir, çoğunlukla image asset ve palette kullanır, desteklenen platformlarda çalışabilir. Bazı kontroller Fusion'a düşer; image asset'ler özelleştirmeyi sınırlar. Yalnız hazır Fluent görünümü yeterliyse **yeni kit yerine onu kullan**. Qavsi kitini gerekçelendiren fark; seçilen Web token/recipe sözleşmesi, public özelleştirme ve StyleKit tabanlı kök stil seçimi olmalıdır. Yerleşik stili sarmalayıp ona yeni ad vermek ek değer değildir. [Qt kontrol stil seçimi](https://doc.qt.io/qt-6/qtquickcontrols-styles.html) ile `SK.StyleKit.style` değişimi farklı mekanizmalardır; birinin seçimi ötekine canlı kit değiştirme özelliği kazandırmaz.

### P1 — F03-02: Mica/Acrylic/Smoke RGBA renk token'larıyla eşitlenmiş

**Yer:** snapshot satır 71–80, 99–101; CONTRACTS §5. `mica = rgba(243,243,243,.85)` gerçek Mica değildir. Microsoft [Mica rehberi](https://learn.microsoft.com/en-us/windows/apps/design/style/mica) malzemeyi **“opaque, dynamic material”** olarak tanımlar: uzun ömürlü pencere tabanı, wallpaper kişiselleştirmesi, active/inactive ve opak fallback politikası birlikte gelir. Şeffaflığı artırmak Mica desteği sağlamaz.

| Malzeme / kaynak | Resmi anlam ve Qt sınırı | Kitin güvenli fallback'i |
|---|---|---|
| [Mica / Mica Alt](https://learn.microsoft.com/en-us/windows/apps/design/style/mica) | Pencere temel katmanı; alt varyant tabbed-titlebar hiyerarşisi için daha güçlü tint. Bir card'a RGBA atamak native backdrop değildir | Kitin light/dark opak tabanı; high-contrast'ta kullanıcı sistem renkleri |
| [Desktop/background Acrylic](https://learn.microsoft.com/en-us/windows/apps/design/style/acrylic) | Geçici, light-dismiss yüzeyler; arka pencere/desktop bağlamı. GPU ve güç maliyeti var | Geçici yüzeyin opak kit rengi; blur zorunlu değil |
| [In-app Acrylic](https://learn.microsoft.com/en-us/windows/apps/design/style/acrylic) | Uygulama içeriğini örnekleyen ayrı blend/overlay kullanımı; background Acrylic ile aynı kaynak değil | Okunaklı opak destek/overlay yüzeyi |
| [Smoke](https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/materials) | Modal UI altını karartan, light/dark'ta translucent black scrim; blur/backdrop değildir | Qt Dialog/Popup modal semantiği + kitten gelen karartma; yeni native bridge gerekmez |
| Katalog `surface` | Frontmatter satır 8 opak `#f3f3f3`, prose satır 78 saydam beyaz: iki farklı sözleşme | Üretim DESIGN.md tek anlam seçer; içerik tabanı/popup/scrim rolleri ayrılır |

Native Windows için iki public yolun sürüm ve bağımlılığı farklıdır:

- [DWM `DWM_SYSTEMBACKDROP_TYPE`](https://learn.microsoft.com/en-us/windows/win32/api/dwmapi/ne-dwmapi-dwm_systembackdrop_type): minimum **Windows 11 build 22621**. `MAINWINDOW`, `TRANSIENTWINDOW`, `TABBEDWINDOW` Windows 11'de sırasıyla Mica, Desktop Acrylic, Mica Alt'a karşılık gelir; Microsoft gelecek OS'de efektin değişebileceğini belirtir. [DwmSetWindowAttribute](https://learn.microsoft.com/en-us/windows/win32/api/dwmapi/nf-dwmapi-dwmsetwindowattribute) `HWND` ve `HRESULT` ister. Salt enum veya `Q_OS_WIN` kontrolü başarı değildir.
- [Windows App SDK controller yolu](https://learn.microsoft.com/en-us/windows/apps/desktop/modernize/ui/apply-mica-win32): ek SDK/runtime/deployment ve Composition hedefi gerektirir; `MicaController.IsSupported()` sonucu kontrol edilir. Mica rehberi Windows build 22000 altı için opak fallback, Mica Alt için App SDK 1.1+ ve Windows 11 build 22000+ belirtir; bunlar DWM enum yolunun 22621 minimumuyla karıştırılmaz. [Güncel XAML rehberi](https://learn.microsoft.com/en-us/windows/apps/develop/ui/system-backdrops) `Window.SystemBackdrop` yanında **App SDK 2.0+ `SystemBackdropElement`** de sunar. Bu XAML element API'si keyfi Qt Quick Item'a doğrudan bağlanmaz. “Windows'ta element backdrop hiç yok” iddiası da artık doğru değildir.

**En küçük giderim:** İlk dilimde native materyal kapalı. Sonraki Windows spike'ında tek public yol seç; QQuickWindow/native handle yaşam döngüsü, compositor/renderer uyumu, alpha/opaque katman yerleşimi, activation, screen/DPR, transparency/battery/high-contrast, surface recreation ve cleanup test edilene kadar `Attached` görsel uygunluk demesin. Kitte tek pencere owner'ı ve opak fallback korunsun. DWM yoluna Windows App SDK'yı koşulsuz ekleme; aynı zamanda controller yolunun dependency'sini gizleme. Windows'ta D3D/RHI varlığı tek başına Mica yeteneğini kanıtlamaz.

[Qt software adaptation](https://doc.qt.io/qt-6/qtquick-visualcanvas-adaptations-software.html) `ShaderEffect` çizmez. Linux/Wayland/X11/EGLFS, macOS ve WASM için Windows-native materyal vaat edilmez; kitin Rectangle/Text/normal border temelli opak yolu kullanılır. Ekran görüntüsü alıp sürekli blur üretmek fallback değildir. Backend'in aynı proses içinde canlı software/GPU geçişi varsayılmaz.

### P1 — F03-03: “WCAG AA” cümlesi state ve erişilebilirlik sözleşmesi değil

**Yer:** snapshot satır 109–116. Hover/pressed/focus yalnız prose; checked/disabled/loading, dark, yüksek kontrast ve error tarifleri yok. [Fluent token modeli](https://fluent2.microsoft.design/design-tokens) global ve semantic alias katmanlarını, light/dark/high-contrast/brand çözümünü ayırır. Sekiz sabit renk bütün state tablosunun yerine geçmez. Snapshot `primary=#0f6cbd`, resmi [brandWeb[80]](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/brandColors.ts) ile uyumludur; palette bütünü yanlış diye atılmamalı.

[Fluent Button erişilebilirliği](https://fluent2.microsoft.design/components/web/react/core/button/usage) etkileşimli durumlardaki metin için 4.5:1, ikon için 3:1 ister; disabled için aynı kontrast koşulu aranmaz. [Genel erişilebilirlik](https://fluent2.microsoft.design/accessibility) ayrıca focus dönüşü, metin büyütme ve bilgi kaybetmeden reflow ister. Web ARIA/forced-colors kuralları Qt'ye isimleri kopyalanarak uygulanmaz: Qt Accessible role/name/checked, gerçek klavye focus'u, platform palette ve okuyucu davranışı ayrıca sınanır. `ContrastPreference` kullanıcı renklerinin tamamı değildir.

**En küçük giderim:** Her desteklenen appearance/state için background/foreground/stroke/focus çiftini tanımla. Focus'u seçili/hover renginin yerine geçen tek state yapma; focus+pressed ve checked+disabled birleşimlerini doğrula. Reduced motion, high-contrast ve unknown koşullarında okunaklı durum korunmalı. Önce doğrudan [QAccessibilityHints](https://doc.qt.io/qt-6/qaccessibilityhints.html) ve [QStyleHints/palette](https://doc.qt.io/qt-6/qstylehints.html) kullanılmalı; yeni OS-facts singleton'ı gerekmiyor. Qt'nin preference enum'larını CSS `forced-colors` renk değerleri sanma; erişilemeyen platform bilgisini `false`a indirme.

Tek kaynak diagnostik sonucu: [Yerel arşiv: diagnostic-result.json](evidence/README.md#review-03).

| Literal çift / örnek | Kontrast | Yorum |
|---|---:|---|
| `on-primary` / `primary` | 5.384 | Tanımlı light metin çifti PASS |
| `on-accent` / `accent` | 6.308 | Tanımlı light metin çifti PASS; secondary rolünün doğruluğunu kanıtlamaz |
| `on-surface` / opak `surface` | 18.925 | Frontmatter okuması PASS |
| `text` / `background` | 15.523 | Tanımlı light metin çifti PASS |
| `#616161` / prose `surface` siyah altında | 2.937 | Katalog alpha'nın varsayımsal bileşiminde AA altı |
| `#616161` / prose `mica` siyah altında | 3.957 | Gerçek Mica ölçümü değil |
| `#616161` / prose `acrylic` siyah altında | 2.126 | Gerçek Acrylic ölçümü değil |

Son üç satır katalogdaki muted renk ile belirtilmiş alpha'ların olası birleşimidir; bu birleşimin gerçek UI'de kullanıldığı iddia edilmiyor. Sadece arka plan/rol sınırı verilmeden “AA uyumlu materyal” denemeyeceğini gösterir. [WCAG contrast yöntemi](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) üzerinden sRGB relative-luminance hesabı yapılmıştır; native renk uzayı/compositor/render/text rasterization test edilmedi.

### P2 — F03-04: Button görünümü, geometri ve hierarchy resmi Web profilinden sapıyor

**Yer:** snapshot satır 30–48. Her iki buton da markalı dolgu ve 8 radius/16 padding kullanıyor. [Resmi Button mapping'i](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/react-components/react-button/library/src/components/Button/useButtonStyles.styles.ts) secondary'yi neutral temel tarif olarak tutuyor. Medium örnekte radius `borderRadiusMedium`, yatay `spacingHorizontalM`, dikey 5px, min-width 96px; `fonts.ts` 14/20 ve semibold verir. [Radius](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/borderRadius.ts) medium=4; [spacing](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/spacings.ts) M=12. Bu katalogdaki `rounded.md=8` ile aynı rol değildir.

**En küçük giderim:** Variationsız buton neutral secondary; tek primary tarifine açık katılım. Card/page radius'larını butonla zorla birleştirme. Medium Web referansının normal ölçüsü 20 line-height + 2×5 padding + 2×1 border = 32px'tir; bu CSS box hesabı Qt'ye körlemesine `height:32` çevirisi değildir. Gerçek font metrikleri/caller boyutu/çeviri ile büyüyen implicit/minimum geometri doğrulanmalı. Büyük touch alanı kit/ürün kararıdır; Merce profili veya bütün kitlerde ortak sabit yükseklik yapılmaz.

Primary/secondary/outline/subtle/transparent görünüm adları resmi Web kaynaklarında ayrıdır; ilk dilimde yalnız ilk ikisi yeterli. Split/compound/menu button ayrı davranış gerektirirse kit içinde ele alınır. Aynı Qt Button'a beş boş variation dosyası açmak kapsam değildir.

### P2 — F03-05: Tipografi, font bulunabilirliği ve dağıtımı ayrı sözleşme istiyor

**Yer:** snapshot satır 15–29, 87–91. Katalog `body-md=1rem/1.6`, `h1=4rem/700/1.05`; bunlar resmi type-ramp rollerinin eşdeğeri değil. [Fluent typography](https://fluent2.microsoft.design/typography) Web Body 1'i Segoe UI 14/20, Windows Body'yi Segoe UI Variable 14/20 verir; platform native font fallback'ini de destekler. `label-caps` bütün label'ları uppercase yapmak için normatif gerekçe değildir; rehber sentence case kullanır.

Kaynaklar kendi aralarında da tamamen aynı değil: aynı tarihte Web rehberi Subtitle 1 için **20/26**, sabitlenmiş [fonts.ts](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/fonts.ts) + [typographyStyles.ts](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/typographyStyles.ts) çözümü **20/28** veriyor. Üretim spec'i bu farkı kayıt altına alıp seçilen roller için kaynağı açıkça seçmeli; hareketli “güncel Fluent” tanımı yeterli değil. İlk Button dilimindeki 14/20 bu farktan etkilenmiyor.

[Microsoft font FAQ](https://learn.microsoft.com/en-us/typography/fonts/font-faq) Windows sistem fontlarını uygulama/device içine kopyalama hakkı vermiyor; Segoe UI Variable'ın Microsoft dışı/non-Windows lisans/kullanımı da kısıtlı. Segoe UI için ayrı lisans yolu bulunması otomatik izin değildir. Kit paketine Windows'tan çıkarılmış Segoe/Segoe Fluent Icons dosyaları konmamalı. Font adıyla sistemdeki yasal fontu kullanmak ile font binary'sini dağıtmak ayrıdır.

[Cascadia Code lisansı](https://github.com/microsoft/cascadia-code/blob/56bcca3f2c1e4cb19458954f0e2bb4635960df91/LICENSE) ayrı **SIL OFL 1.1**; paketlenirse copyright/lisans ve Reserved Font Name koşulları kendi asset envanterinde izlenmeli. İlk buton dilimi monospace/font bundle gerektirmez. [Fluent UI MIT LICENSE](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/LICENSE) bile referenced fonts/icons için ayrı şartlara yönlendirir; MIT kod lisansı bütün Microsoft assets'ini serbestleştirmez.

**En küçük giderim:** Windows'ta mevcut sistem ailesini, diğer OS'de Qt'nin platform sistem fontunu çöz; gerçek family/metrik/fallback kaydı tut. CSS `system-ui`/`-apple-system`/`rem` girdileri Qt'nin tüm platformlarda aynı sonucu vereceği sözleşmesi değildir. [QFont/QFontMetricsF](https://doc.qt.io/qt-6/qfont.html), [QFontDatabase](https://doc.qt.io/qt-6/qfontdatabase.html) ve [Text public font/lineHeight API'leri](https://doc.qt.io/qt-6/qml-qtquick-text.html) yeterli başlangıçtır. Point/pixel dönüşümünde sabit 96/72 çarpanı veya caller font binding'ini ezme yok. Windows GDI backend'de variable axes desteği varsayılmamalı. Font/assets sürümü seçilmeden binary dağıtım uyumu ilan edilmez.

### P2 — F03-06: Motion ilkeleri var; çalıştırılabilir motion reçetesi yok

**Yer:** snapshot satır 4, 70, 101, 116. Beş temel öğeyi sıralamak duration/easing/reduced-motion davranışını tanımlamaz. [Fluent motion](https://fluent2.microsoft.design/motion) fonksiyona göre duration/easing ve üst düzey gezinmede hızlı fade önerir; her family'ye aynı spring/ripple uygulanmaz. Resmi [duration](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/durations.ts) ve [curve](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/global/curves.ts) token'ları kitte kalır.

**En küçük giderim:** İlk Button renk/stroke geçişi için referans kaynak `durationFaster=100ms` ve `curveEasyEase=(.33,0,.67,1)`; reduced-motion durumunda sıfır süreli son durum. Bunlar Fluent kitinin recipe'sidir, Qavsi easing standardı değildir. Stil/tema değişiminde ve high-contrast'a geçerken kritik metin/focus renklerini ara düşük-kontrast değerlerle crossfade etme. Gizli/minimized sahnede sürekli animasyon yok. Loading eklenirse azaltılmış motion'da işlem bilgisi/erişilebilir isim korunmalı. Her yerel animasyon ayrı yazılmadan public Qt animation/delegate desteği kullanılmalı; native kontrol input state machine'i tekrarlanmaz.

## İlk bağımsız dikey dilim

Bu bölüm **uygulama planı**, mevcut ürün listesi değildir. Önce taşınabilir opak yüzey üzerinde **bir Button family** kanıtlanır. Tam Fluent kit veya v1 release kabulü sayılmaz; KIT-AUTHORING §7'nin loading/GPU/accessibility gibi kalan yayın kapıları açık kalır.

| Sınır | İlk somut seçim / sahip |
|---|---|
| Referans hedefi | Fluent 2 Web-derived portable Qt profili; mevcut snapshot + yukarıdaki açık Microsoft düzeltmeleri |
| Sürüm tabanı | Qt 6.12.0, C++20, CMake 3.30 minimum; test edilen toolchain/OS ayrıca kaydedilir |
| Public paket adları — aday | `QavsiFluent::Theme`, `QavsiFluent::Style`; URI `Qavsi.Fluent.Theme`, `Qavsi.Fluent.Style` |
| Theme | Fluent'e ait seçilmiş global/alias rolleri; light/dark ve kitin opak contrast fallback'i; CSS/React runtime dependency yok |
| Public config — minimum | Kitin `Light/Dark/System` seçimi, desteklenen font override'ı; uygulama persistence/ürün touch tercihini sahiplenir. Henüz gerekmeyen brand generator veya universal density enum yok |
| Style | `FluentStyle` doğrudan public `SK.Style`; MerceStyle/Merce.Theme/Merce.Platform veya diğer kit kalıtımı/import'u yok |
| Button | Qt kontrolü; variationsız neutral secondary, önceden tanımlanmış `fluent.primary`; checked Qt `checkable/checked` state'iyle çözülür |
| Diğer görsel parçalar | Testte Label/body ve opak Pane/canvas; bunlar bütün typography/surface family kapsamı sayılmaz |
| Delegate | Önce public StyleKit property mapping; gerekiyorsa yalnız Button focus/background slot'unda küçük kit delegate'i. Ortak çok-kit renderer, private `impl` import'u veya yeni kontrol wrapper'ı yok |
| QObject/scope | Composition root bir Theme ve app-owned Style instance'ını yaşatır; ana QQmlEngine en uzun ömürlü host. Pencere/engine başına farklı StyleKit vaat edilmez |
| Native materyal / font paketi | İlk dilimde yok; yalnız sistem fontu + opak kit çizimi. Basic ikon ihtiyacında native `icon.source/name`, Qt resource/QIcon; yeni registry/provider şart değil |
| Runtime | Dependency yok. Gerçek ortak ihtiyaç bütün [kabul kapılarını](../docs/SHARED-ADMISSION.md) geçene kadar boş Runtime target açılmaz |
| Gallery | Kit seçim/sample role eşlemesi Gallery'nin kendi adaptöründe; kitin Theme'i Gallery interface'ine dönüştürülmez |

### Token, state, geometri ve motion kapsamı

Aşağıdaki her satır **PLAN / NOT IMPLEMENTED**. “Mapping mümkün” ifadesi runtime doğrulaması değildir.

| Family / senaryo | Gerekli Fluent rolleri | İlk dilim sınırı ve gerekli doğrulama |
|---|---|---|
| Button secondary | `colorNeutralBackground1` normal/hover/pressed/selected; matching foreground/stroke | Normal, hover, pointer/keyboard press, visualFocus, disabled; state birleşimleri ayrı |
| Button primary | `colorBrandBackground` normal/hover/pressed/selected + `colorNeutralForegroundOnBrand`; disabled rolleri | Light `brandWeb` kademeleri 80/70/40/60; dark alias farklı kademeler kullanır. Tek hex'i koyulaştıran ortak algoritma yok |
| Checkable Button | Selected tarif + pressed/hover; disabled önceliği, focus katmanı | Aynı control kimliği ve sabit appearance rolü; checked değişince variation listesi mutasyona uğramaz. [Resmi Toggle mapping](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/react-components/react-button/library/src/components/ToggleButton/useToggleButtonStyles.styles.ts) referansı; ikon fill davranışı ilk dilimde yok |
| Font/geometri | 14/20 semibold; radius=4, stroke=1, x-padding=12, y-padding=5, min-width=96 Web referansı | Qt logical coordinates/DPR, gerçek font metrikleri, caller font/padding/width önceliği, 200% metin, uzun Türkçe/Arapça/RTL; sabit 32 yüksekliğe kırpma yok |
| Focus ve contrast | `colorStrokeFocus1/2`, foreground/background; platform palette eşlemesi | Focus+hover/press/checked görülebilir; renk tek başına durum bilgisi değil. OS renkleri unavailable ise kitin belgeli opak fallback'i; sistem contrast uyumu ayrı test |
| Motion | `durationFaster`, `curveEasyEase`; reduced motion sonucu | Normalde seçilen 100ms recipe, azaltmada anlık son değer; tema/contrast geçişi ara renk testi; opacity ile bütün kontrolü soldurma yok |
| Label/Pane fixture | Body 1 ve opak canvas/foreground | Yalnız test sahnesi; Display/Caption/link/badge/card kapsamı değil |
| Loading / Busy / Progress | Gerekli indicator recipe, durum adı, motion azaltması | **Sonraki kapı.** Bu bootstrap loading desteği iddia etmez; ButtonSupport dependency'si zorunlu değil. İş/duplicate-start uygulamada |
| TextField/TextArea/Search/SpinBox | Input idle/hover/focus/error/disabled/placeholder/label rolleri | **Kapsam dışı.** Persistent label/IME/selection/validation davranışı Qt+uygulama; input görseli ileride kitten |
| CheckBox/Radio/Switch; Slider/Range/Dial | Selection/track/thumb/tick recipe'leri | **Kapsam dışı.** Qt değer/interaction semantiği korunur; Button'ın geçmesi bunları kapsamaz |
| ComboBox/Menu/MenuItem/ToolBar | Popup/surface/selection/focus/inset | **Kapsam dışı.** Item/window/native popup farklı yollar; native menüye custom QML görünümü vaat edilmez |
| Popup/Dialog/Drawer | Surface, border/elevation, Smoke scrim | **Kapsam dışı.** Modal/dismiss/focus restore Qt'de; native Acrylic daha sonraki Windows işi |
| Tab/ItemDelegate/navigation; scroll/page göstergeleri | Selected/hover/foreground/geometry | **Kapsam dışı.** Route/layout/768px collapse uygulamaya aittir; snapshot eşiği evrensel Qt breakpoint'i olmaz |

Button'ın somut alias kaynağı [lightColor.ts](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/alias/lightColor.ts) ve [darkColor.ts](https://github.com/microsoft/fluentui/blob/6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676/packages/tokens/src/alias/darkColor.ts). Kaynak comment'lerindeki örnek brand hex'leri gerçek `brandWeb` parametresiyle aynı olmayabilir; import/alias değeri çözülür, comment hex'i token sanılmaz. Bütün Fluent token'larını Qavsi'nin veya Merce'nin tek ortak sözlüğüne çevirmek gereksizdir.

### En küçük uygulama ve kabul sırası

1. Kitin üretim DESIGN.md'sinde yukarıdaki profil/commit/state/units/font/material/fallback farklarını kaydet; katalog snapshot'ını koru. İlk kontrolün kullandığı token alt kümesini typed Theme'e bağla. Normal consumer token generator veya Node/npm gerektirmesin.
2. Tek pencere + Button family'yi public StyleKit mapping ile oluştur. İlk roller sabit olsun; Qt'nin `visualFocus/hovered/down/checked/enabled`, Action, icon ve accessible davranışını kullan. İlk custom delegate yalnız ölçülmüş eksik slot için eklensin.
3. Aynı instance üzerinde light/dark, keyboard/pointer state birleşimleri, RTL/büyük metin ve caller binding'lerini gerçek Fluent recipe'siyle test et. 05'in fixture başarısını bu yeni ürünün sonucu diye sayma. Qt role mutation regression'ı açık expected-failure olarak ayrı yaşasın; MButton workaround'u kopyalanmasın.
4. Önce offscreen/software; sonra seçilen OS/GPU üzerinde çalışan UI ve gerçek input/ekran okuyucu doğrulaması. Appearance uygunluğu için insan incelemesi/karşılaştırma gerekir; bu rapordaki sayısal renk denetimi yeterli değildir.
5. Source ve installed/relocated standalone consumer'ı Merce source/build/import/prefix erişimi olmadan kur. 06'nın metadata/plugin/component/export yöntemini uygula; yeni proof framework yazma. Static QML module ile tam static Qt sonuçlarını ayrı tut.
6. Bu dar bootstrap geçince loading/ikon/ikinci family için gerçek ihtiyacı ölç. Native Windows malzeme ayrı, isteğe bağlı kit target'ında ayrı platform acceptance işi olsun. İlk kit için Runtime açmayı veya tüm family'leri scaffold etmeyi bekleme.

## Ortaklık ve API sahiplik kararı

| Mekanizma / politika | Karar | Somut sınır |
|---|---|---|
| Qt Button state, Action, accessibility, BusyIndicator/Binding, QML lifecycle | **USE EXISTING QT** | Yeni Button/enable guard/state framework'ü yok; loading iş politikası uygulamada |
| ColorScheme, motion/contrast hints, QWindow/renderer/screen-DPR | **USE EXISTING QT** | Qt'nin public bilgisi doğrudan; Window scope'u global primary-screen modeline indirilmez |
| Fluent color alias, type ramp, geometry, density/touch eşlemesi, state precedence | **KEEP KIT/APP** | Fluent Theme/Style sahibi; ortak `Theme`, `Fluent≈Merce primary` tablosu yok |
| Mica/Acrylic nerede kullanılacak, opak fallback ve Smoke | **KEEP KIT/APP** | Kit görünümü; modal/route/iş kararı uygulamada |
| Public DWM/Composition handle lifecycle bridge | **DEFER; bugün kit/proje** | Windows-only aday olabilir. Aynı mekanik sözleşme, Qt açığı, gerçek tüketici, cleanup/fallback ve source/installed proof olmadan `Qavsi.Platform.Materials` kabulü yok |
| Qt'de olmayan Windows transparency/battery fact | **DEFER; bugün kit/proje** | Native yol eklenirse dar ihtiyaç; genel Platform abstraction için gerekçe değil. Native controller politikası varsa onu yeniden yazma |
| Font/icon paketleme ve lisans | **KEEP KIT/PROVIDER; Qt altyapısını kullan** | Asset seçimi/versiyonu/NOTICE ayrı; ilk dilime Segoe/Cascadia/ikon arşivi eklenmez |
| Token değişim/alias doğrulama aracı | **DEFER** | Gerçek ortak build-time ihtiyaç kanıtlanırsa; görsel semantic şema veya Runtime zorunluluğu olmaz |
| Qt/CMake source/installed metadata ve static plugin mekanizması | **USE EXISTING QT/CMAKE** | 06'nın kanıtı tekrar kullanılabilir; yeni paket yöneticisi/registry motoru yok |

**QUALIFIES QAVSI: bu incelemede de boş.** Public Windows API standardı olması onu kitler arasında ortak görünüm veya gerekli Qavsi wrapper'ı yapmaz. Dar Windows bridge adayının birden fazla benzer kod kullanması da [ortaklık kapısını](../docs/SHARED-ADMISSION.md) tek başına geçirmez.

## Gerçekte yapılan doğrulama ve kalan sınırlar

| Kontrol | Sonuç / kanıt | İddia edilmeyen |
|---|---|---|
| Sabit üçüncü taraf snapshot hash'i | **PASS**, [Yerel arşiv: diagnostic](evidence/README.md#review-03), [Yerel arşiv: sonuç](evidence/README.md#review-03) | Katalog tam/yetkili platform spec'i değil |
| Kontrast hesabı self-check'leri | **PASS**; black/white=21, eşit renk=1, simetri, field count, dört pair ve negatif alpha örnekleri assert'li | Gerçek QML metni/render/OS materyali ölçülmedi |
| Birincil kaynak incelemesi | **SOURCE CHECK**; [Yerel arşiv: 55 URL/dosya kaydı](evidence/README.md#review-03), commit/hash ve tmp cache | Upstream kodu Qt'ye port edilmedi; npm tarball/supply-chain kabulü değil |
| StyleKit mapping/lifecycle mekanizması | **05 yeniden kullanıldı**; sabit rol + A/B/A, light/dark, native/public reader; 399 assertion ve iki-engine için 8 assertion | Gerçek Fluent adaptörü, atomic frame, popup, aktif host teardown veya Windows değil |
| Dynamic instance variation | **05'te FAIL — bilinen Qt regression** | Sabit rol proof'u bu hatayı çözmüyor |
| Source/installed/relocated/shared/static-QML paketleme | **06 yeniden kullanıldı**; generic probe target/URI/resource/lint kanıtı | Fluent paketi, tam static Qt, Windows DLL deploy veya font paketi değil |
| Gerçek Fluent build/test/installed consumer | **NOT-RUN** | Yeni Fluent repo/kod/paket henüz yok |
| Windows native Mica/Acrylic/DWM/Composition, active/inactive, transparency/battery/high-contrast | **NOT-RUN** | Windows machine/toolchain/Qt render lane yok |
| Gerçek UI/screenshot, DPI 1x/2x/fractional, text scaling, RTL/IME, focus/reader/input | **NOT-RUN** | CPU kontrast script'i bu testlerin yerine geçmez |
| GPU/software Fluent performansı, Linux/EGLFS/macOS/WASM hedefleri | **NOT-RUN** | Başka alanın offscreen proof'u Fluent desteği ilan etmez |
| Font/ikon binary dağıtım ve bütün asset lisans kabulü | **NOT-RUN** | Font/ikon binary eklenmedi; assets PDF'inin linki cache'lendi, şartları ayrı hukuki kabul olarak incelenmedi |

Tek diagnostik tekrar çalıştırma:

```bash
rtk proxy python3 /private/tmp/qavsi-review-03/diagnostic.py MERCE_SOURCE/qavsi-design/references/fluent/DESIGN.md
```

Geçici klasör yoksa [Yerel arşiv: diagnostic.py](evidence/README.md#review-03) aynı tmp konumuna kopyalanır; sonuç script'in yanında `diagnostic-result.json` olur. Komut exit `0`; yeni bağımlılık, Qt proses/pencere/foreground işlemi veya ürün build'i yok. Tüm shell komutları `rtk` ile ve `login:false` çalıştırıldı. İlk sandbox içi public ağ okuması DNS kısıtına takıldı; aynı salt-okuma fetch'i onaylı ortamda tmp cache'e başarıyla tamamlandı. Otomatik approval reddi yok.

**Atlanan:** Gerçek Fluent implementasyonu/build'i, Windows/native materyal, UI/input/ekran okuyucu, GPU/performance ve ürün deployment testleri yapılmadı.
**Risk:** Kaynak/kontrast ve 05/06 mekanizma kanıtları, henüz var olmayan Fluent kitinin veya native Windows materyalinin kabul sonucu sayılamaz.
