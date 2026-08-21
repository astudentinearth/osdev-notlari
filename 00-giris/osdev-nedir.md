# OSDev nedir

OSDev (operating system development), donanımın üzerinde çalışan ve donanımı diğer programlar adına yöneten yazılımı yazma işidir. Uygulama geliştirmede altta hazır bir işletim sistemi bulunur; bellek, aygıtlar ve dosyalar onun sunduğu arayüzden görünür. OSDev'de o katmanın kendisi yazılır ve altta yalnızca işlemci, bellek ve aygıtlar kalır.

Alan tek bir işten oluşmaz. Firmware'in kernel'i belleğe yüklemesinden, kernel'in interrupt'ları karşılamasına, sürücülerin aygıt register'larını sürmesine ve userspace'in system call'larla hizmet istemesine kadar uzanan bir katman yığını vardır. Bu not, bu katmanların ne olduğunu, hangi problemden doğduklarını ve OSDev'in nerede başlayıp nerede bittiğini tanımlar.

## Problem

Bir işletim sistemi olmasaydı, donanımı kullanan her program donanımın tamamını kendisi sürmek zorunda kalırdı. Bu yalnızca kurgusal bir durum değildir; ilk bilgisayarlarda çalışma biçimi buydu. Program makineye tek başına sahip olur, işini bitirir ve yerini bir sonrakine bırakırdı.

Bu modelin üç somut sorunu vardır:

- **Her program donanımı bilmek zorundadır.** Diskten veri okumak için denetleyicinin register düzenini, ağ üzerinden veri göndermek için kartın model farklarını programın kendisi bilir. Donanım değiştiğinde program değişir.
- **Kaynaklar paylaştırılamaz.** İşlemciyi iki program arasında bölüştürecek bir bileşen yoktur. Bir program sonsuz döngüye girdiğinde makineyi kimse geri alamaz.
- **Koruma yoktur.** Bir programın yazdığı hatalı adres, başka bir programın verisini bozar. Hata, oluştuğu yerde değil çok sonra fark edilir.

İşletim sistemi bu üç soruna karşılık üç iş yapar: donanımı ortak bir arayüzün arkasına alır (soyutlama), sınırlı kaynağı birden çok program arasında bölüştürür (paylaştırma) ve programları hem birbirinden hem kendisinden ayırır (koruma).

Bu işlerin hiçbiri yalnızca yazılımla yapılamaz. Kernel'in diğer programlar üzerindeki yetkisi donanımdan gelir. İşlemci en az iki ayrıcalık seviyesi sunar: x86-64'te ring 0 ve ring 3, ARM64'te EL1 ve EL0, RISC-V'de S-mode ve U-mode. Adres alanlarını birbirinden ayıran MMU (Memory Management Unit), çalışan programdan denetimi geri alan timer interrupt ve ayrıcalıklı komutları kullanıcı modunda engelleyen kontroller, işletim sisteminin dayandığı donanım desteğidir.

## İşletim sisteminin sorumlulukları

Bir işletim sisteminin üstlendiği işler kabaca şu başlıklara ayrılır:

| Sorumluluk | Ne yapar | Bölüm |
| --- | --- | --- |
| İşlemcinin paylaştırılması | process ve thread'leri sıraya koyar, context switch yapar | [06-process](../06-process/) |
| Bellek yönetimi | fiziksel belleği dağıtır, sanal adres alanlarını kurar | [05-memory](../05-memory/) |
| Aygıtların sürülmesi | sürücüler aracılığıyla donanımı sürer, interrupt'ları karşılar | [07-drivers](../07-drivers/) |
| Kalıcı depolama | blok aygıtlarını dosya ve dizin kavramına dönüştürür | [08-filesystems](../08-filesystems/) |
| Program çalıştırma ve arayüz | executable format'ını yükler, system call arayüzünü sunar | [09-userspace](../09-userspace/) |
| Koruma | kullanıcı modu ile kernel modu arasındaki sınırı korur | [04-kernel](../04-kernel/) |

Bu sorumlulukların hepsinin kernel'in içinde olması zorunlu değildir. Sürücülerin ve dosya sistemlerinin kernel'de mi yoksa userspace'te mi çalışacağı bir tasarım kararıdır; kernel mimarisini belirleyen ayrım da budur.

## OSDev neyi kapsar

OSDev'in kapsamı, makineye enerji verildiği andan bir kullanıcı programının çalıştığı ana kadar uzanan katmanların tamamıdır:

- **Firmware ve boot.** BIOS (Basic Input/Output System) veya UEFI (Unified Extensible Firmware Interface) makineyi ayağa kaldırır, bootloader kernel'i belleğe yükler ve denetimi ona devreder. Kernel'in bellek haritası, çalışma modu ve ilk stack düzeni bu aşamada belirlenir. Multiboot2 gibi boot protokolleri bu devir teslimi standartlaştırır.
- **Kernel.** Tanımlayıcı tablolar, interrupt ve exception yönetimi, paging, process yönetimi ve system call mekanizması. Kernel'in donanımla kurduğu sözleşme burada yazılır.
- **Sürücüler.** Aygıtın kendi arayüzünü kernel'in beklediği ortak arayüze çeviren kod. Veri yolu numaralandırması, MMIO veya port I/O erişimi ve interrupt işleme bu katmanın işidir.
- **Userspace.** Program yükleme, libc, system call sarmalayıcıları ve kabuk. Bir sistemin kullanılabilir hale geldiği katman burasıdır.
- **Toolchain ve test ortamı.** Kernel, üzerinde geliştirme yapılan sistemin derleyicisiyle derlenmez. Hedef mimariye uygun bir cross-compiler, linker script ve freestanding C ortamı gerekir. Emülatör ve debugger da bu katmanın parçasıdır.

Bu katmanların hepsini aynı anda yazmak gerekmez. Çoğu proje boot eden ve çıktı verebilen küçük bir kernel ile başlar, sonra interrupt'ları, ardından bellek yönetimini ekler. Kapsamın tamamını görmek yine de gereklidir: erken verilen bir karar, çoğu zaman çok sonra yazılacak katmanı bağlar.

## OSDev neyi kapsamaz

Aşağıdaki işler işletim sistemlerine yakın olsa da bu notların anlamıyla OSDev değildir:

- Mevcut bir işletim sistemi üzerinde uygulama veya sistem programı yazmak. Burada kernel hâlâ altta durur.
- Hazır bir kernel'in etrafında dağıtım hazırlamak. Bu paketleme ve sistem yönetimi işidir.
- Mevcut bir işletim sistemini yapılandırmak, derlemek veya kurmak.

Mevcut bir kernel'e sürücü ya da modül yazmak komşu bir alandır: hedef farklıdır, ancak gereken donanım bilgisi büyük ölçüde ortaktır.

## Kernel tasarım yaklaşımları

Sorumlulukların nasıl bölüştürüleceği kernel mimarisini belirler:

- **Monolitik kernel.** Sürücüler, dosya sistemleri ve bellek yönetimi aynı ayrıcalıklı adres alanında çalışır. Bileşenler arası çağrı ucuzdur; buna karşılık bir sürücüdeki hata tüm sistemi düşürebilir.
- **Mikrokernel.** Kernel yalnızca adres alanı, thread ve mesajlaşma gibi en temel işleri üstlenir. Sürücüler ve dosya sistemleri userspace'te çalışır. Hata izolasyonu güçlüdür; bunun bedeli bileşenler arası her isteğin mesajlaşma maliyeti taşımasıdır.
- **Hibrit yaklaşımlar.** İki uç arasında, hangi bileşenin nerede çalışacağını ayrı ayrı seçen tasarımlar.

Bu ayrım, sonraki bölümlerde tekrar karşılaşılacak bir ödünleşimin ilk örneğidir: performans ile izolasyon çoğu zaman aynı anda en üst düzeyde tutulamaz. Ayrıntı `04-kernel/kernel-mimarileri.md` notunda ele alınır.

## OSDev'i uygulama geliştirmeden ayıran şey

Kernel yazarken altta hatayı yakalayacak bir katman yoktur. Bunun günlük çalışmaya birkaç somut yansıması olur:

- Standart kütüphane yoktur. `printf`, `malloc` ve dosya erişimi, kullanılabilmeleri için önce yazılır.
- Hatalar sessizdir. Yanlış kurulmuş bir tanımlayıcı tablo, hata mesajı yerine makinenin yeniden başlamasıyla sonuçlanır.
- Doğruluk ölçütü dokümantasyondur. Bir davranışın doğru olup olmadığı deneyerek değil, mimari manual'ından doğrulanarak bilinir.
- Test yöntemi farklıdır. Geliştirmenin büyük kısmı emülatör üzerinde ve debugger bağlıyken yapılır; gerçek donanım ayrı bir doğrulama adımıdır.

## Sık yapılan hatalar

- OSDev'i yalnızca kernel yazmaya indirgemek. Boot katmanı ve toolchain atlandığında ilk somut sorun genellikle bu ikisinden çıkar.
- Kernel'i geliştirme yapılan sistemin derleyicisiyle derlemek. Host derleyicisi, altında bir işletim sistemi olduğunu varsayar.
- Emülatörde çalışan kodu doğrulanmış saymak. Emülatörler bazı hataları bağışlar, gerçek donanım bağışlamaz.
- Mimari manual yerine forum ve wiki bilgisine dayanmak. İkincil kaynaklar başlangıç noktasıdır, ölçüt değildir.
- Tek bir mimaride görülen davranışı genel kural saymak. Ayrıcalık seviyeleri, interrupt yönetimi ve bellek modeli mimariden mimariye değişir.

## İlgili notlar

- [00 — Giriş](README.md) — bölümün diğer konuları
- [01 — Mimari](../01-mimari/) — işlemci modeli ve ayrıcalık seviyeleri
- [02 — Boot](../02-boot/) — firmware, bootloader ve boot protokolleri
- [04 — Kernel](../04-kernel/) — kernel'in donanımla kurduğu sözleşme

## Kaynaklar

- Intel® 64 and IA-32 Architectures Software Developer's Manual, Cilt 3A, Bölüm 5 (Protection) ve Bölüm 6 (Interrupt and Exception Handling)
- Arm® Architecture Reference Manual for A-profile architecture, Bölüm D1 — AArch64 System Level Programmers' Model
- The RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Bölüm 1.2 — Privilege Levels
- Unified Extensible Firmware Interface (UEFI) Specification, Sürüm 2.10, Bölüm 2 — Overview; <https://uefi.org/specifications> (erişim: 21 Ağustos 2026)
- The Multiboot2 Specification, Sürüm 2.0, Bölüm 3 — Boot information format
- Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*, Bölüm 2 — Introduction to Operating Systems
- OSDev Wiki, "Getting Started"; <https://wiki.osdev.org/Getting_Started> (erişim: 21 Ağustos 2026)
