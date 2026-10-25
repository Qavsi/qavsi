# Altı alanlı mimari ve uygulanabilirlik incelemesi

**Talep:** Mevcut Material3/Apple/Fluent DESIGN.md kaynakları kullanılsın; altı bağımsız alan mimariyi ve uygulanabilirliği sınasın. Standart olduğu kanıtlanan mekanizmalar Qavsi’ye alınsın; diğerleri kit/uygulamada kalsın.

**Yürütme:** En fazla üç inceleme eşzamanlı; iki dalga. Bu tur ürün migrasyonu, SDK yaması, MButton kaldırma, native masaüstü etkileşimi veya Git commit’i içermez. İzole proof’lar ayrı geçici klasörlerde çalışır.

**Durum:** Altı görev tamamlandı. [Ortak sonuç, düzeltmeler ve sonraki dilim](SUMMARY.md) incelemeye hazır; overall karar **CONDITIONAL**, yeni koşulsuz Runtime kabulü yok.

**2026-10-25 kabul edilen kaynak kararı:** Google Material 3, Microsoft Fluent 2/Windows ve Apple native Liquid Glass için HIG/public AppKit esas alınır. Üç [kit DESIGN.md sözleşmesi](../references/SOURCES.md) bu kararla güncellendi. İlk altı raporun katalog girdileri tarihsel, VoltAgent Apple web belgesi yardımcı referanstır; [Apple ek incelemesi](apple-source-addendum.md) ayrımı kaydeder. Eski Qt/AppKit/packaging proof’ları yeni görünümü doğrulamış veya yeniden çalıştırılmış sayılmaz.

## Görevler

| Alan | Ayrı sorumluluk | Beklenen kanıt | Rapor |
|---|---|---|---|
| 01 Material3 | Kendi token/Theme modeli; family/state/delegate kapsamı; mevcut Merce bağımlılıkları | Mevcut DESIGN.md + Google kaynakları + source/file eşlemesi | [01-material3.md](01-material3.md) |
| 02 Apple | HIG katman ayrımı; native materyal ownership; platform fallback ve erişilebilirlik | Mevcut DESIGN.md + public Apple/Qt kaynakları + native sınırlar | [02-apple.md](02-apple.md) |
| 03 Fluent | Fluent 2 token/geometry/state; Mica/Acrylic ayrımı; font/deploy ve portable kapsam | Mevcut DESIGN.md + Microsoft kaynakları; hedef OS matrisi | [03-fluent.md](03-fluent.md) |
| 04 Ortak altyapı | Her Runtime adayının standartlık ve gereklilik sınaması | Public Qt/OS API karşılaştırması; gerekli yerde Qt-only proof | [04-shared-boundary.md](04-shared-boundary.md) |
| 05 StyleKit runtime | Sabit role + aynı kontrol üzerinde Style değişimi; global scope; binding/guard | Qt 6.12.0 executable assertion/pixel kanıtı | [05-stylekit-runtime.md](05-stylekit-runtime.md) |
| 06 Dağıtım | Source/installed target; optional dependency; QML metadata; prefix/static | İzole CMake build/install/consumer smoke proof’u | [06-delivery.md](06-delivery.md) |

## Sonuçları okuma

- **PASS:** Sadece belirtilen test veya kaynak sözleşmesi geçti.
- **CONDITIONAL:** Yaklaşım uygulanabilir; listelenen kapılar tamamlanmalı.
- **FAIL:** Sınanan beklenti kanıtla çelişti.
- **BLOCKED:** Somut ortam/bağımlılık engeli nedeniyle test yapılamadı.
- **NOT-RUN:** Bu tur çalıştırılmadı; başarı kabul edilmez.

Kaynak incelemesi bir executable testi değildir. Offscreen/software testi Metal/native cam veya Windows Mica kanıtı değildir. Bir packaging prototipi bütün Qavsi kitlerinin kurulu paketinin çalıştığını göstermez.

Paket içindeki belge/kanıt bağlantıları görelidir. Mevcut Merce kaynaklarına atıflar bu incelemenin yerel checkout yollarını ve satırlarını kullanır; ilgili provenance hash’leri snapshot’ı tanımlar. Bunlar taşınabilir SDK dependency’si veya henüz yayımlanmamış bir GitHub permalink’i değildir. Proof’ları yeniden çalıştırmak için raporda belirtilen Qt toolchain’i ve gerektiğinde aynı Merce kaynak snapshot’ı gerekir.

## Ortak modül için kabul ölçütü

1. Aynı ihtiyaç Qt veya standart public platform API’siyle doğrudan çözülüyorsa önce o kullanılır; isim değiştiren wrapper eklenmez.
2. Ortak adayın anlamı, state/ownership/lifecycle’ı kitler arasında aynı olmalı.
3. Kit token, tipografi, geometri, variation katalogu veya navigation politikası içermemeli.
4. Mevcut Qt’ye eklediği dar yetenek somut bir tüketici ihtiyacıyla gösterilmeli.
5. Opsiyonel dependency, source/installed tüketim ve fallback sınırı doğrulanmalı.
6. Yalnız benzer kod veya iki tüketici bulunması yeterli değil; standart sözleşme ve test kanıtı gerekir.
7. Kanıt yoksa aday kit/proje içinde kalır veya ertelenir. Runtime kataloğunda adı bulunması kabul edilmiş olduğu anlamına gelmez.

Kullanılan byte-identical kaynaklar ve atıflar [SOURCES](../references/SOURCES.md), machine-readable digest’ler [sources.json](../references/sources.json) içindedir.


## Public kopya sınırı

Bu raporlar ilk araştırmanın tarihsel sonuçlarıdır. Ham proof kaynakları, loglar,
ekran görüntüleri, yerel SDK kopyaları ve upstream implementation snapshot'ları
bu repoya dahil değildir. Eski atıflar [kanıt kataloğuna](evidence/README.md)
yönlenir; bu bağlantılar ham kanıtın indirilebildiği iddiası değildir.
