# lrpm — architektura

| moduł | rola | zewnętrzne |
|---|---|---|
| `pure.h#` | czysta logika (wersje, URL, tokenizer sysusers, kolejność, XML atrap) — **testowalna bez bibliotek** | — |
| `conf.h#` | `repos.hk` → `[Repo]`, `$releasever/$basearch` | libfasttree `hk_*` |
| `policy.h#` | `compat.hk` → atrapy / wykluczenia / hooki | libfasttree `hk_*` |
| `meta.h#` | repomd → primary → `index.tsv` (gawk), `latest_version`, `search` | curl, gawk, zstd/gzip/xz, sha256sum |
| `instdb.h#` | `installed.xml` (bloki `<package>` z primary): wrap / extract / merge / remove | gawk |
| `solve.h#` | **satsolv**: pool, repo zainstalowane + dostępne + atrapy, zadania install/erase, plan → kroki z indeksu | libsolv (statycznie, przez bit) |
| `fetch.h#` | równoległy curl, sha256, `rpmkeys --checksig` | curl, rpmkeys |
| `extract.h#` | RPM → katalog | bsdtar (zapas: rpm2cpio+cpio) |
| `sysusers.h#` | minimalne sysusers.d → passwd/group/shadow/gshadow | — |
| `hooks.h#` | polecenia po rozpakowaniu (chroot lub żywy system) | chroot, mount |
| `txn.h#` | `prepare` / `install_into` / `sync` / `search` | — |

## Dlaczego `installed.xml` w formacie rpm-md
Solver musi znać `Provides/Requires` zainstalowanych pakietów (inaczej wszystko, co baza już ma, zostałoby dociągnięte
ponownie). Zamiast osobnego formatu kopiujemy bloki `<package>` z `primary.xml` i ładujemy je tym samym loaderem co repo.

## Znane ograniczenia (stan 0.1)
* `primary.xml` zawiera tylko podzbiór plików (`/etc`, `/usr/bin`, `/usr/sbin`…); `Requires: /ścieżka` spoza niego wymaga filelists (nie ładowane).
* Brak weryfikacji podpisu `repomd.xml` (łańcuch: HTTPS + sha256).
* Brak priorytetów repo, brak delta-RPM, brak `modular` (Fedora 43 — brak modułów w repo głównym).
* Kolejność rozpakowania: tylko `filesystem`, `setup` najpierw; brak pełnego porządku transakcji libsolv.
* Cały kod **niezweryfikowany** kompilatorem (sesja bez `h#`/`bit`/libsolv). Pierwszy krok: `bit build` + `h# test src/tests/pure_test.h#`.
