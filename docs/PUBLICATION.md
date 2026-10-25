# İlk mimari kaynak aktarımı

Kullanıcı 2026-10-25 tarihinde iki Public repo için MIT lisansını ve ilk
commitlerin push edilmesini onayladı. Copyright: 2026 Enes Alp.

## Dahil edilen içerik

- Mimari, ADR, kit yazarlığı, teslimat ve geçiş sözleşmeleri.
- Material3, Apple ve Fluent ekosistem hedef DESIGN.md sözleşmeleri.
- Altı alan incelemesinin yazılı sonuçları.
- İncelenen kaynakların sabit URL/sürüm/hash manifestleri.
- VoltAgent Apple web referansı ve özgün MIT lisansı.

Material3'ün uygulanmış kit kodu, public target/URI, örnek ve davranış testleri
[Qavsi/material3](https://github.com/Qavsi/material3) reposundadır.
Bu mimari repo Runtime veya kitin zorunlu build bağımlılığı değildir.

## Public repoya alınmayan araştırma dosyaları

Hesap kurulum ekranı ve üretilmiş ZIP/arşivler; ham loglar/stderr; ekran
görüntüleri; çalıştırılabilir proof kaynakları; Qt/Google implementation
snapshot'ları ve dört tarihsel designmd.app kaynak metni dağıtılmaz.
Orijinal araştırma dosyaları kaynak çalışma alanında korunur; silinmedi.

Eski raporlardaki bu dosya atıfları [kanıt kataloğuna](../reviews/evidence/README.md)
yönlendirilir. URL/hash manifestlerindeki `file`, `path` veya `copy` alanları
orijinal araştırma düzenini tanımlar; bu dosyaların public repoda bulunduğunu
veya kanıtların bu aktarım sırasında yeniden çalıştırıldığını iddia etmez.
Katalog atıf adlarını korur; kişisel makine dizin önekleri genel adlarla gösterilir.

## Lisans sınırı

Bize ait belgeler MIT'dir. [Üçüncü taraf atıfları](../THIRD_PARTY_NOTICES.md)
ve VoltAgent'ın özgün lisansı korunur. Dış kaynakların kopyalanmaması,
upstream lisanslarını MIT'ye dönüştürmez.

İlk commitler source aktarımıdır; stable SDK/tag veya binary paket yayını değildir.
