# x86-64 register'ları

Register, işlemcinin üzerinde doğrudan işlem yapabildiği, çekirdeğin içinde duran sabit sayıdaki saklama birimidir. Bir komut belleği okuyup yazabilir, ancak toplama, karşılaştırma veya kaydırma gibi işlemlerin operandları neredeyse her zaman register'lardadır. Bellek erişimi en iyi durumda önbellekten birkaç çevrim sürerken register erişimi komutun kendi yürütme adımının parçasıdır; aradaki fark, işlemcinin neden az sayıda ama çok hızlı saklama birimiyle çalıştığını açıklar.

x86-64'te bu birimler tek bir küme değildir. Aritmetik ve adresleme için kullanılan genel amaçlı register'ların yanında durum bayraklarını tutan RFLAGS, adres çevirisini ve işlemci özelliklerini yöneten kontrol register'ları, model'e özgü MSR'lar ve SIMD durumu vardır. Kernel yazarken bu kümelerin hepsine dokunulur: kullanıcı modundan gelen bağlam genel amaçlı register'larda, sayfa tablosunun kökü CR3'te, system call giriş adresi bir MSR'da durur. Bu not, x86-64'ün register kümesini, donanımın bu register'lara yüklediği örtük anlamları ve kernel'in bu durumu saklarken uyması gereken kuralları anlatır.

## Ön koşullar

- [01 — Mimari](../01-mimari/) — işlemci modeli ve ayrıcalık seviyeleri
- [Kernel nedir](../04-kernel/kernel-nedir.md) — kernel modunun donanım dayanağı

## Kapsam

Bu not x86-64 mimarisini ve long mode'u esas alır. 32-bit protected mode farkları yalnızca gerektiği yerde belirtilir. Assembly örnekleri NASM (Intel) söz dizimiyle yazılmıştır. Çağrı kuralının tamamı `03-assembly/calling-convention.md`, adresleme biçimleri `03-assembly/adresleme-modlari.md` notunun konusudur.

## Problem

Register kümesinin neden bu kadar küçük olduğu ilk bakışta anlaşılmaz. İşlemci içinde daha fazla saklama birimi bulundurmak mümkündür, ancak sayı arttıkça her komutun register seçmek için harcadığı bit sayısı da artar. Üç operandlı bir komutta 16 register'ı adreslemek 12 bit, 256 register'ı adreslemek 24 bit ister; komut kodlaması büyür, komut önbelleği aynı işi yapmak için daha fazla bayt taşır. Buna ek olarak register dosyası her çevrimde birden çok okuma ve yazma portundan erişilen bir yapıdır ve büyüdükçe erişim gecikmesi artar. Az sayıda register, kodlama yoğunluğu ile erişim hızı arasındaki bu ödünleşimin sonucudur.

x86'nın kendi kümesi ayrıca tarihsel bir yüktür. 8086'nın sekiz register'ı genel amaçlı değildi; her birinin komut kodlamasına gömülmüş bir görevi vardı. Bugün RAX'in çarpma ve bölmede örtük operand olması, RCX'in kaydırma ve `rep` sayacı olarak kullanılması ya da string komutlarının RSI ve RDI'yi zorunlu tutması bu dönemden kalmadır. AMD64, kümeyi 64 bite genişletirken sekiz yeni register (R8–R15) ekledi ve register baskısını belirgin biçimde azalttı, ancak eski örtük rolleri kaldırmadı; geriye uyumluluk bunu gerektiriyordu.

Kernel tarafında sorun bir adım daha gider. Register'lar yalnızca hesabın yapıldığı yer değil, aynı zamanda her yürütme akışının kimliğidir. Bir interrupt geldiğinde kesintiye uğrayan process'in tüm durumu bu register'larda durur; kernel bu durumu eksiksiz saklayıp geri yükleyemezse, program hatayı kesilme anında değil çok sonra, açıklanamaz bir davranış olarak gösterir. Register kümesini tanımak bu yüzden assembly yazmanın değil, doğru context switch yazmanın ön koşuludur.

## Donanım seviyesinde

### Genel amaçlı register'lar

Long mode'da 16 adet 64 bitlik genel amaçlı register vardır. Her birine dört farklı genişlikte erişilebilir; erişim genişliği ayrı bir register değil, aynı register'ın alt kısmıdır.

| 64 bit | 32 bit | 16 bit | 8 bit | Örtük rolü |
| --- | --- | --- | --- | --- |
| RAX | EAX | AX | AL | dönüş değeri; `mul`, `div` ve `imul`'da örtük operand |
| RCX | ECX | CX | CL | kaydırma ve `rep` sayacı; `syscall`'da dönüş adresi |
| RDX | EDX | DX | DL | `RDX:RAX` çiftinin üst yarısı; `in` ve `out`'ta port adresi |
| RBX | EBX | BX | BL | örtük rolü yok |
| RSI | ESI | SI | SIL | string komutlarında kaynak adresi |
| RDI | EDI | DI | DIL | string komutlarında hedef adresi |
| RBP | EBP | BP | BPL | frame pointer (zorunlu değil) |
| RSP | ESP | SP | SPL | stack pointer; `push`, `call` ve interrupt girişinde kullanılır |
| R8–R15 | R8D–R15D | R8W–R15W | R8B–R15B | örtük rolü yok |

Eski dört register'ın ikinci baytına ayrıca AH, BH, CH ve DH adlarıyla erişilebilir. Bu adlar, komutun REX ön ekiyle kodlandığı durumlarda kullanılamaz: aynı kodlama alanı SPL, BPL, SIL ve DIL'e ayrılmıştır. Pratikte AH ile R8B'yi aynı komutta kullanmak mümkün değildir.

R8–R15'e ve 64 bit operand genişliğine erişim REX ön ekiyle sağlanır. Bu ön ek komutu bir bayt uzatır; yeni register'ları kullanan kodun eski register'larla yazılmış eşdeğerinden biraz daha büyük olmasının nedeni budur.

Yazma genişliğinin üst bitlere etkisi, x86-64'ün en sık gözden kaçan kuralıdır:

- 32 bitlik bir hedefe yazmak, register'ın üst 32 bitini **sıfırlar**.
- 8 veya 16 bitlik bir hedefe yazmak, üst bitleri **değiştirmez**.

```asm
; NASM (Intel) söz dizimi
mov rax, 0xFFFFFFFFFFFFFFFF
mov eax, 1                    ; RAX artık 0x0000000000000001

mov rax, 0xFFFFFFFFFFFFFFFF
mov al, 1                     ; RAX artık 0xFFFFFFFFFFFFFF01

xor eax, eax                  ; RAX = 0; iki bayt, bayrakları da değiştirir
```

Bu asimetri kod üretiminde bilinçli olarak kullanılır: 32 bitlik işlem yapan derleyici çıktısının 64 bitlik değerler için de doğru kalması, sıfır genişletmenin bedava gelmesindendir. Aynı asimetri, elle yazılan assembly'de sessiz hataların da kaynağıdır.

RIP (instruction pointer) genel amaçlı register değildir ve doğrudan yazılamaz; yalnızca dallanma komutlarıyla değişir. Long mode'da RIP'e göreli adresleme (`[rel etiket]`) eklendiği için konumdan bağımsız kod yazmak 32-bit moddaki gibi ek numaralar gerektirmez.

### RFLAGS

RFLAGS, komutların ürettiği durum bilgisini ve işlemcinin bazı davranış anahtarlarını tutar. 64 bitlik register'ın üst yarısı ayrılmıştır.

| Bayrak | Bit | Anlamı |
| --- | --- | --- |
| CF | 0 | işaretsiz taşma veya elde |
| PF | 2 | sonucun düşük baytında çift sayıda 1 biti |
| AF | 4 | BCD aritmetiği için yarım elde |
| ZF | 6 | sonuç sıfır |
| SF | 7 | sonucun en yüksek biti |
| TF | 8 | tek adım; her komuttan sonra #DB |
| IF | 9 | maskelenebilir interrupt'lar açık |
| DF | 10 | string komutlarının yönü; 1 ise azalan |
| OF | 11 | işaretli taşma |
| IOPL | 12–13 | port I/O için gereken ayrıcalık seviyesi |
| AC | 18 | hizalama denetimi; SMAP ile birlikte erişim anahtarı |

Bayraklar bir bütün olarak `pushfq` ve `popfq` ile stack üzerinden taşınır. Kernel açısından iki bit ayrıca önemlidir: IF, `cli` ve `sti` ile değiştirilir ve interrupt girişinde işlemci tarafından temizlenebilir; DF ise çağrı kuralının sıfır varsaydığı bir bayraktır.

### Segment register'ları

Long mode'da CS, DS, ES, SS, FS ve GS bulunur, ancak çoğunun anlamı daralmıştır. DS, ES ve SS'in tabanı sıfır kabul edilir ve limit denetimi yapılmaz; bu register'lara yüklenen seçici adres hesabını etkilemez. CS ise hâlâ işlevseldir: geçerli ayrıcalık seviyesi (CPL) CS seçicisinin en düşük iki bitinde tutulur ve descriptor'ının L biti 64 bit modu belirler.

FS ve GS istisnadır. Bu ikisinin taban adresi descriptor'dan değil MSR'lardan gelir ve adres hesabına gerçekten eklenir.

| MSR | Numara | İçerik |
| --- | --- | --- |
| IA32_FS_BASE | 0xC0000100 | FS taban adresi |
| IA32_GS_BASE | 0xC0000101 | etkin GS taban adresi |
| IA32_KERNEL_GS_BASE | 0xC0000102 | `swapgs` ile takas edilecek yedek taban |

`swapgs` komutu IA32_GS_BASE ile IA32_KERNEL_GS_BASE'in içeriğini yer değiştirir. Bu tek komut, kernel'in kullanıcı modundan girişte kendi işlemci başına verisine ulaşmasını sağlayan mekanizmadır: kullanıcı modu GS tabanını kendi amacı için kullanırken (thread local storage), kernel girişte takas yapar ve `gs:` ön ekiyle işlemci başına yapıya erişir. CR4.FSGSBASE açıksa aynı tabanlar `rdfsbase`, `wrfsbase`, `rdgsbase` ve `wrgsbase` komutlarıyla MSR erişimi olmadan okunup yazılabilir; kullanıcı modu da bunu yapabildiği için, bu özellik açıkken GS tabanının belirli bir değerde olduğu varsayılamaz.

### Kontrol register'ları ve MSR'lar

Kontrol register'ları işlemcinin çalışma kipini belirler ve yalnızca ring 0'dan erişilebilir.

| Register | İçerik |
| --- | --- |
| CR0 | PE (protected mode), WP (write protect), PG (paging), TS ve EM (FPU tuzakları) |
| CR2 | son page fault'un oluştuğu doğrusal adres |
| CR3 | etkin sayfa tablosunun fiziksel tabanı ve PCID |
| CR4 | PAE, PGE, OSFXSR, OSXSAVE, PCIDE, FSGSBASE, SMEP, SMAP, UMIP, LA57 |
| CR8 | TPR (Task Priority Register); hangi interrupt önceliklerinin maskeleneceği |

MSR'lar (Model Specific Register) numarayla adreslenen, `rdmsr` ve `wrmsr` ile erişilen genişletilebilir bir register alanıdır. Numara ECX'te verilir, değer EDX:EAX çiftinde taşınır; yani 64 bitlik içerik iki 32 bitlik yarıya bölünür. Kernel başlatmasında en sık dokunulanlar IA32_EFER (long mode, `syscall` etkinleştirmesi ve NX biti), IA32_STAR, IA32_LSTAR ve IA32_FMASK (system call girişi) ile yukarıdaki taban register'larıdır.

Ayrıca DR0–DR3 donanım kesme noktalarının adreslerini, DR6 hangi kesme noktasının tetiklendiğini, DR7 ise etkinleştirme ve tetikleme koşullarını tutar. DR4 ve DR5, DR6 ile DR7'nin takma adlarıdır.

### SIMD ve x87 durumu

Kayan nokta ve vektör işlemleri ayrı bir küme kullanır: x87 yığını (ST0–ST7, MMX ile örtüşür), SSE'nin XMM0–XMM15 register'ları, AVX ile bunların 256 bite genişlemesi (YMM), AVX-512 ile hem 512 bite genişlemesi hem de sayının 32'ye çıkması (ZMM0–ZMM31) ve maskeleme register'ları (K0–K7). Yuvarlama kipi ve istisna maskeleri MXCSR'da tutulur.

Bu kümenin toplam boyutu genel amaçlı register'ların çok üzerindedir ve elle saklanmaz. `xsave` ve `xrstor` komutları, XCR0'da hangi durum bileşenlerinin etkin olduğuna bakarak durumu bir bellek alanına yazar ve geri yükler. XCR0'a yazmak için CR4.OSXSAVE'in açık olması gerekir. Alanın boyutu işlemciye ve etkin bileşenlere göre değiştiğinden `cpuid` ile sorgulanır; sabit varsaymak taşınabilir değildir.

## Kernel tarafında

**Girişte donanımın yaptığı iş sınırlıdır.** Bir interrupt veya exception oluştuğunda işlemci stack'e SS, RSP, RFLAGS, CS ve RIP'i iter; bazı exception'larda ayrıca bir hata kodu ekler. Genel amaçlı register'ların hiçbiri kaydedilmez. Kesintiye uğrayan akışın RAX'i, RDI'si ve diğerleri hâlâ oradadır ve handler'ın yazacağı ilk değerle yok olur. Bu yüzden her interrupt giriş noktası, C koduna geçmeden önce kullanacağı register'ları elle saklar.

**`syscall` daha da azını yapar.** Komut, dönüş adresini RCX'e ve RFLAGS'i R11'e kopyalar, IA32_FMASK'te işaretli bayrakları temizler ve IA32_LSTAR'daki adrese atlar. Stack değiştirilmez: giriş anında RSP hâlâ kullanıcı stack'ini gösterir. Kernel stack'ine geçiş, `swapgs` ile işlemci başına yapıya ulaşıp oradaki kernel stack işaretçisini okuyarak kernel tarafından yapılır. RCX ve R11'in içeriği `sysret` için gerekli olduğundan bu iki register system call boyunca korunmalıdır; Linux'un dördüncü system call argümanını RCX yerine R10'da taşımasının nedeni budur.

**Context switch'te ne kaydedileceği, geçişin nasıl olduğuna bağlıdır.** Kernel'in kendi isteğiyle yaptığı geçiş bir fonksiyon çağrısı gibidir: çağrı kuralı gereği caller-saved register'lar zaten çağıran tarafça korunmuş sayılır, dolayısıyla RBX, RBP, R12–R15, RSP ve dönüş adresini kaydetmek yeterlidir. Interrupt üzerinden yapılan geçişte böyle bir sözleşme yoktur; kesilen akış herhangi bir komutun ortasındadır ve tüm genel amaçlı register'lar ile RFLAGS eksiksiz saklanmalıdır. Kernel bu tam kümeyi genellikle stack'in tepesinde tek bir yapı olarak tutar ve aynı düzeni hem interrupt dönüşünde hem de panic çıktısında kullanır.

**Adres alanı değişimi CR3 üzerinden olur.** Farklı bir process'e geçilirken CR3'e yeni sayfa tablosunun fiziksel adresi yazılır; bu yazma, PCID kullanılmıyorsa global olmayan tüm TLB (Translation Lookaside Buffer) girdilerini geçersiz kılar. Aynı adres alanı içinde kalan thread geçişlerinde CR3'e dokunulmaz.

**SIMD durumu ayrı ele alınır.** Kernel kodu kural olarak SIMD register'larını kullanmaz. Kullansaydı her kernel girişinde kullanıcının vektör durumunu kaydetmek zorunda kalırdı ve bu, kısa bir system call'un maliyetini birkaç katına çıkarırdı. Kullanıcı durumu bu yüzden yalnızca process değiştiğinde `xsave` ile saklanır. CR0'daki TS biti ile saklama, durumun gerçekten kullanıldığı ana ertelenebilir; kernel'lerin bir kısmı tuzak maliyeti nedeniyle bu ertelemeden vazgeçip her geçişte kaydetmeyi tercih eder.

**MSR kurulumu başlatmanın parçasıdır.** `syscall` yolunun çalışması için IA32_EFER.SCE açılmalı; IA32_STAR'a kernel ve kullanıcı segment seçicileri, IA32_LSTAR'a giriş noktasının adresi, IA32_FMASK'e girişte temizlenecek bayraklar (en azından IF ve DF) yazılmalıdır. İşlemci başına verinin adresi IA32_KERNEL_GS_BASE'e yazılır ve bu, her işlemci için ayrı ayrı yapılır; çok işlemcili bir sistemde ikincil işlemciler kendi kurulumlarını kendileri tamamlar.

## Implementasyon notları

**Red zone kernel'de kapatılmalıdır.** System V AMD64 ABI, yaprak fonksiyonların RSP'nin altındaki 128 baytı stack işaretçisini değiştirmeden kullanmasına izin verir. Bu alan kullanıcı modunda güvenlidir, çünkü sinyal çerçevesi onu atlar. Kernel modunda böyle bir garanti yoktur: kernel çalışırken gelen bir interrupt, işlemcinin ittiği çerçeveyi doğrudan bu 128 baytın üzerine yazar. Kernel kodu bu yüzden `-mno-red-zone` ile derlenir. Belirtisi, nadiren ve yalnızca yük altında bozulan yerel değişkenlerdir.

**Kernel derlemesinde SIMD kapatılır.** Derleyici, `memcpy` benzeri işlemleri veya yapı kopyalarını kendiliğinden XMM register'larıyla üretebilir. Kernel'in vektör durumunu saklamadığı bir yerde bu, kullanıcının register'larının sessizce bozulması demektir. GCC ve Clang'de `-mno-sse -mno-sse2 -mno-mmx -mno-80387` ya da kısaca `-mgeneral-regs-only` kullanılır.

**Kısmi register yazımından kaçının.** `mov al, [ptr]` komutu RAX'in üst bitlerini koruduğu için işlemci açısından önceki RAX değerine bağımlıdır; bu yapay bağımlılık yürütmeyi seri hale getirebilir. Genişletme gerekiyorsa `movzx eax, byte [ptr]` (sıfır genişletme) veya `movsx` (işaret genişletme) tercih edilir. Aynı nedenle sıfırlama `mov rax, 0` yerine `xor eax, eax` ile yapılır; ikincisi hem daha kısadır hem de işlemci tarafından bağımlılık kesici olarak tanınır. Bayrakların korunması gereken yerlerde bu değişim geçerli değildir.

**DF bayrağını girişte temizleyin.** Çağrı kuralı, bir fonksiyona girildiğinde DF'in sıfır olduğunu varsayar; derleyicinin ürettiği `rep movsb` bu varsayıma dayanır. Kullanıcı modu DF'i set edip bir system call yaparsa, temizlenmediği takdirde kernel içindeki kopyalama ters yönde çalışır. IA32_FMASK'e DF bitini eklemek `syscall` yolunu kapatır; interrupt ve exception yolunda ayrıca `cld` gerekir.

**`swapgs` eşlemesinin tek kuralı vardır: yalnızca ayrıcalık seviyesi gerçekten değiştiyse çalıştırılır.** Kernel modunda oluşan bir exception'ın handler'ında `swapgs` çalıştırmak, GS tabanını kullanıcının değerine çevirir ve işlemci başına veriye yapılan ilk erişimi geçersiz bir adrese yönlendirir. Karar, girişte itilen CS'in düşük iki bitine bakılarak verilir. NMI ve #DB gibi her yerde oluşabilen exception'larda bu denetim tek başına yetmez; bunlar için IST (Interrupt Stack Table) üzerinden ayrı stack kullanmak ve GS tabanını doğrudan okuyup karar vermek yaygın çözümdür.

**Frame pointer'ı bilinçli seçin.** Derleyiciler varsayılan olarak `-fomit-frame-pointer` uygular ve RBP'yi sıradan bir register olarak kullanır. Bu, register baskısını azaltır ancak panic çıktısında çağrı yığınını stack'i tarayarak tahmin etmek zorunda bırakır. Erken aşamada hata ayıklamanın kolaylığı genellikle bu maliyete değer.

**Register kaydetme düzenini tek yerde tanımlayın.** Interrupt girişinde register'ların stack'e itilme sırası, dönüşte çekilme sırası, `iretq`'nun beklediği çerçeve düzeni ve C tarafındaki yapı tanımı birbirini tutmak zorundadır. Bu dördü ayrı ayrı yazıldığında ortaya çıkan hata dönüşte değil, geri yüklenen yanlış register'ın ilk kullanıldığı yerde görünür.

## Mimariye göre farklar

| Konu | x86-64 | ARM64 | RISC-V |
| --- | --- | --- | --- |
| Genel amaçlı register | 16 | 31 (X0–X30) | 32 (x0–x31) |
| Sıfır register'ı | yok | XZR / WZR | x0 |
| Stack pointer | RSP, kümenin parçası | SP, ayrı; her EL için farklı | x2, sözleşmeye bağlı |
| Durum bayrakları | RFLAGS | PSTATE (NZCV) | yok; karşılaştırma dallanmada |
| Program sayacı | RIP, yazılamaz | PC, doğrudan erişilemez | pc |
| Alt genişliğe yazma | 32 bit sıfır genişletir | W görünümü sıfır genişletir | tam genişlik |
| İşlemci başına taban | GS ve `swapgs` | TPIDR_EL1 | sscratch |
| Sistem ayarları | CR'ler ve MSR'lar | sistem register'ları (`msr`, `mrs`) | CSR'ler (`csrrw` ailesi) |

Fark isimlendirmenin ötesine geçer. ARM64 ve RISC-V'de exception oluştuğunda dönüş adresi ve durum, stack'e itilmek yerine ayrı register'lara (ELR_EL1 ve SPSR_EL1; sepc ve sstatus) yazılır; giriş kodunun ilk işi stack ayarlamak değil, bu register'ları üzerlerine yazılmadan önce güvene almaktır. Bayrak register'ının olmaması RISC-V'de karşılaştırma ile dallanmayı tek komutta birleştirir ve saklanacak durumu bir register azaltır. `swapgs`'in karşılığı RISC-V'de `sscratch` ile bir genel amaçlı register'ı takas etmektir; ARM64'te böyle bir takasa gerek yoktur, çünkü TPIDR_EL1 zaten yalnızca EL1'den erişilebilir.

## Sık yapılan hatalar

- 8 veya 16 bitlik bir yazmanın üst bitleri temizlediğini varsaymak. Yalnızca 32 bitlik yazma sıfır genişletir.
- Interrupt handler'ında yalnızca "kullanılan" register'ları saklamak. C koduna dallanan bir handler'da hangi register'ların değişeceği çağrı kuralına ve derleyici seçimine bağlıdır.
- Kernel kodunu red zone açık derlemek. Hata yük altında ve rastgele görünür.
- `swapgs`'i kernel modunda oluşan exception'da da çalıştırmak ya da girişte yapıp dönüş yollarından birinde unutmak.
- `syscall` girişinde RSP'nin kernel stack'ini gösterdiğini varsaymak. İşlemci stack'i değiştirmez.
- RCX veya R11'i system call yolunda kalıcı değer taşımak için kullanmak; ikisi de `syscall` tarafından ezilir.
- `cld` çalıştırmadan derleyicinin ürettiği string komutlarına güvenmek.
- `xsave` alanı için sabit boyut varsaymak. Boyut işlemciye ve XCR0'daki etkin bileşenlere göre değişir.
- MSR erişiminde 64 bitlik değerin EDX:EAX çiftine bölündüğünü atlamak; üst yarı yazılmadığında değer sessizce budanır.

## İlgili notlar

- [03 — Assembly](README.md) — bölümün diğer konuları
- [Kernel nedir](../04-kernel/kernel-nedir.md) — ayrıcalık seviyeleri ve kernel'e giriş yolları
- [02 — Boot](../02-boot/) — long mode'a geçişte kontrol register'larının kurulumu
- [06 — Process](../06-process/) — context switch ve saklanan bağlam

## Kaynaklar

- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 1, Bölüm 3.4 — Basic Program Execution Registers
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 1, Bölüm 3.4.1.1 — General-Purpose Registers in 64-Bit Mode (sıfır genişletme, REX ve AH/BH/CH/DH kısıtı)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 1, Bölüm 13 — Managing State Using the XSAVE Feature Set (XCR0 ve alan boyutu)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 2.5 — Control Registers
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3B, Bölüm 18 — Debug and Branch Profile Features (DR0–DR7)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 2B, SWAPGS, SYSCALL ve SYSRET komut tanımları
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 4 — Model-Specific Registers (IA32_EFER, IA32_STAR, IA32_LSTAR, IA32_FMASK, FS ve GS taban MSR'ları)
- AMD64 Architecture Programmer's Manual, Cilt 2, Bölüm 3 — System Resources
- System V Application Binary Interface, AMD64 Architecture Processor Supplement, Bölüm 3.2 — Function Calling Sequence (register sınıfları, red zone, DF varsayımı)
- Arm® Architecture Reference Manual for A-profile architecture, Bölüm B1 — AArch64 Application Level Programmers' Model ve Bölüm D1 — AArch64 System Level Programmers' Model
- The RISC-V Instruction Set Manual, Volume I: Unprivileged ISA, Bölüm 2.1 — Programmers' Model for Base Integer ISA
- The RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Bölüm 4.1 — Supervisor CSRs (sscratch, sepc, sstatus)
- OSDev Wiki, "CPU Registers x86-64"; <https://wiki.osdev.org/CPU_Registers_x86-64> (erişim: 23 Ağustos 2026)
