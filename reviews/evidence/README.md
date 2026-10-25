# Tarihsel kanıt kataloğu

Bu public depo mimari/review metinlerini ve kaynak URL/hash manifestlerini
yayımlar. Ham loglar, ekran görüntüleri, çalıştırılabilir proof kaynakları ve
yerel/upstream SDK kopyaları yayımlanmaz; orijinal araştırma arşivinde korunur.
Bu sayfaya yönlenen eski bağlantılar ham dosya indirme bağlantısı değildir.

[Public aktarım kapsamı](../../docs/PUBLICATION.md),
[referans otoriteleri](../../references/SOURCES.md),
[Material3'ün güncel doğrulaması](https://github.com/Qavsi/material3/blob/main/evidence/VERIFICATION.md).

<a id="review-01"></a>
## 01 — Material3 kaynak karşılaştırması

Numeric/alias ve generated recipe karşılaştırmaları tarihsel raporda özetlenir.
[Google kaynak URL/hash kayıtları](01/official-sources.json) korunur.
Google MDC/Compose ve Qt SDK kaynak kopyaları, diagnostic script/çıktıları
ve lint logları bu repoda yoktur.

<a id="review-02"></a>
## 02 — Apple public API incelemesi

Compile-only probe, availability guard ve weak symbol incelemesi raporda
özetlenir. Native runtime/UI kabulü değildir. Yerel AppKit header kopyaları,
probe kaynağı, build/deployment logları ve provenance arşivi dağıtılmaz.

<a id="review-03"></a>
## 03 — Fluent kaynak karşılaştırması

Web/Windows otoriteleri ve kaynak/renk değerlendirmesi raporda tutulur.
Python diagnostic, ham sonuçlar ve kaynak cache'i yerel arşivdedir;
Windows/native materyal desteği iddiası yoktur.

<a id="review-04"></a>
## 04 — Standart ortak mekanizma kabulü

Qt-only temel API kontrollerinin sonuçları raporda özetlenir. Ham Cocoa/offscreen
sonuçları, C++/CMake proof'u ve eski Merce kaynak inceleme snapshot'ları
bu public depoda dağıtılmaz.

<a id="review-05"></a>
## 05 — StyleKit scope ve variation mekanizması

Aynı instance, iki engine scope'u ve dinamik variation testleri tarihsel
raporda özetlenir. Raw loglar, snapshot görüntüleri, prototype ve SHA envanteri
yerel arşivdedir. Mevcut Qt hatasının çözüldüğü iddia edilmez.

<a id="review-06"></a>
## 06 — Source/installed paketleme proof'u

İlk shared/static/moved-prefix proof'unun bulguları tarihsel raporda özetlenir.
Eski command/build/run/lint logları, prototype, CMake metadata kopyaları ve
makine ortam kayıtları bu repoda dağıtılmaz. Güncel bağımsız Material3 paketi
kendi reposunda ayrıca doğrulanır.

<a id="historical-inputs"></a>
## Tarihsel kaynak metinleri

Dört designmd.app girdisinin özgün URL/download/hash kayıtları
[history manifestinde](../../references/history/2026-10-24-sources.json) korunur.
Kaynak metinleri bu repoda yeniden dağıtılmaz; güncel kit otoriteleri değildir.

<a id="local-source-audit"></a>
## Eski yerel kaynak atıfları

Merce, Qt SDK ve macOS SDK dosya/satır atıfları ilk yerel araştırma snapshot'ına
aittir. Bunlar bu repoda yayınlanmış kaynak veya build dependency'si değildir.
Kişisel dizin önekleri public metinde `MERCE_REPOSITORY`, `MERCE_SOURCE`,
`QT_SDK` ve `MACOS_SDK` olarak gösterilir.

<a id="docs-audit"></a>
## İlk belge kontrol kaydı

İlk belge/link/hash kontrolünün ham JSON çıktısı yerel arşivdedir. Public aktarımın
bağlantı kontrolü ayrıca yapılır; ham dosyanın burada bulunduğu iddia edilmez.
