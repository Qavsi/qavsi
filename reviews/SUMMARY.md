# Altı incelemenin ortak sonucu

**Tarih:** 2026-10-24; kaynak düzeltmesi 2026-10-25. **Genel karar: CONDITIONAL — mimari uygulanabilir, ürün kabulü henüz yok.**

Üç tasarım sistemini kendi Theme/token/StyleKit modüllerinde geliştirmek doğru sınırdır. Ortak bir görsel Theme, evrensel variation kataloğu veya yeni kontrol ailesi gerekmiyor. İlk bağımsız kit doğrudan **Qt + kendi Theme/Style** ile başlayabilir; Runtime deposunun açılmasını beklemez.

Altı görev ilk geçici DESIGN.md girdileriyle tamamlandı. Kaynağa bağlı katalog bulguları tarihsel olarak işaretlendi. Kullanıcının son kararı Google Material 3 ve Microsoft Fluent 2/Windows rehberlerini esas almak, Apple native Liquid Glass hedefini korumaktır. Üç kitin güncel DESIGN.md sözleşmesi bu kararla yazıldı; bunlar yeni ürün/render kabulü değildir. Qt/API ve mevcut kod bulguları tasarım girdisinden bağımsızdır; yeni Runtime component’i bugün bütün kabul kapılarını geçmiyor.

## 1. Altı alanın sonucu

| Alan | Karar | Bulgu / test | Uygulamaya etkisi |
|---|---|---|---|
| [01 Material3](01-material3.md) | **CONDITIONAL**; bugünkü bağımsızlık **FAIL** | Mevcut kit Merce’ye bağlı. Seçilen resmi Expressive snapshot’ına göre 3 yükseklik ve 4 icon ölçüsü farklı. Public typography/content sınırı var. | Kaynak edisyonunu sabitle; ilk Small butonu Qt + kendi Theme/Style ile, zorunlu Runtime olmadan doğrula. |
| [02 Apple](02-apple.md) | **CONDITIONAL** | Public availability compile-only probe PASS. Internal handle kullanımı, çoklu owner restore riski ve ayrı QML foreground/native glass composition açığı bulundu. | Önce opak fallback; ardından public lifecycle ve tek owner ile tek regular native yüzey. Gerçek Metal/masaüstü testi şart. |
| [03 Fluent](03-fluent.md) | **CONDITIONAL**; DESIGN.md’nin üretim yeterliliği **FAIL** | Web, Windows/WinUI ve eski Reveal kuralları karışıyor. Dört tanımlı light renk çifti PASS; alpha örneklerinde karşı örnek var. | Önce hazır Qt FluentWinUI3 yeterli mi bak; ayrı kit gerekiyorsa seçilmiş portable recipe ile opak Button dilimi. Native Mica ayrı iş. |
| [04 Ortak altyapı](04-shared-boundary.md) | **CONDITIONAL**; yeni koşulsuz ortak kabul **yok** | Qt-only offscreen 10/10 ve Cocoa 12/12 kontrol PASS. Temel SF/font icon, public motion/contrast ve image-provider lifecycle zaten Qt’de. | Qt’yi yeniden adlandıran wrapper yazma. Yalnız kalan dar farkı ayrı kanıtla. |
| [05 StyleKit runtime](05-stylekit-runtime.md) | Sınanan mekanik yol **PASS**; kit geneli **CONDITIONAL** | Sabit rol + aynı butonda Style A→B→A/theme/state: 27 snapshot, 399 assertion PASS. İki-engine global scope: 8 assertion PASS. Mutation regression: beklenen FAIL. | Sabit Gallery rolleriyle ilerle; per-engine izolasyon veya SDK hatasının düzeldiği iddiası kurma. |
| [06 Dağıtım](06-delivery.md) | Sınanan paketleme yolu **PASS**; ürün dağıtımı **CONDITIONAL** | Shared/STATIC QML modülü; source, installed ve taşınmış prefix PASS. Tooling/import-root ve static plugin export sorunları fixture’da çözüldü. | İlk gerçek paket aynı metadata/export/strict-lint kapısını geçsin. Tam statik Qt/WASM/app deployment sonucu çıkarma. |

Tüm testler ürün SDK’sından ayrı, sınırlı fixture’larda çalıştı. Bir rapor diğerinin proof’unu kullandıysa tekrar çalıştırmış gibi sayılmadı. Native cam veya Windows materyali için bu tur görsel kabul/performans sonucu yok.

## 2. Güncel kaynak ve önceki inceleme girdileri

[Atıf ve digest kaydı](../references/SOURCES.md), [machine-readable envanter](../references/sources.json).

| Kit | Güncel durum | Nasıl kullanılır? |
|---|---|---|
| Material3 | [Güncel DESIGN.md](../design-systems/material3/DESIGN.md) — resmi Google otoritesi kabul edildi | Eski [Yerel arşiv: katalog girdisi](evidence/README.md#historical-inputs) ilk 01’in tarihsel input’u; yeni sözleşmenin ölçü/edition kaydı ayrı. |
| Apple | [Native Liquid Glass DESIGN.md](../design-systems/apple/DESIGN.md) — HIG/public AppKit | [VoltAgent web belgesi](../references/voltagent/apple/DESIGN.md) yardımcı referans; native hedefi değiştirmez. [Kaynak ek incelemesi](apple-source-addendum.md). |
| Fluent | [Güncel DESIGN.md](../design-systems/fluent/DESIGN.md) — resmi Microsoft otoritesi kabul edildi | Eski [Yerel arşiv: katalog girdisi](evidence/README.md#historical-inputs) ilk 03’ün tarihsel input’u; Web/Windows recipe ayrımı korunur. |

VoltAgent commit `3b605329240ce735b510037cca322f1831fdd7f8` envanterinde Material3/Fluent dosyası yoktu. Kullanıcı ardından **resmi Google/Microsoft rehberlerini esas almayı ve Apple native Liquid Glass’ı korumayı** açıkça istedi; kaynak sorusu kapandı. [Resmi kaynak URL/sürüm/hash kaydı](../references/official/sources.json) ve üç yerel kit sözleşmesi bu kararı izler. Uyarlama dosyaları resmi kuruluşların yayımladığı DESIGN.md kopyaları değildir.

Yeni ve tarihsel dosyalar byte-identical/atıflı saklandı. Kaynak promptları talimat sayılmaz. Önceki designmd.app Liquid Glass girdisinin “No circular spinners” iddiası veya eski Fluent RGBA eleştirileri yeni VoltAgent belgesine ait değildir. İlk altı raporun kaynağa bağlı bulguları kendi tarihsel girdisiyle okunur; Qt mekanizma proof’ları yeni tasarımın görsel doğrulaması değildir.

## 3. Ortak alan için karar

[SHARED-ADMISSION](../docs/SHARED-ADMISSION.md) ve 04’teki ayrıntılı 18 alan tablosu uygulanır.

| Sınıf | Bugünkü örnekler | İşlem |
|---|---|---|
| **USE EXISTING QT** | Temel QIcon/image/native SF/font glyph yolu, QQmlEngine image-provider ownership, native Menu, public motion/contrast hints, window/screen/DPR/move/resize, QML/CMake module metadata | Qt’yi doğrudan kullan; aynı API’yi yeni Qavsi adıyla sarmalama. |
| **KEEP KIT/APP** | Token, Theme, typography, geometry, state/motion/ripple, glass yer seçimi, Mica/Acrylic/Smoke politikası, navigation, feedback orkestrasyonu | Sahibi ilgili kit veya uygulama. Benzer kod ortak tasarım semantiği demek değil. |
| **DEFER — bugün projede** | Çoklu icon collection/ikon başına axes/routing, birleşik ButtonSupport, ileri SF config/catalog, Qt’de eksik OS tercihleri, native material bridge, ortak token build-time aracı | Gerçek Qt boşluğu, aynı dar sözleşme, bağımsız tüketim, lifecycle/fallback ve dağıtım kanıtı tamamlanırsa tekrar değerlendir. |
| **QUALIFIES QAVSI** | **Yeni component yok** | Boş runtime scaffold’u veya aday URI/target’ı yayımlama. |

Mevcut Merce AppIcon ve paketli ad API’si korunur. Qt’nin temel icon desteği bulunması mevcut compatibility API’sini silme gerekçesi değildir. İleri ihtiyacın ortaklaştırılması ayrı iştir. Aynı şekilde enabled binding restoration’ın geçmesi birleşik loading/content desteğini ortak standarda dönüştürmez.

Qavsi’nin bugün sağlayacağı ortak değer **belgelenmiş sınırlar, kaynak/uyumluluk kaydı ve doğrulama yöntemi** olabilir; bunun bir runtime library’ye dönüşmesi zorunlu değildir.

## 4. Uygulamadan önce kapanacak somut boşluklar

### 4.1 Tasarım edisyonunu seçmeden recipe taşımama

Material3’te 140 numeric/alias leaf’in çözülmesi ve 11 generated sabitin kaynakla eşleşmesi yalnız üretim zinciri tutarlılığıdır. Seçilen resmi Expressive referansla geometri/type-role karşılaştırması farklı çıktı; başka M3 edisyonuna aynı hüküm genişletilmez.

Fluent’te Web ve Windows type ramp’ı, neutral/brand görünüm ve material katmanları ayrı. İlk öneri Web referanslı portable opak dilimdir; Windows-native eşdeğerlik iddiası yoktur. Hazır Qt FluentWinUI3 ihtiyacı karşılıyorsa yeni stil yazılmaz. Hangi kitin üretileceği, gerçek ek değer ve seçilen referansla gerekçelendirilir.

### 4.2 Public StyleKit typography/content sınırı

TextStyle tam QFont/type-scale nesnesi değildir; genel content-delegate slot’u da yoktur. Family QFont ile çözülebilen native alt küme, exact line-height ve Material FILL gereksiniminden ayrılır.

İlk Small buton text-only veya native image/icon ile başlayabilir. Gerekliyse yalnız kit içinde açık contentItem adaptörü eklenir; background’dan control.font yazılmaz. Caller’ın font binding’i, explicit sıfır spacing/padding/tracking ve sonraki kaynak değişimi regresyonu henüz çalıştırılmadı. 399 assertion bu eksik testi kapatmaz.

### 4.3 Apple’da attachment ile görsel başarı ayrımı

Mevcut kaynakta QWindow::handle() internal API sınırında. Yeni bridge belgeli winId/PlatformSurface lifecycle’ından başlar. WinId’nin native kaynak yaratabileceği ve teardown’da stale handle kullanılamayacağı dikkate alınır.

Tek pencere owner’ı; observer/view/mask ve window-state restore’u yönetir. İkinci owner’ın eski snapshot’ı başka güncel değişikliği ezmemeli. Empty glass contentView ile ayrı Qt foreground’un AppKit vibrancy sisteminden yararlandığı varsayılmaz. Qt Metal root’u başka native view’a reparent edilmez.

İlk geometri item-scale=1 + translation ile sınırlanabilir; DPR=1 zorlaması değildir. Scale/frame doğruluğu radius/clip doğruluğunu kanıtlamaz. Gerçek masaüstü üzerinde okunaklılık veya input başarısızsa native katman yerine güvenli fallback kalır.

### 4.4 Paketlemede runtime link’inden fazlası

06’da runtime geçerken default all_qmllint exit=0 yanlış güven üretti. Public CMake import-root metadata’sı + strict qmllint gerekli. Shared/static backing/plugin ve helper export’ları, generated qmldir linktarget eşleşmesi ve taşınmış prefix birlikte sınanmalı.

Static QML module + shared Qt, tam static Qt değildir. Qt SDK’sının imzalı qml aracında local plugin TeamID kısıtı da ürün metadata başarısızlığıyla karıştırılmaz; kendi diagnostik host ayrı kanıttır.

## 5. Mimariye işlendi

Bu tur ürün kodu yerine aşağıdaki sözleşmeler düzeltildi:

| Belge | Düzeltilen sınır |
|---|---|
| [ARCHITECTURE](../ARCHITECTURE.md) | Optional Runtime, Qt-first, tasarım edisyonu, public typography ve native composition riskleri |
| [CONTRACTS](../docs/CONTRACTS.md) | Font/content, caller binding kapısı, tek native owner, Attached anlamı, Apple/Fluent materyal ayrımı |
| [KIT-AUTHORING](../docs/KIT-AUTHORING.md) | Mevcut DESIGN girdisi, source/platform/hash/deviation kaydı, native font/asset hakları, minimal kit |
| [MIGRATION](../docs/MIGRATION.md) | Otomatik extract yerine admission; M3 Small-first; Apple opaque-first/tek native yüzey; compatibility korunması |
| [DELIVERY](../docs/DELIVERY.md) | Runtime’sız minimal consumer; component/export/import-root/static/strict diagnostics |
| [DECISIONS](../docs/DECISIONS.md) | ADR-009: kullanıcının standart ortaklık ilkesi kabul edilmiş; tek tek API adayları henüz onaylanmış değil |
| [VALIDATION](../docs/VALIDATION.md) | Gerçekte geçen proof’lar, expected failure ve NOT-RUN kapıları |
| [RESEARCH](../docs/RESEARCH.md) | Sabit kaynaklar, üçüncü taraf özet/resmi rehber ayrımı ve kanıt kapsamı |

İncelemeler uygulanmış Merce koduna düzeltme yapmadı. Eski private/lifecycle/visual risklerin artık kodda çözüldüğü söylenemez.

## 6. En küçük sonraki uygulama

1. **Material3 Small Button:** resmi edisyonu sabitle; Qt + kendi Theme/Style, Merce/Runtime’sız minimal source/installed consumer. Native typography alt kümesi/sapması açık; fixed-role, caller binding, light/dark/software ve görsel karşılaştırma.
2. **Apple:** önce opak yüzey/dialog ve bağımsız own Theme/Style. Ardından macOS 26/Cocoa/Metal’de tek regular native yüzey + public lifecycle + tek owner. Gerçek masaüstü, okunaklılık/input/tercih/cleanup geçince ikinci glass bölgesi veya window backdrop.
3. **Fluent:** önce hazır Qt FluentWinUI3 ile ihtiyaç karşılaştırması. Ayrı StyleKit kitinin gerekçesi varsa seçilmiş Web-derived portable neutral/primary Button; sistem fontu ve opak yüzey. Windows-native materyal sonraki bağımsız kapı.
4. **Gallery:** sayfalar somut kitleri bilmez; root/private adaptörler seçer. Aynı kontrol kimliği ve sabit Gallery rolleriyle geçiş; eşzamanlı farklı kit karşılaştırması ayrı süreçler. Açık popup/IME/atomic frame testleri geçmeden genel destek beyanı yok.
5. **Runtime ancak gerçek açık kalırsa:** paketli ikon veya loading gibi kullanım ortaya çıktığında kalan mekanizmayı karşılaştır. Tam admission geçmediyse kod ilgili projede kalır.

Bu sıra bir implementasyon onayı veya bugün yapılmış migration değildir. Üç kitin bütün family’lerini baştan scaffold etmek, Qt’ye SDK yaması yapmak veya MButton’ı kaldırmak gerekmiyor.

## 7. Teslimat sınırı

- İlk mimari incelemede ürün kodu, SDK, MButton ve mevcut kullanıcı worktree değişiklikleri korundu; o inceleme sırasında commit atılmadı.
- Font/ikon binary’si veya yeni üçüncü taraf ürün dependency’si eklenmedi.
- Kaynak/diagnostik/packaging kanıtları [evidence](evidence) altında; tam test sınırı [VALIDATION](../docs/VALIDATION.md) içinde.
- İlk incelemedeki GitHub kurulum kapısı sonradan kapandı: iki Public repo doğrulandı, MIT ve ilk kaynak aktarımı onaylandı. Bu, bütün kitlerin ürün kabulü anlamına gelmez.

**Atlanan:** Gerçek bağımsız kit build/UI kabulü, Metal/native cam, Windows materyal ve performans testleri bu tur yapılmadı.\
**Risk:** Dar Qt mekanizma proof’u veya resmi kaynak karşılaştırması, henüz uygulanmamış tasarım sisteminin görsel/ürün desteği diye sunulmamalı.
