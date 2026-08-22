# Nereden başlanır: yol haritası

OSDev'e başlamak, hangi adımın hangisini gerektirdiğini bilmeden yola çıkmak demektir. Boot edemeyen bir kernel yazılamaz, interrupt kurmadan belleği yönetmek güçleşir, belleği yönetmeden process çalıştırılamaz. Bu not, o bağımlılık zincirini görünür kılar ve başlangıç noktasından çalışan bir sistem çekirdeğine giden makul bir sırayı ortaya koyar.

Bu bir müfredat değildir. Her proje farklı hedefler, farklı mimari kısıtlamalar ve farklı ilgi alanları taşır. Aşağıdaki sıra, gereksiz engellere takılmadan ilerleyebileceğiniz bir yol olarak okunmalıdır; zorunlu bir liste olarak değil.

## Ön koşullar

- [OSDev nedir](osdev-nedir.md)

## Başlamadan önce: hangi soruyu sorduğunuzu bilin

OSDev'e yaklaşmanın iki farklı biçimi vardır. Bunları karıştırmak, çoğu zaman neyi öğrendiğinizden emin olamadığınız belirsiz bir aşama yaratır.

**Anlamak için yazmak.** Bellek yönetiminin nasıl çalıştığını, interrupt'ların CPU'dan kernel'e nasıl ulaştığını, context switch'in gerçekte ne yaptığını kavramak istiyorsunuz. Bu durumda hedef çalışan bir sistem değil, anlaşılan bir mekanizmadır.

**Bir şey inşa etmek.** Belirli bir hedef var: bir oyun konsolu, bir RTOS (Real-Time Operating System), bir UNIX benzeri sistem veya sıfırdan yazan bir kernel. Bu durumda hedef bellidir ve öğrenme bu hedefe ulaşmanın aracıdır.

Bu ayrım, neyi ne zaman öğreneceğinizi etkiler. İlk yaklaşımda daha geniş okumak ve farklı mimari seçenekleri incelemek mantıklıdır. İkinci yaklaşımda dar bir hedef belirleyip ona odaklanmak ve yalnızca o yolda engel oluşturan konuları öğrenmek daha verimlidir.

## Aşama 0: Temel araçları hazırlamak

Kernel yazmaya başlamadan önce üç şeyi sağlam tutmak gerekir: derleyici, emülatör ve debug ortamı. Bu araçları sonradan kurmaya çalışmak, ilk somut sorun onlardan çıktığında anlamlı zaman kaybettirir.

**Cross-compiler.** Kernel, üzerinde geliştirme yapılan sistemin derleyicisiyle derlenmez. Host derleyici, altta bir işletim sistemi olduğunu varsayar ve freestanding (bağımsız) ortam için doğru çıktıyı üretmez. GCC veya Clang'ın hedef mimariye özgü bir cross-compiler kurulumu gerekir. Ayrıntı `cross-compiler.md` notundadır.

**QEMU.** Emülatör olmadan kernel geliştirmek mümkündür; ancak gerçek donanımda her değişiklikten sonra test etmek çok yavaştır. QEMU, boot süreci dahil pek çok x86-64 ve ARM64 özelliğini doğru biçimde emüle eder. Ayrıntı `qemu.md` notundadır.

**GDB ve remote debug.** QEMU, GDB ile bağlanmayı destekler. Bu bağlantı kurulduğunda kernel'in içinde breakpoint koyabilir, register'ların değerini okuyabilir ve belleği inceleyebilirsiniz. Debug çıktısını seri porta yazmak da erken aşamalarda işe yarar.

Bu üçü hazır olduğunda, yazdığınız her değişikliği hızla test edebilir ve nerede takıldığınızı görebilirsiniz.

## Aşama 1: Boot etmek

Kernel'in ilk işi, firmware'in denetimi devrettikten sonra bilinen bir duruma ulaşmaktır. Bu aşamada gerçekleşen işlerin büyük kısmı kernel kodu değil, boot protokolü ve linker script tarafından belirlenir.

**Boot protokolü seçimi.** Kernel'i bellekte nereye yükleyeceğini ve ona nasıl denetim devredeceğini belirlemek için bir protokol gerekir. x86-64 için iki yaygın seçenek vardır:

- **Multiboot2.** GRUB tarafından desteklenen, olgun bir protokol. Bellek haritası, RSDP adresi ve framebuffer bilgisi kernel'e yapılandırılmış olarak geçirilir.
- **Limine Boot Protocol.** Daha yeni, daha sade ve başlangıç dostu bir protokol. Higher-half kernel yüklemesi varsayılandır; bellek haritası ve diğer yapılar tutarlı bir arayüzle sunulur.

**Linker script.** Kernel'in hangi bölümünün bellekte nereye yerleşeceğini linker script belirler. Çalışma zamanında erişilecek adreslerin, yükleme sırasındaki adreslerden farklı olabileceğini anlamak bu aşamanın can alıcı noktasıdır.

**İlk çıktı.** Kernel boot ettiğinde bunu doğrulayabilmek gerekir. En erken aşamada seri porta karakter yazmak veya QEMU'nun `debugcon` aygıtını kullanmak işe yarar. Framebuffer hazırsa, belirli bir konuma piksel yazmak da bir doğrulama yöntemidir.

Bu aşamanın sonunda: kernel derlenir, boot eder ve gözlemlenebilir bir işaret verir.

## Aşama 2: Interrupt ve exception yönetimi

Boot eden bir kernel, CPU'nun ürettiği event'lere yanıt veremeden ciddi bir iş yapamaz. Hatalı bir bellek erişimi, sıfıra bölme ya da donanımdan gelen bir sinyal, kernel'in müdahale etmesi gereken durumlardır.

**Tanımlayıcı tablolar.** x86-64'te IDT (Interrupt Descriptor Table), interrupt ve exception geldiğinde çalışacak handler'ın adresini tutar. IDT'yi kurmak, her giriş için doğru tip ve ayrıcalık seviyesi bayraklarını ayarlamak ve handler'ları yazmak bu adımın temelidir. ARM64'te eşdeğer yapı exception vector tablosudur.

**Exception handler'ları.** CPU'nun ürettiği exception'lar arasında en sık karşılaşılanlar page fault, general protection fault ve double fault'tur. Bu handler'ların çalışması, en azından neyin yanlış gittiğini ekrana yazdırabilmesi kernel geliştirmenin geri kalanını önemli ölçüde kolaylaştırır.

**Timer interrupt.** Zamanlama yapmak, bir süre sonra process çalıştırmak ve kernel'in periyodik işlerini yürütmek için timer interrupt gerekir. x86-64'te PIT (Programmable Interval Timer) veya APIC timer bu amaçla kullanılır. Bu interrupt'ı almanın önemi yalnızca sayaç tutmak değildir; ileride scheduler'ın da dayandığı mekanizma budur.

Bu aşamanın sonunda: kernel exception'ları yakalar, bir hata oluştuğunda bilgi yazdırır ve periyodik timer interrupt alır.

## Aşama 3: Bellek yönetimi

Bellek yönetimi, kernel geliştirmenin teknik açıdan en ağır kısmıdır ve iki katmana ayrılır: fiziksel belleği izlemek ve sanal adresleri düzenlemek.

**Fiziksel bellek yöneticisi.** Boot sırasında firmware'den alınan bellek haritası, hangi aralıkların kullanılabilir olduğunu gösterir. Bu aralıkları izlemek ve talep üzerine page frame tahsis edip geri almak fiziksel bellek yöneticisinin işidir. Yaygın yaklaşımlar: bitmap allocator (her page frame için bir bit) ve free list (boş page frame'lerin bağlı listesi).

**Paging ve sanal adres alanı.** x86-64'te 4 seviyeli sayfa tablosu (PML4, PDPT, PD, PT), ARM64'te 4 veya 5 seviyeli tablo yapısı sanal adresleri fiziksel adreslere çevirir. Kernel'in kendi adres alanını kurmak, kendisini higher-half (üst yarı) adreslere map etmek ve TLB (Translation Lookaside Buffer) tutarlılığını yönetmek bu aşamanın içindedir.

**Kernel heap allocator.** Sanal bellek yönetimi kurulduktan sonra kernel'in kendi içinde dinamik bellek tahsis edebilmesi gerekir. Basit bir `kmalloc` / `kfree` çifti bu gereksinimi karşılar. İlk implementasyon slab allocator veya buddy allocator olabilir; önemli olan arayüzü doğru tanımlamaktır.

Bu aşamanın sonunda: kernel fiziksel belleği izler, kendi adres alanını kurar ve dinamik bellek tahsis edebilir.

## Aşama 4: Process yönetimi ve context switch

Birden fazla programın aynı makinede çalışır gibi görünmesi, kernel'in işlemciyi aralarında paylaştırmasıyla sağlanır. Bu, her process'in yürütme bağlamını (register değerleri, stack işaretçisi, program sayacı) saklamayı ve gerektiğinde geri yüklemeyi gerektirir.

**Process yapısı.** Kernel, her process için en azından şunları tutmalıdır: ayrıcalık seviyesi, sayfa tablosu tabanı, kernel stack'i ve yürütme bağlamı. Bu bilgilerin tutulduğu veri yapısına PCB (Process Control Block) denir.

**Context switch.** Bir process'ten diğerine geçiş, kayıt değerlerini saklamayı ve diğer process'in kayıtlarını yüklemeyi kapsar. Bu geçiş, doğrudan assembly ile yazılır çünkü derleyicinin alışkın olduğu çağrı kuralları burada geçersizdir.

**Scheduler.** Hangi process'in ne zaman çalışacağına karar veren bileşen scheduler'dır. Başlangıç için round-robin yeterlidir: her process sırayla belirli bir süre çalışır. Timer interrupt, context switch'i tetikleyen sinyaldir.

**Kullanıcı modu.** Process'in ring 3 (veya ARM64'te EL0) ayrıcalık seviyesinde çalışması, kernel'in ayrıcalıklı işlemlerden korunmasını sağlar. Bu geçiş, x86-64'te `iretq` komutuyla gerçekleştirilir.

Bu aşamanın sonunda: kernel birden fazla process'i sıraya koyar, aralarında geçiş yapar ve process'ler kullanıcı modunda çalışır.

## Aşama 5: System call arayüzü

Kullanıcı modunda çalışan bir program, donanıma ve kernel kaynaklarına doğrudan erişemez. Kernel'den hizmet istemek için system call mekanizması kullanılır. x86-64'te `syscall` / `sysret` komutları bu geçişi sağlar; ARM64'te `svc` komutu kullanılır.

Minimal bir system call arayüzü şunları kapsar: çıkış (`exit`), bellek tahsisi (`brk` veya `mmap`), yazma (`write`) ve başka bir program çalıştırma (`exec`). Bu dörtle işlevsel bir userspace oluşturulabilir.

## Aşama 6: Dosya sistemi ve sürücüler

Bu aşamaya gelindiğinde çalışan bir kernel vardır. Bundan sonra hangi yöne gidileceği hedefe bağlıdır.

**Sürücüler.** Bir disk, ağ kartı veya klavyeyle iletişim kurmak için ilgili aygıtın register düzenini, interrupt yapısını ve veri yolu protokolünü (PCI, USB, MMIO) anlamak gerekir. Her sürücü nispeten bağımsız bir çalışmadır; birini bitirmek diğerini kolaylaştırır.

**Dosya sistemi.** Kalıcı veriye erişmek için bir dosya sistemi gerekir. Başlangıç için bir bellek içi (in-memory) dosya sistemi veya ext2 gibi sade bir disk formatı uygun başlangıç noktalarıdır.

**VFS (Virtual File System).** Farklı dosya sistemlerini ortak bir arayüzün arkasına almanın yolu bir soyutlama katmanı eklemektir. Bu katman olmadan her dosya sistemi değiştiğinde userspace arayüzü de değişmek zorunda kalır.

## Mimari seçimi hakkında

Bu notlardaki örnekler ağırlıklı olarak x86-64 üzerinden verilmiştir; ARM64 ve RISC-V farkları ilgili bölümlerde belirtilmiştir. Başlangıç mimarisi seçiminde şu değerlendirmeler yapılabilir:

| Mimari | Emülatör desteği | Donanım erişimi | Dokümantasyon |
| --- | --- | --- | --- |
| x86-64 | QEMU, Bochs | Yaygın ve ucuz | Intel SDM, AMD APM |
| ARM64 | QEMU | Raspberry Pi serisi | Arm ARM |
| RISC-V | QEMU | Sınırlı, artan | RISC-V spesifikasyonları |

x86-64, en geniş topluluk desteğine ve ikincil kaynak yoğunluğuna sahiptir. Ancak ISA'nın (Instruction Set Architecture) tarihi karmaşıklığı, bazı kavramları öğrenmeyi zorlaştırır. ARM64 ve RISC-V daha sade bir ayrıcalık modeli sunar.

## Sık yapılan hatalar

- Araçları hazırlamadan kod yazmaya başlamak. İlk bug, çoğu zaman kernel değil build ortamında çıkar.
- Aşamaları atlamak. Page fault handler'ı yazmadan bellek yönetimine geçmek, hatayı sessizce geçiren bir ortam yaratır.
- Emülatördeki davranışı kesin doğru saymak. QEMU bazı hataları bağışlar; aynı kod gerçek donanımda farklı davranabilir.
- İkincil kaynaklara dayanmak. Forum cevabı veya wiki maddesi başlangıç noktasıdır; mimari manual'ı ölçüttür.
- Aynı anda çok fazla şeyi değiştirmek. Küçük adımlarla ilerlemek ve her adımda doğrulama yapmak, neyin bozulduğunu bulmayı kolaylaştırır.

## İlgili notlar

- [OSDev nedir](osdev-nedir.md) — alanın kapsamı ve temel kavramlar
- [Ön gereksinimler](on-gereksinimler.md) — C, assembly ve bilgisayar mimarisi
- [Geliştirme ortamı kurulumu](gelistirme-ortami.md) — Linux / WSL kurulumu
- [Cross-compiler hazırlama](cross-compiler.md) — GCC ve Clang
- [QEMU ile emülasyon](qemu.md) — emülatör kullanımı
- [GDB ile kernel hata ayıklama](gdb.md) — remote debug kurulumu
- [02 — Boot](../02-boot/) — firmware, bootloader ve boot protokolleri
- [05 — Memory](../05-memory/) — fiziksel ve sanal bellek yönetimi
- [06 — Process](../06-process/) — process yönetimi ve scheduler

## Kaynaklar

- OSDev Wiki, "Getting Started"; <https://wiki.osdev.org/Getting_Started> (erişim: 22 Ağustos 2026)
- OSDev Wiki, "Beginner Mistakes"; <https://wiki.osdev.org/Beginner_Mistakes> (erişim: 22 Ağustos 2026)
- Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*, Bölüm 2 — Introduction to Operating Systems; <https://ostep.org> (erişim: 22 Ağustos 2026)
- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 6 (Interrupt and Exception Handling)
- Arm® Architecture Reference Manual for A-profile architecture, Bölüm D1 — AArch64 System Level Programmers' Model
- The RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Bölüm 1 — Introduction
- Limine Boot Protocol spesifikasyonu; <https://github.com/limine-bootloader/limine/blob/trunk/PROTOCOL.md> (erişim: 22 Ağustos 2026)
- The Multiboot2 Specification, Sürüm 2.0
