# Qavsi Material3 — DESIGN.md

**Tarih:** 2026-10-25. **Durum:** Resmi kaynak seçimi kullanıcı tarafından kabul edildi; aşağıdaki Qt uyarlaması uygulama sözleşmesi taslağıdır, ürün desteği beyanı değildir.

## Kaynak ve kapsam

Birincil otorite [Google Material 3](https://m3.material.io/). Uygulanacak family’nin sayısal recipe’si, resmi Google component/token kaynağının tek sabit sürümünden alınır. İlk buton referansı MDC-Android 1.13.0, commit `84c0e6d6d493fcf176d2099683245b78afbeb78d`: [button tokens](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/button/res/values/tokens.xml). Android implementasyonu Qt dependency’si değildir.

M3, Material You ve Expressive isimleri aynı recipe sürümü sayılmaz. Önceki Compose 1.5.0-beta02 karşılaştırması ayrı bir referanstır; farklı size/type/shape değerleri sessizce karıştırılmaz. Her family için edisyon, platform, commit ve sapma kaydı tutulur. [Kaynak digest’leri](../../references/official/sources.json)

Bu dosya Qavsi’nin resmi kaynaklara dayalı proje sözleşmesidir; Google’ın yayımladığı veya onayladığı bir DESIGN.md kopyası değildir. Eski designmd.app dosyası yalnız tarihsel inceleme girdisidir.

## Görsel model ve token sahipliği

Material3 kendi renk, typography, shape, elevation, state-layer ve motion rollerinin sahibidir. Primary/on-primary, container/on-container, surface/on-surface, outline ve error rolleri kullanılan M3 edisyonundan çözülür. Light/dark bir hex’i koyulaştırarak üretilmez; karşılıklı foreground/background çiftleri birlikte tanımlanır.

Kitin typed Theme’i Merce Theme’inden veya ortak bir Qavsi palette’sinden türemez. Görünür değerler ilgili kit token’ından gelir. Kaynak token değerleri Qt component dosyalarına bağımsız magic number olarak kopyalanmaz. Marka girdisi, üretilmiş palette ve render recipe ayrı sahipliktedir.

## Typography ve geometri

[Resmi typography token’ları](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/typography/res/values/tokens.xml) family/size bazında uygulanır. Container, icon, padding, shape ve type role tek recipe oluşturur; yalnız yüksekliği değiştirerek yeni size yayımlanmaz. Paint boyutu ve interaction/touch alanı ayrı doğrulanır.

Qt’nin point/pixel/font çözümlemesi kullanılır. Uzun Türkçe metin, RTL, metin büyütme ve caller’ın açık ölçü/font bağları korunur. StyleKit TextStyle tam line-height/tracking/font-weight şeması değildir. Önce native family QFont alt kümesi; exact line-height veya FILL gerekiyorsa açık, kit içi content adaptörü sınanır. Background delegate’den `control.font` yazılmaz. [Qt tipografi sınırı](../../docs/CONTRACTS.md#26-geometri-ve-tipografi)

## Kontrol ve durum tarifleri

| Alan | Material3 sorumluluğu |
|---|---|
| Button | Filled, tonal, elevated, outlined, text ve icon/FAB tarifleri ayrı coverage alır; var olmayan variation desteklenmiş sayılmaz |
| State | Normal, hover, pressed, visualFocus, checked ve disabled kombinasyonları; state-layer, foreground, outline ve elevation birlikte çözülür |
| Ripple | Kitin interaction feedback’i; clipping/geometri ve disabled kapısı sınanır, kontrolün input davranışı Qt’de kalır |
| TextField/ComboBox | Floating caption, yardımcı/error metni ve indicator; etiket tek çizilir, erişilebilir label ilişkisi ve native editing/model davranışı korunur |
| Selection/value | Check/radio/switch, slider/range/dial için kendi track/indicator recipe’si; Qt value/keyboard state machine’i tekrarlanmaz |
| Popup/dialog/surface | Material3 surface/elevation/scrim hiyerarşisi; Apple camı veya Fluent Mica rolü eklenmez |
| Label/badge | İlgili type/intent/size tarifi; ikinci label veya yeni kontrol ancak davranış gerektiriyorsa |

Material icon FILL kararı kitin checked/interaction tarifindedir; tüm checkable ikonlar otomatik dolmaz. Paketli ikon arayüzü mevcut Merce compatibility alanında korunur; bu kiti kullanmak yeni provider veya ButtonSupport dependency’sini zorunlu kılmaz.

## Motion, accessibility ve fallback

[Resmi motion token’ları](https://github.com/material-components/material-components-android/blob/84c0e6d6d493fcf176d2099683245b78afbeb78d/lib/java/com/google/android/material/motion/res/values/tokens.xml) seçilen family edisyonuna aittir. Basit easing animasyonu fiziksel spring desteği diye sunulmaz. Reduced motion’da gereksiz ripple/transform hareketi durur; işlem ve focus bilgisi görünür kalır.

Keyboard focus, accessible name/role, kontrast, disabled ve error ayrımı renk dışı işaretlerle de anlaşılır olmalı. Software’da ShaderEffect zorunlu değildir; sabit state-layer/outline/indicator ile aynı anlam korunur. GPU ile aynı optik çıktı vaat edilmez.

## İlk uygulama ve kabul

İlk dilim: tek Small normal SK.Button, kendi Theme + doğrudan SK.Style; Merce ve Runtime’sız source/installed consumer. Sonra yalnız doğrulanan appearance/size eklenir. Diğer family’ler için boş dosya üretilmez.

Aynı instance ile light/dark, sabit variation, hover/checked/focus/disabled, explicit sıfır spacing/padding/tracking, canlı font binding’i ve software doğrulanır. Yeni recipe görsel olarak kendi resmi sürümüyle karşılaştırılır. Önceki 399 assertion bu kitin görsel kabulü değildir.

Loading işi uygulamadadır. Mevcut MButton/deney API’si kaldırılmaz; global StyleKit ve dinamik variation sınırı [ortak kontratta](../../docs/CONTRACTS.md) kalır. Yeni ortak mekanizma ancak [kabul kapısıyla](../../docs/SHARED-ADMISSION.md) çıkar.
