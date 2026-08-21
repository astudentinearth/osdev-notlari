# TürkOSDev — OSDev Notları

Türkçe işletim sistemi geliştirme ve düşük seviye programlama kaynakları.

Bu repository, işletim sistemi geliştirmeye ilgi duyan geliştiriciler için Türkçe teknik notlar, açıklamalar, örnekler ve kaynaklar içerir.

Amaç, farklı kaynaklara dağılmış bilgileri tek bir yerde toplamak ve Türkçe bir OSDev bilgi tabanı oluşturmaktır.

## İçerik

Notlar, işletim sistemi geliştirmenin temel bileşenlerinden başlayarak daha ileri konulara doğru ilerler.

* [Giriş](00-giris/) — OSDev nedir, nereden başlanır, geliştirme ortamı
* [Mimari](01-mimari/) — x86, x86-64, ARM64, RISC-V
* [Boot](02-boot/) — BIOS, UEFI, bootloader ve boot protokolleri
* [Assembly](03-assembly/) — assembly, register'lar, calling convention ve interrupt'lar
* [Kernel](04-kernel/) — kernel yapısı, GDT, IDT, exception'lar ve system call'lar
* [Memory](05-memory/) — fiziksel ve sanal bellek, paging, heap ve allocator'lar
* [Process](06-process/) — process'ler, thread'ler, context switch ve scheduling
* [Drivers](07-drivers/) — donanım, PCI, USB, depolama ve giriş aygıtları
* [Filesystems](08-filesystems/) — filesystem'ler, VFS ve filesystem implementasyonları
* [Userspace](09-userspace/) — libc, ELF, executable format'ları ve shell
* [Networking](10-networking/) — Ethernet, ARP, IPv4, UDP ve TCP
* [Graphics](11-graphics/) — framebuffer, DRM, KMS ve grafik sistemleri
* [Kaynaklar](resources/) — kitaplar, dokümantasyonlar, makaleler ve faydalı projeler

## Yaklaşım

Buradaki notlar yalnızca kavramların tanımlarını vermeyi amaçlamaz.

Mümkün olduğunda:

* kavramın neden gerekli olduğu,
* donanım seviyesinde nasıl çalıştığı,
* kernel tarafından nasıl kullanıldığı,
* gerçek bir implementasyonda nasıl ele alınabileceği

açıklanır.

Kod örnekleri, kavramı göstermek için mümkün olduğunca küçük ve anlaşılır tutulur. Bir konunun farklı mimarilerdeki davranışı değişiyorsa bu farklar ayrıca belirtilir.

İngilizce teknik terimler, aramalarda ve resmi dokümantasyonda kullanılabilmeleri için gerektiğinde korunur.

## Kaynaklar

OSDev konusunda resmi dokümantasyon ve mimari referanslar önemlidir. Notlarda kullanılan bilgiler mümkün olduğunca birincil kaynaklara dayandırılır.

Özellikle:

* CPU üreticilerinin architecture manual'ları
* donanım ve protokol spesifikasyonları
* resmi standartlar
* kernel ve toolchain dokümantasyonları
* güvenilir teknik kaynaklar

referans alınır.

Kaynakların amacı yalnızca bilgi vermek değil, okuyucunun konuyu daha derin inceleyebileceği bir başlangıç noktası sağlamaktır.

## Katkıda Bulunma

Bu repository topluluk katkılarına açıktır.

Eksik bir konu ekleyebilir, hatalı veya eksik bir açıklamayı düzeltebilir, örnek kod ekleyebilir veya mevcut bir konunun daha iyi anlatılmasına yardımcı olabilirsiniz.

Yeni bir konu eklerken:

1. Konunun mevcut içerikle nerede yer alacağını belirleyin.
2. Teknik bilgileri güvenilir kaynaklarla doğrulayın.
3. Gereksiz teori yerine anlaşılır ve teknik bir anlatım kullanın.
4. Kod örneklerini mümkün olduğunca küçük ve açıklayıcı tutun.
5. Kullanılan önemli kaynakları notun sonunda belirtin.

Her bölümün README dosyasında, o bölüm için planlanan konular ve durumları listelenir. ⬜ ile işaretli konulardan herhangi birini üstlenebilirsiniz.

Başlamadan önce okunması gereken dosyalar:

* [CONTRIBUTING.md](CONTRIBUTING.md) — katkı süreci ve içerik standartları
* [SABLON.md](SABLON.md) — notların takip ettiği yapı
* [TERIMLER.md](TERIMLER.md) — hangi terimin Türkçe, hangisinin İngilizce kullanıldığı
* [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — topluluk davranış kuralları

## Lisans

Bu repository'nin lisansı için [LICENSE](LICENSE) dosyasına bakabilirsiniz.

---

TürkOSDev topluluğu tarafından geliştirilmektedir.
