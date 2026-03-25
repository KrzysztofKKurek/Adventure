# GraBarda — Prototyp Dungeon RPG Deckbuilder
**Data:** 2026-03-25
**Status:** Zatwierdzony do implementacji

---

## 1. Cel i zakres

Prototyp przeglądarkowej turowej gry RPG opartej na mechanice budowania talii. Pojedynczy bohater sterowany przez gracza eksploruje ręcznie zaprojektowane podziemia, walczy z przeciwnikami kartami, zdobywa skarby i relikty, unika pułapek oraz rozwija talię przez cały run.

**Prototyp:** 2 piętra × ~6 pokoi + boss. Czas jednego runu: ~20–30 minut.
**Docelowa platforma po prototypie:** Unity lub Unreal Engine (fotorealistyczna planszówka).

---

## 2. Stos technologiczny

- **React + TypeScript + Vite** — warstwa UI i zarządzanie stanem
- **Logika gry w czystym TypeScript** — klasy `Card`, `Hero`, `Enemy`, `Room`, `Floor`, `GameEngine` niezależne od UI, gotowe do przepisania na C# (Unity)
- **Styl wizualny prototypu:** Clean UI / board-game look (ikony + typografia, bez ilustracji)

---

## 3. Pętla rozgrywki

```
START RUNU (startowa talia + bohater)
  ↓
MAPA PIĘTRA — gracz wybiera pokój do odwiedzenia
  ↓
  ├─ ⚔  WALKA       → nagroda (karta lub relic)
  ├─ 💰 SKARB       → relic lub rzadka karta (bez wyboru)
  ├─ 🔥 ODPOCZYNEK  → lecz HP albo ulepsz kartę + odnów karty 🔥
  └─ 📜 WYDARZENIE  → narracyjny dylemat (ryzyko vs nagroda)
  ↓
BOSS — koniec piętra → następne piętro
  ↓
WYGRANA (pokonano oba piętra) lub PRZEGRANA (HP = 0 → restart)
```

Gracz nie może cofać się do odwiedzonych pokoi. Każde piętro ma jedno wejście i jeden boss-room na końcu.

---

## 4. Mapa podziemi

- Struktura **grafu skierowanego** — pokoje jako węzły, korytarze jako krawędzie
- Gracz widzi typy wszystkich pokoi z góry (brak fog-of-war na typy)
- Dostępne do wejścia są tylko pokoje **bezpośrednio sąsiadujące** z aktualną pozycją
- Mapa ręcznie zaprojektowana w danych (JSON); każde piętro ma własny plik konfiguracji
- Wizualnie: graf renderowany jako SVG lub siatka z ikonami pokoi

### Typy pokoi

| Typ | Ikona | Opis |
|---|---|---|
| Walka | ⚔ | Turowa walka kartami z 1–2 przeciwnikami |
| Skarb | 💰 | Relic lub rzadka karta — bez wyboru, bierzesz co jest |
| Odpoczynek | 🔥 | Wybór: lecz 30% max HP **albo** ulepsz 1 kartę; odnawia karty 🔥 |
| Wydarzenie | 📜 | Narracyjny dylemat z wyborami (np. HP za kartę, klątwa za relic, pułapka do uniknięcia) |
| Boss | 👹 | Silniejszy przeciwnik, nagroda: relic + przejście na kolejne piętro |

---

## 5. System walki

### Przebieg tury

1. **Tura gracza:** dobierz 5 kart z talii; zagraj dowolną liczbę kart w dowolnej kolejności; kliknij "Koniec tury"
2. **Tura przeciwnika:** wykonuje zapowiedziany zamiar
3. Powtarzaj do śmierci jednej ze stron

### Trzy typy kart (trwałość)

| Typ | Oznaczenie | Zachowanie po zagraniu |
|---|---|---|
| Zwykła | szara ramka ↩ | Trafia na stos odrzuconych; wraca do talii po jej wyczerpaniu |
| Odpoczynkowa | pomarańczowa 🔥 | Trafia na stos wyczerpanych; wraca tylko po wizycie w pokoju Odpoczynku |
| Jednorazowa | fioletowa 💀 | Usuwana z talii na zawsze |

### Wyczerpanie talii

Gdy gracz musi dobrać kartę a talia jest pusta: stos odrzuconych tasuje się i staje się nową talią. Stos wyczerpanych (karty 🔥) **nie** wchodzi do przetasowania — te karty wracają dopiero po odpoczynku.

### Blok i obrażenia

- **Blok** resetuje się na początku każdej tury gracza (nie przenosi się między turami)
- Obrażenia redukowane najpierw przez blok, potem przez HP
- Statusy: Trucizna, Podpalenie, Spowolnienie, Osłabienie (implementowane jako stackowalne efekty)

### Zamiar przeciwnika

Każdy przeciwnik pokazuje swój zamiar na kolejną turę (ikona + wartość), np. "⚔ Atak 12" lub "🛡 Blokuje się". Gracz może się przygotować.

---

## 6. Talia kart

### Startowa talia bohatera (10 kart)

| Karta | Ilość | Typ | Efekt |
|---|---|---|---|
| Cios | 5 | Zwykła | Zadaj 6 obrażeń |
| Blok | 3 | Zwykła | Zyskaj 8 bloku |
| Drugi Oddech | 1 | Odpoczynkowa 🔥 | Dobierz +2 karty |
| Intuicja | 1 | Zwykła | Dobierz 1 kartę, odrzuć 1 kartę |

### Kategorie kart w puli nagród

- **Atakujące** — obrażenia, efekty statusu na wrogu
- **Obronne** — blok, leczenie, tarczowanie
- **Umiejętności** — dobieranie kart, manipulacja talią, buffy
- **Magiczne** — silne efekty obszarowe lub wielokrotne uderzenia
- **Karty umiejętności bohatera** — specjalne akcje (złota ramka 🌟)

---

## 7. Bohater i relikty

### Bohater

- Jeden bohater (typ: Łowca Podziemi) na potrzeby prototypu
- HP nie resetuje się między walkami — każde HP jest cenne
- Brak tradycyjnych poziomów; rozwój tylko przez karty i relikty

### Relikty

Pasywne przedmioty modyfikujące zasady globalnie. Zdobywane ze skarbów i bossów.

Przykłady reliktów na prototyp:

| Relic | Efekt |
|---|---|
| Kryształ Ostrości | Pierwsza karta w turze zadaje +3 dmg |
| Kość Losu | Na początku walki dobierz 1 kartę z wyczerpanych |
| Amulet Krwi | Każde zabójstwo leczy 3 HP |
| Starożytna Tarcza | Na początku każdej walki zyskaj 4 bloku |

---

## 8. Ulepszanie kart

W pokoju Odpoczynku gracz może wybrać 1 kartę z talii do ulepszenia. Ulepszenie wzmacnia efekt (np. Cios → Cios+ zadaje 9 dmg zamiast 6). Każda karta może być ulepszona tylko raz.

---

## 9. Architektura techniczna

### Struktura katalogów

```
src/
  engine/          # Czysta logika gry (bez React)
    GameEngine.ts  # Główna klasa zarządzająca stanem gry
    Card.ts        # Model karty
    Hero.ts        # Model bohatera
    Enemy.ts       # Model przeciwnika + AI zamiaru
    Room.ts        # Model pokoju
    Floor.ts       # Model piętra (graf pokoi)
    Combat.ts      # Silnik walki
    effects/       # StatusEffect, Relic, CardEffect
  ui/              # Komponenty React
    screens/       # CombatScreen, MapScreen, RewardScreen, EventScreen, RestScreen
    components/    # Card, HeroPanel, EnemyPanel, RoomNode, RelicIcon
  data/            # JSON: karty, relikty, piętra, przeciwnicy, eventy
  App.tsx          # Router stanów gry (mapa / walka / nagroda / event / rest)
```

### Zarządzanie stanem

- Stan gry (`GameState`) w `useReducer` lub Zustand na poziomie `App`
- `GameEngine` to pure TS — przyjmuje stan, zwraca nowy stan
- Komponenty React tylko wyświetlają stan i emitują akcje

### Przepływ danych

```
User Action (klik karty)
  → React dispatch(action)
  → GameEngine.processAction(state, action) → newState
  → React rerenders UI
```

---

## 10. Dane i konfiguracja

Piętra, pokoje, karty, relikty i eventy zdefiniowane w plikach JSON w `src/data/`. Pozwala to projektować zawartość bez modyfikowania kodu.

### Format piętra (przykład)

```json
{
  "id": "floor-1",
  "name": "Krypty Zapomnianego Króla",
  "rooms": [
    { "id": "start", "type": "start", "connections": ["room-a", "room-b"] },
    { "id": "room-a", "type": "combat", "enemyId": "skeleton-guard", "connections": ["room-c", "room-d"] },
    { "id": "room-b", "type": "event", "eventId": "mysterious-altar", "connections": ["room-d", "room-e"] },
    { "id": "room-c", "type": "rest", "connections": ["boss"] },
    { "id": "room-d", "type": "treasure", "relicId": "crystal-focus", "connections": ["boss"] },
    { "id": "room-e", "type": "combat", "enemyId": "tomb-rat-swarm", "connections": ["boss"] },
    { "id": "boss", "type": "boss", "enemyId": "bone-knight" }
  ]
}
```

---

## 11. Zakres prototypu (MVP)

### W zakresie

- 1 bohater (Łowca Podziemi)
- 2 piętra z ręcznie zaprojektowanymi mapami
- ~20 unikalnych kart (ulepszalne)
- ~8 reliktów
- ~3 typy przeciwników + 2 bossowie
- ~4 eventy narracyjne
- Wszystkie 5 typów pokoi w pełni działające
- Ekran przegranej i wygranej

### Poza zakresem prototypu

- Wiele klas bohaterów
- Proceduralnie generowane mapy
- Dźwięk i muzyka
- Animacje kart (poza prostym CSS)
- Osiągnięcia i trwały postęp między runami
- Tryb multiplayer
