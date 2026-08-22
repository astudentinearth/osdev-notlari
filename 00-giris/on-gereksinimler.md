# Ön gereksinimler

OSDev'e başlamak için gereken bilgi tek bir dile veya tek bir konuya sığmaz. Kernel yazan kişi aynı anda üç düzlemde çalışır: kodu yazdığı dil (çoğunlukla C ve assembly), kodun üzerinde çalıştığı donanım modeli ve kaynağı çalıştırılabilir bir imaja dönüştüren araç zinciri. Bu üç düzlemden birindeki boşluk, çoğu zaman diğer ikisinde anlaşılmayan bir hata olarak görünür.

Bu not, o üç düzlemde nelerin gerçekten gerekli olduğunu ayırır. Amaç bir okuma listesi vermek değil, hangi konunun hangi somut probleme karşılık geldiğini göstermektir: `volatile` bilmemek sürücü yazarken, struct hizalaması bilmemek tanımlayıcı tablo kurarken, çağrı kuralını bilmemek ilk interrupt handler'da karşınıza çıkar.

## Ön koşullar

- [OSDev nedir](osdev-nedir.md)

## Ne kadarı yeterli

Ön gereksinimlerin tamamını başlamadan önce öğrenmek gerekmez; böyle bir sıra pratikte işlemez de. Gereken şey uzmanlık değil bir eşiktir: bir mimari manual'ının ilgili bölümünü okuyup anlayabilmek ve kendi derleyicinizin ürettiği çıktıyı okuyabilmek. Bu iki yetenek elinizdeyse eksik kalan bilgiyi ihtiyaç anında tamamlayabilirsiniz. Bunlar yoksa, karşılaşılan her sorun kaynağı belirsiz bir tıkanmaya dönüşür.

Aşağıdaki bölümler konuları bu eşiğe göre sıralar. Her başlığın sonunda o bilginin ilk hangi aşamada gerekeceği belirtilmiştir; aşamaların tanımı [yol haritası](yol-haritasi.md) notundadır.

## C bilgisi

Kernel'lerin büyük kısmı C ile yazılır. Kernel'de yazılan C, uygulama geliştirmede yazılan C'den iki noktada ayrılır: altta standart kütüphane yoktur ve yazılan kodun belleğe nasıl yerleştiği doğrudan önemlidir.

**İşaretçiler ve bellek düzeni.** Bir adresi işaretçiye çevirmek, o adresteki baytları belirli bir struct gibi okumak ve işaretçi aritmetiğinin tip boyutuna göre ölçeklendiğini bilmek temel gereksinimdir. Tamsayı ile işaretçi arasındaki dönüşümlerde `uintptr_t` kullanılır; `int` bir adresi tutmak için yeterli değildir.

**Sabit genişlikli tipler.** `int` ve `long` boyutları platforma ve veri modeline göre değişir. Donanımla konuşan her yapıda `stdint.h` içindeki `uint8_t`, `uint32_t`, `uint64_t` gibi tipler kullanılır. Bir sayfa tablosu girdisinin 64 bit olduğu bilgisi tipin adında görünmelidir.

**Bit işlemleri.** Bayrak okuma, maskeleme ve kaydırma kernel kodunun günlük işidir. Kaydırmalarda işaretsiz sabit kullanmak gerekir: 32 bitlik bir `int` üzerinde `1 << 31` tanımsız davranıştır, doğrusu `1u << 31` veya 64 bit için `UINT64_C(1) << 63` biçimidir. Hizalama hesapları da bu gruba girer; bir adresi `n` sınırına yukarı hizalamak `(a + n - 1) & ~(n - 1)` ifadesiyle yapılır ve `n` ikinin kuvveti olmak zorundadır.

**struct düzeni, hizalama ve dolgu.** Derleyici, struct alanlarının arasına hizalama için dolgu (padding) ekler. Donanımın beklediği bir yapıyı (tanımlayıcı tablo girdisi, boot protokolü yapısı, aygıt register bloğu) tanımlarken bu dolgu yapıyı bozar. `__attribute__((packed))` dolguyu kaldırır, ancak bu kez alanlar hizasız kalır: x86-64'te hizasız erişim çalışır fakat yavaştır, bazı mimarilerde ve aygıt belleğinde hata üretir. Daha sağlam yaklaşım, yapıyı donanımın istediği hizaya kendiliğinden oturacak biçimde tasarlamak ve boyutunu derleme zamanında `_Static_assert` ile doğrulamaktır.

**`volatile` ve donanım erişimi.** Derleyici, sonucu kullanılmayan bir okumayı kaldırabilir veya art arda gelen erişimleri birleştirebilir. Aygıt register'ında erişimin kendisi bir yan etkidir; kaldırılamaz.

```c
// Aygıt register'ı sabit bir adrese eşlenmiştir.
// volatile olmadan derleyici bu okumayı döngünün dışına taşıyabilir.
static volatile uint32_t *const durum = (volatile uint32_t *)0xFEE00020;

while ((*durum & (1u << 12)) != 0) {
    // aygıt hâlâ meşgul
}
```

`volatile` yalnızca derleyicinin bu erişimi kaldırmasını, birleştirmesini veya diğer `volatile` erişimlere göre yerini değiştirmesini engeller. İşlemci ve veri yolu tarafındaki sıralama için ayrıca bellek bariyeri gerekir; `volatile` atomiklik de sağlamaz.

**Tanımsız davranış.** Uygulama kodunda "pratikte çalışan" tanımsız davranış, kernel'de optimizasyon seviyesi değiştiğinde kaybolan bir hataya dönüşür. En sık karşılaşılanlar: tip kuralını çiğneyen işaretçi dönüşümleri (strict aliasing), işaretli tamsayı taşması ve null işaretçi varsayımları. Kernel'de adres 0 map edilmiş olabildiği için derleyicinin "buraya erişilemez" çıkarımı da yanlış sonuç verir. Bu yüzden kernel derlemelerinde `-fno-strict-aliasing` ve `-fno-delete-null-pointer-checks` yaygın olarak kullanılır.

**Freestanding C.** Standart iki uyum düzeyi tanımlar: hosted ve freestanding. Freestanding ortamda yalnızca kütüphane işlevi içermeyen başlıklar bulunur: `stddef.h`, `stdint.h`, `stdbool.h`, `limits.h`, `stdarg.h`, `float.h`, `stdalign.h`, `stdnoreturn.h` ve `iso646.h`. `printf` ve `malloc` yoktur, ihtiyaç duyulduğunda yazılır. Buna karşılık derleyici, kaynakta çağrı geçmese bile `memcpy`, `memset`, `memmove` ve `memcmp` çağrıları üretebilir; bu dört işlevi kernel'in sağlaması gerekir.

**Bağlantı (linkage) ve bağlayıcı sembolleri.** `static` ve `extern` anahtar sözcüklerinin sembol görünürlüğüne etkisi, kodu belirli bir bölüme yerleştiren `section` özniteliği ve bağlayıcı betiğinde tanımlanan sembollerin C tarafından nasıl okunacağı bilinmelidir. Bağlayıcı sembolünün değeri değil adresi anlamlıdır; bu yüzden dizi tipiyle tanımlanır:

```c
// Bağlayıcı betiğinde tanımlanmış sembol.
extern char _kernel_sonu[];

uintptr_t ilk_bos_adres = (uintptr_t)_kernel_sonu;
```

**Inline assembly.** GCC ve Clang'ın genişletilmiş `asm` söz dizimi; çıkış ve giriş kısıtları, bozulan register'ların (clobber) bildirilmesi ve `"memory"` clobber'ının anlamı. Port I/O, `cpuid`, control register erişimi ve bariyerler bu yolla yazılır.

Bu başlıkların çoğu Aşama 1'den itibaren gerekir; `volatile` ve inline assembly ilk sürücüde, bağlayıcı sembolleri ilk bellek yöneticisinde karşınıza çıkar.

## Assembly bilgisi

Assembly'yi baştan sona yazabilmek gerekmez. Gereken şey okuyabilmek ve gerektiğinde kısa parçalar yazabilmektir. Kernel'in assembly ile yazılması zorunlu olan kısımları sınırlıdır ama atlanamaz: boot girişi, interrupt giriş kodu, context switch, tanımlayıcı tablo yükleme (`lgdt`, `lidt`), adres alanı değiştirme (`mov cr3`) ve kullanıcı moduna geçiş (`iretq`, `sysretq`).

**İşlemci modeli.** x86-64'te 16 genel amaçlı register, `rip`, `rflags`, control register'lar (`cr0`, `cr2`, `cr3`, `cr4`) ve MSR'ler (Model Specific Register). Bu register'ların hangisinin ne işe yaradığını bilmek, disassembly okumanın ön şartıdır.

**Söz dizimi farkları.** GAS varsayılan olarak AT&T söz dizimini, NASM Intel söz dizimini kullanır. İkisinde işlenen sırası terstir. Hangi söz diziminde çalıştığınızı bilmek ve okuduğunuz örneğin hangisiyle yazıldığını ayırt etmek gerekir.

**Çağrı kuralı.** C ile assembly'nin birbirini çağırabilmesi bir ABI (Application Binary Interface) sözleşmesine dayanır. x86-64 üzerinde System V ABI'de tamsayı argümanları sırasıyla `rdi`, `rsi`, `rdx`, `rcx`, `r8` ve `r9` register'larından geçer, dönüş değeri `rax`'tedir. `rbx`, `rbp` ve `r12`–`r15` çağrılan tarafından korunur. `call` anında yığın 16 bayta hizalı olmalıdır.

Aynı ABI'nin kernel'i doğrudan ilgilendiren bir ayrıntısı daha vardır: `rsp`'nin altındaki 128 baytlık red zone. Yaprak fonksiyonlar bu alanı yığın işaretçisini değiştirmeden kullanabilir. Bir interrupt geldiğinde işlemci ve handler bu alanın üzerine yazar. Bu yüzden kernel kodu `-mno-red-zone` ile derlenir.

```asm
; NASM söz dizimi. Interrupt girişinde işlemci yalnızca bir kısım bilgiyi
; yığına iter; kalan register'ları saklamak handler'ın işidir.
isr_ortak:
    push rax
    push rcx
    ; ... diğer register'ların itilmesi atlandı
    mov  rdi, rsp          ; System V ABI: ilk argüman rdi
    call interrupt_isle
    ; ... register'ların geri yüklenmesi atlandı
    iretq
```

Örnek yalnızca argüman geçişini göstermek için kısaltılmıştır. Eksiksiz bir giriş kodunun tüm genel amaçlı register'ları saklaması, hata kodu iten ve itmeyen vektörleri ayırması ve segment register'larını ele alması gerekir.

**System call kuralı.** `syscall` komutu `rcx` ve `r11` register'larını kendisi kullandığı için Linux'un x86-64 system call arayüzünde dördüncü argüman `rcx` yerine `r10`'dan geçirilir. Bu, çağrı kuralının donanım tarafından şekillendirildiği tipik bir örnektir.

Assembly Aşama 1'den itibaren gerekir ve Aşama 4'te, context switch yazılırken zorunlu hale gelir.

## Bilgisayar mimarisi bilgisi

Kernel donanımın davranışını varsayamaz; ölçüt her zaman mimari manual'ıdır. Aşağıdaki kavramların en azından tanımını bilerek başlamak gerekir, ayrıntıları ilgili bölümlerde ele alınır.

**Ayrıcalık seviyeleri ve mod geçişleri.** Kullanıcı modu ile kernel modu arasındaki ayrımın donanım tarafından zorlandığı, geçişin yalnızca belirli kapılardan (interrupt, exception, system call komutu) yapılabildiği.

**Bellek hiyerarşisi.** Register, önbellek, ana bellek ve disk arasındaki erişim maliyeti farkı. Önbellek satırı kavramı ve hizalamanın performansla ilişkisi.

**Sanal bellek ve MMU.** Sanal adresin fiziksel adrese çevrildiği, çevirinin sayfa tablolarıyla tanımlandığı ve TLB'nin (Translation Lookaside Buffer) bu çevirileri önbelleklediği. Sayfa tablosu değiştiğinde TLB'nin geçersiz kılınması gerektiği.

**Interrupt mekanizması.** Aygıtın sinyal ürettiği, işlemcinin çalışan kodu keserek bir tablodaki adrese daldığı ve dönüşte önceki bağlamı geri yüklediği.

**G/Ç yöntemleri.** x86'da ayrı bir G/Ç adres alanına erişen port I/O (`in`, `out`) ile aygıt register'larının bellek adreslerine eşlendiği MMIO (Memory-Mapped I/O) ayrımı. ARM64 ve RISC-V'de yalnızca MMIO vardır.

**Bayt sıralaması ve hizalama.** x86-64 küçük endian çalışır; ARM64 yapılandırılabilir olsa da pratikte küçük endian kullanılır. Aygıt ve dosya sistemi formatlarında sıralamanın açıkça belirtilmesi gerekir.

**Atomiklik ve bellek sıralaması.** Çok çekirdekli bir sistemde belleğe yapılan erişimlerin programda yazıldığı sırayla görünmeyebileceği. x86-64 görece güçlü bir sıralama modeli sunar, ARM64 zayıf sıralamalıdır ve bariyer kullanımı zorunludur. Bu konu Aşama 4'e kadar ertelenebilir, ancak ilk spinlock yazıldığı an gerekli hale gelir.

**Sayı sistemleri.** Onaltılık gösterim, ikinin tümleyeni ve bit alanlarının okunması. Manual'lardaki tabloların tamamı bu gösterimle yazılmıştır.

## Araç zinciri ve bağlayıcı bilgisi

Kernel geliştirmede yaşanan ilk sorunların önemli bir kısmı kernel kodunda değil, derleme ve bağlama aşamasında çıkar. Bu yüzden araç zincirinin ne yaptığını bilmek ön gereksinimdir.

- **Derleme aşamaları.** Ön işleme, derleme, assembly ve bağlama adımlarının ayrı ayrı ne ürettiği.
- **Nesne dosyası ve semboller.** Sembol tablosu, tanımlı ve tanımsız semboller, yer değiştirme (relocation) kayıtları.
- **ELF formatı.** Bölümler (`.text`, `.rodata`, `.data`, `.bss`), program başlıkları ve giriş noktası. `.bss` bölümünün dosyada yer kaplamadığı, yalnızca boyutuyla tanımlandığı.
- **Bağlayıcı betiği.** Bölümlerin bellekte nereye yerleşeceğini belirleyen `SECTIONS` bloğu, konum sayacı ve yükleme adresi (LMA) ile çalışma adresi (VMA) ayrımı. Higher-half kernel'lerde bu ayrım doğrudan işlevseldir.
- **Make.** Hedef, bağımlılık ve kural üçlüsü. Ayrıntı `build-sistemi.md` notundadır.
- **İnceleme araçları.** `objdump -d`, `readelf -S`, `nm` ve `addr2line`. Bir çökme adresini kaynak satırına geri götürebilmek, hata ayıklamanın temel becerisidir.

Bu grup Aşama 0'da gerekir; ayrıntısı `cross-compiler.md` ve `build-sistemi.md` notlarındadır.

## İşletim sistemi kavramları

Kernel yazmadan önce işletim sistemi teorisini bilmek zorunlu değildir, ancak kavramların adını bilmek okunan kaynakları anlamlı kılar: process ve thread ayrımı, zamanlama, eşzamanlılık ve yarış durumu, karşılıklı dışlama ve deadlock, sanal bellek, dosya sistemi modeli. Bu kavramların tamamı *Operating Systems: Three Easy Pieces* kitabında ücretsiz olarak bulunabilir ve bu notlarla paralel okunabilir.

Teori ile implementasyon arasındaki fark küçümsenmemelidir. Kitapta bir paragrafla anlatılan context switch, gerçekte doğru sırayla yazılması gereken bir assembly rutinidir.

## Gerekmeyenler

Alanın zorluğu hakkındaki yaygın kanılar, gereğinden uzun bir hazırlık aşamasına yol açar. Aşağıdakiler ön gereksinim değildir:

- **C uzmanı olmak.** Yukarıdaki başlıklar C'nin tamamını değil, kernel'de kullanılan alt kümesini kapsar.
- **Assembly'yi baştan yazabilmek.** Okumak ve kısa rutinler yazabilmek yeterlidir.
- **Elektronik veya donanım tasarımı bilmek.** Bir aygıtla konuşmak için gereken bilgi, aygıtın veri sayfasında yazılıdır.
- **C kullanmak zorunda olmak.** Rust (`no_std`), Zig ve C++'ın freestanding alt kümesiyle de kernel yazılabilir. Bu durumda dilin kendi kısıtları (Rust'ta `core`, panic handler ve `unsafe` sınırları) ek bir öğrenme yükü getirir; buna karşılık bulunabilecek örneklerin çoğu C ile yazılmıştır.
- **İleri matematik.** Alan ağırlıklı olarak durum yönetimi ve dikkatli okuma gerektirir.
- **Belirli bir ders veya derece.** Gereken bilginin tamamı kamuya açık kaynaklarda mevcuttur.

## Kendinizi sınamak için

Aşağıdaki soruları yanıtlayabiliyorsanız eşiği geçmişsiniz demektir. Yanıtlayamadıklarınız, önce hangi konuya bakmanız gerektiğini gösterir.

- İki `uint32_t` ve bir `uint8_t` alanı olan bir struct'ın boyutu neden 9 değil 12 bayttır?
- `volatile uint32_t *const p` ile `uint32_t *volatile p` arasındaki fark nedir?
- Bir adresi 4096 sınırına yukarı hizalayan ifadeyi bit işlemleriyle yazabiliyor musunuz?
- `objdump -d` çıktısında bir fonksiyonun ilk argümanının hangi register'dan geldiğini bulabiliyor musunuz?
- 64 bitlik bir sayfa tablosu girdisinde 12. bitin değerini okuyan ifadeyi yazabiliyor musunuz?
- `readelf -S` çıktısında `.bss` bölümü neden dosyada yer kaplamaz?
- Derleyicinin kendiliğinden ürettiği bir `memset` çağrısı, standart kütüphane olmayan bir ortamda neden bağlama hatası verir?

## Sık yapılan hatalar

- Her şeyi önceden öğrenmeye çalışmak. Eksik bilgi, ihtiyaç anında en hızlı öğrenilen bilgidir.
- Standart kütüphane alışkanlıklarını kernel'e taşımak. `printf` ve `malloc`, kullanılmadan önce yazılması gereken işlevlerdir.
- Tanımsız davranışı "çalışıyor" diye görmezden gelmek. Optimizasyon seviyesi veya derleyici sürümü değiştiğinde davranış da değişir.
- `int` ve `long` boyutlarını varsaymak. Donanım yapılarında sabit genişlikli tipler kullanılır.
- Derleyicinin ürettiği koda hiç bakmamak. Disassembly okumak çoğu zaman kaynağa bakmaktan daha hızlı sonuç verir.
- Assembly'yi tamamen atlamak. Kernel'in assembly ile yazılması zorunlu kısmı küçüktür, ancak atlanamaz.
- Yalnızca öğretici takip edip manual'a hiç bakmamak. Öğretici bir yolu gösterir, manual ölçütü tanımlar.

## İlgili notlar

- [OSDev nedir](osdev-nedir.md) — alanın kapsamı ve temel kavramlar
- [Nereden başlanır: yol haritası](yol-haritasi.md) — aşamaların tanımı ve sırası
- [01 — Mimari](../01-mimari/) — işlemci modeli, register'lar ve ayrıcalık seviyeleri
- [03 — Assembly](../03-assembly/) — söz dizimi, çağrı kuralı ve inline assembly
- [Terimler sözlüğü](../TERIMLER.md) — terim kullanımı

## Kaynaklar

- ISO/IEC 9899:2018 (C17), Bölüm 4 — Conformance (freestanding ortam tanımı ve zorunlu başlıklar)
- Brian W. Kernighan, Dennis M. Ritchie, *The C Programming Language*, 2. baskı
- System V Application Binary Interface — AMD64 Architecture Processor Supplement, Sürüm 1.0, Bölüm 3.2 (Function Calling Sequence); <https://gitlab.com/x86-psABIs/x86-64-ABI> (erişim: 22 Ağustos 2026)
- GCC dokümantasyonu, "How to Use Inline Assembly Language in C Code"; <https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html> (erişim: 22 Ağustos 2026)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 1, Bölüm 3 — Basic Execution Environment
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 8.2 — Memory Ordering
- Arm® Architecture Reference Manual for A-profile architecture, Bölüm B2 — The AArch64 Application Level Memory Model
- Tool Interface Standard (TIS) Executable and Linking Format (ELF) Specification, Sürüm 1.2, Bölüm 1 — Object Files
- GNU Binutils dokümantasyonu, "Using ld" — Scripts; <https://sourceware.org/binutils/docs/ld/Scripts.html> (erişim: 22 Ağustos 2026)
- John R. Levine, *Linkers and Loaders*, Bölüm 3 — Object Files
- Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*; <https://ostep.org> (erişim: 22 Ağustos 2026)
- David A. Patterson, John L. Hennessy, *Computer Organization and Design*, 5. baskı, Bölüm 2 — Instructions: Language of the Computer
- OSDev Wiki, "Required Knowledge"; <https://wiki.osdev.org/Required_Knowledge> (erişim: 22 Ağustos 2026)
