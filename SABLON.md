# Not Şablonu

Bu repository'deki her not aynı yapıyı takip eder. Ortak yapı, okuyucunun aradığı bilgiyi her notta aynı yerde bulmasını sağlar ve yazarken neyin eksik kaldığını görünür kılar.

Yeni bir not yazarken aşağıdaki şablonu kopyalayın.

## Bölümlerin anlamı

**Başlık ve giriş.** Notun ilk paragrafı, konunun ne olduğunu ve neden var olduğunu tek başına anlatmalıdır. Okuyucu yalnızca bu paragrafı okusa bile kavramın ne işe yaradığını bilmelidir. "Bu notta X'i inceleyeceğiz" gibi cümlelerle başlamayın.

**Ön koşullar.** Notun anlaşılması başka notlara bağlıysa buraya bağlantı verin. Bağımlılık yoksa bu bölümü tamamen çıkarın.

**Kapsam.** Notun hangi mimariyi, hangi modu veya hangi sürümü anlattığını belirtir. Konu mimariden bağımsızsa bu bölüm çıkarılabilir.

**Problem.** Bu kavram olmasaydı ne olurdu? Hangi ihtiyaçtan doğdu? Bu bölüm çoğu notta en değerli kısımdır ve en sık atlanan kısımdır. Bir mekanizmayı, çözdüğü problemi bilmeden öğrenmek ezber olur.

**Donanım seviyesinde.** CPU, MMU, veri yolu veya aygıt tarafında somut olarak ne gerçekleşiyor? Hangi register'lar, hangi yapılar, hangi sinyaller devrede? Bu bölümdeki iddialar mimari manual'ından doğrulanmalıdır.

**Kernel tarafında.** Kernel'in sorumluluğu nerede başlıyor? Hangi veri yapılarını tutuyor, ne zaman güncelliyor, hangi durumları ele almak zorunda?

**Implementasyon notları.** Gerçek bir kernel yazarken verilecek kararlar, karşılaşılacak tuzaklar, sık yapılan hatalar. Alternatif yaklaşımlar varsa aralarındaki ödünleşim.

**Mimariye göre farklar.** Aynı kavram x86-64, ARM64 ve RISC-V'de farklı ele alınıyorsa fark burada özetlenir. Yalnızca tek mimari anlatılıyorsa bu bölüm çıkarılır.

**Sık yapılan hatalar.** Bu konuda yeni başlayanların düştüğü tuzaklar. Kısa maddeler halinde.

**İlgili notlar.** Konuyla bağlantılı diğer notlara bağlantılar.

**Kaynaklar.** Zorunlu bölüm. Kullanılan birincil kaynaklar, cilt ve bölüm bilgisiyle.

## Şablon

Aşağıdaki bloğu kopyalayıp doldurun. Kullanmadığınız isteğe bağlı bölümleri silin; boş başlık bırakmayın.

````markdown
# Konu Adı

Konunun ne olduğunu ve neden var olduğunu anlatan bir veya iki paragraf.

## Ön koşullar

- [Diğer notun adı](../XX-bolum/dosya.md)

## Kapsam

Bu not x86-64 mimarisini ve long mode'u esas alır.

## Problem

Bu mekanizma olmasaydı hangi sorun ortaya çıkardı?

## Donanım seviyesinde

Donanımın bu konuda ne yaptığı, hangi yapıları ve register'ları kullandığı.

## Kernel tarafında

Kernel'in sorumluluğu, tuttuğu veri yapıları ve ele almak zorunda olduğu durumlar.

## Implementasyon notları

Gerçek bir implementasyonda verilecek kararlar ve ödünleşimler.

## Mimariye göre farklar

| Mimari | Davranış |
| --- | --- |
| x86-64 | |
| ARM64 | |
| RISC-V | |

## Sık yapılan hatalar

- ...

## İlgili notlar

- [İlgili not](../XX-bolum/dosya.md)

## Kaynaklar

- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm X.Y
- ...
````

## Kısa notlar için

Bazı konular yukarıdaki yapının tamamını gerektirmez. Terim açıklaması veya araç kullanımı gibi kısa notlarda şu yapı yeterlidir:

````markdown
# Konu Adı

Giriş paragrafı.

## Nasıl kullanılır

...

## Sık yapılan hatalar

- ...

## Kaynaklar

- ...
````

Hangi yapının uygun olduğundan emin değilseniz uzun şablonla başlayın ve gereksiz bölümleri silin.

## Biçim kuralları

- Her notta yalnızca bir adet birinci seviye başlık (`#`) bulunur.
- Başlıklarda numaralandırma kullanılmaz; sıralama başlıkların kendi düzeninden anlaşılır.
- Tablolar dar tutulur; uzun açıklamalar tablo yerine metne yazılır.
- Görseller `gorseller/` klasörüne konur ve alternatif metin (`alt`) yazılır.
- Notun sonunda **Kaynaklar** dışında bir bölüm bulunmaz.
