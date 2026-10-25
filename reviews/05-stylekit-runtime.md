# 05 — Qt 6.12 StyleKit runtime fizibilitesi

**Tarih:** 2026-10-23. **Karar:** Sabit Gallery rolü + app-owned Style değişimi, aşağıdaki Qt-only kapsamda **PASS**. Bu sonuç gerçek Merce/Material3/Apple adaptörlerinin veya bütün Qavsi sisteminin kabulü değildir.

Runtime modül adları `docs/SHARED-ADMISSION.md` kapsamındaki adaylardır; bu prototip ürün SDK'sı, ortak modül admission'ı veya standart API kabul kararı üretmez.

## Kapsam ve ortam

`ARCHITECTURE.md`, `docs/CONTRACTS.md`, `docs/KIT-AUTHORING.md`, `docs/DELIVERY.md`; mevcut `experiments/stylekit-dynamic-variation` ve `experiments/button-support` kaynakları incelendi. Ürün kodu, MButton, SDK ve Git değiştirilmedi. Prototip/build yalnız `/private/tmp/qavsi-review-05/` içinde oluşturuldu; tekrar üretilebilir kaynak kopyası ve loglar [Yerel arşiv: evidence/05](evidence/README.md#review-05) altında.

| Alan | Gerçek koşul |
|---|---|
| Qt runtime/SDK | `6.12.0`, `QT_SDK` |
| Qt build kimliği | `arm64-little_endian-lp64 shared (dynamic) release`; Apple LLVM `17.0.0 (clang-1700.3.19.1)` |
| Prototip compiler/build | `/usr/bin/c++`, AppleClang `21.0.0.21000101`; C++20, Debug; CMake `4.4.3`, Ninja |
| İşletim sistemi | macOS `26.5.2`, build `25F84`, arm64 |
| QPA/renderer | `QT_QPA_PLATFORM=offscreen`, `QT_QUICK_BACKEND=software`; `graphicsApi() == Software (1)` |
| Qt kaynak referansı | `qtdeclarative` `4332fbbb4c340463dc8cee36dda0c85d932122d8`; inceleme için verilen `qqstylekit.cpp` / `_p.h` kopyaları |
| SDK doğruluğu | `qVersion()`, `QLibraryInfo::build()` ve framework SHA-256 kaydedildi; SDK'nın bu commit'ten yeniden derlendiği ayrıca doğrulanmadı |

Configure, SDK'da `Qt6TaskTree` ve buna bağlı `Qt6QmlAssetDownloaderPrivate` bulunamadığına ilişkin warning verdi. Seçilen modüllerin configure/build/import çalışması başarılı; SDK install/deploy bütünlüğü bu testin konusu değil. Runtime'da yalnız mevcut `Sans Serif` font alias maliyeti uyarısı kaldı. Kanıt: [Yerel arşiv: configure.log](evidence/README.md#review-05), [Yerel arşiv: build.log](evidence/README.md#review-05), [Yerel arşiv: summary.json](evidence/README.md#review-05), [Yerel arşiv: SHA256.json](evidence/README.md#review-05).

## Çalıştırılan senaryolar

| Senaryo | Beklenen | Gözlenen | Sonuç |
|---|---|---|---|
| Aynı `SK.Button`, sabit `gallery.primary`, A→B→A | Kontrol yeniden yaratılmadan yeni kit background/text/data çözülür | Light→Dark→Light × normal/hover/checked × A/B/A: **27 snapshot**, eşleşen native/public-reader değerleri ve boyalı piksel | **PASS** |
| Ziyaret edilmiş hover/checked cache'i | A/B/A sonrasında önceki state değerleri kalmaz | Hover native mouse event ile; checked native property ile kuruldu. Her stile geçişte doğru state renk/data sonucu | **PASS** |
| Kontrol state'i | Kimlik, text, checked, focus ve rol sabit kalır | Her snapshot'ta aynı pointer, `Persistent button`, beklenen checked/hovered, active focus, aynı role listesi | **PASS** |
| Gerçek Light/Dark theme | Style theme değişiminde background/text/data yeni theme'ye çözülür | İki Style'ın public `light`/`dark` Theme bileşenleri ve `themeName` yolu kullanıldı; iki tema + Light'a dönüş eşleşti | **PASS** |
| `StyleReader` ile native çizim uyumu | Ayrı public reader, stock Button'ın background/text çözümüyle eşleşir | `StyleReader.background/text/data`, stock background descriptor/data, `contentItem.color` ve ekran görüntüsünden background pixel aynı sonucu verdi | **PASS** |
| Loading guard sonrası caller binding | Loading sırasında değişen izin güncel değeriyle geri gelir; sonraki bağımlılık değişimleri de işler | `true→loading→permission=false→idle` disabled kaldı; sonra permission=true enabled oldu. Ters yönde loading sırasında true olması guard'ı aşmadı; bitişte enabled oldu | **PASS** |
| İki `QQmlEngine` | Bağımsız stil scope'u oluşmaz | Engine2, engine1'in A değerlerini gördü. Engine2 B atayınca engine1'in mevcut kontrolü de B oldu; A'ya dönüş ikisini etkiledi | **PASS — izolasyon yok** |
| Inactive engine cleanup | Aktif Style ilk host'a geri verildikten sonra ikinci engine silinebilir | A geri seçildi, engine2 silindi, engine1 native data A olarak kaldı | **PASS — bu sıra için** |
| Aynı Style, instance rol ekleme/kaldırma | Mevcut ve sonradan yaratılan instance aynı role göre güncellenir | Eklemede mevcut: `missing/#d3d3d3`; fresh: `true/#90ee90`. Kaldırmada fresh hâlâ `true/#90ee90`. İki rol listesi de aynıydı. Orijinal repro exit `1` | **FAIL — bilinen Qt regression** |
| Aktif stil host'unu silip engine ile devam; null Style reset; styleUrl/style karışımı | Destekli güvenli lifecycle ispatı gerekir | Kaynak riski aşağıda; bilerek çalıştırılmadı | **NOT-RUN** |
| Popup açıkken geçiş, native menü/material, gerçek keyboard/IME, GPU, diğer kontroller/OS'ler, installed/static/WASM tüketici | İlgili üretim hattında ayrıca doğrulanır | Bu prototip kapsamı dışında | **NOT-RUN** |
| Başta loading=true / queued initialization / ilk input; shared Action; custom loading içeriği | Ayrı mevcut ButtonSupport contract kapsamı | Bu inceleme yalnız actual guard'ın binding restore yolunu çalıştırdı | **NOT-RUN** |

Ana çalışma **399 başarılı assertion, 0 başarısız assertion**; iki engine çalışması **8 başarılı assertion** üretti. Sayının kapsam garantisi yoktur; birçok assertion aynı snapshot'ta farklı gözlemlerdir. [Yerel arşiv: same-instance.log](evidence/README.md#review-05), [Yerel arşiv: two-engines.log](evidence/README.md#review-05), [Yerel arşiv: dynamic-variation.log](evidence/README.md#review-05).

### Çözümleme kanıtı

`ProbeStyle` doğrudan public `SK.Style` tabanlıdır. İki app-owned instance pencere/root ömründe tutulur. `ProbeTheme` public `SK.Theme` ve `SK.StyleVariation` kullanır; `gallery.primary` tarifleri bileşenler kurulurken bildirilir. Çalışma boyunca Button'ın `StyleVariation.variations` listesi değiştirilmez. Theme değişimi gerçek `themeName` ve `light`/`dark` bileşenleri üzerinden yapılır.

Örnek gözlemler:

| Kit/theme/state | Native + reader background | Native + reader text | Native + reader `data.probe` | Boyalı background |
|---|---|---|---|---|
| A/light/normal | `#b11a1a` | `#fff1f1` | `A/light/normal` | `#b11a1a` |
| B/light/normal | `#126a26` | `#f1fff4` | `B/light/normal` | `#126a26` |
| A/dark/hover | `#773333` | `#ffc4cc` | `A/dark/hover` | `#773333` |
| B/dark/checked | `#163e66` | `#c4ffda` | `B/dark/checked` | `#163e66` |

Sekiz offscreen PNG kanıta alındı; [Yerel arşiv: A/dark/hover](evidence/README.md#review-05) ve [Yerel arşiv: B/light/normal](evidence/README.md#review-05) ayrıca görsel incelendi. Pixel assertion düz background bölgesini ölçer; metin rengi native getter'dan okunur. Bu bir kit tasarım doğruluğu veya kontrast ölçümü değildir.

Prototip private Qt header, `Qt.labs.StyleKit.impl` import'u veya SDK yaması kullanmaz. Native background'un `backgroundStyle` alanına test amaçlı dinamik introspection yapılır; bu alan ürün API'si olarak önerilmez. Ürün sözleşmesi public StyleReader ve gerçek render üzerinden tutulabilir. `QQuickIconLabel::color` için notify bulunmadığından ölçüm her snapshot'ta `QQmlProperty` ile doğrudan okunur; buna QML binding kurmak stale ölçüm üretir. Nihai log bu ölçüm sorununu giderilmiş hâlde içerir.

Loading doğrulamasında mevcut `ButtonSupport.cpp`, `.h`, `ButtonLoadingGuard.qml` **byte-identical** kopyaları küçük ayrı public QML modülünde derlendi; guard taklit edilmedi. `contentItem` istenmedi, stock SK.Button içeriği kaldı. Bu, guard'ın içerikten bağımsız `enabled` restoration yolunun kanıtıdır; mevcut tüm Merce deneyinin geçtiği anlamına gelmez.

## Pinned kaynak: süreç kapsamı ve lifecycle

[qqstylekit.cpp](https://github.com/qt/qtdeclarative/blob/4332fbbb4c340463dc8cee36dda0c85d932122d8/src/labs/stylekit/qqstylekit.cpp) satır 90–99, `QQStyleKitAttached::s_instance` üzerinden tek attached nesnesi döndürür; parent `QGuiApplication::instance()` olur. İlk erişimin engine'i `m_engine` içinde tutulur. [qqstylekit_p.h](https://github.com/qt/qtdeclarative/blob/4332fbbb4c340463dc8cee36dda0c85d932122d8/src/labs/stylekit/qqstylekit_p.h) içinde hem engine hem Style `QPointer` olarak saklanır. Deneyin iki engine sonucu bu kaynak okumasıyla uyumludur. Tam upstream araştırma snapshot’ları geçici cache’de korunur; teslimatta [Yerel arşiv: URL/hash provenance](evidence/README.md#review-05) bulunur.

Somut sınırlar:

1. **Pencere/engine başına bağımsız stil yok.** Gallery worker'larının ayrı süreç olması doğru karar. Theme/config nesnelerini scoped yapmak bu global scope'u değiştirmez.
2. **Null reset güvenli API yolu değil.** `setStyle()` satır 125–137, atanan `m_style` üzerinde null guard olmadan `reapplyStyle()` çağırır. İlk aktif Style'dan `nullptr`a geçiş reset olarak kullanılmamalı; çökme deneyi çalıştırılmadı.
3. **`styleUrl` / `style` sahipliği karıştırılmamalı.** `setStyleUrl()` sonunda `m_ownsStyle = true`; sonraki `setStyle()` eski nesneye `deleteLater()` çağırır fakat bu flag'i false yapmaz. URL yolundan app-owned A'ya, sonra B'ye geçmek A'nın Qt tarafından silinmesine yol açabilecek sahiplik zinciridir. V1'in yalnız app-owned Style instance yolu seçimi bu riski dışarıda bırakır.
4. **Engine reset yaşam döngüsü kanıtı yok.** QPointer kullanımı tek başına reader/context/cache güvenliğini ispatlamaz. Tek ana host engine'in bütün aktif Style/consumer'lardan uzun yaşaması korunmalı. Deney yalnız aktif host'u yaşatıp inactive observer engine'i kapatmayı doğrular.

Bu davranışlar Qt Labs uygulama ayrıntısına bağlıdır. Qt patch yükselmesinde executable regression tekrar çalıştırılmalıdır; private header ürün bağımlılığı yapılmamalıdır.

## Mimari sonuç ve en küçük gerekli adımlar

- **Sabit Gallery rolleri mekanik olarak uygulanabilir.** Bu yolla Q-05'in mevcut Button kimliği ve çözülmüş style değerleri parçası için blocker bulunmadı. Aynı süreçte tüm pencerelerin değişmesi beklenen global davranıştır.
- **Gerçek adapter kanıtı hâlâ gerekir.** Test kitleri kontrollü Qt-only fixture'lardır; gerçek Merce/M3/Apple'ın public Theme/recipe extension noktaları, bütün family'ler, font/asset/delegate yaşam döngüsü burada sınanmadı. İlk kit entegrasyonunda aynı senaryoyu gerçek public adaptörle çalıştırmak yeterli sonraki adım; yeni ortak Theme veya private import gerekmiyor.
- **Rol mutation desteği beyan edilmemeli.** Orijinal Qt repro hâlâ FAIL. Sabit rolün başarılı olması bu hatayı onarmaz. MButton ve ayrı QTBUG regression korunmalı; alias tarifleri runtime'da eklenmemeli/kaldırılmamalı.
- **Lifecycle sözleşmesi mevcut hâliyle tutulmalı.** Composition root, tek host engine ve app-owned Style nesnelerini açıkça sahiplenmeli; geçiş sırasında eski Style'ı hemen yok etmemeli. `styleUrl`, null reset veya engine teardown workaround eklenmemeli.
- **Stil transaction tamamı kanıtlanmadı.** Loglar 80 ms event-loop settling sonrasını ölçer. Tek frame atomik geçiş, açık popup, custom background delegate değişimi ve native material cleanup için üretim UI testi gerekir; mevcut dokümanın bu konuları ayrı kabul kapısı yapması doğru.

Bu inceleme nedeniyle mimari doküman veya ürün kodu değişikliği zorunlu bulunmadı. Gereken küçük teslimat, bu dar regression'ın gerçek Gallery/adaptör uygulanınca CI'a alınması ve destek beyanının test edilen Qt patch/renderer/family ile sınırlanmasıdır.

## Tekrar üretme komutları

Kaynakların kalıcı kopyası `evidence/05/prototype/` altında. Aynı Qt kurulumu bulunan ortamda kaynak klasörü yeniden `/private/tmp/qavsi-review-05/src/` konumuna kopyalanabilir; ürün kaynaklarına yazma gerekmez. Tüm komutlar `login:false` koşulunda çalıştırıldı.

```bash
rtk QT_SDK/bin/qt-cmake -S /private/tmp/qavsi-review-05/src -B /private/tmp/qavsi-review-05/build -G Ninja -DCMAKE_BUILD_TYPE=Debug
rtk cmake --build /private/tmp/qavsi-review-05/build --parallel 4
rtk env QT_QPA_PLATFORM=offscreen QT_QUICK_BACKEND=software QAVSI_ARTIFACT_DIR=/private/tmp/qavsi-review-05/logs /private/tmp/qavsi-review-05/build/qavsi_review_05
rtk env QT_QPA_PLATFORM=offscreen QT_QUICK_BACKEND=software QAVSI_ARTIFACT_DIR=/private/tmp/qavsi-review-05/logs /private/tmp/qavsi-review-05/build/qavsi_review_05 --engines

rtk QT_SDK/bin/qt-cmake -S MERCE_SOURCE/experiments/stylekit-dynamic-variation -B /private/tmp/qavsi-review-05/dynamic-build -G Ninja -DCMAKE_BUILD_TYPE=Debug
rtk cmake --build /private/tmp/qavsi-review-05/dynamic-build --parallel 4
rtk env QT_QPA_PLATFORM=offscreen QT_QUICK_BACKEND=software /private/tmp/qavsi-review-05/dynamic-build/stylekit_dynamic_variation --verify
```

İlk iki runtime komutu exit `0`; son Qt bug repro komutu beklenen exit `1` verdi. Build/import için özel `QML_IMPORT_PATH` eklenmedi. Yeni dependency, desktop pencere/foreground kontrolü, OS ayarı, ağ mutasyonu, SDK install veya Git işlemi yapılmadı.

Atlanan: gerçek kit adaptörleri, popup/tek-frame atomik geçiş, GPU/platform/deploy ve aktif host teardown. Risk: Qt Labs patch davranışı ve mevcut instance variation mutation hatası üretim kapsamını sınırlamaya devam ediyor.
