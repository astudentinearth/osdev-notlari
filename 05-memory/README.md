# 05 — Memory

Fiziksel bellekten sanal belleğe: bellek haritası, sayfa tabloları, page fault yönetimi, kernel heap'i ve bellek koruma mekanizmaları.

Bellek yönetimi, kernel'in en çok hataya açık ve en çok performans etkisi olan parçasıdır. Bu bölüm hem donanım tarafını hem de allocator tasarımını kapsar.

## Konular

| Konu | Dosya | Durum |
| --- | --- | --- |
| Fiziksel bellek ve bellek haritası | `fiziksel-bellek.md` | ⬜ |
| Fiziksel bellek yöneticisi (bitmap, buddy) | `fiziksel-bellek-yoneticisi.md` | ⬜ |
| Sanal bellek kavramı ve MMU | `sanal-bellek.md` | ⬜ |
| Paging: x86-64 dört seviyeli sayfa tabloları | `paging.md` | ⬜ |
| Sayfa tablosu girdileri ve bayrakları | `sayfa-tablosu-girdileri.md` | ⬜ |
| Page fault yönetimi | `page-fault.md` | ⬜ |
| TLB ve invalidation | `tlb.md` | ⬜ |
| Adres alanı düzeni ve higher half mapping | `adres-alani.md` | ⬜ |
| Kernel heap ve allocator tasarımı | `heap.md` | ⬜ |
| Slab allocator | `slab-allocator.md` | ⬜ |
| Demand paging ve copy-on-write | `demand-paging.md` | ⬜ |
| Bellek eşleme: mmap ve VMA'lar | `mmap.md` | ⬜ |
| Swap ve sayfa değiştirme algoritmaları | `swap.md` | ⬜ |
| Bellek koruma: NX, SMEP, SMAP, KASLR | `bellek-korumasi.md` | ⬜ |
| Bellek hataları ve ayıklama teknikleri | `bellek-hatalari.md` | ⬜ |

---

**Durum işaretleri:** ⬜ yazılmadı · 🟡 yazılıyor · ✅ yayında

Bu bölüme katkıda bulunmadan önce [CONTRIBUTING.md](../CONTRIBUTING.md) ve [SABLON.md](../SABLON.md) dosyalarını okuyun. Listede olmayan bir konu önermek için önce issue açın.
