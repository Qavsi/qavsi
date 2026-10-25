# 06 — Qt 6.12 / CMake paketleme ve dağıtım incelemesi

**Karar: CONDITIONAL.** Qt'nin QML modülü ve CMake config/export modeli bu tasarımı taşıyabiliyor. Küçük proof'ta shared ve statik QML modülü; source consumer, ayrı installed consumer ve taşınmış prefix çalıştı. Ancak public backing target'ın varlığı tek başına QML tooling veya statik plugin keşfi garantisi değil. Aşağıdaki iki P1 teslimat ayrıntısı ilk gerçek pakette uygulanmalı.

**Tarih:** 2026-10-23. **Ortaklaştırma kararı: USE EXISTING QT.** Bu sonuç yeni bir Runtime API'sinin gerekliliğini veya katalogdaki aday modüllerin kabulünü kanıtlamaz. Genel plugin registry/loader framework'ü gerekmiyor; Qt modül metadata'sı ve normal CMake paketleme yeterli. [Ortak kabul sözleşmesi](../docs/SHARED-ADMISSION.md) geçerli.

## Kapsam ve kanıt sınırı

İncelenen tasarım: [ARCHITECTURE](../ARCHITECTURE.md), [DELIVERY](../docs/DELIVERY.md), [MIGRATION](../docs/MIGRATION.md), [ADR-007](../docs/DECISIONS.md). Mevcut Merce `CMakeLists.txt`, `cmake/InstallMerce.cmake`, `MerceConfig.cmake`, `tests/source-consumer` ve `tests/package-consumer` salt okunur referanstı; gerçek ürün derlemesi yapılmadı.

Proof yalnız `/private/tmp/qavsi-review-06/` altında üretildi. Adı `QavsiDeliveryProof`, test sürümü `0.1.3`; yayımlanmış Qavsi paketi değil. Tek QML modülü `Qavsi.Review.Probe`: bir C++ `Mechanism`, bir QML `Probe` ve küçük metin resource'u içeriyor. Theme, ikon kataloğu, StyleKit delegate'i, Merce kodu veya native platform adaptörü içermiyor.

Ortam: Qt **6.12.0**, macOS **26.5.2**, arm64, AppleClang **21.0.0**, CMake **4.4.3**, Ninja **1.13.2**, Release. Kaynaklar `cmake_minimum_required(VERSION 3.30)`, `find_package(Qt6 6.12 ...)`, `qt_standard_project_setup(REQUIRES 6.12)` ve public `cxx_std_20` kullanıyor. Consumer hem public compile feature'ı hem `__cplusplus >= 202002L` koşulunu denetliyor. **Tam CMake 3.30 binary'siyle çalışma NOT-RUN**; kurulu 4.4.3 kullanıldı. [Yerel arşiv: Ortam kaydı](evidence/README.md#review-06)

Runtime smoke'larında `QT_QPA_PLATFORM=offscreen`, `QT_QUICK_BACKEND=software` ayarlıydı. `QML*`, `QT_*`, `DYLD_*` ve dependency ortam yolları önce temizlendi. Ön planda pencere veya masaüstü etkileşimi yoktu. Test; import, C++ registration, QML property ve resource içeriğini doğrular; render edilmiş frame/pixel veya native malzeme doğrulaması yapmaz.

## Öncelikli bulgular ve giderimleri

### P1 / DEL-01 — Link ve runtime geçerken QML tooling bozuk kalabiliyor

**Yer:** DELIVERY §2, §3.2, §4, §6.1. İlk installed/moved consumer yalnız backing target'a bağlandığında build ve runtime geçti; `qmllint` ise `Failed to import Qavsi.Review.Probe` ve `Probe was not found` üretti. `all_qmllint` hedefi bu uyarılarla **exit 0** döndü; `qmllint --max-warnings 0` aynı durumda **255** döndü. Yalnız build/exit-code veya yalnız runtime smoke yeterli kapı değil. [Yerel arşiv: İlk lint](evidence/README.md#review-06), [Yerel arşiv: katı negatif kontrol](evidence/README.md#review-06)

**Giderim:** Paket QML import root'unu CMake metadata'sı olarak versin. Proof'ta source ve installed akış aynı `QavsiDeliveryProof_QML_IMPORT_DIR` değişkenini sağlıyor; consumer `qt_add_qml_module(... IMPORT_PATH "${QavsiDeliveryProof_QML_IMPORT_DIR}")` kullanıyor. Installed değer `configure_package_config_file(... PATH_VARS ...)` ile prefix'e göre hesaplanıyor. Runtime `addImportPath` veya `QML_IMPORT_PATH` ortam değişkeni eklenmedi. Son source ve taşınmış installed consumer lint'leri sıfır uyarıyla geçti. [Yerel arşiv: Final kaynak tüketici](evidence/README.md#review-06), [Yerel arşiv: config şablonu](evidence/README.md#review-06), [Yerel arşiv: final lint](evidence/README.md#review-06)

Qt'nin `TARGET <target>` dependency/import biçimi build-tree target'lar içindir; `find_package` ile gelen imported target için kullanılmaz. Bu nedenle source ve installed yolu aynı URI + açık CMake import metadata'sıyla sınandı. [Qt QML module belgesi](https://doc.qt.io/qt-6/qt-add-qml-module.html)

### P1 / DEL-02 — Backing export'u, statik plugin sözleşmesini tamamlamıyor

**Yer:** DELIVERY §2 ve §4.1. İlk statik QML source consumer derlendi fakat `module "Qavsi.Review.Probe" plugin "QavsiDeliveryProbePlugin" not found` ile açılmadı. Import root düzeltildikten sonra installed scanner başka bir açığı görünür kıldı: `qmldir` içindeki `linktarget QavsiDeliveryProbePlugin`, namespaced export `QavsiDeliveryProof::CorePlugin` ile eşleşmiyordu. [Yerel arşiv: İlk statik hata](evidence/README.md#review-06), [Yerel arşiv: isim eşleşmesi uyarısı](evidence/README.md#review-06)

**Giderim:** Backing ve plugin target'ını export et; `qt_add_qml_module(OUTPUT_TARGETS ...)` ile dönen resource/plugin-init yardımcılarını aynı export set'e dahil et ve object dosyalarını kur. Generated `qmldir` linktarget'ı installed ortamda gerçekten var olmalı. Proof config'i normal CMake alias'ıyla `QavsiDeliveryProbePlugin -> QavsiDeliveryProof::CorePlugin` köprüsünü kuruyor. Generated `qmldir` veya Qt private property'leri değiştirilmedi. [Yerel arşiv: Modül CMake'i](evidence/README.md#review-06)

Son durumda consumer yalnız `QavsiDeliveryProof::Core` bağlayıp Qt import/finalization akışını kullanıyor. Ek `Q_IMPORT_QML_PLUGIN` C++ kodu veya explicit plugin link seçeneği gerekmedi; source ve taşınmış installed statik çalıştırma **staticPlugins=1** verdi. Ayrı bir pozitif kontrolde exported plugin target'ını açık bağlamak da çalıştı. [Yerel arşiv: Source final](evidence/README.md#review-06), [Yerel arşiv: installed final](evidence/README.md#review-06)

Mevcut `InstallMerce.cmake` örneği plugin'i kuruyor fakat plugin target'ını export set'e almıyor; yeni statik paket için olduğu gibi kopyalanmamalı. Bu, mevcut Merce sürümünün destek taahhüdüne ilişkin yeni bir hata beyanı değil. Qt de installed statik backing için `OUTPUT_TARGETS` yardımcılarının kurulmasını istiyor. [Qt QML module komutu](https://doc.qt.io/qt-6/qt-add-qml-module.html)

**SDK ayrıntısı:** Resmi `qt_import_qml_plugins` sayfası non-static Qt'de etkisiz olduğunu yazıyor; bu makinedeki 6.12.0 `Qt6QmlMacros.cmake:4939–5058` ise import tarayıp bulduğu `STATIC_LIBRARY` plugin target'larını bağlıyor. Sonuç yerel SDK ve executable kanıtına dayanıyor; bütün Qt patch'lerine genellenmiyor. SDK değiştirilmedi. [Resmi komut belgesi](https://doc.qt.io/qt-6/qt-import-qml-plugins.html), [Yerel arşiv: yerel SDK parmak izi](evidence/README.md#review-06)

### P2 / DEL-03 — Component export ve dependency closure gerçek pakette ayrıca kurulmalı

**Yer:** DELIVERY §1, §3.2, §4; ADR-002/007. Mevcut `MerceConfig.cmake` bütün temel Qt/StyleKit bağımlılıklarını birlikte buluyor; bu yaklaşım yeni `Icons` Core config'ine taşınırsa Q-01 bozulur. `QavsiRuntime::IconsMaterial` istendiğinde kendi gerekli `Icons` component'i de sağlanmalı; unrelated provider/native component bulunmamalı.

**Giderim:** Her component'in küçük dependency closure'ını açık tanımla; yalnız o closure için `find_dependency` ve export include çalıştır. Kapalı component target/asset/metadata kurmasın; required eksik component `check_required_components` ile configure sırasında reddedilsin. Proof'ta Core configure'ı, opsiyonel backend'e erişilirse fatal hata veren yerel sentinel paket varken geçti; opsiyonel ON pozitif kontrolü sentinel'e ulaştı. `COMPONENTS Provider` açık `Unavailable component: Provider` hatası verdi. Bu guard mekanizması kanıtıdır; gerçek Material/FontAwesome/SF Symbols bağımlılıklarının uygulanmış olduğu anlamına gelmez. [Yerel arşiv: OFF configure](evidence/README.md#review-06), [Yerel arşiv: ON negatif kontrol](evidence/README.md#review-06), [Yerel arşiv: eksik component](evidence/README.md#review-06)

Varsayılan network fetch yoktu; proof'ta FetchContent veya downloader kullanılmadı. Mevcut Merce'nin QtToastify fetch'i de `MERCE_BUILD_NOTIFICATIONS` koşulu içinde; bu sınır extraction sırasında korunmalı.

### P2 / DEL-04 — 0.x sürüm numarası, repo arası ABI veya source hedef uyumu sağlamaz

**Yer:** DELIVERY §3.2/§5 ve MIGRATION §3. `if(TARGET ...)` ile paket bulmayı atlamak, source target'ın istenen Runtime sürümünü sağladığını doğrulamaz. Farklı source ve installed Runtime minor'larını aynı graph'ta sessizce karıştırmamak gerekir.

**Giderim:** İlk sürümde her Runtime paketi graph'a tek origin/sürümden gelsin; gerekli dependency sürümü ve Qt patch/toolchain tuple'ı integration kaydında bulunsun. 0.x için Merce'nin `SameMajorVersion` kuralını kopyalama. Patch uyumu gerçekten taahhüt ediliyorsa `SameMinorVersion`; garanti yoksa exact pin kullan. Proof'un `0.1.3` version dosyası `0.1.0`/`0.1.3` isteklerini kabul etti, `0.1.4`/`0.2.0` isteklerini reddetti. Bu CMake seçimini test eder; gerçek iki kitin ABI uyumunu test etmez. [Yerel arşiv: Version assertions](evidence/README.md#review-06), [CMake 3.30 version helpers](https://cmake.org/cmake/help/v3.30/module/CMakePackageConfigHelpers.html)

SemVer 0.y.z public API'yi stable saymaz. Runtime/kit/Gallery ayrı etiket alabilir; integration consumer kaydı birlikte test edilen commit'leri sabitlemeli. Yeni Runtime edinme zorunluluğu Merce'nin mevcut tek-clone deneyimini kırar; MIGRATION'daki experimental/sonraki kırıcı sürüm kararı doğru. Mevcut 1.x patch sürümüne sessizce uygulanmamalı. [SemVer 2.0](https://semver.org/spec/v2.0.0.html)

### P2 / DEL-05 — Relocatable SDK paketi, imzalı uygulama deployment'ı değildir

**Yer:** DELIVERY §4/§6.2. Qt SDK'sının kendi imzalı `qml` aracı yerel plugin'i farklı Team ID nedeniyle reddetti. Aynı deployed plugin, yalnız Qt'ye link eden kendi diagnostic host'unda geçti. Bu bir QML tip/resource hatası değil; gerçek bir macOS host/signing sınırı. [Yerel arşiv: SDK tool hatası](evidence/README.md#review-06), [Yerel arşiv: Qt-only host final](evidence/README.md#review-06)

Proof plugin'inin backing dylib'e giden RPATH'i URI yerleşimine göre `@loader_path/../../../..`; relocation sonrası plugin-only yükleme çalıştı. Bu literal başka URI derinliğine kopyalanmamalı; gerçek pakette relative yol hesaplanmalı. Qt framework'leri kurulu SDK'dan geldi. Release app bundle, Qt runtime kopyalama, imzalama/notarization ve clean machine smoke ayrı yayın kapılarıdır.

## Kanıtlanan target ve module tasarımı

| Parça | Source kullanım | Installed kullanım |
|---|---|---|
| Public backing | `QavsiDeliveryProof::Core` alias → `QavsiDeliveryProbe` | `EXPORT_NAME Core`, `NAMESPACE QavsiDeliveryProof::` |
| Plugin helper | `QavsiDeliveryProof::CorePlugin` alias → generated plugin | Aynı namespaced imported target |
| Generated qmldir linktarget | Gerçek `QavsiDeliveryProbePlugin` target'ı var | Config'te aynı ada normal alias var |
| QML URI | `Qavsi.Review.Probe` | Aynı URI; `/qt/qml/Qavsi/Review/Probe` resource alanı |
| Tooling import root | Build tree'nin `src` root'u | Prefix'e göre hesaplanan `lib/qml` |
| C++ baseline | Public `cxx_std_20` | Export edilen aynı usage requirement |
| Qt bağımlılığı | Core, Gui, Qml, Quick | Yalnız Core component için aynı dependency bulma |
| Optional provider | OFF; target/asset/metadata yok | Required istek açık unavailable hatası |

Backing lib normal `lib/` altına, plugin + generated qmldir/qmltypes/QML/resources `lib/qml/Qavsi/Review/Probe/` altına kuruldu. Statik sürümde Qt'nin döndürdüğü üç resource object target'ı ve plugin-init object target'ı da export/install edildi. Header public C++ API olmadığı için export edilmedi. Qt'nin generated metadata'sı tek kaynak kaldı; ikinci elle yazılmış qmldir oluşturulmadı. [Qt module query/install modeli](https://doc.qt.io/qt-6/qt-query-qml-module.html)

Config, version ve export dosyaları prefix'e göre relative path kullanıyor. Final shared ve static kurulumundaki **bütün dosya byte'ları** eski source/build/prefix/SDK mutlak yolları için tarandı; sızıntı bulunmadı. Corrected prefix'ler ikinci kez fiziksel olarak taşındı; eski prefix ve modül source/build yolları yokken final consumer configure/build/lint/run tamamlandı. CMake user/system package registry kapalıydı. [Yerel arşiv: Tarama](evidence/README.md#review-06), [Yerel arşiv: izolasyon](evidence/README.md#review-06), [Yerel arşiv: taşıma](evidence/README.md#review-06)

Bu standart paketleme tasarımı, kabul edilen ilk gerçek Runtime component'ine kendi public adıyla uygulanabilir. Buradaki `Core` veya `Probe` isimleri Qavsi API önerisi değildir.

## Test matrisi

| Test | Sonuç | Sınır / belirleyici kanıt |
|---|---|---|
| Shared QML module, source consumer | PASS | Aynı public target; mekanizma=42, resource içeriği doğru; final strict lint 0 |
| Shared, ayrı installed consumer | PASS | Modül source/build eski yolları gizli; import ve resource smoke |
| Shared, final taşınmış prefix | PASS | Yeni `build-final-shared`; configure/build/runtime ve strict lint 0 |
| Static QML module + shared Qt, ilk source consumer | FAIL | İlk metadata düzeninde plugin bulunamadı; başarısızlık saklanmadı |
| Static QML module + shared Qt, final source consumer | PASS | Yalnız backing public target; scanner plugin'i bağladı; staticPlugins=1 |
| Static installed, explicit exported plugin link | PASS | Normal CMake plugin target'ı + generated initializer; manuel C++ import kaldırıldığında da geçti |
| Static QML module + shared Qt, final taşınmış prefix | PASS | `QAVSI_PROOF_EXPLICIT_PLUGIN=OFF`; tek public backing link; staticPlugins=1; strict lint 0 |
| Plugin-only dynamic diagnostic, taşınmış prefix | PASS | Host yalnız Qt'ye link eder; import root açık argv'dir, normal consumer doğrulamasının yerine geçmez |
| Optional OFF + dependency sentinel | PASS | Configure sentinel backend'i bulmaya çalışmadı; font/StyleKit/Merce target'ı yok |
| Optional ON + sentinel | Beklenen ret | Configure exit 1, `OPTIONAL_DEPENDENCY_REACHED` |
| Required bulunmayan component | Beklenen ret | Configure exit 1, `Unavailable component: Provider` |
| İlk tooling metadata negatif kontrolü | FAIL | Default lint exit 0 + import uyarısı; strict lint exit 255 |
| Final source/installed tooling | PASS | Source shared/static ve final installed shared/static strict lint sıfır uyarı |
| Prefix absolute-path taraması | PASS | Final shared/static installed dosyalarda eski source/build/prefix/SDK yolu yok |
| 0.x package version seçimi | PASS | SameMinorVersion için dört assert; gerçek cross-kit ABI testi değil |

Final testlerin tam argv/exit-code kayıtları [Yerel arşiv: commands.json](evidence/README.md#review-06), kaynaklar ve metadata [Yerel arşiv: evidence/06](evidence/README.md#review-06) altında. Ana final loglar: [Yerel arşiv: shared build](evidence/README.md#review-06), [Yerel arşiv: shared run](evidence/README.md#review-06), [Yerel arşiv: shared lint](evidence/README.md#review-06), [Yerel arşiv: static build](evidence/README.md#review-06), [Yerel arşiv: static run](evidence/README.md#review-06), [Yerel arşiv: static lint](evidence/README.md#review-06).

Gerçekte kullanılan final shared komut dizisinin özü:

```bash
rtk QT_SDK/bin/qt-cmake -S /private/tmp/qavsi-review-06/consumer -B /private/tmp/qavsi-review-06/build-final-shared -U QavsiDeliveryProof_DIR -DCMAKE_PREFIX_PATH=/private/tmp/qavsi-review-06/final-prefix-shared
rtk cmake --build /private/tmp/qavsi-review-06/build-final-shared --parallel 4
rtk QT_SDK/bin/qmllint --max-warnings 0 @/private/tmp/qavsi-review-06/build-final-shared/.rcc/qmllint/delivery_consumer.rsp
rtk proxy python3 /private/tmp/qavsi-review-06/run.py run-final-shared rtk /private/tmp/qavsi-review-06/build-final-shared/delivery_consumer
```

Runner son komutta temiz ortam + offscreen/software ayarını sağlar. İlk source/configure/build/install komutları da `commands.json` içinde; final kaynak snapshot'ı `prototype/` altında. Geçici modül source/build klasörleri izolasyon için `.hidden` adıyla bırakıldı.

## Çalıştırılmayanlar

| Kapsam | Durum | Neden |
|---|---|---|
| Gerçek bağımsız Material3/Apple/Fluent paketi, Merce olmadan | NOT-RUN | Bunlar bu teslimatta implement edilmiş/yayımlanmış paketler değil. Proof'ta Merce/StyleKit bağı olmaması Q-02'nin ürün kabulü değildir |
| Tam static Qt SDK | NOT-RUN | Kullanılan macOS Qt framework'leri shared. Statik QML module sonucu tam statik Qt dağıtımına genellenmez |
| Windows DLL/QML plugin deploy, DPI/IME | NOT-RUN | Windows runner/toolchain ve paket uygulaması yok |
| Linux Wayland/X11/EGLFS | NOT-RUN | Hedef runner/cihaz yok; macOS offscreen bunu doğrulamaz |
| WASM browser/font/input/popup | NOT-RUN | WASM build ve browser smoke bu dar proof kapsamında çalıştırılmadı |
| macOS Metal/native materials/real desktop | NOT-RUN | Yalnız offscreen import/resource smoke yetkisi; native materyal kodu yok |
| macOS app bundle signing/notarization/clean machine | NOT-RUN | Proof kurulu Qt SDK'sını kullanıyor; self-contained release app üretilmedi |
| Kit/font/provider gerçek asset ve lisans envanteri | NOT-RUN | Proof'ta font/provider yok; küçük özgün metin resource'u var |
| Exact CMake 3.30 ve Qt 6.12 dışındaki patch/minor | NOT-RUN | Kurulu CMake 4.4.3 ve Qt 6.12.0 kullanıldı |
| Gerçek UI render, perf, accessibility, input | NOT-RUN | Bu alanın testi packaging/import/resource ile sınırlı |

## İlk gerçek sürüme uygulama sırası

1. Shared-admission kapısından geçen ilk gerçek mekanizmayı çıkar; aynı public target/URI, CMake import metadata ve plugin export kurallarını birlikte kur. Genel plugin veya Theme abstraction ekleme.
2. Core-only, tek provider ve required unavailable component testlerini gerçek dependency/asset set'iyle çalıştır. Kaynak ve kurulu consumer'a strict QML diagnostics ekle.
3. Gerçek ilk kitin standalone consumer'ını Merce source/install/import yolları olmadan çalıştır; sonra kurulu paket ve taşınmış prefix'i doğrula. Proof'un dummy mekanizmasını bu başarı yerine sayma.
4. Merce facade bağlanırken eski target/import korunur; yeni dependency edinme koşulu experimental/sonraki kırıcı sürüm hattında duyurulur. Runtime/kit/Gallery BOM kaydı birlikte test edilmiş sürümleri gösterir.
5. Stable destek yalnız gerçekten geçen platform/Qt/toolchain lane'lerine verilir; binary release öncesi OS deployment ve signing kapıları tamamlanır.

Kurulu SDK ve gerçek ürün dosyaları değiştirilmedi; Git komutu, branch/commit/index işlemi yapılmadı. SDK'daki optional Qt AssetDownloader/TaskTree configure uyarısı loglarda korunmuştur; core proof işlemlerini engellemedi. İlk prototip authoring hatası ve ara Ninja recovery notu kanıt README'sinde ayrıldı; final kabul sonuçları yeni `build-final-*` ağaçlarında alındı.

**Atlanan doğrulama:** Tam static Qt, gerçek kitler ve OS deployment/Metal/Windows/Linux/WASM çalıştırılmadı.
**Risk:** Bu proof paketleme yaklaşımını doğrular; Qavsi ürün sürümü, bütün Qt 6.12+ patch'leri veya çapraz platform release uyumu ilan etmez.
