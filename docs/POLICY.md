# compat.hk — polityka zgodności (pakiety Fedory ↔ system bez systemd)

## Problem
Wiele pakietów Fedory ma twarde `Requires: systemd` (często `Requires(post)` z makr `%systemd_post`). LegendaryOS
używa **zenitd** jako PID 1, więc pakiet `systemd` (PID 1, journald, logind…) nie może trafić do obrazu — ale solver
zależności musi uznać wymaganie za spełnione. `dnf` tego nie potrafi; lpm/lepm — tak.

## Rozwiązanie: trzy mechanizmy w jednym pliku `compat.hk`
| sekcja | działanie | po co |
|---|---|---|
| `[stub]` | generuje pakiet-atrapę `legendaryos-compat-<nazwa>` z `Provides: <nazwa> = <wersja>`, ładowany do solvera jak repo rpm-md. **Nie ma plików**, nie jest pobierany ani rozpakowywany. | spełnić `Requires: systemd`, w tym wersjonowane (`systemd = %{version}-%{release}`) |
| `[exclude]` | zadanie `erase` dla niezainstalowanego pakietu = zakaz instalacji (libsolv) | żeby `systemd` nigdy nie wszedł jako zależność |
| `[hooks]` | polecenia po rozpakowaniu (`ldconfig`, `update-ca-trust`, `depmod`…) | zamiennik skryptletów RPM (nie są uruchamiane) |

`version => from-repo:systemd` ustawia wersję atrapy na **aktualną wersję pakietu `systemd` z repozytoriów**
(`sort -V`), więc wymagania typu `systemd = <wersja>` są spełnione niezależnie od aktualizacji Fedory.

## Czego to NIE rozwiązuje
* Pakiety **linkujące** libsystemd (dbus, NetworkManager, polkit) potrzebują prawdziwego `systemd-libs` — zostaje w obrazie (to biblioteka, nie init).
* **logind**: SDDM/Wayland/Blue Environment potrzebują sesji logind. Plan: **elogind** z Copr LegendaryOS albo — gdy Copr
  nie zadziała — z **własnych źródeł** (budowane w obrazie, `/tools`); w obu przypadkach repo Copr dopisujemy do `repos.hk`,
  a atrapa `[stub]` może zapewnić `Provides: systemd-logind`/`logind`. Do czasu zbudowania elogind — ryzyko #1 (ROADMAP M1).
* Skryptlety RPM (`%pre/%post/%trigger*`) nie są wykonywane; skutki odtwarzają `sysusers.h#` (użytkownicy/grupy) i hooki.
  `tmpfiles.d`, `alternatives`, `authselect` — luki do zamknięcia (ROADMAP M2).

## Format
```
[stub]
-> systemd
--> provides => [systemd, systemd-units, systemd-sysv]
--> version  => from-repo:systemd
[exclude]
-> packages => [systemd, systemd-resolved, systemd-oomd-defaults]
[hooks]
-> run => [ldconfig, update-ca-trust, "depmod -a"]
[solver]
-> weak-deps => false
```
Nazwy są walidowane (`pure::safe_name`) — trafiają do poleceń i XML.

## Priorytety repozytoriów
satsolv przy równym priorytecie wybiera **najwyższą wersję ze wszystkich repo** (np. `updates` wygrywa z `fedora`).
Pole priorytetu per repo nie jest jeszcze obsługiwane (wymaga `i32` w `repo::set_priority` — ROADMAP).
