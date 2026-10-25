# 01 — Material3: bağımsız tasarım sistemi ve Qt 6.12 uygulanabilirliği

**2026-10-25 kaynak notu:** Aşağıdaki DESIGN.md karşılaştırmaları ilk incelemenin geçici designmd.app girdisine aittir. Kullanıcı resmi Google rehberlerini esas aldı; [güncel Material3 sözleşmesi](../design-systems/material3/DESIGN.md) bu kararı izler. Google/Qt/mevcut Merce kaynak bulguları kendi sürüm kapsamıyla geçerlidir; yeni sözleşmenin render testi sayılmaz. [Güncel kaynak kaydı](../references/SOURCES.md).

**Tarih:** 2026-10-23. **Karar: CONDITIONAL.** Önerilen `Qt + kitin kendi Theme’i + doğrudan SK.Style` sınırı uygulanabilir. Mevcut `Merce.Style.Material3` bu sınırı sağlamıyor; onun bağımsızlık sonucu **FAIL**. Bu inceleme yeni Material3 paketini derlemedi veya çalıştırmadı; üretim/render uyumu için PASS verilmedi.

İlk bağımsız dilim Runtime paketi gerektirmeden başlayabilir. Renk sistemi, tipografi, ölçüler, state kompozisyonu, motion, ripple ve floating label Material3 kitinde kalmalı. Aynı görünen helper’lar ortak Runtime kabulü anlamına gelmez.

## Kapsam, kaynak ve sürüm sınırı

`ARCHITECTURE.md`, `docs/CONTRACTS.md`, `KIT-AUTHORING.md`, `DELIVERY.md`, `MIGRATION.md`, `SHARED-ADMISSION.md`; mevcut Merce Material3/Theme/action kaynakları, generated recipe’ler, `AppLabel` ve ilgili test kaynakları incelendi. Merce Git kökü `MERCE_REPOSITORY`; bu raporun mimari belge kökü `MERCE_SOURCE/qavsi-design`.

| Girdi | İncelemede kullanılan kesin kimlik | Yetki / sınır |
|---|---|---|
| Kullanıcının mevcut Material3 DESIGN.md girdisi | [Yerel arşiv: Yerel dosya](evidence/README.md#historical-inputs), SHA-256 `1ea35e38148f7173e5459e57f7cb9f6001c395dce995fad7aacd016285c1e8eb` | [designmd.app özeti](https://designmd.app/library/material-design-3); Google spesifikasyonu değil. Dosya değiştirilmedi; digest provenance ile eşleşti. Başka girdi istenirse seçim yeniden kaydedilmeli. |
| Google Material 3 rehberi | [Motion](https://m3.material.io/styles/motion), [color roles](https://m3.material.io/m3/pages/color-roles?s=m3), [Compose tasarım rehberi](https://developer.android.com/develop/ui/compose/designsystems/material3) | Tasarım semantiğinin resmi kaynağı. Bazı detay sayfaları web okumada yalnız JavaScript gereksinimi döndürdü; okunmamış tablolar doğrulanmış sayılmadı. |
| Resmi MDC-Android kaynak referansı | `1.13.0`, commit `84c0e6d6d493fcf176d2099683245b78afbeb78d`; incelenen generated token XML’lerinde `Version: 16.0.2`, `platform=android, audience=3p` | [Sabit kaynak](https://github.com/material-components/material-components-android/tree/84c0e6d6d493fcf176d2099683245b78afbeb78d). M3 ve Expressive ile ilişkili kaynaklar aynı sürümde bulunuyor; her dosya kendi bağlamıyla kullanıldı. M2 `mtrl_*` boyutları M3 diye alınmadı. |
| Ayrı Expressive karşılaştırma snapshot’ı | `androidx.compose.material3:material3-android:1.5.0-beta02:sources` | [Google Maven kaynak arşivi](https://dl.google.com/dl/android/maven2/androidx/compose/material3/material3-android/1.5.0-beta02/material3-android-1.5.0-beta02-sources.jar). 2026 snapshot’ı; Mayıs 2025 Expressive tanımıyla eşit sayılmadı. Button token `v0_11_0`, motion `v0_14_0`, shape `14_1_0`, type scale `v0_103`. Arşiv SHA-256 ve çıkarılan dosya digest’leri [manifestte](evidence/01/official-sources.json). Bu bir Kotlin/Android bağımlılık önerisi değil. |
| Qt | Kurulu SDK `6.12.0`; `Qt6ConfigVersionImpl.cmake:10` | Public QML metadata ve SDK’daki QML dosyalarının byte kopyaları [Yerel arşiv: qt-sources.json](evidence/README.md#review-01) içinde. SDK’nın belirli Git commit’inden yeniden derlendiği bu alanda doğrulanmadı. |

Google’ın [resmi duyurusu](https://blog.google/products-and-platforms/platforms/android/material-3-expressive-android-wearos-launch/) Expressive’ı 13 Mayıs 2025’teki genişleme olarak tanımlar. “Material3 / Material You / Expressive” tek sabit sayısal spesifikasyon kimliği değildir. Üretim kitinin seçtiği edisyon, platform bağlamı, desteklediği güncellemeler ve bilinçli özelleştirmeleri yayımlanmalı. Kit, bütün eski ve yeni token setlerini aynı anda kullanmak zorunda değil.

## Öncelikli bulgular

### M3-01 — P1: mevcut Theme ve kalıtım bağı, bağımsız kit kabulünü engelliyor

**Kanıt:** [Yerel arşiv: MerceMaterial3Style.qml:11](evidence/README.md#local-source-audit) `Merce.Style`, `Merce.Platform`, `Merce.Theme` import ediyor; satır 15 `MerceStyle` tabanı. [Yerel arşiv: MerceStyle.qml:81](evidence/README.md#local-source-audit) renk, spacing, radius, size, state ve typography’yi doğrudan `Theme.*` olarak yayımlıyor. [Yerel arşiv: Material3ButtonTheme.qml:9](evidence/README.md#local-source-audit) ve bütün action variation’ları somut `MerceMaterial3Style` tipine bağlı. [Yerel arşiv: CMakeLists.txt:112](evidence/README.md#local-source-audit) QML bağımlılıklarında ve satır 166–168 link listesinde Merce Style/Theme/Platform/Effects/Foundation var. Diagnostik 32 QML dosyasında, 19 dosyaya yayılmış 25 doğrudan Merce import’u buldu.

**Sonuç:** `MerceStyle` adını `BaseStyle` yapmak veya Theme property’lerini yeni namespace altında aynen sunmak bağımsızlık sağlamaz. Merce `ops/cart` profilleri ve `action.primary.container` sözlüğü Material3’ün zorunlu modeli olamaz. Mevcut Merce adaptörünün bu bağımlılıkları kendi bağlamında kasıtlı; bulgu eski adaptöre toplu refactor talebi değildir.

**En küçük düzeltme:** Yeni `Material3Style` doğrudan public [SK.Style](https://doc.qt.io/qt-6.12/qml-qt-labs-stylekit-style.html) tabanında; kendi light/dark renk rolleri, typography ve tek buton tarifiyle başlasın. Merce adaptörü migration boyunca kalsın. Merce kurulu olmayan source/installed tüketici çalışmadan bağımsızlık beyanı verilmesin.

### M3-02 — P1: DESIGN.md referansı üretim sözleşmesi için eksik ve kendi içinde edisyon karışık

**Kanıt:** [Yerel arşiv: DESIGN.md:65](evidence/README.md#historical-inputs) 2021 Material You ve 2025 Expressive’ı birlikte anıyor; satır 72 spring motion istiyor. YAML satır 14–29 yalnız üç web typography preset’i, satır 30–37 üç radius/spacing değeri içeriyor. Satır 39–48 iki aynı renkli, 12 px radius’lu buton tarifi veriyor. Dark, state opacity, selected/checkable ayrımı, elevation basamakları ve font ölçekleme modeli yok.

**Sonuç:** Bu dosya korunacak başlangıç girdisi; bütün M3 bileşenlerinin normative token şeması olamaz. Eksikleri Merce’nin değerleriyle sessiz doldurmak kullanıcının bağımsız kit hedefini bozar. Resmi kaynakla tamamlanan üretim sözleşmesi, snapshot’tan ayrı tutulmalı.

**Gerekli karar:** İlk sürüm için “M3 temel rol sistemi + hangi Expressive family/token güncellemeleri” açık liste olsun. Android/Wear/web değerleri birbirine çevrilmiş gibi sunulmasın. 2026 resmi snapshot’ının farklılığı Mayıs 2025 metnini kendiliğinden yanlış yapmaz; proje bugün hangi sonucu vaat ettiğini seçmek zorunda.

### M3-03 — P1: mevcut common-button recipe’si resmi Expressive ölçü ve type rol zinciriyle eşleşmiyor

**Kanıt:** [Yerel arşiv: material3.json:173](evidence/README.md#local-source-audit) geometri kaynağını QmlMaterial olarak tanımlıyor. [Yerel arşiv: Generated recipe:4](evidence/README.md#local-source-audit) aynı sayıları taşıyor. Aşağıdaki değerler [Yerel arşiv: diagnostic.json](evidence/README.md#review-01) ile parse edilerek karşılaştırıldı:

| Common-button rolü | Yerel paint height | Compose `1.5.0-beta02` container height | Yerel icon minimum | Aynı resmi size rolünde icon |
|---|---:|---:|---:|---:|
| XS | 32 | 32 | 18 | 20 |
| S | 40 | 40 | 18 | 20 |
| M | 48 | 56 | 24 | 24 |
| L | 56 | 96 | 24 | 32 |
| XL | 72 | 136 | 32 | 40 |

Resmi yükseklik kanıtı: kaynak arşivindeki `tokens/Button{XSmall,Small,Medium,Large,XLarge}Tokens.kt:24–25`; ikonlar aynı dosyalarda satır 36–37. Kalıcı örnek: [Yerel arşiv: ButtonLargeTokens.kt:24](evidence/README.md#review-01). MDC’nin [sabit button token kaynağı](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/button/res/values/tokens.xml#L22) de size icon değerlerini `20/20/24/32/40` olarak verir.

[Yerel arşiv: Material3ButtonSizeVariation.qml:23](evidence/README.md#local-source-audit) padding ve background geometry değiştiriyor; size’a bağlı font/line-height/tracking mapping’i yok. [Yerel arşiv: MerceMaterial3Style.qml:118](evidence/README.md#local-source-audit) bütün normal butonlara Merce `sizeSmall` / `weightMedium` veriyor. Resmi MDC size rolleri `LabelLarge/LabelLarge/TitleMedium/HeadlineSmall/HeadlineLarge` istiyor (`button/res/values/tokens.xml:24,37,50,63,76`); Compose [Yerel arşiv: Button.kt:1640](evidence/README.md#review-01) aynı boy–type rol ilişkisini gösteriyor.

**Sınır:** Bunlar descriptor/paint token karşılaştırmaları; gerçek kontrolün boyu ölçülmedi. Hit target, caller padding ve büyütülmüş metin gerçek Item’ı büyütebilir. Ayrıca aynı resmi arşivde eski `FilledTonalButtonTokens` icon 18 gibi eski tarifler de var; yeni size rolünün 20 değeriyle sessiz karıştırılmamalı.

**En küçük düzeltme:** Mevcut bütün recipe’leri otomatik taşımak yerine seçilen edisyona göre **tek Small** tarifi doğrulansın. Sonra boyut başına typography + container/icon + padding + shape birlikte eklenir. 140 token’ın alias çözümünün doğru olması M3 tasarım doğruluğunu kanıtlamaz.

### M3-04 — P1: public StyleKit slot’ları, typography ve Material FILL için tek başına yeterli değil

**Kanıt:** SDK 6.12 metadata’sında `ControlStyle` public görsel grupları `background/indicator/handle/text`; `contentItem` veya `contentDelegate` property’si yok. [Yerel arşiv: SDK Button.qml:49](evidence/README.md#review-01) kendi `IconLabel` içeriğini oluşturuyor; `background.data.iconFilled` okumuyor. Metadata sonucu [Yerel arşiv: diagnostic.json](evidence/README.md#review-01) ve public [ControlStyle](https://doc.qt.io/qt-6.12/qml-qt-labs-stylekit-controlstyle.html) ile uyumlu.

[TextStyle](https://doc.qt.io/qt-6.12/qml-qt-labs-stylekit-textstyle.html) `pointSize`, `bold`, `italic`, renk, hizalama ve padding sunuyor; `lineHeight`, `letterSpacing`, sayısal `font.weight`, `font.family` veya variable font axis slot’u sunmuyor. Global/family `fonts` ile native caller font özellikleri ayrı yollar. “StyleKit var, 15/30 typography rolü kendiliğinden çalışır” sonucu çıkmaz. [Yerel arşiv: SDK Label.qml:23](evidence/README.md#review-01) font ve rengi alıyor; line-height mapping’i yapmıyor.

Legacy FILL yolu var: [Yerel arşiv: Material3IconButtonVariation.qml:19](evidence/README.md#local-source-audit) selected payload’ı `iconFilled: true` yapıyor; [Yerel arşiv: ButtonContent.qml:103](evidence/README.md#local-source-audit) `AppIcon.filled` değerini native olmayan `control.contentData` alanından alıyor. Bunu sağlayan [Yerel arşiv: MButton.qml:58](evidence/README.md#local-source-audit) legacy style context’i ve satır 62 contentData adaptörü. Bu yol stock `SK.Button` yeteneği diye yayımlanamaz.

**En küçük düzeltme:** İlk buton text-only veya native `icon.source` ile public Qt yolunda çalışsın. Material Symbols FILL gerekiyorsa native control’ün public `contentItem` özelliğine bağlanan **kit içi** dar renderer/adaptör ayrıca sınansın. İkinci input handler, sahte button veya background içinde ikinci label üretilmesin. `DelegateStyle.data` bütünüyle değişir; size + appearance + checked payload bileşimi field merge varsaymadan test edilsin. Shared factory/API kabulü için [04 raporundaki](04-shared-boundary.md) ek değer ve standartlık kapısı ayrıca gerekir.

**Somut uygulanabilirlik sınırı:** Native bütün kontrollerin **exact weight + tracking + line-height** değerleri yalnız `Style/Theme/StyleVariation.text` mapping’iyle sağlanamaz. `fonts.button` gibi family QFont girdileri family/size/numeric weight/letter spacing için kullanılabilir; bu, kontrolün her state/size variation’ında farklı bütün type rolünü çözmek veya native text nesnesine line-height geçirmek değildir. Root `ApplicationWindow.font` da QFont mirasını sağlar; line-height QFont özelliği değildir. Ayrıca geçiş sırasında font cache/binding davranışı kaynak listesinden kanıtlanmaz.

| Uygulama seçeneği | Yapabildiği | V1 sınırı / proof |
|---|---|---|
| **Native font alt kümesi** | Kitin `fonts.button` ve kendi Theme’iyle Small butonda family/size/weight/tracking; native text elide, display ve input korunur | En küçük başlangıç. Tek satır Small ölçü/baseline karşılaştırılır; exact 20-unit line box ve bütün 15/30 typography rolü destekleniyor denmez. Sapma üretim DESIGN.md’sinde açık olmalı. |
| **Public Theme + root/family font bağlantısı** | Kitin public font girdisi app root’una ve uygun StyleKit family font’una bağlanır; plain Qt fallback’ler de app fontunu miras alabilir | Ayrı kontrol ailesi gerekmez. Caller’ın açık alt font property binding’i, theme değişimi ve text scale bu mirasın üstünde gerçekten korunmalı. Line-height ve instance’a özel farklı role eksikliği çözülmez. |
| **Kit içi açık content adaptörü** | Native `SK.Button.contentItem` içinde public Text/Image ile exact line-height, Material FILL ve kit size-role fontu uygulanabilir | Yalnız talep edilen eksik için. Caller font/icon/display/spacing ve text elide/RTL aynı sözleşmeyle tüketilmeli; content replacement’ın farklı family’lere otomatik yayıldığı varsayılmaz. Yeni public `M3.Button/Label/...` ailesi otomatik karar değil. |

Background delegate içinden `control.font = ...` yazmak, mevcut binding’i değiştirerek bu açığı kapatmak değildir; böyle bir yol önerilmiyor. Mevcut MButton ve ButtonSupport deneyindeki başarılı içerik, yeni stock `SK.Button` adaptörünün kanıtı sayılamaz.

**Caller önceliği için gerekli executable proof — bu tur NOT-RUN:** Aynı stock native Button’da `font.pixelSize`, `font.weight`, `font.letterSpacing`, `spacing` ve padding ayrı değişkenlere bağlı kurulur; özellikle `font.letterSpacing: 0`, `spacing: 0`, `padding: 0` açık durumları bulunur. İlk oluşturma → hover/checked → Light/Dark → A/B/A Style geçişi sonrası getter değerleri ve gerçekten çizilen içerik ölçülür; ardından binding’in kaynak değişkenleri yeniden değiştirilip hâlâ güncellendiği doğrulanır. Adaptör zero için `value || default` kullanmamalı. Long text, RTL ve icon/display kombinasyonlarında da sıfır değeri ile caller fontu korunmadan API uyumu ilan edilmemeli.

### M3-05 — P2: Expressive motion iddiası mevcut easing animasyonlarıyla doğrulanmıyor

**Kanıt:** [Yerel arşiv: MerceMaterial3Style.qml:74](evidence/README.md#local-source-audit) Merce `motion.press.duration/easing` kullanıyor. [Yerel arşiv: MerceMotion.h:37](evidence/README.md#local-source-audit) bu preset’i 100 ms `InQuad` olarak tanımlıyor. Floating label [Yerel arşiv: Material3InputBackground.qml:159](evidence/README.md#local-source-audit) 300 ms Merce `OutQuad`; ripple [Yerel arşiv: Material3InteractionFeedback.qml:95](evidence/README.md#local-source-audit) duration/easing tabanlı. Spring isimli sabitlerin Theme’de bulunması bu animasyonların spring olduğu anlamına gelmez.

Google’ın [motion sistemi](https://m3.material.io/styles/motion) spatial ve effects spring’lerini, expressive/standard scheme’lerini ayırıyor. Resmi [Yerel arşiv: ExpressiveMotionTokens.kt:22](evidence/README.md#review-01) default/fast/slow spatial için farklı damping ratio/stiffness; effects için ayrı kritik sönümlü değerler içeriyor. Qt [SpringAnimation](https://doc.qt.io/qt-6.12/qml-qtquick-springanimation.html) başka parametre semantiği ve `spring` için 0–5 yararlı aralığı bildiriyor. Material `stiffness=800`, `dampingRatio=0.6` sayılarını Qt property’lerine doğrudan koymak birim dönüşümü değildir.

**En küçük düzeltme:** Motion policy/token’ları kitte. Önce Qt’nin public animation tipleriyle hedef eğri, kesinti ve retarget davranışı ölçülsün; yeterli değilse yalnız Material3 içinde küçük helper düşünülür. Renk/opacity spatial overshoot’a bağlanmaz. Reduced motion/software’da anlamlı statik state/focus korunur; görünmez veya minimized sahnede devam eden animasyon kalmaz. Sayısal spring uyumu bu tur **NOT-RUN**.

### M3-06 — P2: Merce renk/state ve seed algoritması Material3 semantiğinin yerine geçemez

**Kanıt:** [Yerel arşiv: MerceStyle.qml:97](evidence/README.md#local-source-audit) disabled container’ı verilen **aktif container/content** çiftini tint ederek üretiyor. Material3 action variation’ı bu helper’ı [Yerel arşiv: Material3ButtonVariation.qml:56](evidence/README.md#local-source-audit) üzerinden aynen kullanıyor. Resmi [Yerel arşiv: FilledButtonTokens.kt:28](evidence/README.md#review-01) disabled container için `OnSurface` ve kendi opacity’sini seçiyor; eski tonal tokenında da `OnSurface` kaynak fakat farklı opacity var. Tek Merce helper’ı bütün edisyon/family sonuçlarını temsil etmez.

[Yerel arşiv: Material3ButtonVariations.qml:315](evidence/README.md#local-source-audit) elevated için `surface.containerRaised/content.primary`, tonal için `surface.containerTinted/content.primary` kullanıyor. Resmi [MDC tokenları](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/button/res/values/tokens.xml#L99) sırasıyla `SurfaceContainerLow/Primary` ve `SecondaryContainer/OnSecondaryContainer` rolleri tanımlar. Mevcut Merce rolünün herhangi bir palette aynı hex’i vermesi semantik eşdeğerlik değildir.

Merce’de HCT/tonal palette kütüphanesi kullanılıyor; [Yerel arşiv: BrandDerivation.cpp:199](evidence/README.md#local-source-audit) tek seed’den Merce canvas/contrast/deviation kurallarına göre action çıkarıyor. Bu, tam M3 dynamic color scheme üreticisi diye taşınamaz. [Resmi Android rehberi](https://developer.android.com/develop/ui/compose/designsystems/material3#dynamic_color_schemes) wallpaper kaynağını Android 12+ yeteneği olarak sınırlar ve static light/dark fallback önerir; Qt masaüstünde aynı wallpaper palette erişimi varsayılamaz.

**En küçük düzeltme:** Material3 kendi paired color roles ve family state tarifini tutsun. İlk dilimde doğrulanmış static light/dark scheme yeterli; seed/contrast/variant desteği eklenecekse resmi algoritma sürümü ve test vektörleri kitte sabitlensin. Ortak Qavsi paleti, `primaryContainer = Merce action.primary.container` kuralı veya ortak seed servisi çıkarılmasın.

### M3-07 — P2: floating label kısmen güçlü, tam form family kapsamı henüz yok

**Kanıt:** [Yerel arşiv: Material3InputBackground.qml:14](evidence/README.md#local-source-audit) yalnız `T.TextField` ve `T.ComboBox` tanıyor. Satır 16–17 açık `Accessible.labelledBy` ilişkisini okuyor; satır 55–62 value/IME preedit/focus’u ayırıyor. Satır 123–139 dış label/placeholder görünümünü `Binding` ile yönetiyor; destruction’da restore ediyor. Bu lifecycle korunmalı. Caption ölçüsü ise [Yerel arşiv: MerceMaterial3Style.qml:39](evidence/README.md#local-source-audit) Merce `sizeSmall` ve input fontunu scale ederek elde ediliyor; ayrı M3 resting/supporting label typography sözleşmesi yok.

Source alanında TextArea floating label desteği, supporting/error metin yerleşimi, error announcement ve leading/trailing affordance sözleşmesi tamamlanmış değil. `success/warning/error` outline variation’ı textual error mesajını üretmez; validasyon zamanlaması uygulamada kalmalı. Verilen [Yerel arşiv: DESIGN.md:114](evidence/README.md#historical-inputs) persistent label ve textual error istiyor. Resmi [TextField tokenları](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/textfield/res/values/tokens_textfield.xml) label/input/supporting text ve error durumlarını ayrı tanımlıyor.

**En küçük düzeltme:** Input family eklendiğinde floating label/underline kitte kalsın; native editor/model/IME Qt’de. Açık label ilişkisi, ilk/son harf, preedit, filled/outlined, error metni, RTL, clipping ve style round-trip birlikte sınansın. `Accessible.name` kontrolün value’sundan üretilmiş olabilir; generic caption kaynağı yapılmasın. Search veya çok satırlı input’a otomatik caption vaadi verilmesin.

## Mevcut DESIGN.md gereksinimleri → üretim sözleşmesinde gereken tamamlamalar

Snapshot değiştirilmez. Aşağıdaki farklar kitin kendi üretim DESIGN.md’sinde provenance ile kaydedilir.

| Mevcut girdinin isteği / satırı | Resmi kaynakla tamamlanacak kural | Qt/kit karşılığı |
|---|---|---|
| Dynamic wallpaper / 4, 70 | Scheme rol çiftleri; Android wallpaper kaynağı ile brand/content seed farklı girdiler. [Android rehberi](https://developer.android.com/develop/ui/compose/designsystems/material3#dynamic_color_schemes) | Kit config’i typed light/dark scheme alabilir. Desktop wallpaper keşfi ilk dilimin şartı değil. Desteklenmeyen kaynağın sonucu gömülü scheme. |
| Sabit renkler / 5–13, 77–88 | `primary/on-primary`, container eşleri, secondary/tertiary/error, inverse/outline/surface container rolleri. [Google color roles](https://m3.material.io/m3/pages/color-roles?s=m3) | Kaynakta `on-surface=#000000`, `text=#1d1b20` farkı açık çözülür; runtime tek keyfi siyah/beyaz default eklemez. Contrast gerçek bileşik yüzeyde ölçülür. |
| `h1=4rem/700/1.05`, body, label-caps / 14–29 | M3’ün beş rol × üç boyut ölçeği; kullanılan subset açık. Baseline örneği resmi tokenlarda displayLarge 57/64 regular, labelLarge 14/20 medium. Expressive emphasized stiller ayrı kaynak/edition alanı. | CSS rem/em, Android sp ve Qt point/pixel eşit varsayılmaz. Font family/weight, line height ve tracking birlikte çözülür. Her 30 rolün ilk sürümde uygulanması zorunlu değil; kapsanmayanlar belirtilir. |
| Roboto Flex / Roboto / 92–94 | Font asset’i ve variable axis desteği sistem adından ayrı. | Font bulunamaması, lisans/provenance, offline package ve shaping sınanır. `AppLabel` Merce’de kalır. |
| Radius 8/12/16 ve 12 px button / 30–33, 42, 108 | Resmi [shape tokenları](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/shape/res/values/tokens.xml#L22) none, 4/8/12/16/28/full ve ek Expressive rolleri içerir. Component/state mapping’i tek global radius’tan türemez. | Intentional 12 px button tutulursa özelleştirme diye belgelenir. Capsule/full, round/square, pressed/selected geometri ayrı kit tarifidir. Hit target paint plate’den ayrılır. |
| Spring motion / 72 | Scheme + spatial/effects + fast/default/slow; retarget ve velocity. [Google motion](https://m3.material.io/styles/motion) | Public Qt çözümü önce; sayısal eşleme ölçülmeden “Expressive spring” denmez. Helpers kitte. |
| Blur/transparency / 104 | Bu genel katalog cümlesi bütün M3 yüzeylerini cam yapmaz. Resmi component/elevation/color rolleri esas. | Native blur yeni Material3/Runtime bağımlılığına gerekçe değil. Tonal/elevation plate ve okunaklı fallback yeterli; özel materyal ayrıca scope ister. |
| Responsive grid `<768px` / 99–100 | Breakpoint ve uygulama shell’i ayrı sözleşme; Android/Wear değerleri kopyalanmaz. | Gallery/app yerleşimi; kit kendi ölçülerini korur. Qavsi runtime `768` breakpoint’i yayımlamaz. |
| Focus/hover/pressed, persistent label/errors / 112–114 | State kombinasyonları ve accessibility ilişkileri açık test edilir. | Native control state ve Accessible bağlantıları; outline rengi tek başına error metni yerine geçmez. |
| AA + reduced motion / 119 | Metin/non-text ve büyütülmüş yazı ayrı kabul koşulları. | Public Qt accessibility hints doğrudan okunur; kit motion/fallback politikasını uygular. CSS media query Qt API’si değildir. |

Type-scale sürümleri arasında da fark var: MDC 16.0.2 statik displayLarge tracking değeri ile Compose `v0_103` aynı alanı farklı yuvarlayabiliyor; Compose emphasized bölümünde `TODO update with the generated tokens once available` yorumu hâlâ mevcut (`TypeScaleTokens.kt:249`). Bu yüzden bir platformun tablosu “tüm M3 edisyonlarının değişmez sözleşmesi” diye sunulamaz. Referans seti ve bilinçli sapmalar kayıt altına alınmalı.

## Önerilen Material3 Theme/token sınırı

Bu tablo public API ismi dayatmıyor; kitin kendi modelinde tutulması gereken anlamları gösteriyor. DTCG kullanılırsa yalnız dosya/alias biçimi ortaktır.

| Alan | Material3’te sahiplenilecek veri | Taşınmayacak Merce varsayımı |
|---|---|---|
| Renk | Seçilen edisyonun color scheme rolleri ve eş foreground’lar; static/dynamic giriş kaynağı ayrı | Evrensel `action/content/surface` eşleme zorunluluğu, Merce status/brand seçimi |
| Typography | Kullanılan baseline/emphasized rollerin family, weight, size, line-height, tracking; text scale girdisi | `h1/h2/price/overline` enum’u ve Merce tracking default’u |
| Shape/elevation | Component/state shape rolleri, shadow/tonal surface rolleri | Profile control height’inden bütün radius’ları türetmek |
| Motion | Standard/expressive seçimi ve kullanılan spring/easing tariflerinin sürümü; azaltma sonucu | Merce’nin `press=100ms/InQuad` preset’i |
| Component recipe | Button size+appearance+selection; field filled/outlined; family bazlı geometri | Ortak Runtime variation kataloğu veya `ops/cart` ölçeği |
| Config/lifecycle | App-owned config; geçersiz girişte son geçerli/gömülü M3 scheme; tek StyleKit host’unu aşmayan scope | Merce singleton’ına yazma, gizli preference dosyası, global font/theme reset’i |

Typed model ister QObject ister küçük QML nesneleri olsun, ilk dilimde yalnız kullanılan roller üretilsin. Tam evrensel Theme şeması, generic kit registry, runtime JSON’dan QML üretimi veya her token’a ayrı dosya gerekmez. Generation consumer build’inin işi olmaz.

## Family kapsam matrisi — kaynak varlığı, bağımsızlık ve açık kalan kabul

**Kısmi** = Merce içindeki mapping/delegate kaynakta mevcut; bağımsız Material3 ve resmi tasarım doğruluğu test edilmiş demek değil. **Fallback** = base Merce/Qt çizimi veya yalnız birkaç property; M3 family kabulü yok. Bu tur aşağıdaki family’lerin runtime testleri çalıştırılmadı.

| Family / özellik | Mevcut kanıt | Public StyleKit yeterliği / kit işi | Açık kabul / v1 kapsamı |
|---|---|---|---|
| Filled / outlined / tonal / elevated / text buttons | Kısmi: `Material3ButtonTheme.qml:14–85`; `Material3ButtonVariations.qml:315–329`; inherited intent mapping | Normal/hover/pressed/focused/checked/disabled, border/padding stock mapping; ripple/shadow kit delegate | Rol çiftleri ve tek Small ölçü/type zinciriyle ilk dilim. Diğer varyantlar ayrı kabul. |
| Round / Tool / Delay | Kısmi: `MerceMaterial3Style.qml:82–85` | Native behavior Qt’de; ortak görünüm kit recipe’si | Delay’nin progress semantics’i common Button ile aynı sanılmamalı; v1 destek listesi ayrı. |
| Icon buttons / FILL | Kısmi: variation `:19–44`, kayıt `:209–257`; MButton içerik yolu | Container mapping stock; Material glyph axes/FILL custom content/provider gerektirir | Checked vs checkable, selected vs nonselected; exact `material:` isim, native image override. FILL ilk minimal dilime zorunlu değil. |
| FAB / extended / navigation FAB | Kısmi: `Material3FabVariation.qml`, kayıt `:87–92`; `Material3NavigationItemContent.qml` | Background/geometry stock; icon/label morph kit içeriği | Yeni edisyon sizes/colors/elevation doğrulanmalı; app action/navigation state’i kitte değil. İlk dilim sonrası. |
| Connected / split / segmented | Kısmi: ayrı variation ve joined background; size payload’ları | Geometri/state kitte; native Button/Action/selection Qt’de | Border overlap, RTL, pressed/checked, focus + property/payload priority; “bir buton çalıştı” family kabulü değil. |
| TextField / ComboBox floating label | Kısmi: `MerceMaterial3Style.qml:51–53`; `Material3InputBackground.qml` | Background slot notched outline/underline + caption çizebilir; editor/model Qt’de | Label restore, caller font/palette/padding, IME, persistent/error text ve role kaynakları birlikte test. |
| TextArea / Search / SpinBox | Fallback/kısmi: `textInput:144`, `spinBox:179`, `searchField:251`; floating delegate yalnız Field/Combo | Generic palette/border yeterli temel; gerçek M3 family geometry/affordance kitte | Multiline floating/error/search view davranışı kapsam dışı diye açık yazılmalı. |
| CheckBox / RadioButton | Kısmi: `:88–89`, `:521–570`, `Material3SelectionIndicator.qml` | Native checked/tristate Qt’de; indicator/feedback kitte | Unchecked/checked/indeterminate/disabled × hover/focus; M3 role/shape/size ve ekran okuyucu. |
| Switch | Kısmi: `:90–103`, `:571–611`, track/thumb recipe | Indicator/handle slot yeterli; state/thumb glyph ve hareket kitte | On/off, pressed thumb, optional icon, contrast; actual visual minimum ayrı. |
| Slider / RangeSlider / Dial | Slider/range için custom track/first/second handles; `:106–110`, `:614–690`; dial daha sınırlı | Value/drag/keyboard Qt’de; ticks/bubble/stop/geometry kitte | Crossed/degenerate range, endpoints, RTL/vertical, disabled, label bounds. Dial slider coverage’ını devralmaz. |
| Progress / Busy | Fallback/kısmi: `:682`, `:691`; boy/renk override’ı | Stock basic indicator var; wavy/Expressive loading biçimi ayrı kit renderer’ı | Determinate/indeterminate, inactive/offscreen/reduced motion. Expressive progress/loading desteklenmiş sayılmaz. |
| Menu / MenuItem / toolbar | Kısmi: `:104`, `:363–477`, `:736–762` | Qt popup/keyboard/Action; QML surface ve ripple kitte | Native OS menu görünümü M3 delegate’ini kullanmayabilir; bunu ayrı scope say. Popup width/hover/selection bağımsız sınanmalı. |
| Tab / ItemDelegate / Swipe | Kısmi: `:86–87`, `:267–362`, `:478–520`; özel delegate’ler | Selection/input Qt’de; primary/secondary tab/indicator ve swipe feedback kitte | Selected vs hover, focus, long text, RTL; secondary tab ve navigation bar otomatik kapsam değil. |
| Navigation rail / drawer / sheet | Kısmi: navigation item/surface variation’ları `:843–890` | Kit item/indicator/morph; route/destination/layout/expand gesture app’te | Rail, drawer, navigation bar ve sheets farklı component sözleşmeleri. Şu an tam M3 navigation seti yok. |
| Pane / Frame / GroupBox / Page / Popup / Dialog | Kısmi: `:693–771` | Surface/padding/border stock; multi-layer elevation gerekirse kit shadow delegate | Card/dialog/sheet anatomy, tonal elevation/scrim ve dialog focus/modal Qt yoluyla doğrulanmalı. |
| Scroll / PageIndicator | ScrollBar/ScrollIndicator yalnız radius (`:683–684`); PageIndicator inherited fallback | Native range/scroll Qt’de; family ölçü/state kitte | M3 standardında birebir karşılığı olmayan Qt kontrolleri “Qt fallback” diye yayınla. |
| Label / badge / type roles | Badge variation `:892–923`; AppLabel kendi Merce type enum’u | SK.Label temel text/palette için yeterli; line-height/role/multi-line kit tüketim yolu gerekir | 15/30 role kapsamı, baseline, text scale, long/translated strings. Merce AppLabel taşınmaz. |
| Chips / app bars / date-time pickers / carousel / snackbar | İncelenen Material3 modülünde tam family kanıtı yok | Başka Qt kontrolüne yakınlık, otomatik M3 eşdeğerliği değildir | İlk v1 dışında veya explicit fallback. Yeni shared control kütüphanesi için gerekçe değil. |

Mevcut test kaynakları iyi regression başlıkları içeriyor: [Yerel arşiv: style:125](evidence/README.md#local-source-audit), [Yerel arşiv: fields:50](evidence/README.md#local-source-audit), [Yerel arşiv: menus:131](evidence/README.md#local-source-audit), [Yerel arşiv: tabs:125](evidence/README.md#local-source-audit), [Yerel arşiv: sliders:319](evidence/README.md#local-source-audit), [Yerel arşiv: swipe:114](evidence/README.md#local-source-audit). Bunlar Merce adaptörünün testleri; bağımsız kitin test sonucu olarak devralınamaz. Sayısal referanslar da kitin seçtiği resmi sürüme bağlanmalı.

## Kit / Qt / ortak mekanizma sahipliği kararı

| Mekanizma | Karar | Somut gerekçe |
|---|---|---|
| Native control, Action, focus, keyboard, IME, popup, model/value | **USE EXISTING QT** | Mevcut davranış tekrar yazılmaz; public Qt Templates/Controls sözleşmesi. |
| Light/dark/System, StyleKit mapping, StyleReader, variations/delegate | **USE EXISTING QT** | Public Qt API yeterli; Qavsi style registry/BaseStyle gerekmez. Global scope ve mutation bug’ı ayrı sınır. |
| Motion/contrast OS facts | **USE EXISTING QT**, eksik platform olgusu varsa ayrı inceleme | Qt 6.12 `Application.styleHints.accessibility` / `QAccessibilityHints`; M3 motion policy kitte. |
| Color scheme, dynamic-color algorithm/config, typography, shape/elevation, sizes/state layers | **KEEP KIT/APP** | Tasarım anlamı içeriyor; başka kitin modeliyle aynı değil. App seçer, kit yorumlar. |
| Ripple, focus plate, input caption/underline/notch, tab/slider/switch paint | **KEEP KIT/APP** | Şekil, ink, motion ve component/state politikası M3’e ait. Önce public slot. |
| Material glyph FILL / axes / lookup fallback | **KEEP KIT/APP veya DEFER** | M3 selected policy kitte. Qt QIcon temel glyph ihtiyacını karşılayabiliyor; kalan axis/çoklu paket açığı ayrıca kanıtlanmalı. Mevcut `AppIcon`’ın varlığı tek başına extract nedeni değil. |
| Loading guard/content support | **DEFER ortak kabul; gerekli deney kit/app içinde** | 05 guard restore kanıtı var; loading sunumu, exact content hook ve native/caller binding kapsamı farklı. Buton family’sinin core işlevi bu pakete bağlanmamalı. |
| Token alias/format validator | **DEFER** | DTCG standardı dosya biçimini ortaklaştırabilir; bugün paylaşılan ürün tüketimi ve gerçek ek değer gösterilmedi. M3 şeması/values kitte kalır. |
| Gallery sabit role alias’ları ve layout/shell | **KEEP APP** | Karşılaştırma uygulamasının ihtiyacı; evrensel kit API’si veya token ontolojisi değil. |
| Genel runtime Theme / universal density / easing / variation enum’u | **KEEP KIT/APP; Qavsi’ye alınmamalı** | Standard sözleşme yok; görsel politika zorla birleşiyor. |

Bu alanda bugün unconditional **QUALIFIES QAVSI** sonucu çıkan yeni component yok. [04 ortak sınır raporu](04-shared-boundary.md) ile uyumlu: ilk bağımsız Material3 tüketicisi Qt + kendi Theme/Style’ıyla yeterli olabilir.

## En küçük bağımsız dikey dilim ve kabul testleri

1. **Tek edisyon kaydı + tek Small Button tarifi:** kaynak snapshot sabit kalsın; üretim DESIGN.md’si seçilen M3 color/type/shape referansını ve Expressive kapsamını açıklasın. İlk uygulamada yalnız kullanılan renk eşleri ve `labelLarge` font alt kümesi olsun; exact line-height desteği M3-04’teki seçenek/proof tamamlanana kadar açık sapmadır. `SK.Style` tabanı, kendi Theme/config’i, Light/Dark/System; Merce import/link yok. Flat/text veya outlined ikinci varyantı yalnız ilk varsayılan doğrulandıktan sonra ekle.
2. **Minimal consumer:** normal/focused/disabled native `SK.Button`, ayrıca baştan `checkable: true` kurulmuş örnek. Public background delegate gerekiyorsa yalnız M3 ripple/focus paint’ini çizsin. İlk çalışma text-only/native image icon ile mümkün; Symbols FILL, loading ve tüm XS–XL kataloğu ilk bağımsızlık kanıtının önkoşulu değil. Mevcut MButton’a dokunulmaz.
3. **Merce’siz source ve installed tüketim:** aynı exported Theme/Style target’ları, temiz environment ve taşınmış prefix. Normal tüketici token generator, Node, Merce, Icon provider, Notifications veya window material fetch etmemeli. Packaging protokolünü [06 raporu](06-delivery.md) sahiplenir; bu rapor bunu yeniden çalıştırmadı.
4. **Anlamlı acceptance:** Light/Dark × normal/hover/pressed/keyboard focus/disabled/checked; caller font/padding/width/palette binding’i; Action/Space/Enter; 1x/2x DPR, LTR/RTL, uzun metin ve text scale; software/offscreen ve desteklenen gerçek GPU. Paint plate ve hit target ayrı ölçülür. Beklenen değerler kendi implementasyonundan kopyalanmaz, sürümü sabit resmi recipe + documented sapmadan gelir.
5. **State/variation sınırı:** instance role listesi baştan sabit. Background data payload’ının size/appearance/checked birleşimi ve first-variation priority sınanır. Aynı kontrol üzerinde gerçek M3 Theme ve sonra başka Style’a dönüşte pointer, text, checked, focus ve native binding korunur. 05’in Qt-only başarısı gerçek M3 adapter testi yerine geçmez.
6. **Sonraki dilimler ayrı kabul:** FILL/content ve loading ihtiyacı doğduğunda public slot/native content adapter testi; sonra fields/selection. Floating label için ilişkili dış label yalnız bir kez çizilir, Standard/diğer kit geri dönüşünde restore olur; accessible label/error, preedit ve multiline sınırı test edilir. Spring motion istenecekse trajectory/retarget testinin toleransı ve reduced motion sonucu tanımlanır.

Kontrolün yeniden yaratılması, SDK patch’i, private Qt header’ı veya `Qt.labs.StyleKit.impl` import’u bu dilimin uygulama çözümü değildir. Qt’nin kendi SDK QML dosyalarının iç import’ları, kitin aynı private modüle bağlanmasına izin sayılmaz.

## Bu tur çalıştırılan kontroller ve NOT-RUN alanlar

| Kontrol | Gerçek sonuç / kanıt | Ne kanıtlamıyor? |
|---|---|---|
| DESIGN.md provenance digest | **PASS:** kayıtlı SHA-256 eşleşti; kaynak değişmedi | Kaynağın resmi veya tam spesifikasyon olduğunu göstermiyor. |
| Token/source diagnostik | **PASS:** 140 numeric/alias leaf çözüldü; alias-cycle self-check çalıştı; 11 generated Button sabiti local source ile aynı; 32 QML ve Merce import envanteri üretildi. [Yerel arşiv: diagnose.py](evidence/README.md#review-01), [Yerel arşiv: sonuç](evidence/README.md#review-01) | Bütün generator’ın `build/validate/check` geçtiği, gerçek pixel/geometry veya M3 conformance değil. |
| Resmi size-role karşılaştırması | **FAIL — belirtilen reference eşleşmesi:** 3/5 height, 4/5 icon minimum farklı. Referanslar MDC16.0.2/Compose1.5.0-beta02 olarak ayrı kayıtlı | Her eski M3 edisyonu için aynı ihlal sonucu vermez; gerçek control ölçümü yapılmadı. |
| Qt public API inventory | **PASS — kaynak kanıtı:** installed 6.12 metadata’da contentItem hook ve TextStyle lineHeight yok; SDK Button/Label/StyleAnimation kopyaları kaydedildi | Yeni helper’ın çalıştığı veya bütün public feature kombinasyonlarının desteklendiği değil. |
| Deterministic QML source scanner | **FAIL — ham lint:** 33 dosyada 111 bildirim, exit `1`; [Yerel arşiv: log](evidence/README.md#review-01), [Yerel arşiv: komut/exit](evidence/README.md#review-01). 92 bildirim ordering/id/group notation; diğerleri var/imperative-binding, import, animation/performance kuralları | Bunlar 111 runtime bug veya bağımsızlık bulgusu değildir. Stil incelemesi scope dışı cleanup/refactor yetkisi üretmez. `qmllint` type resolution bu tur **NOT-RUN**. |
| Sabit rol + aynı instance Style/theme geçişi; global scope; guard restore | [05 raporuna](05-stylekit-runtime.md) referans: Qt-only dar kapsam **PASS**, mevcut instance variation mutation **FAIL**. Bu alanda tekrar çalıştırılmadı | Gerçek Material3 recipe, FILL/content, popup veya platform render kabulü değil. |
| Bağımsız paket configure/build/install/consumer | **NOT-RUN, domain 06** | Bu rapor bağımsız M3 package PASS’i vermez; henüz ürün kiti yok. |
| Merce test executable’ları / Qt runtime / ekran görüntüsü | **NOT-RUN**; test kaynakları okundu | Eski Merce test sonucu veya bu raporda geçirilen runtime test yok. |
| Physical spring, GPU ripple/shader, software görünümü, font fallback/text scale, IME, accessibility reader, touch hardware, WASM | **NOT-RUN** | Kaynak mapping’i bu alanları doğrulamıyor. |

QML lint dağılımı: `ORD-1:47`, `STY-3:28`, `STY-1:17`, `BND-1:7`, `IMP-4:4`, `BND-2:2`, `PRF-2:2`, `PRF-3:1`, `PRF-5:1`, `STA-2:1`, `BND-5:1`. Ham bildirimler ayrı logda tutuldu; bu raporun öncelikleri mimari bağımsızlık ve tasarım sözleşmesidir. Binding/layout/lifecycle/state/content yolları kaynak üzerinden izlenmiştir; kullanıcı kapsamı gereği alt ajan veya ürün kodu değişikliği yapılmamıştır.

Tekrar üretme — script yalnız kendi `diagnostic.json` kanıt dosyasını yazar; ürün kaynaklarını okumakla sınırlıdır:

```bash
rtk proxy python3 MERCE_SOURCE/qavsi-design/reviews/evidence/01/diagnose.py
```

## Koordinatör belgelerine gerekli en küçük düzeltmeler

- `MIGRATION / Aşama 2`: “actions dosya ayrımını yeniden kullan” cümlesine **recipe değerlerini taşımadan önce seçilen resmi edisyonla doğrula** şartı eklenmeli. Mevcut 3 height/4 icon farkı bu kapının somut gerekçesi.
- `KIT-AUTHORING / ürün sözleşmesi`: `design edition + platform context + source digest + implemented/fallback family + documented deviation` aynı kayıtta tutulsun. M3/Expressive adları eş anlamlı kullanılmasın.
- `CONTRACTS / geometry-typography-content`: public TextStyle’ın line-height/tracking/numeric weight ve content replacement sınırları somut kabul kapısı olarak eklensin. `background.data.iconFilled` stock SK.Button tarafından tüketiliyormuş gibi yorumlanmasın. Gerekli adapter kitte kanıtlansın; ortak API ayrı kabul.
- `DELIVERY / M3 örneği`: minimum consumer’da `ButtonSupport` ve `IconsMaterial` yalnız kullanım varsa istensin. Mevcut metnin bunları zorunlu paket sanmaya açık örneği, Qt + Theme/Style minimal örneğinden ayrı gösterilsin.
- Mevcut **standard admission**, global StyleKit scope, sabit Gallery rolü, MButton compatibility ve no-SDK-patch kararları korunsun. Bu rapor bunlar için yeni Runtime module’u veya mimari yeniden yazım önermiyor.

Ürün/SDK/MButton/Git, OS tercihleri ve foreground UI değiştirilmedi. İnceleme çıktıları yalnız bu rapora ve `reviews/evidence/01/` altına yazıldı; kaynak font veya yeni dependency eklenmedi.

Atlanan: gerçek bağımsız Material3 build/render ve cihaz doğrulaması. Risk: edisyon seçilmeden Merce recipe’lerini kopyalamak bağımsız fakat yanlış Material3 sonucu üretir; public StyleKit content/typography sınırı ayrıca çözülmelidir.
