# Qavsi Fluent — DESIGN.md

**Tarih:** 2026-10-25. **Durum:** Resmi Microsoft kaynakları kullanıcı tarafından esas alındı. Aşağıdaki Qt uyarlaması taslak sözleşmedir; yeni Fluent kitinin çalıştığı beyanı değildir.

## Otorite, platform ve sürüm

Birincil otorite [Microsoft Fluent 2](https://fluent2.microsoft.design/design-tokens), ilgili resmi control rehberi ve [Windows materials](https://learn.microsoft.com/en-us/windows/apps/design/style/mica) belgeleridir. VoltAgent deposunda ayrı Fluent DESIGN.md bulunmadı; eski designmd.app girdisi tarihsel karşılaştırmadır.

Fluent 2 Web ve Windows/WinUI 3 tarifleri aynı recipe sayılmaz. İlk portable Button karşılaştırması için resmi Web Button ve Microsoft fluentui commit `6d3e9d686434d69ac8fc9ca4c4a93f3afaa3d676` kullanılır. Bu, WinUI pixel/native eşdeğerliği veya bütün kitin nihai platform profili kararı değildir. Windows-native dilimi kendi OS/SDK/recipe kaydıyla ayrıca doğrulanır. [Kaynak envanteri](../../references/official/sources.json), [03 incelemesi](../../reviews/03-fluent.md)

Bu DESIGN.md Microsoft’un yayımladığı dosya değil, resmi kaynaklara dayalı Qavsi proje sözleşmesidir. Önce Qt’nin mevcut FluentWinUI3 stilinin ihtiyacı karşılaması değerlendirilir; yeni StyleKit kitinin ek değeri public recipe/config ve root stil geçişi gibi somut ihtiyaç olmalı. Hazır Qt stili SK.Style kalıtım tabanı yapılmaz.

## Token ve Theme modeli

Fluent’in global değerleri ile semantic alias rolleri ayrılır. Neutral/brand foreground, background, stroke, focus ve selected rolleri kendi Theme’inde çözülür; Merce’nin primary/container şemasına zorunlu çevrilmez.

Light, dark, brand ve high-contrast karşılıkları birlikte tanımlanır. Contrast preference, kullanıcının gerçek sistem renk palette’sinin tamamı değildir. Bilgi yoksa unknown ve kitin belgeli opak fallback’i korunur. Render dosyaları semantic token kullanır; renk veya ölçü adına uymayan bağımsız sabitler üretmez. [Microsoft token modeli](https://fluent2.microsoft.design/design-tokens)

## Typography, geometri ve durum

[Microsoft typography](https://fluent2.microsoft.design/typography) platform type ramp’larını ayırır. Seçilen Web rolü ile Windows Segoe UI Variable rolü sessizce karıştırılmaz. Sistem fontu/fallback metrikleri kaydedilir; Segoe veya ikon fontları Windows’tan kopyalanıp paketlenmez.

Control ölçümü gerçek font, çeviri, RTL ve metin büyümesiyle doğrulanır. Native font alt kümesi ve StyleKit line-height sınırı belgelenir; gerektiğinde küçük kit içi content adaptörü sınanır. Caller’ın width/font/padding/spacing ve açık sıfır bağları korunur; background’dan font bağına yazılmaz.

[Resmi Button rehberi](https://fluent2.microsoft.design/components/web/react/core/button/usage) action hierarchy’si için kullanılır. İlk örnekte neutral secondary ve açıkça seçilmiş primary karşılaştırılır. Outline/subtle/transparent, toggle, split ve compound destekleri uygulandıkça coverage alır. Split/menu interaction davranışı sırf görünüm için yeniden yazılmaz.

Normal, hover, pressed, visualFocus, checked ve disabled kombinasyonları birlikte çözülür. Keyboard focus selected/hover zeminini yok eden tek renk state’i değildir. Qt input/Action/value/model/Accessible davranışı native kontrolde kalır.

## Windows materyali ve portable yüzey

Mica kalıcı pencere tabanı, Acrylic uygun geçici/in-app yüzey ve Smoke modal scrim bağlamındadır. Üçü aynı alpha/blur token’ı değildir. Native Mica translucent bir card recipe’si olarak taklit edilmez. [Mica](https://learn.microsoft.com/en-us/windows/apps/design/style/mica), [Acrylic](https://learn.microsoft.com/en-us/windows/apps/design/style/acrylic)

İlk portable dilim opaktır ve Windows-native dependency istemez. DWM ile Windows App SDK controller yolları farklı minimum OS/deploy koşullarına sahiptir. Native iş açılırsa tek yol, tek window owner, HRESULT/destek kontrolü, surface recreation, active/inactive ve preference cleanup sınanır; HWND veya D3D varlığı yeterli değildir.

Linux/macOS/WASM/software’da Windows native materyal vaat edilmez. Opak kit yüzeyi ve anlamı koruyan border/state/focus kullanılır; screenshot blur fallback’i yoktur.

## Motion ve erişilebilirlik

[Microsoft motion](https://fluent2.microsoft.design/motion) ile uyumlu süre/easing recipe’si kitten gelir. Eski Reveal efekti veya Material ripple’ı Fluent 2 zorunluluğu yapılmaz. Reduced motion’da gereksiz hareket durur, state/iş bilgisi korunur; theme/contrast geçişinde kritik metin ve focus geçici olarak okunmaz hale gelmez.

Metin/ikon, interactive state ve gerçek background üzerinden kontrol edilir; disabled kontrastı ayrı değerlendirilir. Qt Accessible ve platform palette kullanılır; Web ARIA/forced-colors property adları QML’e kopyalanmaz. Loading işi ve duplicate-start uygulamadadır; mevcut ButtonSupport yalnız ihtiyaç varsa ayrıca seçilir, yeni ortak dependency sayılmaz.

## İlk uygulama ve kabul

İlk dilim: tek opak Button örneği, kendi Fluent Theme + doğrudan SK.Style; Qt dışında Merce veya Runtime gerektirmez. Source ve installed consumer, light/dark/contrast, normal/hover/pressed/checked/focus/disabled, caller bağları, software ve uzun/RTL metin test edilir.

TextField/ComboBox/selection/value/popup/navigation/scroll family’leri ayrıca recipe/state/fallback kabulünden geçer. Bir butonun başarısı tüm kontroller veya WinUI 3 desteği değildir. Native Mica/Acrylic, Windows deployment ve performans bu aşamada NOT-RUN’dır.

Ortak mekanizma ancak [Qavsi kabul kapısını](../../docs/SHARED-ADMISSION.md) geçerse çıkar. Global StyleKit/sabit variation ve mevcut MButton sınırı [kontrol kontratında](../../docs/CONTRACTS.md) korunur.
