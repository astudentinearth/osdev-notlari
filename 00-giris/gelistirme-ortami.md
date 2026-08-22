# Geliştirme ortamı kurulumu

Kernel geliştirmede kullanılan araçların neredeyse tamamı POSIX uyumlu bir ortam varsayar. GNU binutils, GCC, GNU Make, QEMU ve GRUB'un imaj araçları bu ortamda geliştirilir, belgelenir ve test edilir. Bulacağınız her örnek Makefile, her derleme betiği ve her hata ayıklama tarifi bir kabuk komutuyla yazılmıştır. Bu yüzden geliştirme ortamı sorusu pratikte tek bir soruya iner: elinizin altında çalışan bir Linux kabuğu var mı?

Bu not, Windows, macOS veya Linux üzerinde çalışan bir makineyi kernel geliştirmeye hazır hale getirmeyi kapsar: hangi kurulum biçiminin hangi durumda uygun olduğu, hangi paketlerin ne için gerektiği ve kurulumun doğru çalıştığının nasıl doğrulanacağı. Hedef mimariye özgü cross-compiler'ın hazırlanması ayrı bir konudur ve `cross-compiler.md` notunda ele alınır.

## Ön koşullar

- [OSDev nedir](osdev-nedir.md)
- [Nereden başlanır: yol haritası](yol-haritasi.md)

## Kapsam

Bu not x86-64 hedefini ve Debian/Ubuntu tabanlı bir dağıtımı esas alır. Paket adları dağıtımdan dağıtıma değişir; araçların kendisi ve kullanım biçimi değişmez.

## Ortam seçenekleri

| Seçenek | Uygun olduğu durum | Dikkat edilmesi gereken |
| --- | --- | --- |
| Doğrudan Linux | Makine zaten Linux çalıştırıyor | Gerçek donanımda test için USB'den açılış kolaydır |
| WSL2 (Windows) | Günlük sistemi Windows olanlar | Dosyalar Linux dosya sisteminde tutulmalı |
| Sanal makine | Windows veya macOS, izole ortam istenmesi | İç içe sanallaştırma kapalıysa QEMU hızlandırması çalışmaz |
| macOS | Mevcut makine macOS | Native format ELF değil Mach-O'dur, cross araçları zorunludur |
| Windows (MSYS2 / Cygwin) | Diğer seçenekler mümkün değilse | Yol, satır sonu ve eksik araç sorunları sık yaşanır |

Windows kullanıyorsanız WSL2 en az sürtünmeli seçenektir: gerçek bir Linux kernel'i üzerinde çalışır, dosya sistemi Windows tarafından da görülebilir ve ayrı bir sanal makine yönetmek gerekmez. macOS'ta Homebrew ile araçların çoğu kurulabilir, ancak sistemin nesne dosyası formatı Mach-O olduğu için ELF üreten bir cross-compiler ve GNU ld kurulumu baştan zorunludur.

## WSL2 kurulumu

Yönetici yetkisiyle açılmış bir PowerShell penceresinde:

```powershell
wsl --install -d Ubuntu-24.04
```

Bu komut WSL bileşenlerini, sanal makine platformunu ve belirtilen dağıtımı kurar. İşlemcinin sanallaştırma desteğinin firmware'de açık olması gerekir. Kurulumdan sonra:

```powershell
wsl --update
wsl --status
```

Kurulumun WSL2 sürümünü kullandığını doğrulayın; WSL1 gerçek bir Linux kernel'i çalıştırmaz ve bazı araçlarda beklenmedik davranışlara yol açar.

**Dosyaların nerede duracağı önemlidir.** Depoyu `/mnt/c/...` altında, yani Windows dosya sisteminde tutmak derleme süresini belirgin biçimde uzatır; bu yol Windows ile Linux arasında bir dosya sistemi köprüsü üzerinden geçer. Çalışma dizinini Linux tarafında (`~/projeler/...`) tutun. Windows tarafından erişmek gerekirse `\\wsl$\Ubuntu-24.04\home\...` yolu kullanılabilir.

**Satır sonları.** Windows tarafında klonlanmış bir depo CRLF satır sonlarıyla gelirse kabuk betikleri `bad interpreter` hatası verir, bağlayıcı betikleri ve assembly dosyaları beklenmedik biçimde ayrıştırılır. Bu depodaki `.gitattributes` dosyası `* text=auto eol=lf` kuralıyla çalışma kopyasını LF'de tutar. Kendi projenizde de aynı kuralı koyun veya klonlamayı WSL içinden yapın.

## Paketler

Debian ve Ubuntu üzerinde temel kurulum:

```sh
sudo apt update
sudo apt install -y build-essential nasm gdb git make \
  qemu-system-x86 xorriso mtools dosfstools \
  grub-pc-bin grub-common \
  bison flex texinfo libgmp-dev libmpc-dev libmpfr-dev
```

Paketlerin ne işe yaradığı:

| Paket | Ne için gerekir |
| --- | --- |
| `build-essential` | GCC, GNU Make ve binutils (`ld`, `objdump`, `readelf`, `nm`) |
| `nasm` | Intel söz dizimli assembly dosyaları |
| `gdb` | QEMU'ya bağlanarak kernel'i adım adım izlemek |
| `qemu-system-x86` | x86-64 hedefinin emülasyonu |
| `xorriso`, `mtools`, `dosfstools` | `grub-mkrescue` ile önyüklenebilir ISO ve FAT imajı üretmek |
| `grub-pc-bin`, `grub-common` | GRUB'un BIOS modülleri ve imaj araçları |
| `bison`, `flex`, `texinfo`, `libgmp-dev`, `libmpc-dev`, `libmpfr-dev` | GCC ve binutils'i kaynaktan derleyerek cross-compiler kurmak |

Son satırdaki paketler yalnızca cross-compiler'ı kaynaktan derleyecekseniz gerekir. Dağıtımın hazır cross paketlerini kullanacaksanız bu adım atlanabilir; seçenekler `cross-compiler.md` notunda karşılaştırılır.

Diğer dağıtımlarda paket adları farklıdır. Fedora'da `dnf install gcc make binutils nasm gdb qemu-system-x86 xorriso mtools grub2-tools-extra`, Arch Linux'ta `pacman -S base-devel nasm gdb qemu-system-x86 xorriso mtools grub` karşılığı kurulumu sağlar.

## Kurulumun doğrulanması

Önce araçların bulunduğunu doğrulayın:

```sh
gcc --version
ld --version
nasm -v
qemu-system-x86_64 --version
gdb --version
grub-mkrescue --version
```

Sürüm numaralarını görmek araçların kurulu olduğunu gösterir, birlikte çalıştıklarını göstermez. Bunun için uçtan uca küçük bir test yapmak gerekir. Aşağıdaki 512 baytlık boot sektörü, BIOS'un yüklediği ilk sektörden tek bir karakter yazar:

```asm
; NASM söz dizimi. BIOS bu sektörü 0x7C00 adresine yükleyip denetimi devreder.
bits 16
org 0x7c00

start:
    mov ah, 0x0e        ; BIOS teletype çıktısı
    mov al, 'T'
    int 0x10
bekle:
    hlt
    jmp bekle

times 510 - ($ - $$) db 0
dw 0xaa55               ; boot sektörü imzası
```

```sh
nasm -f bin deneme.asm -o deneme.img
qemu-system-x86_64 -drive format=raw,file=deneme.img
```

Açılan pencerede tek bir `T` harfi görünüyorsa assembler, imaj üretimi ve emülatör birlikte çalışıyor demektir. Örnek sadeleştirilmiştir: sayfa ve renk seçen `bh`/`bl` register'ları ayarlanmamıştır ve kod 16 bit gerçek modda çalışır. Kernel bu biçimde yazılmaz; bu yalnızca araç zincirini doğrulayan bir testtir.

WSL'de pencere WSLg üzerinden açılır. Grafik arayüzün bulunmadığı bir ortamda `-display curses` seçeneği metin modundaki ekranı terminale çizer.

## QEMU ve GDB'yi birlikte kullanmak

QEMU, GDB'nin bağlanabileceği bir sunucu açabilir. `-s` seçeneği 1234 numaralı porttan dinlemeyi, `-S` ise ilk komut çalıştırılmadan durmayı sağlar:

```sh
qemu-system-x86_64 -drive format=raw,file=deneme.img -s -S
```

Başka bir terminalde:

```sh
gdb -ex "target remote localhost:1234"
```

Kernel'i ELF olarak derlediğinizde sembol tablosunu `file kernel.elf` komutuyla yükleyerek fonksiyon adlarıyla çalışabilirsiniz. Ayrıntılar `gdb.md` notundadır.

**Donanım hızlandırması.** Linux üzerinde `/dev/kvm` erişimi olan bir kullanıcı QEMU'yu `-accel kvm` ile çalıştırarak belirgin hız kazanır; bunun için kullanıcının `kvm` grubunda olması gerekir. WSL2 ve sanal makine kurulumlarında iç içe sanallaştırma her yapılandırmada bulunmaz. Hızlandırma olmadan çalışan yazılım emülasyonu (TCG) geliştirme için yeterlidir, üstelik yorumlayıcı yürütme bazı hataları daha görünür kılar.

## Editör ve dil sunucusu

Kernel kaynağı, host sistemin başlıklarını kullanmadığı için editörün varsayılan yapılandırması çoğu satırı hatalı gösterir. clangd kullanılıyorsa derleme komutlarının `compile_commands.json` dosyasına aktarılması gerekir; `bear -- make` bu dosyayı mevcut Makefile'ı değiştirmeden üretir. Böylece `-ffreestanding`, `-nostdinc` ve hedef mimariye özgü seçenekler dil sunucusuna da bildirilmiş olur.

Windows tarafında bir editör kullanıp WSL içinde derliyorsanız, editörün uzak (remote) eklentisiyle Linux tarafına bağlanması derleyici, hata ayıklayıcı ve dosya yollarının tutarlı kalmasını sağlar.

## Git yapılandırması

```sh
git config --global user.name "Adınız"
git config --global user.email "eposta@ornek.com"
git config --global core.autocrlf false
```

`core.autocrlf` değerinin Linux tarafında `false` kalması gerekir. Windows kurulumlarında varsayılan `true` olabilir; bu ayar dosyaları çalışma kopyasında CRLF'e çevirerek yukarıda anlatılan sorunlara yol açar.

## Sık yapılan hatalar

- Depoyu WSL'de `/mnt/c` altında tutmak. Derleme belirgin biçimde yavaşlar.
- CRLF satır sonlarıyla çalışmak. Kabuk betikleri, bağlayıcı betikleri ve assembly dosyaları sessizce bozulur.
- Host derleyicisini kernel derlemek için kullanmak. Host GCC altında bir işletim sistemi olduğunu varsayar; `-ffreestanding` bu farkın yalnızca bir kısmını kapatır.
- `grub-mkrescue` çalıştırırken `xorriso` veya `mtools` kurmamış olmak. Hata mesajı çoğu zaman eksik paketi doğrudan söylemez.
- QEMU'yu Windows tarafından, imajı ise WSL içinden vererek çalıştırmak. İki taraf farklı yol biçimleri kullanır.
- Kurulumu doğrulamadan kernel yazmaya başlamak. İlk hatanın kaynağını kernel kodunda aramak zaman kaybettirir.
- Sürüm numaralarını görmeyi yeterli saymak. Araçların birlikte çalıştığı ancak uçtan uca bir testle anlaşılır.

## İlgili notlar

- [Ön gereksinimler](on-gereksinimler.md) — C, assembly ve araç zinciri bilgisi
- [Nereden başlanır: yol haritası](yol-haritasi.md) — Aşama 0 ve sonrası
- [02 — Boot](../02-boot/) — boot protokolleri ve imaj üretimi
- [Kaynaklar](../resources/) — araç ve dokümantasyon listeleri

## Kaynaklar

- Microsoft, "How to install Linux on Windows with WSL"; <https://learn.microsoft.com/en-us/windows/wsl/install> (erişim: 22 Ağustos 2026)
- Microsoft, "Comparing WSL Versions"; <https://learn.microsoft.com/en-us/windows/wsl/compare-versions> (erişim: 22 Ağustos 2026)
- QEMU dokümantasyonu, "QEMU System Emulation User's Guide" — Invocation ve Debugging; <https://www.qemu.org/docs/master/system/index.html> (erişim: 22 Ağustos 2026)
- NASM — The Netwide Assembler Manual, Bölüm 2 — Running NASM; <https://www.nasm.us/doc/> (erişim: 22 Ağustos 2026)
- GNU GRUB Manual, "Making a GRUB bootable CD-ROM"; <https://www.gnu.org/software/grub/manual/grub/grub.html> (erişim: 22 Ağustos 2026)
- GNU Make Manual, Bölüm 2 — An Introduction to Makefiles; <https://www.gnu.org/software/make/manual/make.html> (erişim: 22 Ağustos 2026)
- GDB dokümantasyonu, "Connecting to a Remote Target"; <https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html> (erişim: 22 Ağustos 2026)
- clangd dokümantasyonu, "JSON Compilation Database"; <https://clangd.llvm.org/design/compile-commands> (erişim: 22 Ağustos 2026)
- OSDev Wiki, "QEMU"; <https://wiki.osdev.org/QEMU> (erişim: 22 Ağustos 2026)
