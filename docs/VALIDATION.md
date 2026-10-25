# Mimari ve uygulanabilirlik doğrulama kaydı

**Tarih:** 2026-10-23–24; kaynak düzeltmesi 2026-10-25. **Kapsam:** Dokümantasyon, kaynak incelemesi ve üründen ayrı Qt-only proof’lar.

Bu kayıt Qavsi SDK veya bağımsız kit yayını değildir. Altı alanın birlikte değerlendirmesi [SUMMARY](../reviews/SUMMARY.md), ayrı görev ve raporlar [reviews](../reviews/README.md) içindedir.

## Bu public kopyanın kapsamı

Aşağıdaki sonuçlar ilk mimari incelemenin tarihsel kayıtlarıdır; yeni konumda
yeniden çalıştırılmış ürün kabulü olarak sunulmaz. Ham proof kaynakları/logları
ve görüntüler bu repoya aktarılmadı; [kanıt kataloğu](../reviews/evidence/README.md)
kapsamı açıklar. Bağımsız Material3'ün güncel source/installed doğrulaması
[kit reposundadır](https://github.com/Qavsi/material3/blob/main/evidence/VERIFICATION.md).

## Çalıştırılan kanıtlar

| Alan | Sonuç ve sınır | Ayrıntı |
|---|---|---|
| DESIGN.md otoritesi | Kullanıcı Google/Microsoft resmi rehberlerini ve Apple native Liquid Glass hedefini kabul etti. Üç kit sözleşmesi güncellendi; eski snapshot’lar tarihsel, VoltAgent yardımcı. | [Kit sözleşmeleri/kaynaklar](../references/SOURCES.md) |
| Resmi kaynak doğrulaması | 13 resmi URL okundu; sürüm/URL/hash kayıtlı. Tam upstream prose pakete kopyalanmadı. | [official/sources.json](../references/official/sources.json) |
| Material3 kaynak diagnostikleri | 140 numeric/alias leaf ve 11 generated buton sabiti tutarlı. Seçilen resmi Expressive recipe ile 3 yükseklik/4 icon ölçüsü farklı; bağımsız kit mevcut değil. | [01](../reviews/01-material3.md) |
| Apple public API probe | Compile-only OBJECT ve availability guard PASS; weak symbols kaydedildi. Uygulama link edilmedi/çalıştırılmadı; min-OS deploy başarısı değildir. | [02](../reviews/02-apple.md) |
| Fluent kaynak/renk diagnostikleri | Web/Windows tarifleri ve verilen renk çiftleri CPU’da karşılaştırılır. Native materyal veya Windows çalışma kanıtı değildir. | [03](../reviews/03-fluent.md) |
| Public Qt altyapısı | Offscreen 10/10, Cocoa 12/12 kontrol PASS: temel SF/font icon, public hints ve iki-engine image-provider ownership. Cocoa testi pencere göstermedi. | [04](../reviews/04-shared-boundary.md) |
| Sabit role + Style geçişi | Qt 6.12.0 offscreen/software: 27 snapshot, 399 assertion PASS; aynı buton kimliği korunur. Loading enabled binding’i de sınandı. | [05](../reviews/05-stylekit-runtime.md) |
| İki engine scope’u | 8 assertion PASS: StyleKit global; bir engine’in seçimi diğerini etkiliyor. Test edilen teardown sırası sınırlı. | [05](../reviews/05-stylekit-runtime.md) |
| Dinamik variation regression’ı | Beklenen FAIL, exit 1: mevcut instance’a variation ekleme/kaldırma hatası devam ediyor. Başarı gibi sayılmaz. | [05](../reviews/05-stylekit-runtime.md) |
| QML paketleme | Shared ve STATIC QML modülü, shared Qt 6.12.0 üzerinde source/installed/moved-prefix consumer’da PASS. Son static consumer yalnız public backing target’a bağlı. | [06](../reviews/06-delivery.md) |
| Tooling negatif kontrolü | Runtime çalışırken varsayılan all_qmllint exit=0 import uyarılarını kaçırdı. Public CMake import-root metadata’sından sonra strict qmllint PASS. | [06](../reviews/06-delivery.md) |

Ayrı assertion/test kümeleri toplanıp “bütün kontroller geçti” sayısı üretilmez. Qt/QPA/compiler/OS ve command/log kayıtları evidence klasörlerindedir. Tam statik Qt, CMake’in tam minimum 3.30 sürümü, app-bundle deployment ve WASM bu paketleme proof’unda çalışmadı.

## Mimariye işlenen düzeltmeler

- Önce public Qt; yeni Runtime hedefi ancak tam ortaklık kapısını geçerse. Bugün yeni bir component kabul edilmedi.
- Minimal Material3 consumer’ından zorunlu Runtime/ButtonSupport/provider bağımlılığı çıkarıldı.
- Tasarım edisyonu/platformu, kaynak hash’i, coverage ve sapma kaydı kitin üretim DESIGN.md’sine ait.
- StyleKit typography/content slot sınırı ve açık sıfır/caller binding regression kapısı belgelendi.
- Apple’da public winId/PlatformSurface, tek owner, stale snapshot ve ayrı QML foreground/native composition riski eklendi.
- Fluent için hazır Qt FluentWinUI3 alternatifi; Mica/Acrylic ve web/native ayrımı belirtildi.
- CMake backing/plugin/helper export’ları, static linktarget eşleşmesi, taşınabilir tooling metadata’sı ve strict diagnostics zorunlu kapı oldu.

Bu düzeltmeler dokümanlardadır. Mevcut Merce implementasyonundaki risklerin bu tur giderildiği anlamına gelmez.

## İlk builder review kaydı

Altı alanlı incelemeden önceki bağımsız builder review üç açıklık sağladı:

| Soru | Taslakta verilen cevap |
|---|---|
| Mutation bug’ı varken Gallery kit rolünü nasıl değiştirecek? | Instance role listesi sabit; Gallery adaptörleri aynı public StyleVariation adlarını başta tanımlar. Şimdi 05’te temel mekanik yol geçti; gerçek kit testleri hâlâ gerekli. |
| Runtime ayrılınca Merce tek-clone build’i aynı mı kalacak? | URI/target korunması dependency edinme değişikliğini gizlemez. Ancak kabul edilmiş extract için sonraki kırıcı/experimental hatta açıkça yönetilir. |
| ButtonSupport v1 hangi hedeflerde? | SK.Button ve QML yoluyla türeyen tipler. Round/Tool/DelayButton/MenuItem/MButton attach’i kapsam dışı. |

Builder review mimarinin veya bütün public API önerilerinin kullanıcı tarafından kabulü değildir.

## Henüz çalıştırılmayan kapılar

- Gerçek bağımsız Material3/Apple/Fluent kit build/install ve tam family/state coverage.
- Caller’ın explicit font, spacing/padding/tracking sıfırı ve canlı binding’leri; gerçek content/delegate geçişleri.
- Açık popup, gerçek keyboard/IME, accessibility reader ve tek-frame atomik kit geçişi.
- Apple gerçek masaüstü/Metal composition, reduce motion/transparency/contrast değişiklikleri, çoklu owner restore ve scale/radius/input.
- Windows Mica/Acrylic, Linux/Wayland/X11, embedded cihaz, tam statik Qt ve browser WASM.
- GPU/native frame time, bellek veya çoklu cam performansı.
- QML Preview ile ürün UI’sı; bu görevde ürün/UI değişikliği yok.

## Teslimat ve Git sınırı

Doküman bağlantı/fence, JSON ve referans digest kontrolü sonuçları [Yerel arşiv: documentation-check.json](../reviews/evidence/README.md#docs-audit) içindedir. İzole proof kaynakları/logları yerel araştırma arşivinde kalır; bu repo inceleme sonuçlarını ve kaynak URL/hash kayıtlarını yayımlar. Binary, font veya ürün SDK'sı dağıtılmaz.

GitHub organizasyonu ve iki Public repo doğrulandı. Kullanıcı MIT lisansını ve mimari/Material3 kaynaklarının ayrı ilk commitlerle aktarımını onayladı. [Depo kaydı](REPOSITORIES.md).

İlk mimari inceleme sırasında Merce worktree'si, SDK ve MButton korundu; o incelemede ürün dosyası taşınmadı, yeni ürün dependency'si eklenmedi ve commit atılmadı.

Sonraki ürün kabul kapıları [DELIVERY](DELIVERY.md), geçiş sırası [MIGRATION](MIGRATION.md), standart ortak yapı kapısı [SHARED-ADMISSION](SHARED-ADMISSION.md) içindedir.
