# Solver (satsolv / libsolv) — lrpm 0.2.0

lrpm używa prawdziwego solvera SAT (`use "bit -> satsolv" from "sv"`), nie heurystyki. Żądanie (`solve::resolve_req`):

| pole | znaczenie |
|---|---|
| `want` | instaluj: `nazwa`, `nazwa>=1.2`, `nazwa=1.2-3`, `glob*`, `provides:/usr/bin/x` |
| `erase` | usuń (te same specyfikacje) |
| `update` | aktualizuj wskazane |
| `mode` | `update-all`, `dist-upgrade`, `verify` |
| `opts` | `best no-weak cleandeps allow-downgrade allow-archchange allow-vendorchange allow-uninstall keep-orphans focus-installed strict-priority install-also-updates lock=N favor=N disfavor=N multiversion=N` |

Wynik: kroki do pobrania (`steps`), `removals` (NEVRA do usunięcia), `actions` (install/upgrade/downgrade/reinstall/change/obsoletes/erase),
`warnings` (specyfikacja bez dopasowania) albo `problems` (opis konfliktu z libsolv).
Atrapy kompatybilności (`legendaryos-compat-*`, np. `systemd`) są odfiltrowane; wykluczenia z `compat.hk` działają jako `erase`.
Wszystkie specyfikacje i opcje są walidowane przed solverem (nic nie trafia do powłoki).

Interfejs płaski (typy proste): `rp::txn::flat_solve(...)` → wiersze `E/P/W/A/R/S/B`; `rp::txn::check_request` — walidacja bez sieci.

Testy: `LEPM=/ścieżka/lepm bash scripts/solver-e2e.sh` (sztuczne repo file://; 14 scenariuszy: zależności, wersje, globy, provides,
konflikty, update-all, obsoletes, cleandeps, wstrzykiwanie).

CLI: `lepm solve …` (pakiety/rootfs), `lpm solve …` (żywy system, względem installed.xml; tylko plan).
Wymaga: gawk, curl, bsdtar (lrpm), statyczne libsolv (`satsolv-main/scripts/bit-native.sh`).
