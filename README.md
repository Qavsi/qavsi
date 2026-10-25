# Qavsi
**Design systems for Qt Quick**

Qt 6.12+ üzerinde bağımsız tasarım sistemleri ve küçük, isteğe bağlı ortak altyapı modülleri için mimari öneri.

**Tarih:** 2026-10-24; kaynak düzeltmesi 2026-10-25\
**Durum:** İncelemeye hazır taslak; uygulama kodu veya yayınlanmış SDK değildir.\
**Karar sahibi:** Enes Alp. Taslaktaki teknik öneriler kullanıcı tarafından ayrıca kabul edilmedikçe kabul edilmiş karar sayılmaz.

## Okuma sırası

Önce [altı incelemenin ortak sonucu](reviews/SUMMARY.md): hangi varsayımlar geçti, hangi eksikler bulundu ve ilk bağımsız dilim nasıl başlamalı?

1. [Mimari](ARCHITECTURE.md): konumlandırma, bağlam, modüller, çalışma senaryoları, kalite hedefleri.
2. [Kontrol ve entegrasyon sözleşmeleri](docs/CONTRACTS.md): StyleKit, ButtonSupport, ikonlar, pencere malzemeleri, erişilebilirlik.
3. [Derleme, dağıtım ve doğrulama](docs/DELIVERY.md): CMake paketleri, QML modülleri, statik bağlama, CI, sürümleme.
4. [Merce’den geçiş](docs/MIGRATION.md): mevcut dosyaların hedefi, küçük geçiş dilimleri, geri dönüş.
5. [Karar kayıtları](docs/DECISIONS.md): alternatifler, gerekçeler ve bedeller.
6. [Araştırma ve kanıtlar](docs/RESEARCH.md): incelenen sürümler, birincil kaynaklar, doğrulanan sınırlar.
7. [Yeni kit geliştirme](docs/KIT-AUTHORING.md): sıfırdan kit ekleme, Gallery adaptörü ve dışarıdan özelleştirme.
8. [Doğrulama kaydı](docs/VALIDATION.md): yapılan kontrol, builder review sonucu ve henüz çalıştırılmamış testler.
9. [Tasarım kaynakları ve DESIGN.md sözleşmeleri](references/SOURCES.md): kabul edilen resmi Google/Microsoft/Apple otoriteleri, yardımcı web referansı ve tarihsel girdiler.
10. [Ortak yapı kabulü](docs/SHARED-ADMISSION.md): Qt’nin mevcut işlevi, kit politikası ve Qavsi adayı ayrımı.
11. [Altı alanlı inceleme](reviews/README.md): bağımsız sorumluluklar, test/kanıt sınırları ve raporlar.
12. [GitHub başlangıç planı](docs/REPOSITORIES.md): ilk iki repo ve yayın öncesi kararlar.

## Kabul edilen başlangıç

- Çatı adı Qavsi.
- Merce kendi tasarım sistemi olarak kimliğini korur.
- Material3, Apple ve ileride Fluent kendi token ve Theme modellerinin sahibi olur.
- Ortak parçalar tasarım sistemlerinden bağımsız modüller olarak ayrılır.
- Yalnız standart olduğu ve gerekli ek değeri kanıtlanan ortak mekanizma Qavsi’ye alınır; diğerleri kit/uygulamada kalır.
- Material3 için resmi Google, Fluent için resmi Microsoft rehberleri esas alınır. Apple’da native Liquid Glass hedefi HIG/public AppKit ile korunur; VoltAgent web analizi yardımcı referanstır.
- Qt 6.12 taban alınır.
- Mevcut SDK değiştirilmez. Bu taslak MButton’ı kaldırmaz veya mevcut uygulamayı taşımaz.

## Önerilen depo sınırları

Kitlerin güncel görsel sözleşmeleri: [Material3 DESIGN.md](design-systems/material3/DESIGN.md), [Fluent DESIGN.md](design-systems/fluent/DESIGN.md), [Apple native Liquid Glass DESIGN.md](design-systems/apple/DESIGN.md). Bunlar ekosistem hedef sözleşmeleridir. Material3'ün uygulanmış ilk diliminin güncel görsel sözleşmesi [kit reposunun DESIGN.md'sindedir](https://github.com/Qavsi/material3/blob/main/DESIGN.md). Bu mimari reposu ürün modülü veya zorunlu superbuild içermez.

| Depo | Sorumluluk | Durum |
|---|---|---|
| `qavsi` | Bu mimari, karar kayıtları ve ekosistem yol haritası | MIT lisanslı mimari deposu |
| `runtime` | Yalnız kabul kapılarını geçen dar ortak mekanizmalar | İlk kanıtlanmış ihtiyaç oluştuğunda; bugün yeni kabul yok |
| `merce` | Merce token’ları, Theme, Style ve Merce’ye özgü bileşenler | Mevcut projenin kontrollü geçişi |
| `material3` | Bağımsız Material3 Theme ve StyleKit uygulaması | İlk bağımsız kit |
| `apple` | Apple görünümü, platform malzemesi politikası ve fallback | Ayrı kit |
| `gallery` | Paketleri kullanan örnekler, karşılaştırma ve entegrasyon doğrulaması | Gerçek tüketici |
| `fluent` | Bağımsız Fluent uygulaması | Henüz açılmayacak |

Bu liste boş kütüphane depoları üretme talimatı değildir. Önce organizasyon ve dokümantasyon merkezi; sonra doğrulanmış ilk modül. Yerel çalışma klasörü Git deposu veya zorunlu superbuild olmaz.

[Qavsi organizasyonu](https://github.com/Qavsi) kullanıcı tarafından oluşturuldu
ve 2026-10-25 tarihinde doğrulandı. Bağımsız Material3 Small Button dilimi yerelde
uygulandı; source/installed ve macOS Metal/software kanıtları kitin kendi
`evidence/` dizininde tutuluyor. Kullanıcının seçimiyle
[Qavsi/qavsi](https://github.com/Qavsi/qavsi) ve
[Qavsi/material3](https://github.com/Qavsi/material3) Public olarak oluşturuldu.
Kullanıcı MIT lisansını ve ilk kaynak commitlerinin yayımlanmasını onayladı. Mimari belgeler bu repoda, bağımsız ilk kit Material3 reposunda tutulur. [Aktarım kapsamı](docs/PUBLICATION.md), [lisans ve referanslar](THIRD_PARTY_NOTICES.md).

## En önemli sınırlar

Runtime URI/target listeleri kabul edilmiş ürün kapsamı değil, [kabul kapısına](docs/SHARED-ADMISSION.md) tabi adaylardır.

- Qavsi Runtime’da ortak `Theme.colors`, `spacing.medium` veya evrensel typography ölçeği bulunmaz.
- Bağımsız kitler `MerceStyle`’dan türemez; Qt StyleKit’in `Style` tipinden başlar.
- Qt 6.12.0 StyleKit stili **uygulama genelinde** tutar. İki QQmlEngine bağımsız stil izolasyonu sağlamaz.
- `StyleKit.style` kontrol görünümünü değiştirir; uygulamanın navigation düzenini, sayfa yerleşimini veya native pencere malzemesini kendiliğinden yeniden tasarlamaz.
- Kontrol davranışı Qt’de kalır. Yeni `Qavsi.Button` / `Qavsi.ComboBox` ailesi üretilmez.
- Runtime variation değişimi, QTBUG-148461 için yayımlanan minimal örnek geçmeden desteklenmiş sayılmaz.

## Bu teslimatta yapılan doğrulama

Altı incelemenin geçici katalog girdileri tarihsel korundu. Kullanıcı son kararıyla Google/Microsoft resmi rehberlerini ve Apple native Liquid Glass hedefini kabul etti. Üç kitin DESIGN.md sözleşmesi bu karara göre yazıldı; resmi kaynakların URL/sürüm/hash’i kaydedildi. Kaynağa bağlı eski bulgular yeni belgeye aktarılmaz; yeni sözleşmeler ürün/render kabulü değildir. [Güncel kaynak durumu](references/SOURCES.md)

Qt 6.12.0 offscreen/software proof’unda sabit role sahip aynı butonun Style/theme/state geçişi 399 assertion ile geçti; iki engine’in global StyleKit’i paylaştığı ayrıca doğrulandı. Shared ve STATIC QML modülleri source/installed/taşınmış-prefix consumer ile geçti; tooling için public import-root metadata’sı ve strict qmllint gereksinimi bulundu. Dinamik variation regression’ı beklenen şekilde hâlâ başarısız. Bunlar [dar mekanizma testleridir](docs/VALIDATION.md); gerçek bağımsız kit, native cam/Windows materyali veya performans başarısı değildir.

KDE Kirigami ve Quickshell sabit commit’lerde mimari referanstır; zorunlu dependency veya kopyalanmış framework değiller. Ortak Qavsi Runtime SDK'sı oluşturulmadı; ilk bağımsız Material3 Small Button dilimi ayrı repoda geliştirilir.

Merce’nin mevcut worktree değişiklikleri, SDK kurulumu ve MButton korunur. Ürün koduna üçüncü taraf implementasyon vendored edilmedi. MIT lisanslı VoltAgent referansı özgün lisansıyla korunur. Diğer tarihsel kaynakların URL/hash kayıtları ve inceleme sonuçları yayımlanır; yerel proof kaynakları, ham loglar, ekran görüntüleri ve vendor snapshot'ları bu repoya dağıtılmaz. [Kanıt kataloğu](reviews/evidence/README.md).
