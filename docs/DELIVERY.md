# Qavsi derleme, dağıtım ve doğrulama

**Durum:** Ekosistem teslimat sözleşmesi. [Material3'ün ilk bağımsız dilimi](https://github.com/Qavsi/material3) source/installed tüketimiyle ayrı repoda tutulur; aşağıdaki diğer Runtime/kit örnekleri hedef tüketim biçimidir, yayımlanmış modül listesi değildir.

Runtime target/component adları [ortak yapı kabul kapısına](SHARED-ADMISSION.md) tabi adaylardır. Bu listede görünmek extraction/yayın kararı değildir; doğrudan Qt ile çözülen iş için yeni paket açılmaz.

## 1. Paket sınırları

Bir Git deposu tek ve dev bir link target’ı olmak zorunda değildir. Runtime tek sürümde yayımlanır; tüketici yalnız seçtiği component’leri ister. İlk sürümde ayrı C++ plugin-discovery framework’ü geliştirilmez; Qt’nin QML modül sistemi kullanılır.

| Depo / CMake paketi | Önerilen component / public target | QML URI |
|---|---|---|
| runtime / `QavsiRuntime` | `Icons` / `QavsiRuntime::Icons` | `Qavsi.Icons` |
| runtime / QavsiRuntime | `IconsMaterial` / `QavsiRuntime::IconsMaterial` | `Qavsi.Icons.Material` |
| runtime / QavsiRuntime | `IconsFontAwesome` / `QavsiRuntime::IconsFontAwesome` | `Qavsi.Icons.FontAwesome` |
| runtime / QavsiRuntime | `IconsSFSymbols` / `QavsiRuntime::IconsSFSymbols` | `Qavsi.Icons.SFSymbols` |
| runtime / QavsiRuntime | `ButtonSupport` / `QavsiRuntime::ButtonSupport` | `Qavsi.ButtonSupport` |
| runtime / QavsiRuntime | `Platform` / `QavsiRuntime::Platform` | `Qavsi.Platform` |
| runtime / QavsiRuntime | `PlatformMaterials` / `QavsiRuntime::PlatformMaterials` | `Qavsi.Platform.Materials` |
| runtime / QavsiRuntime | `PlatformMenus` / `QavsiRuntime::PlatformMenus` | `Qavsi.Platform.Menus` |
| material3 / `QavsiMaterial3` | `Theme`, `Style` / `QavsiMaterial3::Theme`, `QavsiMaterial3::Style` | `Qavsi.Material3.Theme`, `Qavsi.Material3.Style` |
| apple / `QavsiApple` | `Theme`, `Style` / `QavsiApple::Theme`, `QavsiApple::Style` | `Qavsi.Apple.Theme`, `Qavsi.Apple.Style` |
| merce / mevcut `Merce` | Mevcut `Merce::*` target’ları korunur | Mevcut `Merce.*` URI’leri korunur |
| gallery | Uygulamalar; tüketiciye library export edilmez | Gallery’ye ait private URI’ler |

`PlatformMaterials` ve `PlatformMenus` isimleri API spike’ında daraltılabilir; ikisi v1’in otomatik zorunlu kapsamı değildir. Desteklenmeyen OS’de bir public component ya bilinen fallback uygulaması sağlar ya configure sırasında açıkça unavailable olarak raporlanır. “Target var ama import anında çöker” kabul edilmez.

Yeni kitin ayrı Runtime modülüne ihtiyacı yoksa boş bir Runtime bağımlılığı eklenmez. Örneğin yalnız temel StyleKit mapping yapan bir kit doğrudan Qt + kendi Theme’ini kullanabilir.

### 1.1 Bağımlılığın teknik anlamı

- Icons Core: Core, Gui, Qml, Quick. StyleKit, Material fontu, Merce Theme veya AppKit gerekmez.
- ButtonSupport: Qt Quick / Controls / StyleKit ve kullandığı Icons Core adaptörü; somut kit yoktur.
- Platform: OS tercihleri ve Qt pencere olguları. Native malzeme kodu ayrı component’te kalır.
- Font sağlayıcısı: yalnız kendi font/metadata veya SVG/native backend bağımlılığı.
- Kit Style: kendi Theme’i, Qt StyleKit ve gerçekten kullandığı Runtime component’leri.
- QWindowKit: gerekirse yalnız isteğe bağlı window-chrome adaptöründe. Icons veya Theme build’i bunu fetch etmez.
- QtToastify: Merce Notifications’ın mevcut bağımlılığıdır. Runtime’ın ortak dependency’si yapılmaz.
- Node/npm: token üreticisini geliştirme aşamasında kullanabilir; normal paket tüketicisine taşınmaz.

Shared veya static Qt seçimi uygulamanın toolchain’iyle tutarlı olur. Aynı process içinde farklı Qt ABI’larına göre üretilmiş kitlerin yüklenmesi desteklenmez.

## 2. Kaynak dizini ve QML metadata

Runtime’da public URI’yle hizalı düzen önerilir:

```text
runtime/
├── CMakeLists.txt
├── cmake/
│   ├── QavsiRuntimeConfig.cmake.in
│   └── component export kuralları
├── src/Qavsi/
│   ├── Icons/
│   │   ├── CMakeLists.txt
│   │   ├── AppIcon.qml
│   │   ├── iconregistry.h/.cpp
│   │   ├── internal/
│   │   ├── Material/
│   │   ├── FontAwesome/
│   │   └── SFSymbols/
│   ├── ButtonSupport/
│   └── Platform/
│       ├── Materials/
│       └── Menus/
├── tests/
│   ├── unit/
│   ├── qml/
│   └── installed-consumer/
└── examples/
```

Bu dosya listesi scaffold üretim talimatı değildir. İlk taşınan modülün gerçekten gerektirdiği dosyalar açılır.

`qt_add_qml_module`; qmldir, qmltypes, registration ve resource metadata’nın sahibidir. Elle qmldir yazarak ikinci doğruluk kaynağı yaratılmaz. Backing target ve plugin ayrımı Qt’nin normal dağıtım modeliyle kurulur; son uygulamanın statik plugin keşfi ayrıca sınanır. [Qt QML modül komutu](https://doc.qt.io/qt-6/qt-add-qml-module.html)

Internal renderer bir dosya olabilir; public QML tipine dönüşmesi gerekmez. `internal/` adı tek başına erişim engeli değildir: QML registration/export metadata’sında private/internal durumu açık tanımlanır. Başka kit internal dosyaya göreli import yapamaz.

Üretilmiş kaynaklar build tree içinde üretilir veya kitin ilan ettiği committed-output politikasıyla saklanır. Normal consumer kaynak ağacına yazmaz. Kaynak ve kurulu consumer aynı resource/URI davranışını alır.

## 3. Tüketici deneyimi

### 3.1 Kurulu paketten tek kit

Aşağıdaki örnek hedef API taslağıdır; bugün `find_package(QavsiMaterial3)` çalışıyor anlamına gelmez.

```cmake
cmake_minimum_required(VERSION 3.30)
project(MyApp LANGUAGES CXX)

find_package(Qt6 6.12 REQUIRED COMPONENTS Quick Qml QuickControls2 LabsStyleKit)
qt_standard_project_setup(REQUIRES 6.12)

find_package(QavsiMaterial3 0.1 REQUIRED COMPONENTS Style)

qt_add_executable(myapp main.cpp)
qt_add_qml_module(myapp
    URI MyApp
    QML_FILES Main.qml
    IMPORT_PATH "${QavsiMaterial3_QML_IMPORT_DIR}"
)

target_compile_features(myapp PRIVATE cxx_std_20)
target_link_libraries(myapp PRIVATE
    Qt6::Quick
    QavsiMaterial3::Style
)
```

Bu minimal consumer text-only veya Qt’nin native image/icon yolu ile başlar; Runtime, ButtonSupport veya özel font sağlayıcısı istemez. Kit Style kendi Theme target’ını doğru transitif bağlar; uygulama dependency zincirini tahmin etmez. İleride kabul edilmiş bir optional provider/destek kullanılırsa ilgili public import ve target son uygulamada ayrıca bulunur. Statik build’de bu görünür link/import ilişkisinin plugin taramasına yetmesi consumer test’iyle doğrulanır.

Örnekteki paket bazlı QML_IMPORT_DIR değişkenleri önerilen public CMake tooling metadata’sıdır; source ve installed paket aynı bilgiyi sağlar. Bu metadata runtime import-path override’ı değildir. Kabul edilmemiş Runtime adayları ilk gerçek kitin zorunlu dependency’si yapılmaz.

QML başlangıcında seçilen Style yalnız composition root’ta atanır. Sayfaların stil adına koşul eklemesi gerekmez; uygulamaya ait büyük yüzeyler için kitin public sunum adaptörü kullanılır. Kontrol StyleKit’i, keyfi `Rectangle` veya özel navigation layout’unu otomatik dönüştürmez.

### 3.2 Kaynaktan geliştirme

`add_subdirectory` ve kurulu paket aynı public target’ları verir. Kit, “önce bütün ekosistemi kur” şartı koymaz.

Önerilen dependency davranışı:

1. Gerekli target zaten varsa onu kullan.
2. Yoksa config-mode `find_package` ile kurulu paketi bul.
3. Yoksa eksik component ve nasıl sağlanacağı açık hata olsun.

Varsayılan configure sırasında internetten otomatik fetch yapılmaz. Uygulama isterse kendi dependency yönetiminde sabit commit ile FetchContent kullanır. Kütüphane parent’ın genel `BUILD_TESTING`, compiler flags, output directory veya dependency cache değişkenlerini zorla değiştirmez.

Mevcut target’ın bulunması sürümünün uygun olduğu anlamına gelmez. Source ve installed origin aynı graph’ta karıştırılmaz; parent’ın sabitlediği commit/version ve gerekli minimum aralık doğrulanır. Sürüm bilgisi olmayan target sessizce uyumlu sayılmaz.

Bir kaynak workspace’inde parent önce gereken Runtime target’larını, sonra kiti ekleyebilir. Her child kendi başına da build edilebilir. Gerçek consumer test’leri her iki yolu kullanır; özel `QML_IMPORT_PATH` ortam değişkenleriyle gizlenen eksik metadata kabul edilmez.

Bir Runtime component’i ancak kabul edilip Merce gerçekten onu kullanırsa açık dependency olur; bugün böyle bir ekleme yok. O koşul oluşursa eski URI/target’ların korunması tek-clone bağımsız build’in de aynen korunacağı sözü değildir. Kırıcı dependency edinme değişikliği mevcut yayımlanmış Merce sürümlerine geriye dönük uygulanmaz; gizli vendoring/otomatik fetch yerine paket metadata’sı veya parent source workspace kullanılır. [Geçiş kararı](MIGRATION.md)

### 3.3 Public C++ yüzeyi

QML tüketimi için gerekmeyen header export edilmez. Provider kaydı public C++ isterse küçük interface ayrı adlandırılır; Qt private sınıflar, objc nesneleri ve kit Theme sınıfları header’a sızmaz.

QObject ownership, thread ve callback ömrü sözleşmenin parçasıdır. Public ABI’si değişebilen 0.x sürümlerde aynı minor serisi test edilir; binary uyumluluk henüz garanti edilmez. Yeni plugin API’si çıkmadan bağımsız üçüncü taraf native plugin ABI’si vaat edilmez.

## 4. Install ve deploy

Her paket aşağıdakileri doğru üretir:

- Config + version dosyası ve component bazlı export target’ları.
- Sadece seçili component’in `find_dependency` çağrıları.
- QML import klasörleri, plugin binary’si, qmltypes ve gerekli resources.
- Kendi lisans/NOTICE ve font provenance dosyaları.
- Yeniden konumlandırılabilir prefix; build makinesinin mutlak source/SDK yolu sızmaz.

Kullanıcının CMAKE_INSTALL_PREFIX’i değiştirilmez. RPATH ve deployment Qt/OS kurallarıyla çözülür; elle bütün makinenin import path’ini değiştirmek çözüm sayılmaz. [Qt Quick deployment](https://doc.qt.io/qt-6/qtquick-deployment.html)

Install test’i yalnız library link’i değildir: ayrı klasörde consumer configure/build edilir, source ağacı ve build output erişimi kaldırılır, temiz environment ile uygulama başlatılır. Mümkün olduğunda prefix başka dizine taşınıp aynı kontrol tekrarlanır. macOS app bundle, Windows DLL/plugin yerleşimi ve Linux rpath ayrı dağıtım konularıdır.

QML tooling import-root’u da public CMake metadata’sıdır; yalnız target export’u bunu otomatik sağlamayabilir. Source ve installed yolları URI, target ve qmllint/qmltypes davranışı bakımından birlikte sınanır. Runtime ve build geçerken tooling import’unun başarısız kaldığı negatif kontrol [06 raporunda](../reviews/06-delivery.md) kayıtlıdır.

### 4.1 Statik QML modülü, tooling ve WebAssembly

[06 proof’u](../reviews/06-delivery.md) shared Qt 6.12.0 üzerinde hem shared hem STATIC QML modülünü source, installed ve taşınmış prefix ile çalıştırdı. Bu, tam statik Qt veya WASM başarısı değildir.

İlk gerçek paket şu ayrıntıları birlikte sağlamalı:

- Backing **ve plugin** target’ı aynı source/installed public adlarıyla export edilir.
- `qt_add_qml_module(OUTPUT_TARGETS ...)` yardımcı target’ları ve gereken object dosyaları da export/install edilir.
- Generated `qmldir` içindeki `linktarget`, installed consumer’ın gerçekten gördüğü CMake target’ıyla eşleşir. Proof’ta namespaced export için normal CMake alias’ı yeterli oldu; qmldir elle düzenlenmedi.
- Paket relocatable **CMake QML import-root metadata’sı** verir. Consumer bunu `qt_add_qml_module(IMPORT_PATH ...)` üzerinden kullanır; runtime environment path hilesi yapılmaz.
- Plugin scan/finalization yalnız seçili modülün plugin’ini bağlar; bütün plugin’ler körlemesine link edilmez.
- Runtime import ve resource smoke’u **ayrıca strict qmllint** ile tamamlanır. Varsayılan all_qmllint exit=0 import uyarılarını saklayabildi; `qmllint --max-warnings 0` negatif kontrolü yakaladı.

Yerel 6.12.0 SDK ile komut belgesinin statik plugin davranışı arasında fark gözlendi; source/import/export metadata ve executable kanıtı test edilen patch’e bağlanır. Tek başına `qt_import_qml_plugins` çağrısını veya library link’ini “statik dağıtım tamam” saymayız.

WASM native OS materyali, SF Symbols ve desktop chrome sağlamaz. Bu modüller kapalıyken desteklenen kit QML fallback’iyle açılmalıdır. Ağdan font indirme varsayılanı yoktur; gerekliyse app bunu açık yönetir. WASM smoke test gerçek browser’da import, font, input ve popup etkileşimini kapsar. Masaüstü screenshot’ı WASM doğrulaması sayılmaz.


### 4.2 Offline ve embedded

Gerekli kaynaklar ve Qt toolchain sağlandıktan sonra configure/build çevrimdışı yapılabilir. Fonts/provider metadata yalnız seçili paketlerdedir. Linux EGLFS/QPA ortamında desktop blur veya sistem menüsü olmadığı açıkça raporlanır.

EGLFS ve software renderer aynı kavram değildir. Embedded hedefte RHI/GPU çalışabilir; software testi ayrı koşudur. Touch target, fiziksel panel/DPI kalibrasyonu, IME ve font shaping gerçek cihazda doğrulanır.

## 5. Sürüm ve uyumluluk

Runtime, her kit ve Gallery ayrı SemVer etiketi alır. Ortak release train zorunlu değildir. Başlangıçta 0.x sürüm kullanılır; public API kararlılık beyanı özellik bazında yapılır.

0.x için SameMajorVersion bütün 0.y sürümlerini uyumlu ilan edecek şekilde kopyalanmaz. Patch uyumu gerçekten taahhüt ediliyorsa SameMinorVersion; aksi halde exact pin seçilir. Proof’taki CMake version seçimi gerçek kitler arası ABI uyumu yerine geçmez.

Compatibility kaydı en az şu alanları tutar:

| Alan | Amaç |
|---|---|
| Runtime sürümü / commit | Kullanılan ortak sözleşme |
| Kit sürümü / commit | Görsel ve public API davranışı |
| Qt patch + toolchain | Labs API ve ABI sınırı |
| OS + QPA + renderer | Gerçek çalışma yolu |
| Build shared/static | Plugin ve resource doğruluğu |
| Provider sürümü | Font/catalog/asset tekrar üretilebilirliği |
| Test raporu | Hangi kombinasyonun gerçekten geçtiği |

Minimum Qt 6.12, “gelecek tüm Qt sürümlerinde doğrulandı” demek değildir. CI önce 6.12.0 ve seçilen desteklenen patch’i sınar; daha yeni minor deneysel lane olur. Private karantina varsa sürüm aralığı dar ve hata açık tutulur.

Qt Labs API yükseltmesi bağımsız migration işidir. Sağlayıcının iç iyileştirmesi patch olabilir; visible token değişimi veya geometry farkı screenshot baseline ve CHANGELOG gerektirir. Public URI/target, provider adlandırması, payload biçimi ve documented variation davranışı compatibility yüzeyidir.

Yeni public API deprecation süresi yayımlanmadan geriye uyumluluk sözü verilmez. Merce’nin mevcut API’si yeni ekosisteme taşınıyor diye aniden silinmez.

## 6. CI ve kalite kapıları

### 6.1 Hızlı PR hattı

- Configure/build; QML import/type diagnostics.
- Strict QML diagnostics: import-root/type uyarıları sıfır; yalnız all_qmllint exit code’u yeterli değil.
- C++ unit ve anlamlı QML contract test’leri.
- Seçili modül kombinasyonları: minimum core, tipik kit, tüm opsiyoneller.
- Offscreen software smoke + davranış test’leri.
- Token değiştiyse yalnız ilgili kitin generate/validate/check adımları.
- Source consumer ve installed consumer.
- Public API / kaynak bağımlılık kontrolü: Runtime kit import etmez, Material3 Merce’ye link etmez.
- Yeni veya değişen contract’ın regression testi.

Test sayısını artırmak hedef değildir. Uygulama kodunu kopyalayan test, sabit color literal karşılaştırması veya reversible doküman değişikliğine ayrı ürün test’i yazılmaz.

### 6.2 Platform / yayın hattı

| Lane | En az doğrulanacak konu |
|---|---|
| macOS Metal | Kit render, gerçek native cam, popup/input, multi-screen |
| macOS software | Opak fallback, loading, font/ikon, focus |
| Linux Wayland | Popup placement, keyboard, screen/DPR, olmayan materyal fallback’i |
| Linux X11 | Aynı public pencere/menü sözleşmesi |
| Windows | Import/deploy, font/IME, DPI, native olmayan fallback |
| Static consumer | Plugin discovery, resource ve provider kaydı |
| WASM | Browser’da asset/font/input ve opsiyonel native modüller kapalı |
| Embedded hedef | Belirlenmiş cihazda EGLFS/touch/performance |

Bu hedef matristir; tüm lane’ler bugün çalıştırılmış değildir. Üzerinde runner/cihaz olmayan satır “not tested” kalır. Offscreen başarı, gerçek AppKit/Metal veya klavye/IME doğrulamasının yerine geçmez.

GitHub Actions sürümleri immutable commit SHA’ya sabitlenir. Secret ve workflow izinleri minimum tutulur. Dışarıdan gelen PR kodu ayrı izin sınırında çalıştırılır; repo içindeki bir QML örneğinin güvenli veri dosyası olduğu varsayılmaz.

### 6.3 Davranış test senaryoları

`ARCHITECTURE` içindeki Q-01…Q-14 bunlara bağlanır:

1. Button loading başta true / sonradan true; izin binding’i loading sırasında değişir; bitişte doğru restore olur.
2. Mouse, Space/Enter ve Action tekrar tetiklemesi; guard’ın sınırı açık test edilir.
3. Aynı kontrol kimliğiyle kit/theme değişimi; focus, checked, text ve model değeri korunur.
4. Runtime variation ekleme/kaldırma; fresh-instance pozitif kontrol; Qt bug’ı ayrı raporlanır.
5. Popup selected ve hover bağımsız; popup dışına çıkış, ilk/son satır, uzun metin, RTL.
6. Floating label tekliği; accessible ilişki; Standard/Glass dış label restore.
7. Eksik provider/ikon, import sırası, disabled provider ve engine yıkımı.
8. Reduce Motion / Reduce Transparency / Increase Contrast gerçekten değişince uyum.
9. Native materyal aktifken scroll/clip/hide/minimize/restore, DPR/window değişimi ve cleanup.
10. Dış font/palette/padding binding’i; text scaling ve uzun/çevirilmiş metin.
11. Birden çok QQmlEngine’in farklı StyleKit izolasyonu sağlamadığını gösteren Qt-only regression.
12. Shared Action sahibi uygulamanın loading duplicate-start kontrolü.

Qt bug’ında expected-failure sonucu kitin destek beyanına dönüşmez. Yakın bir property veya başka kontrol geçmiş olsa da mevcut Button mutation testi ayrıca çalışır.

### 6.4 Görsel doğrulama

Her kit kendi DESIGN.md ve referans sürümüyle karşılaştırılır. QmlMaterial veya Apple ekran görüntüsü sadece görsel referanstır; pixel-identical native font/system material garantisi verilmez.

Sabit senaryolar: light/dark, normal/hover/pressed/focus/disabled/checked/loading/error; 1x/2x DPR; LTR/RTL; normal/büyük metin; GPU/software; OS azaltma tercihleri.

Native cam için en az açık, koyu ve ayrıntılı gerçek masaüstü arka planı kullanılır. Ekran görüntüsü yanında durum diagnostikleri kaydedilir; “NSGlassEffectView oluştu” tek başına doğru çıktı değildir.

İnsanla çalışma yalnız sol monitörde yapılır. Ekran koordinatı sabitlenmez; ekran geometrisi her oturum okunur. Sağ monitörde foreground veya imleç kontrolü yapılmaz. İnsan fiziksel resize testini üstlenmişse otomasyonla tekrar ele geçirilmez.

## 7. Performans protokolü

Önce standart Qt kontrol sahnesi ve kitin normal fallback sahnesi ölçülür. Native cam, animasyon ve çoklu yüzey maliyeti bunlara göre değerlendirilir.

**Ölçüm seti:**

- İlk pencere/ilk kullanılabilir frame süresi; font/provider eager yükleme maliyeti.
- Idle CPU, animasyon sırasında CPU/GPU, frame-time p50/p95 ve takılma sayısı.
- RSS/peak memory, native view ve QObject sayısı; tekrar aç/kapa sonrası büyüme.
- 1, 4 ve 12 cam yüzey; 1, 25 ve 100 görünür loading/ikon örneği, ürün gereksinimine göre.
- Aktif, inactive, minimized ve görünmez durumlarda animasyon/observer davranışı.
- Provider katalog listeleme ve sanallaştırılmış SF Symbols gezinmesi.

60 Hz için 16.67 ms frame bütçesi yalnız başlangıç hedefidir. Kabul eşiği seçilen cihazda referans sahne ve ürün yüküyle belirlenir; bu belge performans sonucu üretmez. Debug QML profiler kaydı ve Release gerçek cihaz ölçümü ayrı tutulur. Kaynak tüketimi için uydurma sabit MB hedefi konmaz.

Native malzemede masaüstü arka planı, OS sürümü, pencere aktifliği, ekran/DPR ve surface sayısı rapora yazılır. Her surface için desktop screenshot capture veya ayrı sürekli blur pass üretilmediği kontrol edilir.

## 8. QML Preview ve geliştirme araçları

QML Preview MCP, uygun Debug executable’a bağlanır. QML/JS değişimi ve stylesheet/delegate denemeleri hızlı karşılaştırılır. C++ tipi, native adaptör, font registration, resource veya qmltypes değiştiğinde rebuild/restart gerekebilir. [Qt preview sınırı](https://doc.qt.io/qt-6/qtqml-tooling-qmlpreview.html)

Preview tooling ayrı makine aracı olarak kalır; ürünün CMake dependency listesine, install paketine ve zorunlu CI yoluna eklenmez. Debug portu üretim binary’sinde açılmaz. Reload sırasında eski native callback/observer kalması leak testinin parçasıdır.

Quickshell’in engine-generation mekanizmasını taşımak yerine public Qt tooling kullanılır. Preview; test, installed-consumer doğrulaması veya gerçek performans ölçümünün yerine geçmez.

## 9. Yayın kontrol listesi

Bir modülün ilk sürümü ancak şu bilgilerle yayımlanır: scope, public API, destek matrisi, kaynak/installed kullanım, fallback davranışı, bilinen Qt sınırları, CHANGELOG ve lisans/provenance envanteri.

Mimari dokümanlar ve ilk bağımsız Material3 kiti kullanıcı kararıyla MIT lisanslıdır. Mevcut Merce MIT lisansı korunur; üçüncü taraf Apple/font asset dağıtım şartları veya gelecekteki Runtime kapsamı otomatik kabul edilmez. Bu belge hukuki uygunluk veya bütün üçüncü taraf asset’lerin yeniden dağıtım izni beyanı değildir.
