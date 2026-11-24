# lrpm — silnik RPM dla LegendaryOS (bez dnf)

Wspólna biblioteka bit (H#) dla **lpm** (na urządzeniu) i **lepm** (przy budowie obrazów):

```
repos.hk ─▶ meta (repomd → primary.xml → index.tsv) ─┐
installed.xml ───────────────────────────────────────┼─▶ solve (satsolv/libsolv) ─▶ fetch (curl + sha256 + GPG)
compat.hk (atrapy, wykluczenia, hooki) ──────────────┘                                  │
                                                       extract (bsdtar) ◀───────────────┘ ─▶ sysusers ─▶ hooks ─▶ installed.xml
```

* **Zero dnf.** Wymagane narzędzia: `curl`, `gawk`, `bsdtar`, `zstd`/`gzip`/`xz`, `sha256sum`, `rpmkeys` (pakiet `rpm`; tylko gdy `gpgcheck => true`).
* **Biblioteki wyłącznie przez bit:** `use "bit -> libfasttree" from "ft"` i `use "bit -> satsolv" from "sv"` w `src/lib.h#`. Brak kopii cudzego kodu.
* **Problem „pakiety wymagają systemd”** rozwiązuje polityka `compat.hk` — zob. [docs/POLICY.md](docs/POLICY.md).
* Architektura i ograniczenia: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

API wysokiego poziomu: `txn::prepare(Request, download)`, `txn::install_into(root, Prepared, Request)`, `txn::sync`, `txn::search`.

**Status:** kod napisany bez dostępu do kompilatora H#, libsolv i Fedory — **niezweryfikowany**. Testy czystej logiki: `src/tests/pure_test.h#`.
