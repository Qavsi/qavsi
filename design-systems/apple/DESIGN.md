# Qavsi Apple — native Liquid Glass DESIGN.md

**Tarih:** 2026-10-25. **Durum:** Native Liquid Glass hedefi ve resmi Apple otoritesi kullanıcı tarafından korunmuştur. Qt uyarlaması henüz uygulanmış/validasyonu tamamlanmış ürün değildir.

## Kaynak ve hedef

Birincil otorite [Apple HIG materials](https://developer.apple.com/design/human-interface-guidelines/materials), [typography](https://developer.apple.com/design/human-interface-guidelines/typography) ve public [NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview) API’sidir. Native Glass API’si macOS 26+ availability guard ile kullanılır; Qt 6.12/Cocoa/Metal ilk hedef çalışma yoludur. [Kaynak kayıtları](../../references/official/sources.json)

VoltAgent’ın [Apple DESIGN.md](../../references/voltagent/apple/DESIGN.md) dosyası apple.com web pazarlama analizidir; yardımcı görsel referans olarak kalır. Web hero/type-scale, CSS blur, palette veya pill CTA, native kitin otoritesi değildir. Eski designmd.app glassmorphism girdisi tarihsel kalır. Bu dosya resmi Apple belgesinin kopyası değil, Qavsi uyarlama sözleşmesidir.

## Malzeme hiyerarşisi

| Alan | Tasarım kararı |
|---|---|
| Navigation ve üst panel | İçeriğin üzerinde ayrı functional katman; ilk native tarif regular Liquid Glass |
| İçerik/canvas/card/form | Standart malzeme veya kitin okunaklı opak yüzeyi; tüm content camlaştırılmaz |
| Alert/popover | İşlev/yoğunluğa göre regular malzeme uygun olabilir; genel cam yasağı veya zorunlu saydamlık yok |
| Clear Glass | Yalnız görsel olarak zengin arka planın öncelikli olduğu uygun bağlam; varsayılan panel malzemesi değil |
| Window backdrop | Masaüstü arkası ile pencere içi yüzey farklı mekanizmalar; aynı pikselde iki native blur/glass katmanı oluşmaz |

Malzemenin nerede kullanılacağı Apple kit politikasında; route, navigation bilgi mimarisi ve uygulama yerleşimi uygulamada kalır. Her badge/kontrol yeni native view sahibi açmaz. Ortak Runtime’da AppleTheme veya evrensel glass enum’u yayımlanmaz.

## Typography, kontrol ve state

Sistem typography rolleri ve gerçek font metrikleri kullanılır; web belgesindeki sayılar macOS kontrol ölçeği yapılmaz. Native font adını kullanmak font binary’si dağıtmak değildir. Metin büyütme, locale, RTL ve caller font/ölçü bağları korunur.

Standart kontroller Qt’nin focus/input/Action/checked/model/IME davranışını korur. Hover, pressed, keyboard visualFocus, checked, disabled ve error görünümü kitte çözülür. Native camın aktif olması focus göstergesini veya foreground kontrastını gereksiz kılmaz. Form ComboBox ile toolbar menüsü ayrı sözleşmededir.

Yüzey tint’i, foreground, stroke ve fallback rengi Apple Theme/recipe’sinin sahibidir. Etiket ve dialog okunaklılığı gerçek birleşik arka plan üzerinde sınanır; sabit bir hex çiftinin oranı tek başına native cam testi değildir.

## Native entegrasyon sınırı

Public winId/PlatformSurface lifecycle’ı, GUI thread ve pencere başına tek owner kullanılır. Owner view/mask/observer ve değiştirdiği window state’ini yönetir; başka güncel değişiklikleri eski snapshot ile ezmez. Qt/Metal root view yeniden sahiplenilmez. Internal QWindow::handle() yeni public sözleşmeye taşınmaz.

NSGlassEffectView’in attach edilmesi, ayrı Qt sahnesindeki metnin native contentView/vibrancy sistemine katıldığını kanıtlamaz. Attached mekanik durumdur; optik/HIG uygunluk sonucu değildir. Gerekirse kit foreground/tint’i güçlendirir veya fallback seçer. Masaüstü capture alıp sürekli blur üretmek varsayılan çözüm değildir.

İlk geometri item scale=1 + translation ile sınırlanabilir; DPR=1 zorunluluğu değildir. Radius/clip, ekran değişimi, minimize/restore, duplicate-owner ve destruction testleri geçmeden genel transform desteği verilmez. Mevcut kaynak riskleri [02 incelemesindedir](../../reviews/02-apple.md); bu belgede ürün kodu düzeltilmiş sayılmaz.

## Tema, motion, loading ve software

Light/dark/system, reduce motion, reduce transparency ve increased contrast kit recipe’sine birlikte yansır. Qt’nin public motion/contrast bilgisi doğrudan kullanılır; Qt’de olmayan tercih kitte dar OS köprüsüyle ele alınır. Bilinmeyen tercih false sayılmaz; güvenli opak başlangıç mümkündür.

Reduced motion’da gereksiz transform/morphing durur. [HIG progress indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators) doğrultusunda alan/iş bağlamına uygun gösterge seçilir; circular spinner kategorik yasaklanmaz. Mevcut ButtonSupport’un spinner+yazı varsayılanı evrensel Apple kuralı değildir. Accessible isim ve işlem bilgisi kaybolmaz.

Software, desteklenmeyen OS/QPA veya native başarısızlıkta cam kapalı; kitin shadersız, opak ve okunaklı yüzeyi kullanılır. Eski OS’de NSVisualEffectView yolu varsa standart materyal diye adlandırılır; Liquid Glass başarısı sayılmaz.

## İlk uygulama ve kabul

Önce bağımsız kendi Theme/Style ile opak surface ve okunaklı dialog. Sonra tek regular NSGlassEffectView navigation/üst panel örneği. Gerçek masaüstü/Metal’de görünüm, metin, input, OS tercihi ve cleanup geçince ikinci bölge veya window backdrop eklenir.

Source/installed consumer Merce’den bağımsız olmalı. Karşılaştırma light/dark, parlak/yoğun desktop, ekran/DPR değişimi, software ve reduced preference durumlarını kapsar; insan UI incelemesi yalnız sol monitördedir. Önceki compile-only probe bu native/görsel kabulün yerine geçmez.
