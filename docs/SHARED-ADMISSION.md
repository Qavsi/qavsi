# Ortak yapı kabul sözleşmesi

**İlke:** Kullanıcı kararı: standart ortak mekanizmalar Qavsi’ye alınabilir; aksi halde ilgili kit veya uygulama içinde kalır. Aşağıdaki kapılar bu ilkenin önerilen teknik uygulamasıdır.

## 1. Karar sırası

1. **Doğrudan Qt / public platform özelliği:** İhtiyaç zaten karşılanıyorsa onu kullan. Yeni isim veren wrapper veya yinelenen state modeli üretme.
2. **Kit ya da uygulama davranışı:** Görünüm, token semantiği, geometri, interaction tercihi veya ürün politikası içeriyorsa kendi projesinde tut.
3. **Dar ortak mekanizma adayı:** Qt’nin karşılamadığı farkı ve aynı sözleşmeye ihtiyaç duyan tüketicileri göster.
4. **Kanıtlanan ortak modül:** Aşağıdaki kapıları geçerse Qavsi’de seçilebilir bir module/target olarak çıkar.

Benzer kod, aynı alan adı veya iki tüketici bulunması tek başına yeterli değildir. Yeni kitte kullanılabilir görünmesi de gerçek tüketici kanıtı yerine geçmez.

## 2. Zorunlu kapılar

| Kapı | Aranan kanıt | Geçmiyorsa |
|---|---|---|
| Standart sözleşme | Public Qt/OS standardı veya kitlerden bağımsız açık ve sınanabilir davranış; kaynağı ve sürümü belli | Kit/uygulama içinde kalır |
| Ek değer | Qt’nin doğrudan sunduğu yol ile karşılaştırma; yalnız eksik dar yeteneğin tanımı | Qt kullanılır; wrapper eklenmez |
| Aynı anlam | Tüketicilerin aynı girdiyi, sonucu, hatayı, lifecycle’ı beklediği gösterilir | Ayrı implementasyonlar kabul edilir |
| Görsel bağımsızlık | Ortak şemada kit token, font ölçeği, radius, variation adı veya navigation politikası bulunmaz | Kitte tutulur |
| Gerçek tüketim | Varsayımsal gelecek kit yerine çalıştırılabilir tüketici veya açık public-standard conformance örneği | Aday olarak ertelenir |
| Sahiplik ve fallback | QObject/engine/window/thread sahibi; unavailable/error/cleanup davranışı testli | Extract edilmez |
| Dağıtım | Optional dependency; source/installed target ve QML import doğruluğu | Public modül yayımlanmaz |
| Değişim maliyeti | API/ABI/version ve lisans/asset sınırı; ortaklaştırmanın bakım bedeli | Yerel tutmak tercih edilir |

Bu ölçütler iki farklı kitin aynı görünmesi gerektiği anlamına gelmez. Aynı mekanizmayı kullanmak, aynı Theme veya görsel tarif kullanmak değildir.

## 3. Kabul durumları

- **USE EXISTING QT:** Doğrudan Qt API’si kullanılır; yeni Qavsi public API’si yok.
- **QUALIFIES QAVSI:** Kanıtlanan dar mekanizma; extract etmek için ayrıca implementasyon planı gerekir.
- **KEEP KIT/APP:** İlgili tasarım sistemi veya uygulama sahiplenir.
- **DEFER:** İhtiyaç veya ortak sözleşme henüz yeterince kanıtlanmamış.

Önceki Runtime modül kataloğu **aday kataloğudur**. İçinde bir URI/target adı olması kabul kararı değildir. Aynı isimlerin hiç yayımlanmaması mümkündür.

## 4. Somut aday değerlendirme formu

Her extract önerisi tek kısa kayıtla değerlendirilir:

- İhtiyaç ve gerçek consumer’lar.
- Mevcut Qt/public platform alternatifi ve neden yeterli/yetersiz.
- Ortak mekanizmanın girdileri, çıktıları, hata ve ownership modeli.
- Kitte kalan politika/token/renderer alanları.
- Kullanılan public API ve desteklenen sürüm/platform.
- Çalıştırılan proof/contract test’leri ve NOT-RUN kapsam.
- Optional build/install/asset sınırı.
- Kabul durumu, gerekçe ve yeniden değerlendirme koşulu.

Bir wrapper’ın test geçmesi, wrapper’ın gerekli olduğunu kanıtlamaz. Önce gereklilik ve standartlık, sonra implementasyon doğruluğu değerlendirilir.

## 5. İlk doğrulama stratejisi

[Altı alanlı inceleme](../reviews/README.md) mevcut adayları bu ölçütlerle sınar. Kaynak incelemesi, Qt-only public API proof’u, StyleKit runtime proof’u ve packaging proof’u ayrı kanıt türleridir.

Runtime tarafına genel Theme, evrensel variation kataloğu, badge/button/control ailesi veya kit seçme registry’si taşınmaz. Gallery’nin ortak örnek adları ve sunum adaptörleri kendi uygulamasında kalır.

Pencere sistem taşıma/resize, native menü, sistem sembolleri veya OS motion/contrast için doğrudan public Qt yeterliyse Qavsi katmanı eklenmez. Gerçek bir public API açığı varsa yalnız o eksik dar mekanizma aday olur; bütün Platform adı altında geniş bir abstraction otomatik kabul edilmez.
