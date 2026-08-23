# Kernel nedir

Kernel, işletim sisteminin donanımın en yüksek ayrıcalık seviyesinde çalışan, makine açıldıktan sonra kapanana kadar bellekte kalan ve işlemci zamanı, bellek ve aygıtlar üzerindeki son sözü söyleyen parçasıdır. Bir programın bellek ayırması, dosya okuması ya da ekrana bir şey yazması, doğrudan donanıma dokunmasıyla değil; kernel'in izin verdiği bir giriş noktasından geçmesiyle olur.

Kernel ile işletim sistemi aynı şey değildir. Bir işletim sistemi kernel'in yanında kabuk, servisler, kütüphaneler ve araçlardan oluşur; bunların hepsi userspace'te çalışır. Kernel bu yığının yalnızca ayrıcalıklı katmanıdır. Bu not, o katmanın hangi problemden doğduğunu, donanımın ona hangi mekanizmaları verdiğini ve kernel'in bu mekanizmalar üzerine ne kurduğunu anlatır.

## Ön koşullar

- [OSDev nedir](../00-giris/osdev-nedir.md)

## Kapsam

Somut örnekler x86-64 ve long mode üzerinden verilir. ARM64 ve RISC-V karşılıkları ayrı bir bölümde özetlenir. Bu not kernel'in ne olduğunu tanımlar; sorumlulukların monolitik, mikrokernel veya hibrit tasarımlar arasında nasıl bölüştürüldüğü `04-kernel/kernel-mimarileri.md` notunun konusudur.

## Problem

Donanımı programlar adına yöneten bir kod parçası gerektiği açıktır. Asıl soru, bu kodun neden ayrı ve ayrıcalıklı bir bileşen olmak zorunda olduğudur. Aynı iş, her programa bağlanan bir kütüphane ile de yapılabilirmiş gibi görünür.

Yapılamaz, çünkü kütüphane çağıranın insafındadır. Programın adres alanında duran kod, o programın atlayabileceği, değiştirebileceği ve yok sayabileceği bir koddur. Donanımı paylaştıran bileşenin üç özelliği sağlaması gerekir ve üçü de kütüphaneyle sağlanamaz:

- **Atlanamaz olmalıdır.** Disk denetleyicisine yazma yetkisi bir programın elindeyse, o program başka bir programın dosyasını okuyabilir. Yetkinin, programın erişemeyeceği bir yerde durması gerekir.
- **Denetimi geri alabilmelidir.** Çalışan program hiçbir şey çağırmasa bile — sonsuz döngüye girse bile — sistemin ondan işlemciyi geri alabilmesi gerekir. Yalnızca çağrıldığında çalışan bir kod bunu yapamaz.
- **Bütünü görmelidir.** Hangi fiziksel sayfanın kime ait olduğu, hangi process'in çalışmayı beklediği ya da bir dosyanın hangi bloklarda durduğu, tek bir programın bilgisiyle yanıtlanamaz. Bu durum bilgisinin tek bir yerde ve tutarlı tutulması gerekir.

Bu üç özellik yalnızca yazılımla elde edilemez. Kernel'in diğer programlar üzerindeki yetkisi işlemcinin ayrıcalık seviyelerinden, MMU'dan (Memory Management Unit) ve interrupt mekanizmasından gelir. Kernel, donanımın sunduğu bu ayrımın ayrıcalıklı tarafında duran koddur; tanımı da esas olarak budur.

Bunun bedeli vardır. Kernel'i koruyacak bir üst katman yoktur: kernel modunda oluşan bir hatanın yakalayıcısı yine kernel'dir, kernel'in bozduğu bir veri yapısını düzeltecek başka bir bileşen yoktur. Ayrıcalık, aynı ölçüde sorumluluk getirir.

## Donanım seviyesinde

Kernel'i mümkün kılan donanım desteği dört başlıkta toplanır.

**Ayrıcalık seviyeleri.** x86-64'te dört ring tanımlıdır, pratikte ring 0 (kernel modu) ve ring 3 (kullanıcı modu) kullanılır. Geçerli ayrıcalık seviyesi CPL (Current Privilege Level), CS segment seçicisinin en düşük iki bitinde tutulur; ayrı bir "kernel modundayım" bayrağı yoktur. Ayrıcalıklı komutlar — `lgdt`, `lidt`, `hlt`, `invlpg`, `wrmsr` ve CR register'larına yazma — ring 3'te çalıştırılmaya kalkışıldığında yürütülmez, işlemci #GP (General Protection) exception'ı üretir. Yani koruma, kernel'in yaptığı bir denetim değil, işlemcinin komut çözme aşamasında uyguladığı bir kuraldır.

**Adres çevirisi.** MMU, programın kullandığı sanal adresleri sayfa tabloları üzerinden fiziksel adreslere çevirir. Etkin sayfa tablosunun kökü CR3 register'ında durur ve CR3'e yazmak ayrıcalıklı bir işlemdir. Sayfa tablosu girdilerindeki U/S bayrağı, o sayfaya ring 3'ten erişilip erişilemeyeceğini belirler: kernel'in kodu ve verisi kullanıcı sayfa tablolarında haritalı olsa bile, bu bayrak sıfır olduğu sürece kullanıcı modundan okunamaz. Böylece koruma her erişimde, donanım tarafından ve ek maliyet olmadan uygulanır.

**Denetimli giriş noktaları.** Kullanıcı modundaki bir programın kernel'e girmesinin tek yolu donanımın tanımladığı geçişlerdir; adres verip kernel'in ortasına atlamak mümkün değildir.

| Giriş yolu | Ne zaman oluşur | Nereye gider |
| --- | --- | --- |
| interrupt | aygıt sinyal verdiğinde, timer dolduğunda | IDT'de vektöre karşılık gelen girdi |
| exception | hatalı komut, page fault, sıfıra bölme | IDT'deki ilgili exception vektörü |
| system call | program bilerek kernel'den hizmet istediğinde | IA32_LSTAR MSR'ında yazan adres |

Her üç durumda da işlemci ayrıcalık seviyesini yükseltir ve yürütmeyi kernel'in önceden belirlediği bir adrese aktarır. Interrupt ve exception yolunda ayrıcalık seviyesi değiştiği için stack de değiştirilir: işlemci TSS'teki (Task State Segment) RSP0 alanından yeni stack işaretçisini alır. `syscall` komutu ise stack'i kendiliğinden değiştirmez; hedef adresi IA32_LSTAR, hedef segment seçicilerini IA32_STAR ve girişte temizlenecek RFLAGS bitlerini IA32_FMASK MSR'ları belirler, kernel stack'ine geçişi kernel kendisi yapar.

**Denetimi geri alma.** Bir timer, ayarlanan süre dolduğunda interrupt üretir. Kernel'in, çalışan programın işbirliğine ihtiyaç duymadan işlemciyi geri almasını sağlayan mekanizma budur. Preemptive scheduling'in dayandığı donanım desteği de aynı mekanizmadır; timer interrupt olmadan bir program işlemciyi kendi isteğiyle bırakana kadar tutar.

Bu dördünün üstüne, kernel'i kendi hatalarına karşı koruyan yardımcı özellikler eklenir. x86-64'te SMEP (Supervisor Mode Execution Prevention) kernel modundayken kullanıcı sayfalarındaki kodun çalıştırılmasını, SMAP (Supervisor Mode Access Prevention) ise bu sayfalara istemsiz erişimi engeller; ikisi de CR4 register'ından açılır. SMAP açıkken kernel'in kullanıcı belleğine bilerek erişmesi gereken kısa bölgeler `stac` ve `clac` komutlarıyla işaretlenir.

## Kernel tarafında

**Devralma.** Kernel çalışmaya başladığında makine, bootloader'ın bıraktığı durumdadır: geçici bir GDT, geçici sayfa tabloları, boot sırasında kullanılan bir stack ve bellekte bir yerde duran boot bilgisi yapısı. Bu yapıların hiçbiri kalıcı değildir. Kernel'in ilk işi kendi GDT'sini, IDT'sini, sayfa tablolarını ve stack'ini kurmak, bootloader'ın verdiği bellek haritasını okuyup kendi yapılarına aktarmaktır. Bu aktarım tamamlanmadan, bootloader'ın bıraktığı bellek bölgeleri yeniden kullanılamaz.

**Tuttuğu yapılar.** Kernel'in bellekte kalıcı olarak tuttuğu durum kabaca şudur:

| Yapı | Ne için |
| --- | --- |
| GDT, IDT, TSS | işlemciyle kurulan sözleşme: segmentler, interrupt vektörleri, kernel stack'leri |
| sayfa tabloları | her adres alanı için bir hiyerarşi, fiziksel bellek allocator'ının durumu |
| process ve thread tabloları | çalışan işlerin durumu, kayıtlı register bağlamları, run queue |
| aygıt ve sürücü kayıtları | hangi interrupt vektörünün hangi handler'a gittiği, aygıt durumları |
| dosya sistemi durumu | bağlı dosya sistemleri, açık dosya tabloları, önbellekler |

**Çalışma biçimi.** Kernel'in, başlatma aşamasından sonra kendine ait sürekli dönen bir ana döngüsü yoktur. Başlatma bittiğinde kernel yürütmeyi ilk kullanıcı process'ine devreder ve o andan sonra yalnızca yukarıdaki üç giriş yolundan biriyle çalışır: bir interrupt gelir, bir exception oluşur ya da bir system call yapılır. Bu anlamda kernel bir program değil, olaylara yanıt veren bir kod topluluğudur. Yapacak iş kalmadığında bile kernel "beklemez"; işlemciyi `hlt` ile bir sonraki interrupt'a kadar durduran boş bir idle akışı çalışır.

Her giriş aynı iskeleti izler: değişecek register'lar kaydedilir, kernel stack'ine geçilir, iş yapılır, durum geri yüklenir ve `iretq` ya da `sysret` ile kullanıcı moduna dönülür. Bu simetrinin bozulduğu her yer — kaydedilmeyen bir register, dengelenmeyen bir stack — hatayı dönüşün kendisinde değil, çoğu zaman çok sonra gösterir.

**Zorunlu olarak ele aldığı durumlar.** Kernel kodunun uyması gereken kısıtlar userspace kodunun kısıtlarından farklıdır:

- **Bağlam ayrımı.** Interrupt handler'ı içinde çalışan kod, kesintiye uğrattığı process'in bağlamındadır ve bloke olamaz. Uykuya geçen ya da kilit bekleyen bir interrupt handler'ı, hiç çalışmayacak bir kodu bekliyor olabilir. Uzun işler, handler'ın kısa bir kısmıyla kuyruğa alınıp sonradan yapılır.
- **Yeniden girilebilirlik ve eşzamanlılık.** Aynı kernel kodu, bir işlemcide interrupt tarafından bölünürken başka bir işlemcide baştan çalışıyor olabilir. Paylaşılan her yapının erişimi kilitle ya da atomik işlemle korunmalıdır.
- **Sınırlı stack.** Kernel stack'i sabit boyutludur; yaygın seçim thread başına 8–16 KiB'dir. Büyük yerel diziler ve derin özyineleme taşmaya yol açar.
- **Standart kütüphane yokluğu.** Kernel freestanding bir C ortamında derlenir. `malloc`, `printf` ve dosya erişimi yoktur; kullanılacaklarsa önce yazılmaları gerekir.
- **Güvenilmeyen girdi.** Userspace'ten gelen her işaretçi ve her uzunluk düşmanca kabul edilir. Doğrulanmadan dereference edilen bir kullanıcı işaretçisi, kernel'in kendi yetkisiyle rastgele belleğe erişmesi demektir.
- **Kurtarılamaz hata.** Kernel modunda ele alınmayan bir exception'ın devredileceği bir üst katman yoktur. Bu durumda yapılacak iş, durumu okunabilir biçimde yazıp sistemi durdurmaktır.

## Implementasyon notları

**Başlatma sırası hata ayıklanabilirliğe göre kurulur.** Yaygın sıra şudur: en erken çıktı yolu (seri port ya da framebuffer) → GDT → IDT ve exception handler'ları → fiziksel bellek allocator'ı → kernel'in kendi sayfa tabloları → heap → timer → scheduler. Çıktının ve exception handler'larının başa alınmasının nedeni işlevsel değildir: bu ikisi kurulmadan yapılan bir hata, ekranda hiçbir iz bırakmadan makinenin yeniden başlamasıyla sonuçlanır. Handler'ı olan bir kernel'de aynı hata, vektör numarası ve hata koduyla birlikte görünür.

**Kernel her adres alanına haritalanır.** Kernel genellikle sanal adres alanının üst yarısına (higher half) yerleştirilir ve tüm process'lerin sayfa tablolarında aynı yerde görünür; kullanıcı girdilerinden farkı U/S bayrağının sıfır olmasıdır. Bunun nedeni maliyettir: kernel her adres alanında zaten haritalı olduğu için system call girişinde CR3 değiştirmek ve TLB'yi (Translation Lookaside Buffer) boşaltmak gerekmez. Meltdown sınıfı yan kanal açıkları bu tercihi kısmen geri aldı; KPTI (Kernel Page Table Isolation) uygulayan kernel'ler kullanıcı sayfa tablosunda kernel'in yalnızca giriş kodunu bırakır ve girişte adres alanını değiştirir. Yeni bir kernel'de basit haritalamayla başlayıp ayrımı sonradan eklemek makul bir sıradır.

**Panic yolu bağımsız olmalıdır.** Panic mesajını yazan kod heap allocator'ına, scheduler'a ya da sürücü altyapısına bağımlıysa, tam da bu bileşenler bozulduğunda çalışmaz. Panic çıktısı, doğrudan seri porta ya da framebuffer'a yazan, ayrı ve mümkün olduğunca az bağımlılıklı bir yol olarak tutulur.

**SMP kararı erken verilir.** Tek işlemci varsayımıyla yazılmış veri yapılarını sonradan çok işlemciye açmak, hemen her paylaşılan yapıya dokunmayı gerektirir. Başlangıçta tek işlemcide çalışmak makul bir tercihtir; paylaşılan yapıların erişim noktalarını baştan tek bir yerde toplamak bu geçişi ucuzlatır.

**Neyin kernel'de olacağı ayrı bir karardır.** Sürücülerin ve dosya sistemlerinin kernel'de mi userspace'te mi çalışacağı, bu notun tanımladığı ayrıcalık sınırını değiştirmez; yalnızca sınırın hangi tarafında ne kadar kod olacağını belirler. Ödünleşim `04-kernel/kernel-mimarileri.md` notunda ele alınır.

## Mimariye göre farklar

| Mimari | Kernel'in seviyesi | Kernel'e giriş | Vektör tablosu | Sayfa tablosu kökü |
| --- | --- | --- | --- | --- |
| x86-64 | ring 0 | `syscall`, `int`, interrupt ve exception | IDT (IDTR), `syscall` için IA32_LSTAR | CR3 |
| ARM64 | EL1 | `svc`, interrupt ve exception | VBAR_EL1'in gösterdiği tablo | TTBR0_EL1 ve TTBR1_EL1 |
| RISC-V | S-mode | `ecall`, interrupt ve exception | stvec | satp |

Farklar isimlendirmenin ötesine geçer. ARM64'te kullanıcı ve kernel adres alanları iki ayrı taban register'ıyla yönetilir; higher half yerleşimi mimarinin doğal sonucudur. Ayrıca kernel'in altında hypervisor için EL2, firmware için EL3 seviyeleri bulunur. RISC-V'de kernel S-mode'da çalışır ve altında, M-mode'da, SBI (Supervisor Binary Interface) hizmetlerini sunan bir firmware katmanı vardır; x86-64'te doğrudan kernel'in yaptığı bazı işler burada firmware çağrısına dönüşür. SMAP'in karşılığı ARM64'te PAN (Privileged Access Never), RISC-V'de sstatus register'ındaki SUM (permit Supervisor User Memory access) bitidir.

## Sık yapılan hatalar

- Kernel ile işletim sistemini eşitlemek. Kabuk, servisler ve kütüphaneler kernel'in parçası değildir; bu ayrım kaybolduğunda "bunu kernel'de mi yapmalıyım" sorusu yanıtlanamaz hale gelir.
- Bootloader'ın kurduğu geçici GDT'ye, sayfa tablolarına ve boot bilgisi yapısına süresiz güvenmek.
- Exception handler'larını sonraya bırakmak. Handler yokken oluşan bir fault ikinci bir fault'u tetikler ve triple fault ile sessiz bir yeniden başlatmaya dönüşür; hatanın nedeni görünmez.
- Kullanıcı işaretçisini doğrulamadan dereference etmek. Erişim kernel'in yetkisiyle yapılır, dolayısıyla hatalı ya da kötü niyetli bir adres doğrudan kernel belleğine erişim demektir.
- Interrupt handler'ı içinde uzun iş yapmak ya da bloke olabilecek bir kilit almak.
- Kernel stack'inde büyük yerel tampon kullanmak. Guard page yoksa taşma hata vermez, komşu veriyi sessizce bozar.
- Host sisteminin derleyicisi ve başlık dosyalarıyla derlemek. Host toolchain'i altta bir işletim sistemi ve çalışan bir libc olduğunu varsayar.

## İlgili notlar

- [04 — Kernel](README.md) — bölümün diğer konuları
- [OSDev nedir](../00-giris/osdev-nedir.md) — alanın kapsamı ve katmanlar
- [01 — Mimari](../01-mimari/) — ayrıcalık seviyeleri ve işlemci modeli
- [02 — Boot](../02-boot/) — kernel'in devraldığı makine durumu
- [05 — Memory](../05-memory/) — paging ve adres alanları
- [06 — Process](../06-process/) — scheduling ve context switch

## Kaynaklar

- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 2 — System Architecture Overview
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 5 — Protection (CPL, ayrıcalıklı komutlar, SMEP ve SMAP)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 6 — Interrupt and Exception Handling (IDT, TSS ve stack değişimi)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 2B, SYSCALL ve SYSRET komut tanımları (IA32_STAR, IA32_LSTAR, IA32_FMASK)
- Arm® Architecture Reference Manual for A-profile architecture, Bölüm D1 — AArch64 System Level Programmers' Model (exception seviyeleri, VBAR_EL1, PAN)
- The RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Bölüm 1.2 — Privilege Levels ve Bölüm 4 — Supervisor-Level ISA (stvec, satp, sstatus.SUM)
- RISC-V Supervisor Binary Interface Specification, Bölüm 1 — Introduction
- Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*, Bölüm 6 — Mechanism: Limited Direct Execution
- OSDev Wiki, "Kernel"; <https://wiki.osdev.org/Kernel> (erişim: 23 Ağustos 2026)
