# Qavsi kontrol ve entegrasyon sözleşmeleri

**Durum:** Öneri. Buradaki Qavsi tipleri yeni pakette henüz uygulanmadı. Mevcut deney ve Merce kodu ile karıştırılmamalı.\
**Taban:** Qt 6.12.0 public QML/C++ API’si.

Yeni Runtime arayüzleri [standartlık ve ek değer kapısını](SHARED-ADMISSION.md) geçmeden çıkarılmaz. Örnek Qavsi namespace’leri adaydır; Qt’nin doğrudan karşıladığı iş için yeni wrapper gerekmez.

Görsel otoriteler kabul edildi: Material3’te Google, Fluent’te Microsoft, native Apple Liquid Glass’ta HIG/public AppKit. Kit recipe/state/fallback sözleşmeleri [Material3 DESIGN](../design-systems/material3/DESIGN.md), [Fluent DESIGN](../design-systems/fluent/DESIGN.md) ve [Apple DESIGN](../design-systems/apple/DESIGN.md) içindedir. Bu ortak dosya kitlere aynı token, loading sunumu veya malzeme dayatmaz.

## 1. Sorumluluk tablosu

| İhtiyaç | Doğru katman | Yanlış katmana taşınmayacak şey |
|---|---|---|
| Hover/press/focus/checked | Native control + kitin StyleKit state mapping’i | ButtonSupport içinde renk hesabı |
| Alternatif görünüm | İlgili kitin variation’ı | Runtime’da evrensel variant enum’u |
| Yazı/boşluk/boyut | Kit Theme’i ve native çözülmüş control özellikleri | Bütün kitlere Merce ölçeği |
| Loading işi | Uygulama servisi/modeli | UI’nin ağ/iş mantığı |
| Loading sunumu/tek buton engeli | Mevcut ButtonSupport deneyi + kit renderer’ı; ortaklık ayrıca kanıtlanır | Shared Action’ı otomatik kapatmak |
| Paketli ikon | Mevcut Merce arayüzü; Qt üzerinde kalan routing ihtiyacı | Global “save eşdeğeri” tablosu veya gereksiz yeni Icons Core |
| Camın nerede kullanılacağı | Apple veya ilgili kit | Platform modülünde navigation tasarımı |
| Native view/handle yönetimi | Kit/projedeki dar native bridge; ortak extract henüz kabul edilmedi | Sayfanın AppKit çağırması |
| Navigation bilgi mimarisi | Uygulama | Runtime’ın destination listesi |
| Form etiketi | Kitin input renderer’ı + Accessible ilişkisi | Playground’da style adına göre visible koşulu |

## 2. StyleKit kullanım sözleşmesi

### 2.1 Stil seçimi

Uygulama bir composition root’ta aktif Style instance’ını seçer. Qt StyleKit native kontrol davranışını korur. Sistemleri seçmek için sayfalara `if (selectedStyle === ...)` koşulları yayılmaz.

```qml
// ÖNERİ: Qavsi modülü henüz uygulanmadı.
import QtQuick
import Qt.labs.StyleKit as SK
import Qavsi.Material3.Style as M3

SK.ApplicationWindow {
    SK.StyleKit.style: M3.Material3Style {}
}
```

Bu bir süreçte tek aktif stildir. Aynı proses içinde başka bir root’ta başka stil atamak ikinci pencereyi izole etmez.

Native pencere malzemesi için ek root bağlantısı gerekebilir. `StyleKit.style`, native OS camını tek başına etkinleştiren API değildir. Sayfa kontrolleri bu bağlantıyı bilmez.

### 2.2 Theme kavramlarını karıştırmama

- `SK.Theme`: StyleKit state/özellik eşleme katmanı.
- `Merce.Theme`: mevcut Merce token runtime’ı.
- `Qavsi.Material3.Theme`: önerilen Material3’e ait kaynak ve config modeli.

Aynı “Theme” kelimesi aynı QObject sözleşmesi anlamına gelmez. QML import alias’ları kullanılır; yeni modüllerde import sürümü yerine paket sürümlemesi tercih edilir. Mevcut Merce sürümlü URI’leri migration sırasında kırılmaz. [Qt modül rehberi](https://doc.qt.io/qt-6/qtqml-writing-a-module.html#versions)

Her kitin typed config’i kendi alanlarını tanımlar. Aynı “brand” girdisi tüm kitlerde aynı renk çıktısı veya aynı algoritma demek değildir. Konfigürasyon uygulamaya ait olur; Runtime gizli bir global tercih dosyasını kendiliğinden yazmaz.

### 2.3 Style / variation / delegate

**Style / theme mapping:** Kontrol family’lerinin normal ve etkileşim durumları, font/palette ve geometri kaynakları.

**Variation:** Kitin yayınladığı alternatif tarife açık katılım. Default kontrol variationsız kullanılabilmeli. İkinci variation bağımsız bir eksen gerektiğinde eklenebilir; bütün özellikleri variation’a dönüştürmeyiz.

**Delegate:** Belirli slot’un çizimi. Native kontrolün state machine’ini tekrar uygulamaz. Background, indicator ve handle gibi public slot’larla çalışır. Bir `background.delegate` kendiliğinden arbitrary `contentItem` değiştirme hook’u değildir. [ControlStyle](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-controlstyle.html), [DelegateStyle](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-delegatestyle.html)

### 2.4 Variation önceliği

- Aynı listede aynı property için ilk variation kazanır.
- Kontrolün kendi variation’ı atadan gelene göre önceliklidir.
- Style ve aktif theme’de aynı isim bulunması “bütün nesneyi shadow etme” değildir; property çözümleme sırası uygulanır.
- Tanımlanmayan isimler Qt tarafından sessizce yok sayılır.
- Parent’a uygulanan instance role descendant popup/menu/helper kontrolüne de ulaşabilir.

Bu yüzden `["destructive", "outline"]` gibi iki rol aynı fill/text property’lerini tanımlıyorsa sıraya bağımlılık vardır. Bir kit bu bileşimi ya açıkça destekleyip test eder ya da iki ayrı görünüm olarak yayınlar. “Bağımsız eksen” diyerek property çakışmasını gizlemeyiz.

Ortak Runtime bir variation isim kataloğu yayınlamaz. Gallery’nin stil uyarlayıcısı, kullandığı kitte hangi rollerin bulunduğunu bilir. Semantic açıdan gerekli bir rolün yokluğu görmezden gelinmez; örnek ya desteklenmez olarak gösterilir ya kitin uygun public API’sine eşlenir. [Variation kuralları](https://doc.qt.io/qt-6/qml-qt-labs-stylekit-stylevariation.html)

Qt bug’ı süresince kitler arası karşılaştırma sayfasının instance role listesi sabittir. Gallery’ye ait sabit alias isimleri her kitin Gallery adaptöründe başlangıçta public StyleVariation tarifleriyle tanımlanır; kit değişiminde listeler yeniden yazılmaz. Kit-specific rollerde karşılığı olmayan senaryo unsupported olarak gösterilir veya ayrı süreçte açılır. Bu bir Qt alias API’si değildir. [Ayrıntı](KIT-AUTHORING.md)

### 2.5 QObject data payload

`DelegateStyle.data` bir QObject olarak bütün halinde yayılır; field bazında birleştirilmez. Boyut variation’ının `data` nesnesi atayıp intent’in iconFill bilgisini yanlışlıkla silmesi engellenir.

Yeni custom data yalnız built-in property yetersizse kullanılır. Her family payload’ının sahibi bellidir; state override’ı eksiksiz payload veya açık bir factory kullanır. Kitler arası bütün token’ları taşıyan `data.tokens` torbası üretilmez.

Geçiş döneminde `iconFilled` gibi dar renderer ipuçları desteklenebilir. Bunlar Qt’nin yerleşik API’si değil, Qavsi/legacy adaptörünün açık sözleşmesidir. Adı, yokken davranışı ve hangi icon provider’ların uyguladığı test edilir.

### 2.6 Geometri ve tipografi

Style descriptor üzerindeki `width/height/minimumWidth/minimumHeight` ile gerçek QML Item’ın `implicitWidth/implicitHeight` özellikleri ayrıdır. Descriptor’da var olmayan `implicitHeight` kullanılmaz; sıradan QML ölçüm bağları bu gerekçeyle silinmez.

Caller’ın açık font, width, height, padding, palette ve icon bağları önceliğini korur. Renderer kendi görünümü için kullanıcının font bağını yeniden yazmaz. Qt’nin hangi override’ı hangi tipte çözdüğü native referans kontrolle karşılaştırılır.

**Qt 6.12 sınırı:** StyleKit `TextStyle`, tam bir typography recipe’si değildir. `pointSize`, `bold`, `italic` gibi alanları vardır; `lineHeight`, `letterSpacing`, numeric font weight ve font family burada ayrı state/variation property’leri değildir. `fonts.button` gibi family font’ları QFont yoluyla family, weight ve tracking sağlayabilir; bu, her size/state için tam type-scale çözümü veya line-height desteği anlamına gelmez. QFont line height taşımaz. [01 public API incelemesi](../reviews/01-material3.md)

Bu nedenle ilk bağımsız Material3 buton native font alt kümesiyle, belgelenmiş line-box farkıyla başlayabilir. Exact line height veya instance başına Material FILL gerekirse ilgili kit native `contentItem` üzerinde açık, küçük bir content adaptörü sağlar. `ControlStyle` içinde genel bir content-delegate slot’u yoktur; background delegate’in `data.iconFilled` alanını stock SK.Button kendiliğinden tüketmez. Background içinden `control.font` yazıp caller binding’ini bozmak çözüm değildir. Bu ihtiyaç yeni ortak renderer veya yeni buton ailesini otomatik gerekçelendirmez.

**Zorunlu regression, henüz çalıştırılmadı:** caller’ın bound `font.pixelSize`, numeric `font.weight`, `font.letterSpacing: 0`, `spacing: 0` ve `padding: 0` girdileri; hover/checked, theme ve A→B→A stil değişiminden sonra korunmalı. Sonra binding’in kaynak değişkeni yeniden değiştirilip bağın canlı kaldığı görülmeli. `value || default` gibi sıfırı yok sayan çözüm kullanılmaz. Uzun metin, RTL ve icon/display ölçümü ayrıca sınanır; 05’in temel state proof’u bu typography kontratını kanıtlamaz.

Token’ların fiziksel birimleri belirtilir. Qt TextStyle pointSize dönüşümünde sabit DPI oranı kullanılmaz; public Qt ölçüm API’si kullanılır. Bir dokunma minimumu bütün kitlerde aynı radius, font veya kontrol yüksekliği demek değildir. Metin büyütüldüğünde basit sabit yükseklikle kesilme üretilmez.

Alignment, text elide, layout direction ve RTL native kontrolle aynı anlamı taşır. `AlignAbsolute` fiziksel, diğer left/right hizaları mantıksal kabul edilir. Spinner için ayrılan ölçü içerik slot’udur; üst variation padding’i bu küçük alanı yeniden tüketmemelidir.

## 3. SK.Button + ButtonSupport — mevcut deney ve ortak aday

Bu sözleşme mevcut deneyin hedef davranışını açıklar. Binding restoration proof’u geçti; bu, birleşik loading/content API’sinin ortak standart veya yeni Runtime modülü olarak kabulü değildir. Görsel politika kitte, duplicate-start iş/uygulamada kalır; deney kendi projesinde korunur. [04 kabul incelemesi](../reviews/04-shared-boundary.md), [05 runtime proof’u](../reviews/05-stylekit-runtime.md)

### 3.1 V1 API kapsamı

Mevcut deneydeki sınırlı model korunur:

**V1 hedefi yalnız SK.Button ve ondan QML yoluyla türetilen uygulama tipleridir.** SK.RoundButton, SK.ToolButton, SK.DelayButton, MenuItem ve MButton’a ayrıca attach etme bu sürümün kapsamı değildir. Hepsinin AbstractButton davranışı taşıması aynı loading/geometry sözleşmesini sağladıklarını kanıtlamaz. Desteklenmeyen hedef diagnostic ile reddedilir; native özellikleri değiştirilmez. Kapsam genişletmek ayrı contract testi ve açık API sürümü gerektirir.

| Alan | Anlam |
|---|---|
| `loading` | Uygulamanın iş durumu; ButtonSupport bunu kendi başına başlatmaz |
| `loadingText` | Yalnız görüntülenen işlem yazısı; boşsa orijinal text |
| `loadingPresentation` | WithText veya IndicatorOnly |
| `contentItem` | Açıkça atanabilen destek içeriği |
| `control` | Attached nesnenin desteklenen native hedefi; kullanıcı tarafından yazılmaz |

Yeni size, variant, extraVariations, selectedStyle veya ayrı iconName API’si eklenmez. Native `icon.name`, `icon.source` ve `display` korunur.

### 3.2 Örnek kullanım

```qml
// ÖNERİ: yeni namespace; mevcut deney hâlen Merce.Experiments.ButtonSupport.
import Qt.labs.StyleKit as SK
import Qavsi.ButtonSupport as Support

SK.Button {
    text: qsTr("Kaydet")
    icon.name: "material:save"
    enabled: permissions.canSave

    Support.ButtonSupport.loading: saveJob.running
    Support.ButtonSupport.loadingText: qsTr("Kaydediliyor…")
    contentItem: Support.ButtonSupport.contentItem

    onClicked: saveJob.start()
}
```

`permissions` ve `saveJob` uygulamanın nesneleridir. Bu örnek uygulama servisi tasarlamaz. Default buton için variation zorunlu değildir. Örnek modül oluşmadan kopyalanıp çalışan QML kabul edilmez.

### 3.3 Loading ve enabled

Attached nesne her butona aittir; application-global “loading state” yoktur. Loading guard içerik hiç istenmese bile çalışır. Desteklenmeyen hedefe attach edilirse bir diagnostic üretilir ve hedef mutasyona uğratılmaz.

Guard, caller’ın QML construction binding’leri kurulduktan sonra etkinleşir. Geçici `Binding { when: loading; property: "enabled"; value: false }` yaklaşımı önceki binding/value’yu Qt’nin normal geri yükleme davranışıyla korur. “Başta enabled değerini bool olarak sakla, bitince true yap” yaklaşımı kullanılmaz. [Qt Binding](https://doc.qt.io/qt-6/qml-qtqml-binding.html#when-prop)

İşlem sırasında caller’ın enabled bağımlılığı değişirse bitişte yeni sonuç değerlendirilir. İşlem sırasında imperative `enabled = ...` ataması guard ile yarışır; desteklenen sözleşme tek caller-owned binding’dir. İş zaten loading iken yaratılan kontrol, queued initialization aralığı ve ilk input teslimi özellikle test edilir.

Guard yalnız o butonu engeller. Aynı Action’ın başka buton, shortcut, menü veya backend üzerinden tetiklenmesini engellemez. İş servisi kendi duplicate-start politikasına sahip olmalıdır.

### 3.4 İçerik ve render ayrımı

- Native butonun kimliği, sinyalleri, Action’ı ve event işlemesi korunur.
- Renderer, native control font/icon/display değerlerini ve active StyleReader’ın çözülmüş text/padding bilgilerini tüketir.
- Native buton background’u korunur; içerik adaptörü ikinci background çizmez.
- Caller kendi `contentItem` nesnesini atayabilir; loading guard bundan bağımsız kalır.
- Attached nesnenin verdiği Item’ın QObject sahibi support, görsel parent’ı control olabilir; bu iki sahiplik ayrı yönetilir.
- C++ tarafında QQuickButton private header’ına bağlanılmaz. Public QML template tipiyle desteklenen hedef doğrulanır.
- Ortak renderer hiçbir somut kit veya Merce.Theme import etmez.

Default içerik native SK.BusyIndicator’ı kullanabilir; kit kendi indicator delegate’ini sağlar. Ortak helper spinner türünü seçmez. Gerekli bir content factory sözleşmesi çıkarsa ayrı bir review yapılır; mevcut MButton’ın factory API’si otomatik Runtime standardı olmaz.

### 3.5 Genişlik ve görünür yazı

Idle ve busy metin/ikon boyutları aktif font ile ölçülür; loading sırasında daha büyük footprint için yer ayrılır. IndicatorOnly label’ı gizler, butonun accessible ismini veya native text’ini silmez. Caller’ın açık width bağı kazanır; uzun metin bunun içinde elide edilir.

Default sunum spinner + işlem yazısıdır. Spinner-only açık tercihtir. Spinner’ın kendi dolu kutusu yoktur; button background’u ayrı katmandır. Reduced motion’da görünür bir statik işaret kalır. Gizli/minimized sahnede animasyon çalışmaz.

### 3.6 Checkable ve dolu/boş ikon

`checkable` kontrolün yeteneği, `checked` güncel durumudur. Style variation’ı başlangıçta sabit verilmiş toggle, checked değişti diye rol listesini değiştirmez.

Material glyph FILL kararı Material3 style state mapping’ine aittir. ButtonSupport bütün checked butonları kendiliğinden filled yapmaz. SF `star` ile `star.fill` farklı açık isimlerdir; Font Awesome solid/regular da açık paket seçimi olarak kalır.

### 3.7 Qt hatası süresince kapsam

SK.Button + ButtonSupport v1 deneyi, instance variation listesinin ömrü boyunca sabit olduğu kapsamda ilerler. Runtime role/size/appearance değişimi zorunlu legacy kullanım MButton’da kalır.

MButton mevcut butonu koruyup private style context’i yeniden yaratıyor. Bu workaround yeni ButtonSupport’a kopyalanmaz. İçerik reader’ını yenilemek native background resolver’ını onarmaz. Global theme/reset tetiklemek de yerel çözüm değildir. QTBUG minimal regression ayrı test olarak yaşar.

## 4. İkonlar

Mevcut Merce paketli ikon sözleşmesi korunur. Yeni Qavsi.Icons ve provider modülleri henüz kabul edilmemiş adaylardır; aşağıdaki taslak public Qt’nin sunduğu işin yerine yeniden implementasyon talimatı değildir.

### 4.1 AppIcon adı ve paket arayüzü

`AppIcon` adı korunur. Yeni import yolu `Qavsi.Icons` olur; Merce.Foundation üzerinden mevcut kullanım facade ile uyumlu tutulabilir.

Önerilen yeni Core kullanımında çizim ölçüsü ve renk caller/kit tarafından açık verilir; Merce’nin varsayılanları Merce facade’ında kalır:

```qml
// ÖNERİ: yeni paket henüz uygulanmadı.
import Qavsi.Icons

AppIcon {
    name: "fa-solid:star"
    size: pagePresentation.iconSize
    color: pagePresentation.iconColor
}
```

`material:save`, `fa-solid:star`, `fa-regular:star`, `sf:star.fill` gibi tam paket isimleri kullanılır. `save` gibi ortak isimler yalnız Gallery/uygulama adaptöründe bulunur. İkon koleksiyonları arasında eşdeğer ad tablosu eklenmez.

### 4.2 Qt’nin sağladığı iş tekrar yazılmaz

Görsel dosya, standard QIcon tema ve platform image altyapısı için Qt kullanılır. Qt 6.12 QIcon, temel native SF Symbols ve named/ligature font ikonlarını zaten sunar; yeni provider gerekçesi bunların yokluğu olamaz. Qt-only Cocoa/offscreen proof’unda temel yollar çalıştı. [Qt QIcon](https://doc.qt.io/qt-6/qicon.html), [04 kanıtı](../reviews/04-shared-boundary.md)

Aynı anda farklı koleksiyonlar, ikon başına font eksenleri veya explicit paket routing’i gerekiyorsa yalnız bu kalan fark ayrı proof ister. Global QIcon theme her ikon çiziminde değiştirilmez. Merce’nin `paket:ikon` biçimi, Qt’nin veya freedesktop icon-theme standardının kendisi değildir.

Mevcut Merce AppIcon’daki QtQuick.Controls.impl.IconImage kullanımı yeni Icons Core’un public temel bağımlılığı yapılmaz. Font provider public Text/font yolunu, image/native provider Qt image/QIcon yolunu kullanır. Renklenebilir raster maske gerekiyorsa public CPU image işlemi provider içinde sınanır; software’da çizilmeyen shader zorunlu tint yolu olmaz. SVG backend’i gerekirse yalnız o provider’ın optional bağımlılığıdır.

Önemli kapsam farkı:

| Tüketici | Paketli ad desteği |
|---|---|
| Qavsi AppIcon | Sağlayıcı kayıtlıysa evet |
| SK.Button + destek contentItem | Adaptör üzerinden evet |
| Stock SK.Button contentItem | Qt’nin normal icon altyapısı; paket parser’ı kendiliğinden eklenmez |
| Native menü | Menü adaptörünün QIcon/image dönüşümü ayrıca gerekir |
| Diğer native kontroller | İlgili kitin public delegate/adapter entegrasyonu ayrıca doğrulanır |

`icon.name: "material:save"` yazmak bütün Qt kontrollerini otomatik olarak değiştiren global bir QIcon theme kaydı değildir. Bu açıklık korunmadan “tüm ikonlar çalışıyor” beyanı yapılmaz.

### 4.3 Provider sözleşmesi

Provider: desteklediği paket adını, tam ikon adını doğrulamayı, çizim veya QIcon/image üretimini ve hata durumunu sağlar. Backend’in font veya SVG olması tüketici API’sini değiştirmez.

Kayıt engine kapsamındadır. Ayrı engine’e QObject taşınmaz. Aynı pakete çakışan provider veya core namespace override reddedilir. QML import sırası tesadüfüne güvenilmez: optional provider modülünün engine initialization’ında kayıt açıkça tanımlanır ve “ilk AppIcon’dan önce/sonra import” durumları test edilir.

Yeni engine registry/factory yazmadan önce public QQmlEngine image-provider/plugin initialization kullanılır. Qt-only iki-engine proof’unda bu ownership/izolasyon çalıştı. Mevcut Merce registry’si bir compatibility implementasyonudur; otomatik olarak yeni Runtime standardı sayılmaz.

Material/font metadata yalnız kendi modülü açıldığında paketlenir. Icons Core’un fallback’i Material Symbols fontunu zorunlu hale getirmez. Yeni bağımsız Core, explicit source veya küçük nötr bir eksik-ikon çizimi kullanır. Legacy Merce fallback davranışı facade’da sürdürülüp deprecation ayrı değerlendirilir.

### 4.4 Provider’a ait görünüm

Material: weight, optical size, FILL ve renderType sağlayıcı tarafından uygulanır; seçilen kontrolün state politikası kitten gelir. Mevcut karşılaştırma için wght=400, opsz=24 ve desteklenen renderer seçimi Material3’ün tercihidir, Icons Core’un evrensel default’u değildir.

Font Awesome: solid/regular/brands koleksiyonlarını açık tutar; selected diye başka pakete geçmez. Mevcut kaynak SVG ise onu fontmuş gibi yeniden paketlemeyiz.

SF Symbols: native OS katalog sürümünü kullanır. Bir font dosyası indirildiği veya tüm OS sürümlerinde aynı katalog bulunduğu iddia edilmez. macOS dışı, eksik symbol ve eski OS için açık missing/source davranışı vardır. Katalog listesi sanallaştırılır; tüm semboller için eager image üretimi yapılmaz.

### 4.5 Performans ve hata davranışı

Glyph/image cache gerekiyorsa provider, ad+appearance+size+DPR anahtarını ve temizleme sınırını açık tanımlar. Tema renginin cache key’e dahil olup olmadığı çizim yöntemine göre belirlenir. Renkli raster çıktıyla tint edilebilir maskenin önbelleği aynı değildir.

Eksik paket/ikon diagnostic’i engine ömründe kontrollü üretilir; her frame log basılmaz. Metadata/provider hatası UI’yi durdurmaz. Kaynak override ve fallback çizimi software üzerinde de doğrulanır.

## 5. Platform ve native pencere

### 5.1 Olgu ile politika ayrımı

Qt’nin doğrudan verdiği OS motion/contrast, renderer, window/screen/DPR ve sistem pencere işlemleri public Qt API’sinden okunur; yeni Platform singleton’ında yeniden paketlenmez. Qt’de olmayan tercih için dar adaptör kendi projesinde kalır ve ortaklık kapısı gerektirir. Apple kit “navigation için native cam istiyorum” kararını verir; ortak mekanizma somut kit adını sormaz.

Destek koşulları: OS API/SDK availability, QPA backend, gerçek window handle, scene graph hazırlığı, alpha surface, renderer, OS accessibility tercihleri ve desteklenen geometri. Tek global `Platform.isMacOS` yeterli değildir.

### 5.2 Pencere ve yüzey iki farklı mekanizma

- **Window backdrop:** Pencere arkasındaki OS/compositor içeriği.
- **Surface material:** Belirli navigation/header yüzeyinin native malzemesi.
- **QML decoration:** Kitin içerik, border, tint ve fallback çizimi.

Bu roller bir araya gelebilir; aynı piksele iki native cam katmanı bindirilmez. Window background blur ile surface Liquid Glass’ın aynı şey olduğu söylenmez.

Kitin public mekanik API’sinin son alan isimleri ilk spike ile kesinleşir. Minimum sözleşme: window/target, etkinlik, talep edilen native effect, mekanik durum (`Unavailable/Pending/Attached/Failed`), desteklenmeme nedeni ve cleanup. `Attached` native kaynağın bağlandığını anlatır; doğru optik çıktı veya HIG uyumu anlamına gelmez. Mevcut Merce `active` özelliği bu taslakta yeniden adlandırılmaz. Evrensel MaterialTheme üretilmez.

### 5.3 Tek owner

Native malzeme kullanan her QQuickWindow için tek owner vardır. View, mask, observer ve native property değişikliklerini bu owner yönetir. Bu invariant bir Qavsi MaterialCoordinator sınıfını otomatik gerekli kılmaz; mevcut bridge kit/projede kalır. Ortak bir coordinator ancak aynı standart lifecycle ve gerçek bağımsız tüketim kanıtıyla çıkar.

İkinci root/native owner isteği sessizce ayrı snapshot açmaz: aynı owner’a yönlenir veya açık diagnostic ile reddedilir. A/B attach-detach sıralarında eski snapshot’ın yeni window state’ini ezmediği test edilir. Birden fazla view’ın aynı container’ı kullanması tek owner kanıtı değildir; bu risk mevcut Merce kodunda kaynak incelemesiyle görüldü, bu tur runtime’da sınanmadı. [02 sahiplik incelemesi](../reviews/02-apple.md)

Sayfa doğrudan NSView yaratmaz. Kitin public root entegrasyonu kendi native bridge/owner’ına bağlanır. Request nesnesi veya style yok olursa eski request serbest bırakılır. Native mekanizma kapanınca değiştirilmiş window state’i geri yüklenir; başka bir owner’ın yeni değişikliği eski snapshot ile ezilmez.

Bu coordinator ayrı bir pencere yöneticisi değildir. Public minimumla başlanır; kanıtlanmamış genel compositor/region engine’i yazılmaz.

Apple NSGlassEffectView ve Windows Mica/Acrylic tek evrensel “cam” enum’unda eşitlenmez. Native API’nin public olması, tasarım sistemleri arasında ortak materyal semantiğini ispatlamaz.

### 5.4 Native view sınırları

macOS’ta public AppKit API’leri ve runtime availability guard kullanılır. QQuickWindow’un content view/Metal layer hiyerarşisi yeniden sahiplenilmez. Private Qt native interface’leri public API gibi gösterilmez.

NSGlassEffectView’in bağlanması, ayrı Qt Metal sahnesindeki yazıya AppKit vibrancy’si sağlamaz. Mevcut bridge’de glass `contentView` boş, QML foreground ayrı sahnededir; bu nedenle native instance’ın varlığı okunaklılık veya doğru Liquid Glass composition kanıtı değildir. İlk native dilim tek regular yüzey ve kitin okunaklı foreground/tint’i ile sınanır; gerçek masaüstü üzerinde light/dark/contrast ölçümü başarısızsa opak fallback kullanılır. Qt root view’ını glass contentView içine taşıyarak ownership/input sorunu üretilmez. [02 composition bulgusu](../reviews/02-apple.md)

Public header’da görünmek tek başına public destek sözleşmesi değildir: Qt 6.12 kaynaklarında `QWindow::handle()` internal olarak belgelenir. Yeni bridge belgeli `winId()` ve PlatformSurface yaşam döngüsünden başlamalı; WId→NSView eşlemesi Qt’nin platform dokümanına ve test edilen sürüme bağlanmalıdır. Mevcut Merce implementasyonu bu mimari turunda değiştirilmez. [Qt 6.12 QWindow kaynağı](https://github.com/qt/qtbase/blob/025bdad181de241e81bf853c8a2d7bf3d19261a9/src/gui/kernel/qwindow.cpp#L2215)

Native view input’u yutmaz; Qt kontrolleri hit testing, focus ve erişilebilirlik sahibi kalır. Opacity, clip, visibility, stacking, scroll ve pencere değişimi izlenir. Keyfi rotation/3D transform veya desteklenmeyen clip için sessiz yanlış çizim yerine QML fallback kullanılır.

İlk native dikey dilim, transform desteğini dar tutabilir: item ölçeği 1 ve yalnız translation; bu DPR=1 demek değildir. Scene rect’i ölçeklemek corner radius/clip’in de aynı geometriye dönüştüğünü kanıtlamaz. Scale/nonuniform transform ancak radius/clip dahil gerçek native test geçince açılır. Genel transform motoru yazmak zorunlu değildir.

Geometri pencerenin ekran/DPR bilgisinden hesaplanır. Primary screen singleton’ı kullanılarak bütün pencerelere aynı ölçek uygulanmaz. Window surface yeniden yaratılması, ekran değişimi ve minimize/restore cleanup senaryosudur.

Window framebuffer clear için yapısal RGBA(0,0,0,0) gerekebilir; bu Theme rengi değildir. Diğer görünür yüzeyler kit token’larını tüketir. QML ve native renk uzayı/premultiplied alpha sınırı Metal üzerinde ayrıca doğrulanır.

### 5.5 Pencere chrome’u

Önce public Qt/QWindow sistem taşıma ve resize API’leri kullanılır. Başlığın boş alanları drag bölgesidir; input, ComboBox, buton ve native window düğmeleri bu bölgeden hariçtir.

Double-click maximize/restore, fullscreen, traffic-light konumu, kenar/köşe resize ve cursor gerçek input ile doğrulanır. Başlığın yüksekliği veya logo boşluğu Runtime token’ı değildir; kit/uygulama geometry’sidir.

QWindowKit kullanımı ancak ayrı ve isteğe bağlı window-chrome adaptörü olabilir. Native camın, tasarım token’larının veya input stillerinin sahibi olamaz. Kullanılmadığında fetch/build bağımlılığı oluşmaz. [QWindow public işlemler](https://doc.qt.io/qt-6/qwindow.html#startSystemMove)

### 5.6 Apple görünümü ve fallback

Apple kit, Liquid Glass’ı navigation/control katmanlarında; standart malzemeleri içerik yüzeylerinde kullanır. Bu, bütün dialog/popover’larda glass yasağı değildir: Apple’ın regular malzemesi uygun kontrol/alert bağlamında kullanılabilir; karar içerik yoğunluğu ve okunaklılığa bağlıdır. Native cam desteği “her şeyi saydamlaştır” anlamına gelmez. [Apple materials](https://developer.apple.com/design/human-interface-guidelines/materials)

İlk incelemede kullanılan tarihsel designmd.app Liquid Glass dosyasındaki “No circular spinners” iddiası Apple standardı değildir; yeni VoltAgent Apple dosyasına aitmiş gibi gösterilmez. Apple.com web pazarlama tarifi de native Liquid Glass şartnamesi değildir. İlk native kabul software opak yüzey/okunaklı dialog’dan başlar; ardından macOS 26/Cocoa/Metal’de tek regular yüzey doğrulanır. Window backdrop veya ikinci glass bölgesi ancak sonra açılır. Eski OS’de NSVisualEffectView yolu Liquid Glass diye etiketlenmez. [Apple progress indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators), [güncel kaynak ayrımı](../references/SOURCES.md)

Software veya desteklenmeyen platformda opak/tinted okunaklı kit yüzeyi kalır. Reduced transparency ve increased contrast, estetik tercihten önce gelir. Desktop screenshot capture yaparak cam taklidi üretmek varsayılan fallback değildir.

### 5.7 Fluent materyali ve taşınabilir ilk dilim

Mica, Acrylic ve Smoke kitin farklı katmanlarıdır; aynı blur/alpha token’ının üç adı değildir. Mica uzun ömürlü pencere tabanı, desktop Acrylic geçici yüzey, Smoke modal scrim bağlamındadır. İlk Fluent dilimi opak kit yüzeyidir; Windows native malzeme gerektirmez. Web referanslı portable kit ile Windows/WinUI görünümü ayrı kapsam olarak seçilir. [03 materyal incelemesi](../reviews/03-fluent.md)

Native Windows işi açılırsa DWM backdrop yolu ile Windows App SDK controller yolu karıştırılmaz: public DWM enum’u Windows 11 build 22621 minimumuna, App SDK yolu kendi OS/runtime/deploy koşullarına bağlıdır. HWND/renderer/activation/tercihler, surface recreation ve cleanup testi olmadan destek beyanı verilmez. XAML’e ait element API’si keyfi Qt Quick Item’a doğrudan taşınmaz. Bu köprü bugün kit/projede kalır; ortak Apple/Fluent “glass” abstraction’ı doğurmaz.

## 6. Toolbar seçicileri, ComboBox, input ve badge

### 6.1 Toolbar seçicisi

Toolbar’daki theme/kit/profile seçicisi bir menü etkileşimidir; editable veya form-value ComboBox ile aynı kontrol sözleşmesi olmak zorunda değildir.

Önce Qt Quick Controls Menu’nün public native popup desteği değerlendirilir. Qt’nin yeterli olduğu yerde ayrı NSMenu implementation’ı yazılmaz. QML fallback aynı model/seçim/Action verisini tüketir. Native menüde custom QML delegate görünümünün aynen çizileceği varsayılmaz. [Qt Menu](https://doc.qt.io/qt-6/qml-qtquick-controls-menu.html)

Ayrı NSMenu adaptörü gerekirse menü açıkken model mutation, cancel, disabled item, checkmark, separator, klavye, focus return ve QObject yıkımı test edilir. Modal/yerel event loop reentrancy göz önüne alınır.

### 6.2 Form ComboBox

Form ComboBox native Qt value/model/editable semantics’ini korur. Popup içerik genişliği, scrollbar/inset ve delegate geometri hesabı tek yerde yapılır. Hover, keyboard highlight ve selected state ayrı kaynaklardır; cursor popup’tan çıkınca hover kalmaz, selection silinmez.

Merce’de kabul edilmiş 4 px iç boşluk Merce’nin token’ı üzerinden korunur. Başka bir kitin popup inset’i aynı olmak zorunda değildir. Seçili ve hover plate’leri aynı kullanılabilir genişliği, ilk/son satır dahil doldurur.

### 6.3 Floating label

Material3 floating caption kendi input delegate’ine aittir. Native `Accessible.labelFor/labelledBy` ilişkisi kullanılır. Accessible.name, otomatik olarak mevcut value’dan türeyebileceği için caption kaynağı gibi yorumlanmaz.

Aynı etiketi iki kez çizmek yerine yalnız açıkça ilişkilendirilmiş dış label gerektiğinde gizlenir; style/fallback değişiminde binding geri yüklenir. Search, çok satırlı input ve unnamed kontrol kendiliğinden form caption kazanmaz. [Qt Accessible](https://doc.qt.io/qt-6/qml-qtquick-accessible.html)

### 6.4 Badge

Badge görsel tarifiyse SK.Label + kit custom delegate yeterlidir. Intent/size kitin variation modelinde kalır. Native glass veya background objesi her badge instance’ında davranış bileşeni olarak gömülmez.

Tek label hedeflenir; ikinci label/Loader yalnız yaşam döngüsü veya gerçek davranış ihtiyacı kanıtlandığında kalır. Mevcut MBadge geçişi ayrı test edilir; yeni Runtime’a `Qavsi.Badge` eklemek varsayılan yol değildir.

### 6.5 Navigation

Destination listesi, route, section başlığı, genişleme politikası ve responsive layout uygulamadadır. Kit yalnız navigation item görünümü veya gerçek farklı bir interaction pattern’i sağlıyorsa kendi public bileşenini yayınlar.

Her uygulamaya aynı sidebar geometrisi veya aynı sol selection bar zorlanmaz. Gallery’nin navigation bilgi içeriği stil denemesi yüzünden değişmez. Collapse/expand ve accessibility semantics’i uygulama kontrolünün sözleşmesinde kalır.

## 7. Motion, software ve erişilebilirlik

Öncelik sırası: platform olgusu → uygulamanın azaltma tercihi → kitin güvenli render yolu → caller’ın desteklenen override’ı.

OS reduced motion true ise uygulama tercihi false diyerek bunu iptal etmez. Şeffaflık ve yüksek kontrast için platform desteği yoksa “bilinmiyor” bilgisi korunur; güvenli fallback mümkündür. Her kit kendi motion duration/easing veya spring modelini tutar; ortak Runtime easing kataloğu yoktur.

ShaderEffect software renderer’da çizilmez; görünmez shader nesnesini tutup başarı saymayız. Aynı anlamı veren Rectangle/Shape/Text/native indicator yolu seçilir. Shape renderer tercihi Qt’nin destek bilgisiyle değerlendirilir. [Software adaptation](https://doc.qt.io/qt-6/qtquick-visualcanvas-adaptations-software.html)

Her kontrol için: Tab/Shift+Tab, Space/Enter, mnemonic/Action, disabled, accessible name/role, label ilişkileri ve mümkünse ekran okuyucu kontrolü bulunur. Görsel focus yalnız mouse click sonrasında yanlışlıkla kalıyorsa native visualFocus ile karşılaştırılır; bütün focus göstergeleri kaldırılarak sorun saklanmaz.

Kontrast hedefleri metin ve non-text için ayrı belirtilir; ölçüm gerçek bileşik background üzerinden yapılır. Native camda değişken masaüstü nedeniyle tek sabit hex karşılaştırması yeterli değildir. Tasarım kiti increased contrast/reduced transparency durumunda ölçülebilir opak çift kullanır.

## 8. Kontrol family kapsam matrisi

| Family | StyleKit mapping | Gerekirse kit delegate’i | Runtime davranışı |
|---|---|---|---|
| Button / Round / Tool / Delay | Font, padding, state, outline | Ripple, geometry, shadow | ButtonSupport yalnız desteklenen Button kapsamı |
| TextField / TextArea / Search | Font/palette/state | Floating label ve underline | Yeni input state machine yok |
| ComboBox / SpinBox | Model/input davranışı native | Popup/indicator/label | Toolbar menü adaptörü ayrı |
| Check / Radio / Switch | Selection state native | Track/handle/checkmark | Action/checked tekrar yazılmaz |
| Slider / Range / Dial | Value/drag/keyboard native | Track, thumb, tick, value bubble | Yeni değer modeli yok |
| Progress / Busy | Native value/running | Kit indicator renderer’ı | İş/ilerleme servisi uygulamada |
| Menu / MenuItem / Toolbar | State ve spacing | QML fallback görünümü | Native menü opsiyonel |
| Tab / ItemDelegate / Swipe | Native selection/input | Kit selection/background | Navigation route uygulamada |
| Popup / Dialog / Drawer | Focus/modal/dismiss native | Kit yüzeyi ve scrim | Feedback orkestrasyonu ayrı |
| Pane / Frame / GroupBox / Page | Content surface ve spacing | Material/background | Yeni layout yöneticisi yok |
| ScrollBar / ScrollIndicator / PageIndicator | Native range/state | Kit şekli/indicator | Sanallaştırma uygulama/view’da |
| Label / Badge | Font/palette | Kit text/badge renderer’ı | Evrensel typography wrapper’ı yok |

Bir family’nin mapping’i yoksa hangi Qt fallback’inin göründüğü açık kaydedilir. Default fallback’in varlığı “kit bütün kontrolleri kapsıyor” beyanı değildir. Başlangıçta her kit kendi coverage tablosunu yayınlar; referans kit içindeki hiçbir native bileşen otomatik olarak Qavsi’ye taşınmış sayılmaz.
