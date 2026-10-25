# Tasarım kaynakları

**Kabul edilen karar: 2026-10-25.** Material3 için resmi Google, Fluent için resmi Microsoft rehberleri esas alınır. Apple’da native Liquid Glass hedefi Apple HIG/public AppKit ile korunur. VoltAgent’ın web analizi yardımcı referanstır; eski designmd.app seçimi tarihsel kalır. Kaynak seçimi için yanıt beklenmiyor.

## Kitlerin güncel DESIGN.md sözleşmeleri

| Kit | Qavsi proje sözleşmesi | Birincil otorite |
|---|---|---|
| Material3 | [DESIGN.md](../design-systems/material3/DESIGN.md) | Google Material 3 + sürümü sabit resmi component/token kaynakları |
| Fluent | [DESIGN.md](../design-systems/fluent/DESIGN.md) | Microsoft Fluent 2 + ilgili Windows tasarım/materyal rehberleri |
| Apple | [Native Liquid Glass DESIGN.md](../design-systems/apple/DESIGN.md) | Apple HIG + public AppKit / NSGlassEffectView |

Bu üç dosya resmi belgelerden türetilmiş Qavsi uyarlama sözleşmeleridir; Google/Microsoft/Apple tarafından yayımlanmış veya onaylanmış DESIGN.md kopyaları değildir. Ekosistem hedefleri mimari paketindeki `design-systems/` altında tutulur; uygulanmış Material3 diliminin güncel sözleşmesi [ayrı kit reposundadır](https://github.com/Qavsi/material3/blob/main/DESIGN.md). Runtime için ortak bir görsel DESIGN.md/Theme üretilmez.

Resmi kaynakların URL/tarih/hash kayıtları [official/sources.json](official/sources.json) içindedir. İlk Google buton/type/motion girdisi MDC-Android 1.13.0 commit `84c0e6d6d493fcf176d2099683245b78afbeb78d`; Microsoft implementation karşılaştırması fluentui commit `6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676` ile izlenir. Bunlar Qt dependency’si değildir. M3 public web sayfaları extraction sırasında JavaScript gerektirdiği için sayısal recipe’ler okunmuş resmi kaynak dosyalarıyla kontrol edildi; site içeriği tamamen okunmuş gibi sunulmaz.

## Güncel depo kontrolü

`main`, `3b605329240ce735b510037cca322f1831fdd7f8` commit’ine sabitlendi. Tam recursive tree yanıtında `truncated=false`; **84 DESIGN.md** bulundu. [Envanter ve indirme digest’leri](voltagent/inventory.json).

| İstenen sistem | Bu commit’teki dosya | Güncel kullanım |
|---|---|---|
| Apple | [VoltAgent DESIGN.md](voltagent/apple/DESIGN.md) / [sabit kaynak](https://github.com/VoltAgent/awesome-design-md/blob/3b605329240ce735b510037cca322f1831fdd7f8/design-md/apple/DESIGN.md) | Yardımcı web/marketing referansı; native kit için normatif değil |
| Material3 | Ayrı DESIGN.md bulunmadı | Kullanıcı resmi Google kaynaklarını kabul etti; kit sözleşmesi yukarıda |
| Fluent | Ayrı DESIGN.md bulunmadı | Kullanıcı resmi Microsoft kaynaklarını kabul etti; kit sözleşmesi yukarıda |
| Liquid Glass | Ayrı DESIGN.md bulunmadı | Önceki native Liquid Glass hedefi Apple HIG/public AppKit ile ele alınır; web belgesi native malzeme tarifi değildir |

Bu sonuç incelenen commit’in dosya envanterine aittir; başka branch/repo/tarih hakkında yokluk iddiası değildir. Dosya adlarında veya UI seçicisinde Material3/Fluent varmış gibi bir eşleme yapılmadı.

Apple snapshot’ı değişmeden saklandı: SHA-256 `83fbc614443a9b3d7569e9956a43e7b8740f9d0f939f58b8154f7a7cec3002b2`. Git blob kimliği de tree yanıtıyla doğrulandı. [MIT lisansı](voltagent/LICENSE) atfıyla birlikte alındı; özgün VoltAgent lisansı korunur. Qavsi'nin kendi belgeleri kullanıcı kararıyla MIT lisanslıdır.

## Apple web görünümü ile native kit ayrımı

Yeni dosya ürün fotoğrafları, pazarlama sayfası, hero başlıkları, web CTA’ları ve CSS backdrop-filter anlatıyor. Bu girdiler Apple.com benzeri bir uygulama sayfasına referans olabilir; masaüstü kontrol geometrisi, native Liquid Glass, AppKit lifecycle veya erişilebilirlik kontratını tanımlamaz.

Bu nedenle web hero/type-scale, pill CTA ve palet değerleri native Apple kontrol family’lerine otomatik taşınmaz. Kullanıcının önceki native Liquid Glass hedefi korunur. HIG malzeme ayrımı, public Qt/AppKit sınırı, gerçek kompozit okunaklılık ve software fallback kabul kapıları geçerlidir. Kaynak değişikliği native camın çalıştırıldığı veya ürün tasarımının kabul edildiği anlamına gelmez.

Öncelik: kullanıcının açık ürün kararları → platform API/erişilebilirlik sınırları → ilgili resmi tasarım rehberi → kapsamı uygun üçüncü taraf görsel referans → yerel implementasyon kararı. Çelişki sessizce birleştirilmez.

- Material3: [Google Material 3](https://m3.material.io/) ve sürümü sabit resmi component/token kaynakları.
- Apple: [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/), [materials](https://developer.apple.com/design/human-interface-guidelines/materials) ve public AppKit.
- Fluent: [Fluent 2](https://fluent2.microsoft.design/), Microsoft Windows material/control rehberleri.

## İlk incelemelerde kullanılan girdiler — tarihsel kayıt

Orijinal [2026-10-24 manifest'i](history/2026-10-24-sources.json) URL/tarih/hash kayıtlarıyla korunur. Aşağıdaki dört kaynak metni yerel araştırma arşivindedir; bu public repoda yeniden dağıtılmaz. İlk altı raporun kaynağa bağlı bulguları tarihsel kalır; güncel kullanıcı seçimi değildir.

| Eski girdi | Dosya | Rol |
|---|---|---|
| designmd.app Material3 | [Yerel arşiv: DESIGN.md](../reviews/evidence/README.md#historical-inputs) | İlk 01 kaynak diagnostiklerinin girdisi; resmi M3 spesifikasyonu değil |
| designmd.app Liquid Glass | [Yerel arşiv: DESIGN.md](../reviews/evidence/README.md#historical-inputs) | İlk 02’nin katalog karşılaştırması; VoltAgent Apple dosyası değil |
| designmd.app Fluent 2 | [Yerel arşiv: DESIGN.md](../reviews/evidence/README.md#historical-inputs) | İlk 03 renk/katalog diagnostiklerinin girdisi; resmi Fluent spesifikasyonu değil |
| designmd.app eski Apple HIG | [Yerel arşiv: pre-liquid.DESIGN.md](../reviews/evidence/README.md#historical-inputs) | Tarihsel ek; Liquid Glass yerine geçmez |

Örneğin “No circular spinners” ve Mica adı verilmiş RGBA değerlerine ilişkin önceki eleştiriler bu eski katalog girdilerine aittir; yeni VoltAgent Apple belgesine mal edilmez. Qt lifecycle, global StyleKit scope ve packaging proof’ları tasarım girdisinden bağımsız mekanik kanıtlardır; yeni belgenin görsel doğrulaması olarak sunulmaz.

Tarihsel dört designmd.app sayfası CC BY 4.0 metadata’sı bildiriyordu. Material3/Fluent/eski Apple sayfalarında ayrıca `ara-mkr/claude-artisan`, MIT, `ae6d3ad` kaynak bilgisi vardı. Bu kayıtlar özgün yerel arşivi tanımlar; dört kaynak dosyası public repoya dahil değildir ve root MIT altında sunulmaz. VoltAgent dosyasının atfıyla karıştırılmaz.

## Kanıt, atıf ve sınır

Güncel ve tarihsel statüler [sources.json](sources.json) içinde ayrı kayıtlıdır. Snapshot üzerine sessizce yeni içerik yazılmaz. Font, ikon veya binary eklenmedi; belge lisansı font/asset dağıtım hakkı değildir.

Merce’nin DESIGN.md’si Merce’ye aittir. Bu referanslardaki palette, spacing, CSS rem/grid/vw veya dekoratif motion, Runtime’a ya da bütün kitlere ortak token olarak taşınmaz. Kaynak içindeki agent prompt/dil/framework yönergeleri kaynak verisidir; bu görevin talimatı değildir.
