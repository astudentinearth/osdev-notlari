# 09 — Userspace

Kernel'in üzerinde çalışan dünya: çalıştırılabilir dosya format'ları, program yükleme, libc, system call arayüzü ve ilk kullanıcı süreçleri.

Bir kernel'i "işletim sistemi" haline getiren şey büyük ölçüde bu bölümdür.

## Konular

| Konu | Dosya | Durum |
| --- | --- | --- |
| Kernel ve userspace ayrımı | `kernel-userspace-ayrimi.md` | ⬜ |
| ELF format'ı | `elf.md` | ⬜ |
| Diğer format'lar (PE, a.out, flat binary) | `diger-formatlar.md` | ⬜ |
| Program yükleyici (loader) | `loader.md` | ⬜ |
| Statik ve dinamik linkleme | `linkleme.md` | ⬜ |
| Dinamik linker, PLT ve GOT | `dinamik-linker.md` | ⬜ |
| System call arayüzü (userspace tarafı) | `syscall-arayuzu.md` | ⬜ |
| libc nedir, hangi seçenekler var | `libc.md` | ⬜ |
| Kendi mini libc'ni yazmak | `mini-libc.md` | ⬜ |
| POSIX ve uyumluluk | `posix.md` | ⬜ |
| init süreci | `init.md` | ⬜ |
| Shell yazımı | `shell.md` | ⬜ |
| Toolchain'i kendi işletim sistemine port etmek | `toolchain-port.md` | ⬜ |

---

**Durum işaretleri:** ⬜ yazılmadı · 🟡 yazılıyor · ✅ yayında

Bu bölüme katkıda bulunmadan önce [CONTRIBUTING.md](../CONTRIBUTING.md) ve [SABLON.md](../SABLON.md) dosyalarını okuyun. Listede olmayan bir konu önermek için önce issue açın.
