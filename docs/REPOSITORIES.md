# GitHub başlangıç planı

2026-10-25: [Qavsi](https://github.com/Qavsi) organizasyonu kullanıcı tarafından
oluşturuldu; ilk GitHub API kontrolünde boş olduğu doğrulandı. Kullanıcının
Public seçimiyle aşağıdaki iki repo oluşturuldu. Kullanıcı daha sonra MIT
lisansını, mimari/kit ayrımını ve ilk commitlerin push edilmesini onayladı.

## İlk iki depo

| Depo | Başlangıç içeriği | Sınır |
|---|---|---|
| [Qavsi/qavsi](https://github.com/Qavsi/qavsi) — Public | Mimari, ADR, kit yazarlığı, kaynak ve doğrulama kayıtları | Ortak Theme, runtime motoru veya zorunlu superbuild yok |
| [Qavsi/material3](https://github.com/Qavsi/material3) — Public | Bağımsız Qt 6.12 Theme/Style, Small Button delegate/shader/painter, örnek ve testler | Merce/Runtime bağımlılığı yok; source/installed aynı target/URI |

Yerel çalışma düzeni aynı üst klasör altında iki bağımsız clone'dur: `qavsi/`
ve `material3/`. Üst klasör Git deposu veya zorunlu superbuild değildir. İlk
araştırma dosyaları Merce çalışma alanındaki geçici `qavsi-design/` ve
`qavsi-material3/` dizinlerinden aktarılmıştır; tüketim için Merce gerekmez.
Remote adı public CMake target veya QML URI'yi değiştirmez:
`QavsiMaterial3::Style`, `Qavsi.Material3.Style` korunur.

`runtime` için kabul edilmiş ortak mekanizma yoktur. Apple, Fluent ve Gallery,
gerçek dikey dilimleri başladığında açılır. Merce'nin taşınması ayrı iştir;
mevcut Git geçmişi ve worktree değişiklikleri bu hazırlıkla değiştirilmez.

## Oluşturma ve yayın sınırı

İlk iki repo için Public görünürlük ve **MIT** lisansı kullanıcı tarafından
seçildi. Copyright: **2026 Enes Alp**. MIT bize ait kod/belgeleri kapsar;
üçüncü taraf referansların kendi lisans ve kaynak kayıtları korunur.
[THIRD_PARTY_NOTICES.md](../THIRD_PARTY_NOTICES.md), [aktarım kapsamı](PUBLICATION.md).

Repo oluşturma, dosyaları yayımlama ve stable sürüm çıkarma ayrı adımlardır.
Repolar başlangıçta boş oluşturuldu. İlk kaynak aktarımı mimari ve Material3
kitini ayrı Git geçmişleriyle yayımlar. Stable sürüm/tag veya binary paket
yayını bu ilk commitlerin kapsamı değildir.

## İlk Material3 çıkış kapısı

- Qt 6.12, kendi typed Theme ve doğrudan StyleKit Style.
- Aynı Small Button'da tema, klavye focus'u ve reduced motion.
- Metal shader; software painter; shader asset'i backing library içinde.
- Source ve taşınmış-prefix installed consumer; tüketicide ShaderTools zorunluluğu yok.
- Ölçümlerin donanım/backend/build bilgisi ve kapsam sınırlarıyla saklanması.
- Commit öncesinde QML Preview üzerinden sol ekranda çalışan örnek.

Bu kapı yalnız ilk Small Filled dilimidir; bütün Material3 kontrol ailesinin veya
stable SDK'nın tamamlandığı anlamına gelmez.
