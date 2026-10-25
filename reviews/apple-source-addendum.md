# Apple kaynağı düzeltmesi — ek inceleme

**Tarih:** 2026-10-25. **Karar:** Kaynak eşleşmesi **PASS**; Apple web görünümü için başlangıç girdisi **CONDITIONAL**; masaüstü Liquid Glass/HIG spesifikasyonu olduğu varsayımı **FAIL**.

**Son kullanıcı kararı:** Apple native Liquid Glass hedefi korunur; [güncel DESIGN.md](../design-systems/apple/DESIGN.md) HIG/public AppKit’i esas alır. Material3/Fluent otoriteleri Google/Microsoft olarak kabul edildi. Bu ek rapor web dosyasının kapsamını kaydeder; güncel kit sözleşmelerinin veya native görünümün test sonucu değildir.

Kullanıcının belirttiği depo [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md/tree/3b605329240ce735b510037cca322f1831fdd7f8), sabit commit `3b605329240ce735b510037cca322f1831fdd7f8`. İncelenen [Apple dosyasının](../references/voltagent/apple/DESIGN.md) SHA-256 değeri `83fbc614443a9b3d7569e9956a43e7b8740f9d0f939f58b8154f7a7cec3002b2`; yerel byte hash’i doğrulandı. Kaynak metnin lisansı [MIT, © 2026 VoltAgent](../references/voltagent/LICENSE).

## Yeni girdinin gerçek kapsamı

Dosya **apple.com ürün pazarlama/store web arayüzünün üçüncü taraf analizidir**. Kendi açıklaması web varlığını anlatıyor; incelenen sayfalar homepage, environment, store, iPhone satın alma ve accessories. [DESIGN:278](../references/voltagent/apple/DESIGN.md), [kaynak sayfalar:296](../references/voltagent/apple/DESIGN.md), [sabit upstream](https://github.com/VoltAgent/awesome-design-md/blob/3b605329240ce735b510037cca322f1831fdd7f8/design-md/apple/DESIGN.md#L276)

Cam örnekleri iki web barındaki CSS `backdrop-filter` tarifidir; AppKit `NSGlassEffectView`/`NSVisualEffectView`, macOS availability veya pencere lifecycle sözleşmesi verilmez. [Blur:397](../references/voltagent/apple/DESIGN.md), [sub-nav:434](../references/voltagent/apple/DESIGN.md), [sticky bar:476](../references/voltagent/apple/DESIGN.md). CSS blur, native Liquid Glass eşdeğerliği kanıtı değildir.

## Eski girdiden anlamlı farklar

| Alan | Eski designmd.app Liquid Glass | Yeni VoltAgent Apple |
|---|---|---|
| Optik | Morphing, iridescence ve chromatic aberration önerisi. [Yerel arşiv: Eski:111](evidence/README.md#historical-inputs) | Dekoratif gradient yok; shadow ürün fotoğrafına ayrılıyor. [Yeni:326](../references/voltagent/apple/DESIGN.md), [:400](../references/voltagent/apple/DESIGN.md) |
| Renk | Saf siyah yasak; vurgu `#0066FF`. [Yerel arşiv: Eski:103](evidence/README.md#historical-inputs) | Siyah global nav; Action Blue `#0066cc`. [Yeni:299](../references/voltagent/apple/DESIGN.md), [:498](../references/voltagent/apple/DESIGN.md) |
| Tipografi | Body 16px; display weight 700. [Yerel arşiv: Eski:49](evidence/README.md#historical-inputs) | Body 17px, headline ağırlıkla 600; SF Display/Text ve tracking tarifleri. [Yeni:339](../references/voltagent/apple/DESIGN.md), [:346](../references/voltagent/apple/DESIGN.md) |
| Buton | Orta radius, hover lift, basışta translation. [Yerel arşiv: Eski:91](evidence/README.md#historical-inputs) | Pill CTA ve basışta `scale(0.95)`. [Yeni:438](../references/voltagent/apple/DESIGN.md) |
| Spinner | Circular spinner yasağı var. [Yerel arşiv: Eski:96](evidence/README.md#historical-inputs) | Spinner/loading sözleşmesi yok; eski yasak yeni kaynağa atfedilemez. |

Bu iki estetik sessizce birleştirilmemeli. Önceki 02 raporundaki **eski kaynağa özgü** zorunlu morphing, spinner yasağı, saf siyah yasağı, Lucide/Heroicons önerisi ve 16px tipografi eleştirileri yeni dosyanın bulguları değildir. Yeni kaynağın basış scale’ını bütün macOS kontrollerinin platform standardı saymak da doğru olmaz.

## Önceki 02 sonuçlarının durumu

| Önceki sonuç | Yeni kaynak sonrası değerlendirme |
|---|---|
| R02-01 — `QWindow::handle()` internal; belgeli native handle/surface sınırı | **Geçerli teknik sınır.** Qt kaynağına dayanır; görsel kaynak değişimi bunu değiştirmez. Yeni kit doğrulandı anlamına gelmez. |
| R02-02/03 — Tek owner, güvenli restore, Qt/Metal kökünü koruma; QML foreground’un native `contentView` dışında kalması | **Native entegrasyon seçilirse geçerli.** Yeni dosya bu entegrasyonu gerektirmiyor; web barındaki blur’dan AppKit ihtiyacı çıkarılamaz. |
| R02-04/05 — Transform/radius, per-window DPR, preference unknown ve observer cleanup | **Koşullu teknik riskler geçerli.** Önceki kaynak incelemesi kapsamındalar; yeni kaynak veya yeni çalıştırma kanıtı değiller. |
| HIG functional/content malzeme ayrımı, dialog okunabilirliği | **Masaüstü HIG uyumu hedeflenirse geçerli.** Web referansının uygulanmış özelliği veya native optik doğrulaması diye sunulamaz. |
| Tipografi ilkeleri | **Sistem font API’si, okunabilirlik, locale/ölçek ve varlık lisansı sınırı geçerli.** Eski 16px yorumu emekli. Yeni 17px, CSS stack/tracking veya Inter önerisi Qt’ye otomatik aktarılmaz; kitin seçilmiş görünüm politikasıdır. |
| Kit bağımsızlığı ve ortak bridge kararı | **Geçerli.** Theme/recipe/yerleşim kit/uygulamada; Qt’nin sunduğu mekanizma doğrudan Qt’de. Ortak native bridge **DEFER**. Yeni dosya aynı lifecycle’a ihtiyaç duyan ikinci gerçek consumer kanıtı sağlamıyor. |

Önceki teknik kanıtların sınırı [02 raporu](02-apple.md) ve [Yerel arşiv: provenance](evidence/README.md#review-02) içinde değişmeden kalır. Önceki availability derlemesi yalnız public API bildirimi içindi; bu yeni tasarım girdisini veya ondan üretilecek kiti doğrulamaz.

## Planlamaya etkisi ve açık sınır

Bu görüşmedeki native Apple Liquid Glass hedefi kullanıcı tarafından daha önce seçildi ve korunur. Yeni web dosyası bu hedef için kapsamı sınırlı bir görsel referanstır; native glass ihtiyacı bu dosyadan türetilmez. Önceki 02’nin native dilimi mevcut kullanıcı hedefi için koşullu spike olarak kalır; web belgesi bunu doğrulamış veya yerini almış sayılmaz. Ortak Runtime’a yeni Theme, palet, spacing, radius, hover veya material API’si kabul edilmez.

Kaynak kendi eksiklerini de belirtiyor: validation/error state ve bazı dark-mode karşılıkları yok; blur değeri token olarak kesinleştirilmemiş. [Known Gaps:557](../references/voltagent/apple/DESIGN.md). `Iteration Guide` içindeki hover belgelemesini yasaklayan ifade kaynak verisidir; görevin veya Qt kitinin talimatı değildir. [DESIGN:550](../references/voltagent/apple/DESIGN.md)

[Envanter](../references/voltagent/inventory.json) tam, 84 DESIGN.md içeriyor; ayrı Material3/Fluent/Liquid Glass girdisi bulunmuyor. Bu ek rapor diğer kitler için kaynak seçmez veya eski kaynağı kullanıcı onayı yerine koymaz.

**Doğrulama:** Yeni Apple dosyası ve lisans okundu; hash ve envanter tutarlılığı kontrol edildi. Ürün/SDK/OS/UI değiştirilmedi; native proof, build, runtime veya görsel test tekrar çalıştırılmadı.

Atlanan: yeni kaynaktan kit implementasyonu ve masaüstü uygunluk testleri. Risk: Apple web pazarlama analizi, macOS HIG veya Liquid Glass destek kanıtı olarak etiketlenmemeli.
