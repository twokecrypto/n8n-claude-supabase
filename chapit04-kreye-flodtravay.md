# Kominote TWOKECRYPTO
# Chapit 4: Kreye Flodtravay nan N8N

## Rezime Chapit

Nan chapit sa a, nou pral aprann:
- Diferan tip nod yo (deklanchè, aksyon, lojik)
- Kijan pou travay ak done nan N8N
- Lojik kondisyonèl (SI/ALÒS)
- Boukle ak iterasyon
- Jesyon erè
- Meyè pratik pou kreye flodtravay

---

## 4.1 Tip Nod yo an Detay

### 4.1.1 Nod Deklanchè

Nod deklanchè yo se pwen depa flodtravay ou a. Yo detèmine kilè flodtravay la pral egzekite.

```
┌─────────────────────────────────────────────────────────────────┐
│                   KATEGORI DEKLANCHÈ                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  DEKLANCHÈ TAN                                          │   │
│  │  ─────────────────                                      │   │
│  │  • Orè (Schedule) - Egzekite sou yon orè defini        │   │
│  │  • Cron - Ekspresyon cron pèsonalize                   │   │
│  │  • Entèval - Chak X minit/èdtan                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  DEKLANCHÈ EVÈNMAN                                      │   │
│  │  ──────────────────                                     │   │
│  │  • Webhook - Resevwa done nan yon URL                  │   │
│  │  • Imèl - Lè ou resevwa yon imèl                       │   │
│  │  • Fichye - Lè yon fichye chanje                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  DEKLANCHÈ APLIKASYON                                   │   │
│  │  ─────────────────────                                  │   │
│  │  • Slack - Nouvo mesaj                                 │   │
│  │  • GitHub - Nouvo angajman (commit)                    │   │
│  │  • Supabase - Chanjman nan baz done                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Egzanp: Deklanchè Webhook

Yon webhook se yon URL ki resevwa done lè yon evènman rive:

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Deklanchè Webhook                       [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Metòd HTTP: [POST ▼]                                          │
│                                                                 │
│  Chemen:     [/webhook-test/premye________________]            │
│                                                                 │
│  URL Konplè:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ https://votredomèn.com/webhook-test/premye              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Otentifikasyon: [Okenn ▼]                                     │
│                                                                 │
│  Opsyon:                                                        │
│  ☑ Reponn imedyatman                                           │
│  ☐ JSON pètèt nan kò rekèt                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Egzanp: Deklanchè Orè ak Cron

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Deklanchè Orè                           [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Mòd: [Ekspresyon Cron ▼]                                      │
│                                                                 │
│  Ekspresyon Cron: [0 9 * * 1-5____________________]            │
│                                                                 │
│  Sa vle di: Chak jou semèn nan 9è nan maten                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  FORMAT CRON                                            │   │
│  │                                                         │   │
│  │  ┌───────────── minit (0 - 59)                         │   │
│  │  │ ┌─────────── èdtan (0 - 23)                         │   │
│  │  │ │ ┌───────── jou nan mwa (1 - 31)                   │   │
│  │  │ │ │ ┌─────── mwa (1 - 12)                           │   │
│  │  │ │ │ │ ┌───── jou nan semèn (0 - 6)                  │   │
│  │  │ │ │ │ │                                             │   │
│  │  * * * * *                                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  EGZANP KOMEN:                                                  │
│  • 0 * * * *     = Chak èdtan                                  │
│  • 0 0 * * *     = Chak jou a minwi                            │
│  • 0 9 * * 1     = Chak lendi a 9è                             │
│  • */15 * * * *  = Chak 15 minit                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.1.2 Nod Aksyon

Nod aksyon yo fè travay reyèl nan flodtravay ou a.

```
┌─────────────────────────────────────────────────────────────────┐
│                    KATEGORI AKSYON                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KOMINIKASYON                                                   │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  📧 Imèl   │  │  💬 Slack  │  │  📱 SMS    │               │
│  │  Voye     │  │  Mesaj     │  │  Twilio    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
│  BAZ DONE                                                       │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │ 🐘 Postgres│  │ 🔥 Supabase│  │ 📊 MySQL   │               │
│  │  Li/Ekri  │  │  CRUD      │  │  Kesyon    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
│  API AK WÈB                                                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  🌐 HTTP   │  │  🔗 API    │  │  📄 JSON   │               │
│  │  Rekèt    │  │  REST      │  │  Konvèti   │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
│  ENTÈLIJANS ATIFISYÈL                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  🤖 OpenAI │  │  🧠 Claude │  │  📝 NLP    │               │
│  │  GPT      │  │  Anthropic │  │  Analiz    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Egzanp: Nod Rekèt HTTP

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Rekèt HTTP                              [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Metòd: [POST ▼]                                               │
│                                                                 │
│  URL: [https://api.example.com/itilizatè_____________]         │
│                                                                 │
│  Otentifikasyon: [Antèt API ▼]                                 │
│                                                                 │
│  Akreditasyon: [API Mwen ▼]                                    │
│                                                                 │
│  ▼ KÒ REKÈT                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Tip Kontni: [JSON ▼]                                   │   │
│  │                                                         │   │
│  │  Kò:                                                    │   │
│  │  {                                                      │   │
│  │    "non": "{{ $json.non }}",                           │   │
│  │    "imèl": "{{ $json.imèl }}"                          │   │
│  │  }                                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ▶ ANTÈT                                                       │
│  ▶ PARAMÈT KESYON                                              │
│  ▶ OPSYON                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.1.3 Nod Lojik

Nod lojik yo kontwole fason done sikile nan flodtravay ou a.

```
┌─────────────────────────────────────────────────────────────────┐
│                      NOD LOJIK                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KONTWÒL FLO                                                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  ❓ SI     │  │  🔀 Chanje │  │  🔗 Mèje   │               │
│  │  (If)     │  │  (Switch)  │  │  (Merge)   │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
│  TRANSFÒMASYON DONE                                            │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  📊 Separe │  │  🔧 Sèt    │  │  📝 Fonksyon│              │
│  │  (Split)  │  │  (Set)     │  │  (Function)│               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
│  ITERASYON                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  🔁 Boukle │  │  📋 Separe │  │  ⏸️ Tann   │               │
│  │  (Loop)   │  │  Pa Pakèt  │  │  (Wait)    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Travay ak Done

### 4.2.1 Estrikti Done nan N8N

N8N itilize yon estrikti done ki rele **eleman** (items). Chak nod resevwa eleman yo epi pwodui nouvo eleman yo.

```
┌─────────────────────────────────────────────────────────────────┐
│                   ESTRIKTI DONE N8N                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Chak eleman gen fòma sa a:                                    │
│                                                                 │
│  {                                                             │
│    "json": {                                                   │
│      // Done prensipal ou yo                                   │
│      "non": "Jan",                                             │
│      "laj": 30,                                                │
│      "vil": "Pòtoprens"                                        │
│    },                                                          │
│    "binary": {                                                 │
│      // Done binè (imaj, fichye, elatriye)                    │
│    }                                                           │
│  }                                                             │
│                                                                 │
│  Plizyè eleman = Yon tablo:                                    │
│  [                                                             │
│    { "json": { "non": "Jan" } },                              │
│    { "json": { "non": "Mari" } },                             │
│    { "json": { "non": "Pyè" } }                               │
│  ]                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2.2 Aksede Done ak Ekspresyon

N8N itilize ekspresyon pou aksede done yo. Ekspresyon yo kòmanse ak `{{ }}`.

```
┌─────────────────────────────────────────────────────────────────┐
│                  EKSPRESYON N8N                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EKSPRESYON BAZ                                                 │
│  ─────────────────────────────────────                          │
│                                                                 │
│  {{ $json.non }}        → Aksede chan "non" nan done aktyèl    │
│  {{ $json.itilizatè.imèl }} → Aksede done inbrike             │
│  {{ $json.lis[0] }}     → Premye eleman nan yon lis           │
│                                                                 │
│  REFERANS NOD                                                   │
│  ─────────────────────────────────────                          │
│                                                                 │
│  {{ $node["Non Nod"].json.done }}                              │
│  → Aksede done nan yon nod espesifik                           │
│                                                                 │
│  {{ $('Non Nod').item.json.done }}                             │
│  → Nouvo fason (plis kout)                                     │
│                                                                 │
│  VARYAB ESPESYAL                                                │
│  ─────────────────────────────────────                          │
│                                                                 │
│  {{ $now }}             → Dat ak lè aktyèl                     │
│  {{ $today }}           → Dat jodi a                           │
│  {{ $itemIndex }}       → Endèks eleman aktyèl                 │
│  {{ $runIndex }}        → Nimewo egzekisyon                    │
│  {{ $workflow.name }}   → Non flodtravay la                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2.3 Nod "Sèt" (Set)

Nod Sèt pèmèt ou kreye oswa modifye done:

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Sèt                                     [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Mòd: [Manyèl ▼]                                               │
│                                                                 │
│  VALÈ YO                                                        │
│  ─────────────────────────────────────                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Non:        [nonKonplè________________________]        │   │
│  │  Tip:        [Tèks ▼]                                   │   │
│  │  Valè:       [{{ $json.prenon }} {{ $json.siyati }}]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Non:        [lajAnAne______________________________]   │   │
│  │  Tip:        [Nimewo ▼]                                 │   │
│  │  Valè:       [{{ $json.laj * 365 }}________________]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [+ Ajoute Valè]                                               │
│                                                                 │
│  OPSYON                                                         │
│  ☑ Kenbe done egzistan yo                                      │
│  ☐ Ranplase tout done yo                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2.4 Nod "Fonksyon" (Function)

Pou transfòmasyon avanse, itilize nod Fonksyon:

```javascript
// Kòd nan nod Fonksyon

// Aksede done antre yo
const eleman = $input.all();

// Trete chak eleman
const rezilta = eleman.map(elem => {
  const done = elem.json;

  return {
    json: {
      nonMajiskil: done.non.toUpperCase(),
      imèlValid: done.imèl.includes('@'),
      dateKreye: new Date().toISOString(),
      mesajPèsonalize: `Bonjou ${done.non}, byenvini!`
    }
  };
});

return rezilta;
```

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Nod Fonksyon                            [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KÒD JAVASCRIPT                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  // Aksede done antre yo                               │   │
│  │  const eleman = $input.all();                          │   │
│  │                                                         │   │
│  │  // Trete chak eleman                                  │   │
│  │  const rezilta = eleman.map(elem => {                  │   │
│  │    const done = elem.json;                             │   │
│  │                                                         │   │
│  │    return {                                            │   │
│  │      json: {                                           │   │
│  │        nonMajiskil: done.non.toUpperCase(),           │   │
│  │        imèlValid: done.imèl.includes('@')             │   │
│  │      }                                                 │   │
│  │    };                                                  │   │
│  │  });                                                   │   │
│  │                                                         │   │
│  │  return rezilta;                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [Tès Kòd]                                [Aplike]             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.3 Lojik Kondisyonèl (SI/ALÒS)

### 4.3.1 Nod SI (If)

Nod SI pèmèt ou kreye branch kondisyonèl nan flodtravay ou a.

```
┌─────────────────────────────────────────────────────────────────┐
│                    NOD SI (IF)                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌─────────────┐                              │
│                    │             │                              │
│     Antre ───────▶│     SI      │                              │
│                    │             │                              │
│                    └──────┬──────┘                              │
│                           │                                     │
│              ┌────────────┴────────────┐                        │
│              │                         │                        │
│              ▼                         ▼                        │
│       ┌─────────────┐           ┌─────────────┐                │
│       │    VRE      │           │    FO       │                │
│       │  (True)     │           │  (False)    │                │
│       └─────────────┘           └─────────────┘                │
│                                                                 │
│  Si kondisyon an vre,      Si kondisyon an fo,                 │
│  done ale nan branch VRE   done ale nan branch FO              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3.2 Konfigirasyon Nod SI

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: SI                                      [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KONDISYON                                                      │
│  ─────────────────────────────────────                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  Valè 1:    [{{ $json.laj }}_____________________]     │   │
│  │                                                         │   │
│  │  Operasyon: [Pi Gran Pase ▼]                           │   │
│  │                                                         │   │
│  │  Valè 2:    [18____________________________________]   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [+ Ajoute Kondisyon]                                          │
│                                                                 │
│  Konbinezon: [TOUT (AND) ▼]                                    │
│                                                                 │
│  Opsyon:                                                        │
│  • TOUT (AND) - Tout kondisyon dwe vre                         │
│  • YOUN (OR) - Omwen yon kondisyon dwe vre                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3.3 Operasyon Kondisyonèl Disponib

| Operasyon | Deskripsyon | Egzanp |
|-----------|-------------|--------|
| Egal | Valè yo idantik | `laj == 18` |
| Pa Egal | Valè yo diferan | `estati != "aktif"` |
| Pi Gran Pase | Premye valè pi gran | `laj > 21` |
| Pi Piti Pase | Premye valè pi piti | `pri < 100` |
| Kontni | Tèks gen yon valè | `imèl kontni "@"` |
| Kòmanse Ak | Tèks kòmanse ak | `non kòmanse ak "J"` |
| Fini Ak | Tèks fini ak | `fichye fini ak ".pdf"` |
| Vid | Valè vid | `non vid` |
| Pa Vid | Valè pa vid | `imèl pa vid` |

### 4.3.4 Egzanp: Flodtravay ak Lojik Kondisyonèl

```
┌─────────────────────────────────────────────────────────────────┐
│  EGZANP: VERIFYE LAJ ITILIZATÈ                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐                            │
│  │  Webhook    │───▶│    SI       │                            │
│  │  (Resevwa   │    │  laj >= 18  │                            │
│  │   done)     │    └──────┬──────┘                            │
│  └─────────────┘           │                                   │
│                    ┌───────┴───────┐                           │
│                    │               │                           │
│                    ▼               ▼                           │
│             ┌─────────────┐ ┌─────────────┐                    │
│             │    VRE      │ │    FO       │                    │
│             └──────┬──────┘ └──────┬──────┘                    │
│                    │               │                           │
│                    ▼               ▼                           │
│             ┌─────────────┐ ┌─────────────┐                    │
│             │ Voye Imèl   │ │ Voye Imèl   │                    │
│             │ Byenvini    │ │ Refize      │                    │
│             └─────────────┘ └─────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3.5 Nod Chanje (Switch)

Pou plizyè kondisyon, itilize nod Chanje:

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Chanje (Switch)                         [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Valè pou Konpare: [{{ $json.kategori }}_______________]       │
│                                                                 │
│  WOUT YO                                                        │
│  ─────────────────────────────────────                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Wout 1: kategori_a                                     │   │
│  │  Valè: [kategori_a_________________________________]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Wout 2: kategori_b                                     │   │
│  │  Valè: [kategori_b_________________________________]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Wout 3: kategori_c                                     │   │
│  │  Valè: [kategori_c_________________________________]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [+ Ajoute Wout]                                               │
│                                                                 │
│  ☑ Enkli wout "Lòt" pou valè ki pa matche                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│  DYAGRAM CHANJE (SWITCH)                                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        ┌─────────────┐                          │
│                        │             │                          │
│         Antre ───────▶│   CHANJE    │                          │
│                        │             │                          │
│                        └──────┬──────┘                          │
│                               │                                 │
│          ┌──────────┬────────┼────────┬──────────┐             │
│          │          │        │        │          │             │
│          ▼          ▼        ▼        ▼          ▼             │
│    ┌──────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐       │
│    │ Wout A  │ │Wout B│ │Wout C│ │Wout D│ │  Lòt    │       │
│    └──────────┘ └──────┘ └──────┘ └──────┘ └──────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.4 Boukle ak Iterasyon

### 4.4.1 Kijan Iterasyon Fonksyone nan N8N

Pa defo, N8N trete chak eleman nan yon lis separeman. Sa vle di si ou gen 10 eleman, nod la egzekite 10 fwa.

```
┌─────────────────────────────────────────────────────────────────┐
│              KONPÒTMAN ITERASYON PA DEFO                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Antre: 3 eleman                                               │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ [                                                      │    │
│  │   { "json": { "non": "Jan" } },                       │    │
│  │   { "json": { "non": "Mari" } },                      │    │
│  │   { "json": { "non": "Pyè" } }                        │    │
│  │ ]                                                      │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│                         │                                       │
│                         ▼                                       │
│                  ┌─────────────┐                                │
│                  │   Nod A     │ ──▶ Egzekite 3 fwa            │
│                  └─────────────┘                                │
│                         │                                       │
│                         ▼                                       │
│                                                                 │
│  Sòti: 3 eleman trete                                          │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ [                                                      │    │
│  │   { "json": { "non": "JAN" } },                       │    │
│  │   { "json": { "non": "MARI" } },                      │    │
│  │   { "json": { "non": "PYÈ" } }                        │    │
│  │ ]                                                      │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.4.2 Nod "Separe Pa Pakèt" (Split In Batches)

Pou trete eleman yo pa pakèt:

```
┌─────────────────────────────────────────────────────────────────┐
│  Konfigirasyon: Separe Pa Pakèt                         [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Gwosè Pakèt: [10_____________________________________]        │
│                                                                 │
│  Sa vle di: Trete 10 eleman alafwa                             │
│                                                                 │
│  EGZANP:                                                        │
│  ─────────────────────────────────────                          │
│                                                                 │
│  Antre: 25 eleman                                              │
│                                                                 │
│  Pakèt 1: Eleman 1-10                                          │
│  Pakèt 2: Eleman 11-20                                         │
│  Pakèt 3: Eleman 21-25                                         │
│                                                                 │
│  Opsyon:                                                        │
│  ☑ Kontinye sou erè (pa kanpe si yon pakèt echwe)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.4.3 Boukle ak Nod "Boukle Sou Eleman" (Loop Over Items)

```
┌─────────────────────────────────────────────────────────────────┐
│  BOUKLE SOU ELEMAN                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐                                               │
│  │  Deklanchè  │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  Rekipere   │ ──▶ Retounen lis itilizatè                   │
│  │  Itilizatè  │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────────────────────────────────────┐               │
│  │           BOUKLE SOU ELEMAN                 │               │
│  │  ┌───────────────────────────────────────┐  │               │
│  │  │                                       │  │               │
│  │  │  ┌─────────────┐    ┌─────────────┐  │  │               │
│  │  │  │  Trete      │───▶│  Voye       │  │  │               │
│  │  │  │  Done       │    │  Imèl       │  │  │               │
│  │  │  └─────────────┘    └─────────────┘  │  │               │
│  │  │                                       │  │               │
│  │  └───────────────────────────────────────┘  │               │
│  └─────────────────────────────────────────────┘               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  Rezime     │                                               │
│  └─────────────┘                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.4.4 Egzanp Kòd: Boukle Manyèl

```javascript
// Nan nod Fonksyon, ou ka fè boukle manyèl

const eleman = $input.all();
const rezilta = [];

for (let i = 0; i < eleman.length; i++) {
  const elem = eleman[i].json;

  // Trete chak eleman
  rezilta.push({
    json: {
      endèks: i,
      non: elem.non,
      trete: true,
      datTretman: new Date().toISOString()
    }
  });
}

return rezilta;
```

---

## 4.5 Jesyon Erè

### 4.5.1 Poukisa Jesyon Erè Enpòtan?

Nan mond reyèl la, erè ka rive:
- API pa disponib
- Done an move fòma
- Limit debit depase
- Pwoblèm koneksyon

N8N bay plizyè fason pou jere erè yo.

```
┌─────────────────────────────────────────────────────────────────┐
│                    TIP ERÈ KOMEN                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ERÈ KONEKSYON                                          │   │
│  │  • API pa reponn                                        │   │
│  │  • Timeout depase                                       │   │
│  │  • Sètifika SSL envalid                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ERÈ OTENTIFIKASYON                                     │   │
│  │  • Kle API envalid                                      │   │
│  │  • Tokèn ekspire                                        │   │
│  │  • Pèmisyon refize                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ERÈ DONE                                               │   │
│  │  • Chan obligatwa manke                                 │   │
│  │  • Tip done move                                        │   │
│  │  • Valè ki pa akseptab                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.5.2 Opsyon Jesyon Erè nan Nod

Chak nod gen opsyon pou jere erè:

```
┌─────────────────────────────────────────────────────────────────┐
│  Paramèt: Jesyon Erè                                    [X]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Sou Erè: [Kontinye ▼]                                         │
│                                                                 │
│  Opsyon disponib:                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Sispann Egzekisyon                                   │   │
│  │    Kanpe flodtravay la imedyatman                       │   │
│  │                                                         │   │
│  │  • Kontinye                                             │   │
│  │    Inyore erè a epi kontinye                           │   │
│  │                                                         │   │
│  │  • Kontinye ak Sòti Erè                                │   │
│  │    Kontinye men voye done nan branch erè               │   │
│  │                                                         │   │
│  │  • Kontinye ak Antèt Erè                               │   │
│  │    Kontinye ak enfòmasyon erè nan done yo              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Tantativ Ankò: [3_______]                                     │
│  Tann ant tantativ: [1000___] ms                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.5.3 Nod "Erè Deklanchè" (Error Trigger)

Ou ka kreye yon flodtravay espesyal pou jere erè:

```
┌─────────────────────────────────────────────────────────────────┐
│  FLODTRAVAY JESYON ERÈ                                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │   Erè       │───▶│  Prepare    │───▶│  Voye       │        │
│  │  Deklanchè  │    │  Mesaj      │    │  Alèt Imèl  │        │
│  └─────────────┘    └─────────────┘    └─────────────┘        │
│                                                                 │
│  Nod Erè Deklanchè resevwa:                                    │
│  {                                                             │
│    "execution": {                                              │
│      "id": "abc123",                                           │
│      "url": "https://...",                                     │
│      "error": {                                                │
│        "message": "Koneksyon refize",                         │
│        "node": "Rekèt HTTP"                                    │
│      }                                                         │
│    },                                                          │
│    "workflow": {                                               │
│      "id": "xyz789",                                           │
│      "name": "Flodtravay Prensipal"                           │
│    }                                                           │
│  }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.5.4 Blòk Eseye-Trape (Try-Catch) ak Fonksyon

```javascript
// Nan nod Fonksyon, itilize try-catch

const eleman = $input.all();
const reziltaSiksè = [];
const reziltaErè = [];

for (const elem of eleman) {
  try {
    // Trete done
    const done = elem.json;

    // Validasyon
    if (!done.imèl) {
      throw new Error('Imèl obligatwa');
    }

    if (!done.imèl.includes('@')) {
      throw new Error('Imèl envalid');
    }

    // Si tout bagay bon
    reziltaSiksè.push({
      json: {
        ...done,
        valide: true
      }
    });

  } catch (erè) {
    // Jere erè
    reziltaErè.push({
      json: {
        doneOrijinal: elem.json,
        mesajErè: erè.message,
        datErè: new Date().toISOString()
      }
    });
  }
}

// Retounen tou de rezilta yo
return {
  siksè: reziltaSiksè,
  erè: reziltaErè
};
```

### 4.5.5 Egzanp: Flodtravay ak Jesyon Erè Konplè

```
┌─────────────────────────────────────────────────────────────────┐
│  FLODTRAVAY AK JESYON ERÈ KONPLÈ                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐                                               │
│  │  Webhook    │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  Valide     │                                               │
│  │  Done       │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│    ┌────┴────┐                                                 │
│    │   SI    │                                                 │
│    │  valid  │                                                 │
│    └────┬────┘                                                 │
│    ┌────┴────┐                                                 │
│    │        │                                                  │
│    ▼        ▼                                                  │
│  ┌─────┐  ┌─────┐                                              │
│  │ VRE │  │ FO  │                                              │
│  └──┬──┘  └──┬──┘                                              │
│     │        │                                                 │
│     ▼        ▼                                                 │
│  ┌─────────────┐  ┌─────────────┐                              │
│  │  Rekèt API  │  │  Retounen   │                              │
│  │ (ak tantativ│  │  Erè        │                              │
│  │  ankò)      │  │  Validasyon │                              │
│  └──────┬──────┘  └─────────────┘                              │
│         │                                                       │
│    ┌────┴────┐                                                 │
│    │   SI    │                                                 │
│    │ siksè   │                                                 │
│    └────┬────┘                                                 │
│    ┌────┴────┐                                                 │
│    │        │                                                  │
│    ▼        ▼                                                  │
│  ┌─────┐  ┌─────┐                                              │
│  │ VRE │  │ FO  │                                              │
│  └──┬──┘  └──┬──┘                                              │
│     │        │                                                 │
│     ▼        ▼                                                 │
│  ┌─────────────┐  ┌─────────────┐                              │
│  │  Retounen   │  │  Voye Alèt  │                              │
│  │  Siksè      │  │  + Retounen │                              │
│  │             │  │  Erè        │                              │
│  └─────────────┘  └─────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.6 Meyè Pratik

### 4.6.1 Òganizasyon Flodtravay

```
┌─────────────────────────────────────────────────────────────────┐
│               MEYÈ PRATIK ÒGANIZASYON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. BAY NON KLÈ                                                │
│     ────────────────                                            │
│     ✅ Bon:  "Senkronize-Itilizatè-Supabase-Mailchimp"        │
│     ❌ Move: "Flodtravay1"                                      │
│                                                                 │
│  2. AJOUTE NÒT                                                 │
│     ────────────────                                            │
│     Itilize nod "Nòt" pou dokimante flodtravay ou              │
│                                                                 │
│  3. GWOUP NOD YO                                               │
│     ────────────────                                            │
│     Ranje nod yo nan lòd lojik, de goch a dwat                 │
│                                                                 │
│  4. ITILIZE ETIKÈT                                             │
│     ────────────────                                            │
│     Ajoute etikèt pou kategori: "pwodiksyon", "tès", elatriye │
│                                                                 │
│  5. VÈSYON                                                     │
│     ────────────────                                            │
│     Sove vèsyon flodtravay ou avan gwo chanjman                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.6.2 Pèfòmans

```
┌─────────────────────────────────────────────────────────────────┐
│                 MEYÈ PRATIK PÈFÒMANS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. LIMITE DONE                                                │
│     ────────────────                                            │
│     Pa rekipere plis done pase sa ou bezwen                    │
│     Itilize filtè ak limitasyon nan rekèt yo                   │
│                                                                 │
│  2. ITILIZE PAKÈT                                              │
│     ────────────────                                            │
│     Pou gwo kantite done, itilize "Separe Pa Pakèt"            │
│                                                                 │
│  3. EVITE BOUKLE ENFINI                                        │
│     ────────────────                                            │
│     Toujou mete kondisyon sòti nan boukle yo                   │
│                                                                 │
│  4. KACHE DONE                                                 │
│     ────────────────                                            │
│     Si posib, kache rezilta ki pa chanje souvan                │
│                                                                 │
│  5. PARALÈL VS SEKANSYÈL                                       │
│     ────────────────                                            │
│     Egzekite nod endepandan yo an paralèl                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.6.3 Sekirite

```
┌─────────────────────────────────────────────────────────────────┐
│                  MEYÈ PRATIK SEKIRITE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. AKREDITASYON                                               │
│     ────────────────                                            │
│     • Toujou itilize sistèm akreditasyon N8N                   │
│     • Pa janm mete kle API dirèkteman nan nod                  │
│     • Renouvle kle yo regilyèman                               │
│                                                                 │
│  2. VALIDE ANTRE                                               │
│     ────────────────                                            │
│     • Toujou valide done ki soti nan webhook                   │
│     • Verifye tip done yo                                      │
│     • Netwaye done avan itilize                                │
│                                                                 │
│  3. WEBHOOK                                                    │
│     ────────────────                                            │
│     • Itilize HTTPS toujou                                     │
│     • Mete otentifikasyon sou webhook yo                       │
│     • Valide siyati si disponib                                │
│                                                                 │
│  4. DONE SANSIB                                                │
│     ────────────────                                            │
│     • Pa loge done sansib                                      │
│     • Mask done nan mesaj erè                                  │
│     • Respekte règleman RGPD/PDPA                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.6.4 Tès ak Debogaj

```
┌─────────────────────────────────────────────────────────────────┐
│              MEYÈ PRATIK TÈS AK DEBOGAJ                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. TÈS ENDIVIDYÈL                                             │
│     ────────────────                                            │
│     Tès chak nod separeman avan konekte yo                     │
│                                                                 │
│  2. DONE TÈS                                                   │
│     ────────────────                                            │
│     Kreye done tès ki reprezante sitiyasyon reyèl              │
│     Enkli ka espesyal: lis vid, valè nil, elatriye            │
│                                                                 │
│  3. OPSYON DEBOGAJ                                             │
│     ────────────────                                            │
│     • Klike sou nod pou wè done antre/sòti                    │
│     • Itilize nod "Debogaj" pou loge enfòmasyon               │
│     • Gade istwa egzekisyon yo                                 │
│                                                                 │
│  4. ANVIWÒNMAN SEPARE                                          │
│     ────────────────                                            │
│     • Devlopman: Pou kreye ak tès                              │
│     • Staging: Pou tès final                                   │
│     • Pwodiksyon: Pou itilizatè reyèl                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.7 Egzanp Konplè: Flodtravay Notifikasyon

Annou kreye yon flodtravay konplè ki:
1. Resevwa done nan yon webhook
2. Valide done yo
3. Deside ki tip notifikasyon pou voye
4. Voye notifikasyon
5. Jere erè

```
┌─────────────────────────────────────────────────────────────────┐
│  FLODTRAVAY: Sistèm Notifikasyon                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐                                               │
│  │  Webhook    │ ◀── Resevwa: {tip, mesaj, desinatè}          │
│  │  Resevwa    │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │   Valide    │ ◀── Verifye chan obligatwa yo                │
│  │   Done      │                                               │
│  └──────┬──────┘                                               │
│         │                                                       │
│    ┌────┴────┐                                                 │
│    │   SI    │                                                 │
│    │  valid  │                                                 │
│    └────┬────┘                                                 │
│    ┌────┴────┐                                                 │
│    │        │                                                  │
│    ▼        ▼                                                  │
│  ┌─────┐  ┌───────────────┐                                   │
│  │ VRE │  │      FO       │                                   │
│  └──┬──┘  │  (Retounen    │                                   │
│     │     │   Erè 400)    │                                   │
│     │     └───────────────┘                                   │
│     ▼                                                          │
│  ┌─────────────┐                                               │
│  │   CHANJE    │ ◀── Baze sou "tip"                           │
│  │   (Switch)  │                                               │
│  └──────┬──────┘                                               │
│    ┌────┼────┬────┐                                            │
│    │    │    │    │                                            │
│    ▼    ▼    ▼    ▼                                            │
│  ┌────┐┌────┐┌────┐┌────┐                                     │
│  │imèl││sms ││push││lòt │                                     │
│  └──┬─┘└──┬─┘└──┬─┘└──┬─┘                                     │
│     │     │     │     │                                        │
│     ▼     ▼     ▼     ▼                                        │
│  ┌─────┐┌─────┐┌─────┐┌─────┐                                 │
│  │Voye ││Voye ││Voye ││Loge │                                 │
│  │Imèl ││SMS  ││Push ││Erè  │                                 │
│  └──┬──┘└──┬──┘└──┬──┘└──┬──┘                                 │
│     │      │      │      │                                     │
│     └──────┴──────┴──────┘                                     │
│              │                                                  │
│              ▼                                                  │
│       ┌─────────────┐                                          │
│       │    Mèje     │                                          │
│       └──────┬──────┘                                          │
│              │                                                  │
│              ▼                                                  │
│       ┌─────────────┐                                          │
│       │  Retounen   │                                          │
│       │  Rezilta    │                                          │
│       └─────────────┘                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Kòd pou Nod Validasyon

```javascript
// Nod: Valide Done

const eleman = $input.all();

return eleman.map(elem => {
  const done = elem.json;
  const erè = [];

  // Verifye chan obligatwa yo
  if (!done.tip) {
    erè.push("Chan 'tip' obligatwa");
  }

  if (!done.mesaj) {
    erè.push("Chan 'mesaj' obligatwa");
  }

  if (!done.desinatè) {
    erè.push("Chan 'desinatè' obligatwa");
  }

  // Verifye tip valid
  const tipValid = ['imèl', 'sms', 'push'];
  if (done.tip && !tipValid.includes(done.tip)) {
    erè.push(`Tip '${done.tip}' pa valid. Tip valid: ${tipValid.join(', ')}`);
  }

  return {
    json: {
      ...done,
      valid: erè.length === 0,
      erè: erè
    }
  };
});
```

---

## 4.8 Rezime

Nan chapit sa a, nou te aprann:

| Sijè | Sa Nou Te Aprann |
|------|------------------|
| Tip Nod | Deklanchè, Aksyon, Lojik |
| Done | Estrikti eleman, ekspresyon, transfòmasyon |
| Kondisyon | SI/ALÒS, Chanje, operasyon konparezon |
| Iterasyon | Boukle, pakèt, trete plizyè eleman |
| Erè | Jesyon erè, tantativ ankò, alèt |
| Pratik | Òganizasyon, pèfòmans, sekirite, tès |

---

## 4.9 Egzèsis Pratik

### Egzèsis 1: Lojik Kondisyonèl
Kreye yon flodtravay ki:
- Resevwa yon nimewo nan yon webhook
- Verifye si li pozitif, negatif, oswa zewo
- Retounen yon mesaj diferan pou chak ka

### Egzèsis 2: Boukle ak Transfòmasyon
Kreye yon flodtravay ki:
- Rekipere yon lis itilizatè nan yon API
- Filtre itilizatè ki gen plis pase 25 an
- Transfòme done yo (ajoute chan "kategorilaj")
- Retounen rezilta a

### Egzèsis 3: Jesyon Erè
Modifye flodtravay egzèsis 2 pou:
- Jere ka kote API pa disponib
- Voye yon alèt imèl si gen erè
- Loge erè yo nan yon fichye

### Egzèsis 4: Flodtravay Konplè
Kreye yon flodtravay ki:
- Deklanche chak jou a 8è nan maten
- Rekipere meteo nan yon API
- Si tanperati > 30°C, voye alèt chalè
- Si tanperati < 10°C, voye alèt fredi
- Sinon, voye rapò nòmal
- Jere tout erè posib

---

## 4.10 Kesyon Revizyon

1. Ki diferans ant nod deklanchè ak nod aksyon?
2. Kijan ou aksede done nan nod anvan an?
3. Ki diferans ant nod SI ak nod Chanje?
4. Kijan N8N trete plizyè eleman pa defo?
5. Site twa fason pou jere erè nan N8N.
6. Poukisa li enpòtan pou valide done ki soti nan webhook?

---

## 4.11 Referans Rapid

### Ekspresyon Komen

```
┌─────────────────────────────────────────────────────────────────┐
│                  EKSPRESYON REFERANS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AKSEDE DONE                                                    │
│  {{ $json.chan }}           - Chan nan done aktyèl             │
│  {{ $json.objè.sousChan }}  - Done inbrike                     │
│  {{ $json.lis[0] }}         - Premye eleman nan lis            │
│                                                                 │
│  REFERANS NOD                                                   │
│  {{ $('Non').item.json }}   - Done nan nod espesifik          │
│  {{ $input.all() }}         - Tout eleman antre                │
│  {{ $input.first() }}       - Premye eleman antre              │
│                                                                 │
│  VARYAB SISTÈM                                                  │
│  {{ $now }}                 - Dat/lè aktyèl                    │
│  {{ $today }}               - Dat jodi a                       │
│  {{ $itemIndex }}           - Endèks eleman                    │
│  {{ $workflow.name }}       - Non flodtravay                   │
│  {{ $execution.id }}        - ID egzekisyon                    │
│                                                                 │
│  MANIPILASYON TÈKS                                              │
│  {{ $json.non.toUpperCase() }}     - Majiskil                  │
│  {{ $json.non.toLowerCase() }}     - Miniskil                  │
│  {{ $json.non.trim() }}            - Retire espas             │
│  {{ $json.tèks.split(',') }}       - Separe tèks              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

**Nan pwochen chapit la**, nou pral aprann kijan pou entegre N8N ak lòt sèvis tankou Supabase ak Claude AI pou kreye solisyon otomatizasyon pwisan.
