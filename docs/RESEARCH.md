# Araştırma, kaynaklar ve kanıt sınırları

**İnceleme tarihi:** 2026-10-23–24.\
**Yöntem:** GitHub `gh` CLI kontrolü, sabitlenmiş resmi kaynaklar, üç mevcut DESIGN.md girdisi ve altı bağımsız alan incelemesi. Qt-only davranış ve CMake dağıtım proof’ları ayrıca çalıştırıldı; sonuçları kaynak incelemesinden ayrı kaydedildi. KDE/Quickshell veya yeni tasarım kitleri build edilmedi; ürüne referans implementasyon kopyalanmadı.

Bu dosyada **kaynak gerçeği**, **Qavsi önerisi** ve **henüz test edilmemiş hedef** ayrı tutulur. Ana belgedeki yeni modül/target/API adları uygulanmış bir SDK’nın belgesi değildir.

## 1. Sabitlenen kaynak snapshot’ları

| Kaynak | İncelenen ref / commit | Tarih |
|---|---|---|
| Qt qtdeclarative | `v6.12.0` → `4332fbbb4c340463dc8cee36dda0c85d932122d8` | Qt 6.12.0 tag’i |
| Qt qtbase | `v6.12.0` → `025bdad181de241e81bf853c8a2d7bf3d19261a9` | QIcon / QWindow / public API sınırı |
| KDE Kirigami | `f0344dc7e40748564a1edec29cab404bfd879530` | 2026-10-23T14:34:53Z |
| Quickshell | `9b0f339b19057f4731bf3b48293307d7ef241671` | 2026-10-21T23:48:02-07:00 |
| DTCG format | Yayımlanmış `2025.10` sürümü | 2025-10-28 Final Community Group Report |

Qt’nin annotated `v6.12.0` tag object’i `c3ab6d3094e8b843c6fc3e04ed3e0f40d4092447`; yukarıdaki değer onun işaret ettiği commit’tir. Hareketli `dev` branch’i mimari sözleşmenin tek sürüm referansı yapılmadı.

KDE için resmi GitHub mirror’ı [KDE/kirigami](https://github.com/KDE/kirigami), Quickshell için projenin işaret ettiği [GitHub mirror](https://github.com/quickshell-mirror/quickshell) kullanıldı. Quickshell upstream bağlantısı [git.outfoxxed.me](https://git.outfoxxed.me/quickshell/quickshell) üzerinden de izlenebilir.

## 2. KDE / Kirigami: ne öğrendik?

| İncelenen kaynak | Kaynakta görülen gerçek | Qavsi’ye çıkarım |
|---|---|---|
| [src/CMakeLists.txt](https://github.com/KDE/kirigami/blob/f0344dc7e40748564a1edec29cab404bfd879530/src/CMakeLists.txt) | Platform, layout, primitive/template/control/delegate/dialog/form alanları ayrı düzenleniyor | Namespace ve dosya sınırı sorumluluğa göre olmalı |
| [platform/CMakeLists.txt](https://github.com/KDE/kirigami/blob/f0344dc7e40748564a1edec29cab404bfd879530/src/platform/CMakeLists.txt) | KirigamiPlatform ayrı paket/target; install tüketimi ele alınıyor; GuiPrivate bağımlılığı var | Ayrı paket sınırını örnek al; private Qt bağımlılığını otomatik kopyalama |
| [platformpluginfactory.h](https://github.com/KDE/kirigami/blob/f0344dc7e40748564a1edec29cab404bfd879530/src/platform/platformpluginfactory.h) | Platform theme/units üretimi ve plugin discovery için dar factory var | Platform politikası ile uygulama UI’sini ayır; henüz gerekmeyen factory sistemini taşıma |
| [platformtheme.h](https://github.com/KDE/kirigami/blob/f0344dc7e40748564a1edec29cab404bfd879530/src/platform/platformtheme.h) | Theme’ın color set/group ve inheritance modeli açık | Kapsam/inheritance belgelensin; bu Theme Qavsi’nin evrensel modeli yapılmasın |

KDE Frameworks’ten alınan ilke, bağımsız kütüphane/tüketici sınırıdır. Bütün KDE Frameworks, KConfig/DBus servisleri veya ECM bu görevde incelenmiş/bağımlılık olarak seçilmiş sayılmaz. Kirigami’nin platform ve tasarım yaklaşımı kendi ürün bağlamına aittir.

Kirigami belgeleri: [API platform modülü](https://api.kde.org/kirigamiplatform-module.html), [geliştirici başlangıcı](https://develop.kde.org/docs/getting-started/kirigami/). Qavsi, Kirigami’nin Qt Quick Controls üzerine geliştirdiği geniş UI framework’ünü tekrar üretmeye çalışmaz.

## 3. Quickshell: ne öğrendik?

| İncelenen kaynak | Kaynakta görülen gerçek | Qavsi’ye çıkarım |
|---|---|---|
| [src/CMakeLists.txt](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/CMakeLists.txt) | Core, io, widgets, window, platform/services ayrı; Wayland/X11/DBus seçenekleri var | Optional modül kapalıyken gerçekten build dışı kalmalı |
| [core/CMakeLists.txt](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/core/CMakeLists.txt) | QML core modülü executable ekosistemine bağlı; QuickPrivate kullanılıyor | Toolkit’in ürün bağlamı ve private bağımlılığı Qavsi’ye taşınmasın |
| [widgets/CMakeLists.txt](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/widgets/CMakeLists.txt) | Widgets için açık namespace/modül ayrımı | Özelliği public URI ve seçilebilir target ile sınırla |
| [generation.cpp](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/core/generation.cpp) | Generation başına yeni QQmlEngine; reload sonrası eski generation yıkılıyor | Engine/state ownership açık olmalı; genel state koruma garantisi verilemez |
| [reload.hpp](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/core/reload.hpp) | Reloadable, eşleme kimliği ve lifecycle hook’ları var | Reload davranışı kendiliğinden oluşmaz; ayrı sözleşmedir |
| [rootwrapper.cpp](https://github.com/quickshell-mirror/quickshell/blob/9b0f339b19057f4731bf3b48293307d7ef241671/src/core/rootwrapper.cpp) | Aday generation yükleme ve başarısızlığı ele alma var | Geçersiz yeni aday çalışır durumdaki UI’yi bozmamalı |

Quickshell’in [resmi tanıtımı](https://quickshell.outfoxxed.me/docs/about) onu shell toolkit bağlamında sunar. Qavsi’de ayrı bir launcher/engine manager, Linux shell servis katmanı veya Quickshell uyumluluğu gerekmiyor. Öğrenilen ilke modülerlik ve açık lifecycle; reload implementasyonu kopyalanmıyor.

## 4. Qt 6.12.0 StyleKit — kritik kaynak doğrulaması

### 4.1 Stil scope’u engine değil process

[qqstylekit.cpp satır 91–100](https://github.com/qt/qtdeclarative/blob/4332fbbb4c340463dc8cee36dda0c85d932122d8/src/labs/stylekit/qqstylekit.cpp#L91) static `s_instance`’ı döndürüyor; nesnenin parent’ı QGuiApplication. İlk engine kaydı tutulsa da sonraki engine için yeni attached instance oluşturulmuyor.

**Doğrulanan gerçek:** İki QQmlEngine bağımsız stil scope’u sağlamaz.

Bu artık yalnız kaynak bulgusu değil: [05 proof’u](../reviews/05-stylekit-runtime.md) iki engine’de birinin Style değişiminin diğerini etkilediğini 8 assertion ile doğruladı. Aynı rapordaki ayrı 27 snapshot / 399 assertion, sabit role sahip tek SK.Button’ın A→B→A state/theme geçişini sınadı. Gerçek kit adapter ve typography doğrulaması değildir.

**Önerimiz:** Gallery eşzamanlı kit karşılaştırmasını ayrı süreçlerle yapar. Bu, Qt kaynaklarından çıkan mimari sonuçtur; Gallery’de henüz uygulanmış bir worker testi değildir.

### 4.2 Style nesnesi sahipliği

[Aynı kaynağın setter/loader bölümü](https://github.com/qt/qtdeclarative/blob/4332fbbb4c340463dc8cee36dda0c85d932122d8/src/labs/stylekit/qqstylekit.cpp#L126) doğrudan nesne atamasıyla URL’den üretilen nesne sahipliğini ayrı ele alıyor.

**Önerimiz:** App-owned Style instance’larıyla tek atama yolu. `style` / `styleUrl` karışımı ve `nullptr` ile reset kullanımına dayalı protokol yok. Window/engine yıkıp açmak stil izolasyon çözümü değildir.

### 4.3 Public QML / private C++ ayrımı

StyleKit’in public QML tipi kullanılır; kaynağının C++ implementation header’larının private olması bu header’ları export etme izni vermez. Labs API değişim riski sürüm matrisine yazılır. [Qt StyleKit](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-stylekit.html)

Qt quick controls’ün normal style seçimi, StyleKit’in uygulama içi Style nesnesi ve kitin Theme config’i üç ayrı katmandır. C++ Qt Quick Controls style ayarını her sayfada değiştiren çözüm önerilmez.

## 5. Public API karşılaştırma notları

| Konu | Doğrulanan sınır | Mimariye etkisi |
|---|---|---|
| [Style](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-style.html) | Theme/fallback ve kontrol family eşlemeleri var | Fallback’in varlığı tam kit coverage değildir |
| [StyleVariation](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-stylevariation.html) | İlk variation önceliği; descendant etkisi; bilinmeyen adlar yok sayılır | Kit role çakışmalarını ve helper popup etkisini test eder |
| [ControlStyle](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-controlstyle.html) | Style descriptor geometri alanları kontrol mapping’ine aittir | Normal QML implicitWidth/Height ile karıştırılmaz |
| [DelegateStyle](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-delegatestyle.html) | Public slot delegate’i ve QObject data payload’ı var | Background delegate arbitrary contentItem hook’u değildir; data alan bazında merge edilmez |
| [StyleReader](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-stylereader.html) | Aktif stilin çözümlenmiş değerlerini okur; state girdileri ve font çıktısı ayrıdır | Ortak içerik değerleri tüketir; reader cache’ine yazarak Style değiştirmez |
| [Binding](https://doc.qt.io/qt-6/qml-qtqml-binding.html) | Koşullu override sonrası önceki binding/value geri yüklenebilir | Loading guard eski bool’u saklayıp true yazmaz |
| [Attached properties](https://doc.qt.io/qt-6/qtqml-cppintegration-definetypes.html#providing-attached-properties) | Hedef nesneyle bağlı ayrı support nesnesi üretilebilir | Loading state buton başına; otomatik davranış scope’u açık |
| [Accessible](https://doc.qt.io/qt-6/qml-qtquick-accessible.html) | labelFor/labelledBy ve announce gibi public yollar var | Label ilişkisi açık kurulur; hayali Accessible.busy property’si kullanılmaz |
| [QAccessibilityHints](https://doc.qt.io/qt-6/qaccessibilityhints.html) | motionPreference 6.12; contrastPreference 6.10; transparency alanı yok | Qt public hints önce, reduced transparency için gerekli OS köprüsü ayrıca |
| [Software adaptation](https://doc.qt.io/qt-6/qtquick-visualcanvas-adaptations-software.html) | ShaderEffect software’da render edilmez | Gerçek shadersız fallback gerekir |
| [Shape](https://doc.qt.io/qt-6/qml-qtquick-shapes-shape.html) | Renderer seçenekleri ve destek koşulları var | CurveRendering adı her backend’de aynı performans/destek garantisi değildir |
| [QML module](https://doc.qt.io/qt-6/qt-add-qml-module.html) | Backing/plugin, registration/resource metadata kuralları var | Public target ve import deployment birlikte test edilir |
| [Qt Menu](https://doc.qt.io/qt-6/qml-qtquick-controls-menu.html) | Native popup yolu mevcut; native çizimde custom QML delegate kısıtları var | Önce Qt public native menu; form ComboBox ayrı |
| [QWindow](https://doc.qt.io/qt-6/qwindow.html#startSystemMove) | Public sistem taşıma/resize yolları var | Özel event injection veya zorunlu QWindowKit temel çözüm olmaz |
| [QML Preview](https://doc.qt.io/qt-6/qtqml-tooling-qmlpreview.html) | Debug uygulamada QML/JS geliştirme akışı | Ürün runtime’ı veya tam C++ hot reload garantisi değil |

Bu tablo doküman ve kaynak incelemesidir. Bütün hedef kombinasyonlarının çalıştırıldığı anlamına gelmez.

## 6. Apple, token biçimi ve dokümantasyon yaklaşımı

- [Apple materials HIG](https://developer.apple.com/design/human-interface-guidelines/materials): glass kullanımını katman ve okunaklılık bağlamında değerlendiriyoruz. Her içerik yüzeyini aynı saydam malzemeye dönüştürme sonucu çıkarmıyoruz.
- [NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview): native public mekanizma ayrı OS availability ve native view entegrasyonu gerektirir. Bir instance’ın yaratılması doğru composition/performance kanıtı değildir.
- [DTCG 2025.10 format](https://www.designtokens.org/TR/2025.10/format/): yayımlanmış değişim biçimi referansıdır; tasarım sistemlerini aynı semantic token isimlerine zorlayan bir standart değildir. W3C Recommendation olarak sunulmaz.
- [arc42](https://arc42.org/overview/): ana belge 12 görünüm yaklaşımından uyarlanır. Bu bir uyumluluk sertifikası veya tüm arc42 araç zincirini uyguladığımız iddiası değildir.

Material3 için [01 incelemesi](../reviews/01-material3.md) MDC-Android 1.13.0 commit `84c0e6d6d493fcf176d2099683245b78afbeb78d` ile ayrı Compose Material3 1.5.0-beta02 kaynaklarını sürüm/platform ayrımını koruyarak karşılaştırdı. Snapshot içindeki farklı token nesilleri tek edisyon gibi birleştirilmedi. [03 Fluent incelemesi](../reviews/03-fluent.md) Microsoft fluentui commit `6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676` ve resmi Windows/Web belgelerini ayırdı. Her rapor kaynak digest’lerini kendi provenance kaydında tutar.

QmlMaterial zorunlu binary/source dependency seçilmedi; mevcut karşılaştırma deneylerinin görsel/davranış referansı olarak kalır. Bu görev QmlMaterial’ın yeni commit’lerinin feature audit’i değildir.

İlk altı incelemenin DESIGN.md girdileri geçici designmd.app snapshot’larıydı. VoltAgent `3b605329240ce735b510037cca322f1831fdd7f8` envanterindeki Apple dosyası web analizi çıktı; Material3/Fluent ayrı dosyası yoktu. Kullanıcı son kararıyla Google Material 3 ve Microsoft Fluent 2/Windows rehberlerini esas almayı, Apple native Liquid Glass hedefini HIG/public AppKit ile korumayı kabul etti. Üç kitin DESIGN.md sözleşmesi güncellendi; VoltAgent yalnız yardımcı referanstır. [Resmi kaynak kayıtları](../references/official/sources.json), [güncel/tarihsel atıflar](../references/SOURCES.md). Kaynak içindeki ajan talimatları veri sayılır.

Bu güncellemede 13 resmi kaynak URL’si okunup hash/tarih kaydı alındı. M3 public site extraction’ı JavaScript gerektirdi; sayısal recipe için sabit resmi Google kaynakları kullanıldı. Apple’ın Markdown uçları web aracı tarafından content-type nedeniyle okunamadığında public URL’ler salt-okuma indirmeyle doğrulandı. Tam upstream prose deliverable’a kopyalanmadı. Bu kaynak incelemesi yeni ürün build/render/native testi değildir.

## 7. Mevcut Merce kaynak incelemesinin sonucu

İncelenen worktree’de:

- Material3 CMake/QML kodu MerceStyle, Theme, Effects, Foundation ve Platform’a bağlı.
- Foundation aynı target’ta AppLabel/Surface ile icon provider/Material font/native SF altyapısını paketliyor.
- Merce AppLabel ve AppIcon default’ları Merce Theme modelinden geliyor.
- Platform native materyal ve accessibility kodu içeriyor; capability modelinde primary-screen yaklaşımı var.
- Window integration ve kit politikası ayrı dosyalara çıkmış olsa da mevcut kit kalıtımı bağımsız sistem hedefini tamamlamıyor.
- MButton ve ButtonSupport deneyi aynı statüde değil: MButton mevcut tüketici yüzeyi, support deneysel.
- Merce Notifications source-only; QtToastify entegrasyonu installed core’un zorunlu parçası değil.
- Material3 actions family’sinde theme/variation/delegate ayrımı başlamış; bu düzen bağımsız kitte korunabilir.

Bunlar snapshot incelemesidir. [Geçiş haritası](MIGRATION.md) dosya bazlı hedefi açıklar. Bu bulgular ürün kodunda otomatik düzeltildi diye yorumlanmamalı.

## 8. GitHub kurulumu — gerçek durum

`gh api user` ile etkin hesap `tech-alp` doğrulandı. `gh api users/qavsi` 404 ve `gh search repos qavsi` boş sonuç verdi. Bu kontrol, kontrol anında GitHub kullanıcı/organizasyon ad alanının kullanılabilir göründüğünü gösterir.

`gh org` public komutu yalnız listeleme sağlıyor; yeni organizasyon için resmi GitHub web kurulumu gerekiyor. [gh org](https://cli.github.com/manual/gh_org), [GitHub organizasyon oluşturma](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch)

Bu ilk ad uygunluğu kontrolü tarihsel kayıttır. Sonrasında kullanıcı Qavsi organizasyonunu oluşturdu; iki Public repo GitHub API ile doğrulandı ve MIT lisansı ile ilk kaynak aktarımı onaylandı. [Güncel depo kaydı](REPOSITORIES.md).

## 9. Altı alanlı inceleme ve çalıştırılan proof’lar

Sonuçların ortak değerlendirmesi [reviews/SUMMARY.md](../reviews/SUMMARY.md), ayrı görevler ve raporlar [reviews/README.md](../reviews/README.md) içindedir.

| Alan | Kanıtın niteliği | Ne kanıtlamaz? |
|---|---|---|
| Material3 | Google kaynak/token karşılaştırması, mevcut Merce import ve recipe diagnostikleri | Bağımsız Material3 kiti veya exact typography render başarısı |
| Apple | HIG/kaynak ownership incelemesi; public API için compile-only OBJECT probe | Link/deploy, gerçek native cam veya Metal okunaklılığı |
| Fluent | Microsoft Web/Windows kaynak ayrımı ve CPU renk diagnostikleri | Windows/WinUI/Mica/Acrylic çalışma veya native materyal kontrastı |
| Ortak altyapı | Qt-only offscreen/Cocoa ikon, public hints ve image-provider ownership proof’u | İleri SF config, tüm font eksenleri veya ortak Runtime kabulü |
| StyleKit | Aynı instance/state/theme geçişi, global scope ve mevcut loading guard binding’i | Açık popup/IME, bütün content delegate’leri veya atomik kit geçişi |
| Dağıtım | Shared ve STATIC QML modülü; source/installed/moved-prefix consumer + strict tooling | Tam statik Qt, WASM, tüm kit paketleri veya uygulama deployment’ı |

Araştırma checkout’ları ürün dependency’si değildir. Kanıt için yazılan küçük kaynaklar/loglar yerel araştırma arşivinde kalır; public kopyadaki [kanıt kataloğu](../reviews/evidence/README.md) dağıtım sınırını, korunmuş manifestler dış kaynak URL/hash bilgilerini açıklar. Apple/Microsoft tam doküman metinleri araştırma cache’inde kalır; dağıtılacak mimari pakete alınmaz. Asset veya font eklenmedi.

## 10. Yapılmayanlar ve yorum sınırı

Bu teslimatta:

- Mimari metin, kaynak digest’leri ve bağlantı tutarlılığı denetlenir; altı alanın bulguları sözleşmelere işlenir.
- Mevcut Merce build/UI testi tekrar çalıştırılmış sayılmaz; ürün kodu değişmedi.
- Qt-only CMake proof’u bir Qavsi ürün SDK’sının kurulduğu anlamına gelmez. Yeni kit build/deploy, CI lane veya native performans sonucu yoktur.
- SDK patch’i, MButton silinmesi ve Merce kod taşıması yapılmaz.
- GitHub formunu açmak organizasyonun tamamlandığı anlamına gelmez.

Uygulama aşamasında kaynak kanıtı; contract testi, installed consumer ve gerçek cihaz/renderer kanıtıyla tamamlanmalıdır.
