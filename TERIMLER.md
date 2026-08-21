# Terimler Sözlüğü

Türkçe teknik içerikte en sık karşılaşılan sorun tutarsızlıktır: aynı kavram bir notta "kesme", diğerinde "interrupt", üçüncüsünde "kesinti" olarak geçer. Okuyucu bunun aynı şey olduğunu anlamak zorunda kalır.

Bu dosya, repository genelinde hangi terimin nasıl kullanılacağını belirler. Katkı yaparken buradaki kullanımlara uyun.

## Temel ilke

**Kavram Türkçe anlatılır, terim aranabilir kalır.**

Okuyucu bir notu bitirdiğinde, konuyu İngilizce dokümantasyonda ve mimari manual'larında aramaya devam edebilmelidir. Bu yüzden:

- Yerleşik ve anlamı net bir Türkçe karşılığı olan terimler Türkçe yazılır.
- Karşılığı yerleşmemiş, çevirisi anlamı bulanıklaştıran veya kısaltmalarla birlikte kullanılan terimler İngilizce bırakılır.
- Bir terim ilk geçtiğinde diğer dildeki karşılığı parantez içinde verilir: "sanal bellek (virtual memory)".

Amaç, metni İngilizce kelimelerle doldurmak değildir. Türkçesi varken İngilizcesini kullanmak da en az tersi kadar sorunludur.

## Türkçe kullanılan terimler

| İngilizce | Türkçe kullanım | Not |
| --- | --- | --- |
| memory | bellek | "hafıza" kullanılmaz |
| physical memory | fiziksel bellek | |
| virtual memory | sanal bellek | |
| address | adres | |
| address space | adres alanı | |
| memory management | bellek yönetimi | |
| page | sayfa | |
| page table | sayfa tablosu | |
| entry | girdi | "sayfa tablosu girdisi" |
| flag | bayrak | |
| cache | önbellek | |
| buffer | tampon | |
| pointer | işaretçi | |
| array | dizi | |
| queue | kuyruk | |
| linked list | bağlı liste | |
| processor / CPU | işlemci | CPU kısaltması da kullanılabilir |
| core | çekirdek | yalnızca CPU çekirdeği için |
| instruction | komut | |
| instruction set | komut seti | |
| bus | veri yolu | |
| device | aygıt | "cihaz" da kabul edilir, not içinde tutarlı olun |
| driver | sürücü | |
| hardware / software | donanım / yazılım | |
| privilege level | ayrıcalık seviyesi | |
| user mode / kernel mode | kullanıcı modu / kernel modu | |
| scheduling | zamanlama | eylem için; bkz. `scheduler` |
| layer | katman | |
| packet | paket | |
| frame (ağ) | çerçeve | yalnızca ağ çerçevesi için; bkz. `page frame` |
| header (protokol) | başlık | |
| routing | yönlendirme | |
| file / file system | dosya / dosya sistemi | |
| directory | dizin | |
| block device | blok aygıtı | |
| partition | bölüm | |
| compiler | derleyici | |
| debugging | hata ayıklama | |
| emulator | emülatör | |
| alignment | hizalama | |
| overflow | taşma | |
| lock | kilit | bkz. `spinlock` |
| performance | performans | |
| default | varsayılan | |
| version | sürüm | |
| feature | özellik | |
| implementation | implementasyon | "gerçekleme" kullanılmaz |
| specification | spesifikasyon | |
| documentation | dokümantasyon | |

## İngilizce bırakılan terimler

Bu terimler çevrilmez. Türkçe ek gerektiğinde kesme işareti kullanılır.

| Terim | Türkçe açıklaması | Neden çevrilmiyor |
| --- | --- | --- |
| kernel | işletim sisteminin çekirdek katmanı | "çekirdek" CPU çekirdeği (core) ile karışır |
| userspace | kullanıcı alanı | yerleşik karşılığı yok |
| process | çalışan program örneği | "süreç" günlük dilde çok geniş |
| thread | process içindeki yürütme akışı | "iş parçacığı" yaygın kullanılmıyor |
| scheduler | process'leri sıraya koyan bileşen | "zamanlayıcı" timer ile karışır |
| timer | zamanlama donanımı (PIT, HPET, APIC timer) | "sayaç" counter ile karışır |
| context switch | yürütme bağlamının değiştirilmesi | parçalı çevirisi anlaşılmıyor |
| interrupt | donanım veya yazılım kesmesi | IDT, IRQ, APIC ile birlikte kullanılıyor |
| exception | işlemcinin ürettiği olağan dışı durum | fault, trap, abort ile birlikte anılıyor |
| fault / trap / abort | exception alt türleri | manual'daki adlarıyla kullanılır |
| page fault | bulunmayan sayfaya erişim hatası | doğrudan aranan bir terim |
| page frame | fiziksel bellekteki sayfa boyutunda birim | "çerçeve" ağ çerçevesiyle karışır |
| paging | sayfalama mekanizması | "sayfalama" başka anlamlarda da kullanılıyor |
| stack | son giren ilk çıkar yapı, çağrı yığını | "yığın" heap için de kullanılıyor |
| heap | dinamik ayrılan bellek bölgesi | "yığın" stack ile karışır |
| allocator | bellek ayırıcı | |
| register | işlemci içi saklama birimi | "yazmaç" aramada karşılık bulmuyor |
| segment / descriptor | GDT ve segmentation kavramları | tablo adlarıyla birlikte kullanılıyor |
| offset | başlangıçtan itibaren kayma | |
| boot / bootloader | açılış süreci ve açılış yükleyicisi | |
| firmware | donanım üzerindeki yerleşik yazılım | |
| system call | userspace'in kernel'den hizmet isteme yolu | "sistem çağrısı" da kabul edilir, tercih edilen İngilizcedir |
| spinlock | bekleyerek dönen kilit | |
| deadlock | karşılıklı kilitlenme | |
| race condition | yarış durumu | |
| linker / linker script | bağlayıcı ve betiği | |
| assembler | assembly çevirici | |
| toolchain | derleme araç zinciri | |
| framebuffer | ekran belleği | |
| socket | ağ uç noktası | |
| journaling | günlükleme | |
| endianness | bayt sıralaması | |

## Ek alma kuralları

İngilizce bırakılan terimlere Türkçe ek getirilirken kesme işareti kullanılır:

- Doğru: `IDT'ye`, `register'ların`, `page fault'lar`, `kernel'in`, `stack'te`
- Yanlış: `IDTye`, `registerların`, `page faultlar`

Kısaltmalarda ek, kısaltmanın **okunuşuna** göre seçilir:

- `IDT'ye` (i-de-te), `GDT'nin` (ge-de-te), `CPU'yu` (si-pi-yu), `APIC'i` (eypik)

Sonu sessizle biten İngilizce kelimelerde ek uyumu okunuşa göre yapılır: `cache'i` (keş-i), `page'i` (peyc-i).

## Kısaltmalar

Kısaltmalar bir notta ilk geçtiği yerde açılır, sonrasında kısaltma olarak kullanılır:

> IDT (Interrupt Descriptor Table), interrupt geldiğinde çalışacak handler'ın adresini tutar. IDT'nin her girdisi 16 bayttır.

Kısaltmanın Türkçe açılımı verilmez; İngilizce açılımı verilir. Amaç okuyucunun kısaltmayı dokümantasyonda tanıyabilmesidir.

## Tartışmaya açık kararlar

Aşağıdaki tercihler bilinçli olarak yapılmıştır, ancak tartışılabilir. Değiştirilmesini öneriyorsanız issue açın; karar değişirse tüm notlar birlikte güncellenir.

- **interrupt / kesme.** "Kesme" Türkçede oldukça yerleşmiş bir karşılıktır. Buna rağmen İngilizcesi tercih edildi, çünkü IDT, IRQ, interrupt handler, interrupt vector gibi terimlerle sürekli birlikte geçiyor ve karışık kullanım metni bozuyor.
- **process / süreç.** "Süreç" günlük Türkçede çok geniş bir anlam taşıdığı için teknik metinde belirsizlik yaratıyor.
- **stack / heap.** İkisinin de yaygın çevirisi "yığın" olduğu için ikisi de İngilizce bırakıldı.

## Yeni terim ekleme

Sözlükte olmayan bir terimle karşılaşırsanız:

1. Terimi ilgili tabloya ekleyin ve kararınızın gerekçesini `Not` sütununa yazın.
2. Değişikliği notunuzla aynı pull request içinde gönderin.
3. Kararın tartışılması gerektiğini düşünüyorsanız pull request açıklamasında belirtin.

Sözlük, notlar yazıldıkça büyüyecek bir dosyadır. Eksik olması normaldir; tutarsız olması değildir.
