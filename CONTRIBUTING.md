# Katkıda Bulunma Rehberi

TürkOSDev'e katkıda bulunmak istediğiniz için teşekkürler. Bu doküman, katkıların nasıl yapılacağını ve içeriğin uyması gereken standartları açıklar.

Bu repository'nin amacı, dağınık halde bulunan işletim sistemi geliştirme bilgisini Türkçe, doğrulanabilir ve tutarlı bir bilgi tabanında toplamaktır. Bu yüzden burada içerik standartları, katkı miktarından daha önemlidir.

## İçindekiler

- [Nasıl katkıda bulunabilirim](#nasıl-katkıda-bulunabilirim)
- [Katkı süreci](#katkı-süreci)
- [Dosya ve klasör düzeni](#dosya-ve-klasör-düzeni)
- [İçerik standartları](#i̇çerik-standartları)
- [Dil ve terim kullanımı](#dil-ve-terim-kullanımı)
- [Kaynak gösterme](#kaynak-gösterme)
- [Kod örnekleri](#kod-örnekleri)
- [Commit mesajları](#commit-mesajları)
- [Pull request kontrol listesi](#pull-request-kontrol-listesi)
- [İnceleme süreci](#i̇nceleme-süreci)

## Nasıl katkıda bulunabilirim

Katkı yalnızca sıfırdan not yazmak demek değildir. Aşağıdakilerin hepsi değerlidir:

- **Yeni not yazmak.** Bölüm README'lerinde ⬜ ile işaretlenmiş konulardan birini üstlenebilirsiniz.
- **Mevcut notu düzeltmek.** Teknik hata, eksik açıklama veya yanlış bilgi bildirmek ya da doğrudan düzeltmek.
- **Anlatımı iyileştirmek.** Karışık bir bölümü yeniden yazmak, eksik bir ara adımı eklemek.
- **Kaynak eklemek.** `resources/` bölümüne güvenilir kaynak eklemek.
- **Diyagram eklemek.** Bir kavramı görselleştiren şema hazırlamak.
- **İnceleme yapmak.** Açık pull request'leri teknik açıdan gözden geçirmek.

Ne yapacağınızdan emin değilseniz `good first issue` etiketli issue'lara bakabilirsiniz.

## Katkı süreci

1. **Önce issue açın.** Yeni bir not yazacaksanız veya kapsamlı bir değişiklik yapacaksanız, çalışmaya başlamadan önce issue açın. Bu, aynı konuyu iki kişinin yazmasını önler ve kapsamın önceden konuşulmasını sağlar. Yazım hatası gibi küçük düzeltmeler için doğrudan pull request açabilirsiniz.
2. **Konuyu üstlenin.** İlgili issue'da çalışmaya başladığınızı belirtin. Bölüm README'sindeki durumu 🟡 yapan değişikliği de pull request'inize dahil edebilirsiniz.
3. **Fork alın ve dal oluşturun.** Dal adı için `konu/kisa-aciklama` veya `duzeltme/kisa-aciklama` biçimini kullanın.
4. **Yazın.** [SABLON.md](SABLON.md) dosyasındaki yapıyı takip edin.
5. **Kontrol listesini gözden geçirin.** Aşağıdaki [pull request kontrol listesine](#pull-request-kontrol-listesi) bakın.
6. **Pull request açın.** Hangi issue'yu kapattığını belirtin.

Bir konuyu üstlenip devam edemezseniz bunu issue'da belirtmeniz yeterlidir; kimse için sorun değildir. Üstlenilip uzun süre ilerlemeyen konular yeniden açığa alınır.

## Dosya ve klasör düzeni

Notlar konuya göre numaralı bölüm klasörlerinde tutulur. Bir notun hangi bölüme gireceğinden emin değilseniz issue'da sorun.

Dosya adlandırma kuralları:

- Yalnızca küçük harf, rakam ve tire kullanın: `sayfa-tablosu-girdileri.md`
- Türkçe karakter kullanmayın: `ayricalik-seviyeleri.md` (`ayrıcalık` değil)
- Boşluk ve alt çizgi kullanmayın.
- Dosya adı, bölüm README'sinde planlanan adla aynı olmalıdır.
- Bir notun görselleri, aynı bölümdeki `gorseller/` klasörüne konur ve nota özgü bir ön ek alır: `gorseller/paging-4-seviye.svg`

Yeni bir not eklediğinizde, bölüm README'sindeki tabloda o satırın durumunu ✅ yapın ve dosya adını bağlantıya dönüştürün.

## İçerik standartları

Bu repository'deki notlar tanım listesi değildir. Her not, mümkün olduğunda şu dört soruyu yanıtlamalıdır:

1. **Bu kavram neden var?** Hangi problemi çözüyor, olmasaydı ne olurdu?
2. **Donanım seviyesinde ne oluyor?** CPU, MMU, veri yolu veya aygıt tarafında somut olarak ne gerçekleşiyor?
3. **Kernel bunu nasıl kullanıyor?** Kernel'in sorumluluğu nerede başlıyor, hangi yapıları tutuyor?
4. **Gerçek bir implementasyonda nasıl ele alınır?** Hangi kararlar verilir, hangi tuzaklar vardır?

Bunun dışında:

- **Mimariye özgü davranışları ayırın.** Bir konu x86-64, ARM64 ve RISC-V'de farklı davranıyorsa bu farkı açıkça belirtin. Yalnızca tek bir mimariyi anlatıyorsanız, notun başında bunu yazın.
- **Doğrulanmamış bilgi yazmayın.** Emin değilseniz ya kaynağa bakın ya da o kısmı yazmayın. "Sanırım", "galiba" içeren cümleler yerine bilinen kısmı yazıp bilinmeyeni açıkça belirtin.
- **Eski ve güncel ayrımını yapın.** Bir yöntem artık kullanılmıyorsa (örneğin x86-64'te segmentation'ın büyük ölçüde devre dışı kalması) bunu belirtin. Tarihsel bilgiyi silmek yerine bağlamlandırın.
- **Ön koşulları belirtin.** Not başka bir notu gerektiriyorsa başında bağlantı verin.
- **Kısaltmaları ilk geçtiği yerde açın:** "IDT (Interrupt Descriptor Table)" gibi.
- **Kopyala-yapıştır yapmayın.** Başka bir kaynaktan çeviri yapıyorsanız bunu belirtin ve kaynağın lisansına uyun. Telif hakkı ihlali içeren katkılar kabul edilmez.

## Dil ve terim kullanımı

Türkçe teknik içerikte en sık karşılaşılan sorun, aynı kavramın farklı yerlerde farklı adlarla anılmasıdır. Bu repository'de terim kullanımı [TERIMLER.md](TERIMLER.md) dosyasındaki sözlüğe bağlıdır.

Temel ilke: **kavram Türkçe anlatılır, terim aranabilir kalır.** Okuyucu notu okuduktan sonra konuyu İngilizce dokümantasyonda aramaya devam edebilmelidir.

- Yerleşik bir Türkçe karşılığı olan terimleri Türkçe kullanın.
- Yerleşik karşılığı olmayan veya çevirisi anlamı bulanıklaştıran terimleri İngilizce bırakın: `page fault`, `context switch`, `bootloader`.
- İngilizce bırakılan terimlere Türkçe ek getirirken kesme işareti kullanın: `IDT'ye`, `page fault'lar`, `register'ların`.
- Bir terimi ilk kullandığınızda parantez içinde diğer dildeki karşılığını verin.
- Sözlükte olmayan bir terimle karşılaşırsanız, notunuzla birlikte [TERIMLER.md](TERIMLER.md) dosyasına da ekleyin.

Yazım hakkında:

- İkinci tekil şahıs ("yapıyorsun") yerine tarafsız anlatım ("yapılır", "yapabilirsiniz") tercih edin.
- Cümleleri kısa tutun. Uzun bir cümleyi ikiye bölmek neredeyse her zaman daha iyidir.
- Gereksiz ilerlemeci ifadelerden kaçının: "gördüğünüz gibi", "kolayca", "basitçe". Bir şeyin kolay olduğunu söylemek, zorlanan okuyucuya yardımcı olmaz.

## Kaynak gösterme

Her notun sonunda **Kaynaklar** başlığı bulunmalıdır.

- Öncelik sırası: mimari manual'ları ve resmi spesifikasyonlar → kernel ve toolchain dokümantasyonu → güvenilir teknik kaynaklar → topluluk wiki'leri.
- Bir manual'a referans verirken cilt, bölüm ve mümkünse sürüm bilgisini yazın: *Intel SDM, Cilt 3A, Bölüm 6.14*
- Bağlantı verirken erişim tarihini ekleyin.
- Yapay zekâ çıktısını kaynak olarak göstermeyin. Bir aracı taslak için kullanmanız sorun değildir, ancak yazdığınız her teknik iddiadan siz sorumlusunuz ve iddia birincil kaynaktan doğrulanmalıdır.

## Kod örnekleri

Şu an bu repository'de derlenebilir örnek proje bulunmuyor. Notların içinde, kavramı göstermek amacıyla kod parçaları kullanılabilir.

- Kod parçalarını kısa tutun. Amaç çalışan bir sistem vermek değil, kavramı göstermektir.
- Dil belirtilmiş kod bloğu kullanın (`c`, `asm`, `ld`, `sh`).
- Assembly örneklerinde hangi söz diziminin (NASM / GAS) kullanıldığını belirtin.
- Örneği kısaltmak için atlanan yerleri açıkça gösterin.
- Yorumları Türkçe yazın.
- Eksik hata kontrolü veya sabit varsayım gibi basitleştirmeler yaptıysanız kod bloğunun altında belirtin.

## Commit mesajları

Commit mesajları kısa ve açıklayıcı olmalıdır. Ön ek İngilizce, açıklama Türkçe yazılır:

```text
docs: 04-kernel/gdt.md notu
fix: paging notunda yanlış bayrak açıklaması
docs: resources/kitaplar.md listesine iki kitap
style: 02-boot README tablosunun biçimi
chore: issue şablonu güncellendi
ci: bağlantı denetimi iş akışı
```

Kullanılan ön ekler:

| Ön ek | Ne zaman |
| --- | --- |
| `docs` | not ekleme, düzenleme, kaynak ekleme |
| `fix` | teknik hata veya yanlış bilgi düzeltmesi |
| `style` | yalnızca biçim değişikliği, içerik aynı |
| `chore` | repository düzeni, şablonlar, yapılandırma |
| `ci` | iş akışı ve denetim ayarları |

Bir commit tek bir işi yapmalıdır. Not eklerken bölüm README'sini güncellemek aynı commit'e girebilir.

## Pull request kontrol listesi

Pull request açmadan önce:

- [ ] Not [SABLON.md](SABLON.md) yapısına uyuyor.
- [ ] Sonunda **Kaynaklar** başlığı var ve kaynaklar birincil.
- [ ] Terimler [TERIMLER.md](TERIMLER.md) ile tutarlı; yeni terimler sözlüğe eklendi.
- [ ] Dosya adı kurallara uygun (küçük harf, tire, Türkçe karakter yok).
- [ ] Bölüm README'sindeki tablo güncellendi (durum ✅, dosya adı bağlantı yapıldı).
- [ ] Tüm bağlantılar çalışıyor.
- [ ] Kod bloklarında dil belirtilmiş.
- [ ] Yazım denetimi yapıldı.

## İnceleme süreci

Pull request'ler teknik doğruluk ve anlatım açısından incelenir. İnceleme sırasında kaynak istenebilir veya belirli bir iddianın manual'daki karşılığı sorulabilir. Bu, katkının kalitesiz olduğu anlamına gelmez; ortak standardın parçasıdır.

Değişiklik istenen bir pull request kapatılmaz, düzeltme beklenir. Uzun süre yanıt gelmezse pull request kapatılabilir; hazır olduğunuzda yeniden açabilirsiniz.

## Sorular

Emin olamadığınız her durumda issue açıp sormak, tahmin edip yazmaktan iyidir.
