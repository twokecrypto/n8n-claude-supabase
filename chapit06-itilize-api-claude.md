# Kominote TWOKECRYPTO
# Chapit 6: Itilize API Claude

## Rezime Chapit

Nan chapit sa a, nou pral aprann kijan pou itilize API Claude nan pratik. Nou pral fè premye apèl API nou, aprann ekri envit efikas, jere repons yo, ak aplike bon pratik pou travay ak entèlijans atifisyèl.

---

## 6.1 Konprann Estrikti API a

### 6.1.1 Kisa Yon API Ye?

API (Entèfas Pwogramasyonèl Aplikasyon) se yon fason pou aplikasyon ou komunike ak sèvis Claude. Ou voye yon demann, epi ou resevwa yon repons.

```
┌─────────────────────────────────────────────────────────────┐
│                    FLOU API CLAUDE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────┐                   ┌───────────────┐     │
│  │  APLIKASYON   │                   │   SÈVÈ        │     │
│  │     OU        │                   │   ANTHROPIC   │     │
│  └───────┬───────┘                   └───────┬───────┘     │
│          │                                   │              │
│          │  1. Demann HTTP POST              │              │
│          │  ─────────────────────────────>   │              │
│          │  - Kle API                        │              │
│          │  - Modèl                          │              │
│          │  - Mesaj yo                       │              │
│          │                                   │              │
│          │  2. Repons JSON                   │              │
│          │  <─────────────────────────────   │              │
│          │  - Tèks jenere                    │              │
│          │  - Jeton itilize                  │              │
│          │  - Metadone                       │              │
│          │                                   │              │
└─────────────────────────────────────────────────────────────┘
```

### 6.1.2 Pwen Tèminal API

API Claude itilize pwen tèminal sa a:

```
https://api.anthropic.com/v1/messages
```

### 6.1.3 Estrikti Demann

Chak demann API gen eleman sa yo:

```
┌─────────────────────────────────────────────────────────────┐
│              ESTRIKTI DEMANN API                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ANTÈT (Headers):                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ x-api-key: [Kle API ou]                             │   │
│  │ anthropic-version: 2023-06-01                       │   │
│  │ content-type: application/json                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  KÒ DEMANN (Body):                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ {                                                   │   │
│  │   "model": "claude-sonnet-4-20250514",              │   │
│  │   "max_tokens": 1024,                               │   │
│  │   "messages": [...]                                 │   │
│  │ }                                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.1.4 Paramèt Enpòtan

| Paramèt | Obligatwa | Deskripsyon |
|---------|-----------|-------------|
| model | Wi | Non modèl Claude pou itilize |
| max_tokens | Wi | Kantite maksimòm jeton nan repons |
| messages | Wi | Lis mesaj yo nan konvèsasyon an |
| system | Non | Enstriksyon sistèm pou Claude |
| temperature | Non | Kontwole kreyativite (0.0 - 1.0) |
| top_p | Non | Kontwole divèsite repons |
| stop_sequences | Non | Sekans ki fè Claude sispann |

---

## 6.2 Fè Premye Apèl API Ou

### 6.2.1 Itilize cURL

Men yon egzanp konplè ak cURL:

```bash
curl https://api.anthropic.com/v1/messages \
  --header "x-api-key: $ANTHROPIC_API_KEY" \
  --header "anthropic-version: 2023-06-01" \
  --header "content-type: application/json" \
  --data '{
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 1024,
    "messages": [
      {
        "role": "user",
        "content": "Bonjou! Eksplike kisa entèlijans atifisyèl ye an yon paragraf."
      }
    ]
  }'
```

### 6.2.2 Estrikti Repons

Lè ou voye demann lan, ou ap resevwa yon repons JSON konsa:

```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Entèlijans atifisyèl se yon branch nan syans òdinatè ki konsantre sou kreye sistèm ki kapab fè travay ki nòmalman mande entèlijans moun. Sa enkli aprann, rezònman, rezoud pwoblèm, konprann langaj natirèl, ak pèsepsyon. Modèl entèlijans atifisyèl modèn yo, tankou mwen menm, itilize gwo kantite done pou aprann modèl ak konpòtman, sa ki pèmèt yo jenere tèks, analize enfòmasyon, ak ede ak divès travay."
    }
  ],
  "model": "claude-sonnet-4-20250514",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 28,
    "output_tokens": 127
  }
}
```

### 6.2.3 Konprann Repons lan

```
┌─────────────────────────────────────────────────────────────┐
│              ELEMAN REPONS API                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  id              Idantifyan inik mesaj la                  │
│  type            Toujou "message"                          │
│  role            "assistant" pou repons Claude             │
│  content         Lis blòk kontni (tèks, imaj, elatriye)    │
│  model           Modèl ki te itilize                       │
│  stop_reason     Poukisa Claude te sispann                 │
│                  - "end_turn": fini nòmalman               │
│                  - "max_tokens": rive nan limit            │
│                  - "stop_sequence": jwenn sekans stop      │
│  usage           Estatistik jeton                          │
│                  - input_tokens: jeton antre               │
│                  - output_tokens: jeton sòti               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.2.4 Egzanp Ak Python

```python
import anthropic
import os

# Kreye kliyan ak kle API
kliyan = anthropic.Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY")
)

# Voye demann
repons = kliyan.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": "Bonjou! Eksplike kisa entèlijans atifisyèl ye."
        }
    ]
)

# Afiche repons lan
print(repons.content[0].text)

# Afiche estatistik jeton
print(f"Jeton antre: {repons.usage.input_tokens}")
print(f"Jeton sòti: {repons.usage.output_tokens}")
```

### 6.2.5 Egzanp Ak JavaScript/Node.js

```javascript
import Anthropic from '@anthropic-ai/sdk';

// Kreye kliyan
const kliyan = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Fonksyon asenkwon pou voye mesaj
async function voyeMesaj(kontni) {
  const repons = await kliyan.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: kontni
      }
    ]
  });

  return repons;
}

// Itilize fonksyon an
async function prensipal() {
  const repons = await voyeMesaj("Bonjou! Eksplike kisa entèlijans atifisyèl ye.");

  console.log("Repons:", repons.content[0].text);
  console.log("Jeton antre:", repons.usage.input_tokens);
  console.log("Jeton sòti:", repons.usage.output_tokens);
}

prensipal();
```

---

## 6.3 Ekri Envit Efikas

### 6.3.1 Prensip Envit Efikas

```
┌─────────────────────────────────────────────────────────────┐
│           PRENSIP ENVIT EFIKAS                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. KLÈTE                                                  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Mal:  "Di m sou bagay la"                       │    │
│     │ Bon:  "Eksplike kijan fotosintèz fonksyone      │    │
│     │        nan plant yo"                            │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  2. KONTÈKS                                                │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Bay enfòmasyon sou sitiyasyon an                │    │
│     │ Eksplike piblik la                              │    │
│     │ Presize fòma ou vle                             │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  3. ESPESIFIK                                              │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Mande egzakteman sa ou bezwen                   │    │
│     │ Presize longè repons                            │    │
│     │ Mande fòma (lis, paragraf, tablo)               │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.3.2 Egzanp Envit Bon ak Mal

**Egzanp 1: Mande Eksplikasyon**

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Eksplike baz done"
    }
  ]
}
```
*Pwoblèm: Twò vag, pa gen kontèks*

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Eksplike kisa yon baz done relasyonèl ye pou yon moun ki fèk kòmanse aprann pwogramasyon. Enkli 2-3 egzanp senp ak yon analoji ak lavi reyèl."
    }
  ]
}
```
*Bon: Klè, gen kontèks, espesifik*

**Egzanp 2: Mande Kòd**

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Ekri kòd pou konekte ak baz done"
    }
  ]
}
```
*Pwoblèm: Ki langaj? Ki baz done? Ki aksyon?*

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Ekri yon fonksyon Python ki konekte ak yon baz done PostgreSQL epi retounen tout itilizatè ki gen laj plis pase 18 an. Itilize bibliyotèk psycopg2. Enkli jesyon erè."
    }
  ]
}
```
*Bon: Langaj klè, baz done klè, aksyon klè*

### 6.3.3 Itilize Mesaj Sistèm

Mesaj sistèm pèmèt ou defini konpòtman Claude:

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "Ou se yon asistan pwogramasyon ki espesyalize nan Python. Toujou bay kòd ak kòmantè an Kreyòl Ayisyen. Bay eksplikasyon klè pou chak etap.",
  "messages": [
    {
      "role": "user",
      "content": "Ekri yon fonksyon ki kalkile mwayèn yon lis nonm."
    }
  ]
}
```

**Repons Posib:**

```python
def kalkile_mwayen(lis_nonm):
    """
    Kalkile mwayèn yon lis nonm.

    Agiman:
        lis_nonm: Lis ki gen nonm yo

    Retounen:
        Mwayèn nonm yo, oswa 0 si lis la vid
    """
    # Tcheke si lis la vid
    if not lis_nonm:
        return 0

    # Kalkile sòm tout nonm yo
    som_total = sum(lis_nonm)

    # Divize pa kantite nonm yo
    kantite = len(lis_nonm)

    # Retounen mwayèn lan
    return som_total / kantite


# Egzanp itilizasyon
nòt_yo = [85, 90, 78, 92, 88]
mwayen = kalkile_mwayen(nòt_yo)
print(f"Mwayèn nòt yo: {mwayen}")  # Rezilta: 86.6
```

### 6.3.4 Teknik Avanse: Envit ak Egzanp

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "Ou se yon tradiksyon entèlijans atifisyèl. Tradui tèks yo nan fòma sa a: ORIJINAL -> TRADIKSYON",
  "messages": [
    {
      "role": "user",
      "content": "Tradui: Hello"
    },
    {
      "role": "assistant",
      "content": "Hello -> Bonjou"
    },
    {
      "role": "user",
      "content": "Tradui: Thank you"
    },
    {
      "role": "assistant",
      "content": "Thank you -> Mèsi"
    },
    {
      "role": "user",
      "content": "Tradui: How are you today?"
    }
  ]
}
```

---

## 6.4 Jere Repons yo

### 6.4.1 Trete Repons Tèks

```python
import anthropic

kliyan = anthropic.Anthropic()

repons = kliyan.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Bay yon lis 3 fwi tropikal."}
    ]
)

# Jwenn tèks repons lan
teks_repons = repons.content[0].text
print(teks_repons)

# Tcheke rezon ki fè Claude te sispann
if repons.stop_reason == "end_turn":
    print("Claude fini repons li nòmalman")
elif repons.stop_reason == "max_tokens":
    print("Avètisman: Repons la te koupe paske li te rive nan limit jeton")
```

### 6.4.2 Trete Plizyè Blòk Kontni

Pafwa repons lan gen plizyè blòk:

```python
repons = kliyan.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Dekri imaj sa a ak ekri kòd ki analize li."}
    ]
)

# Itere sou tout blòk kontni yo
for blok in repons.content:
    if blok.type == "text":
        print(f"Tèks: {blok.text}")
    elif blok.type == "tool_use":
        print(f"Zouti itilize: {blok.name}")
```

### 6.4.3 Siveye Itilizasyon Jeton

```python
def analize_itilizasyon(repons):
    """
    Analize itilizasyon jeton nan yon repons.
    """
    jeton_antre = repons.usage.input_tokens
    jeton_soti = repons.usage.output_tokens
    total_jeton = jeton_antre + jeton_soti

    # Kalkile pri (egzanp ak Claude 3.5 Sonnet)
    pri_antre = jeton_antre * (3.00 / 1_000_000)
    pri_soti = jeton_soti * (15.00 / 1_000_000)
    pri_total = pri_antre + pri_soti

    print(f"Jeton Antre: {jeton_antre}")
    print(f"Jeton Sòti: {jeton_soti}")
    print(f"Total Jeton: {total_jeton}")
    print(f"Pri Estime: ${pri_total:.6f}")

    return {
        "jeton_antre": jeton_antre,
        "jeton_soti": jeton_soti,
        "total": total_jeton,
        "pri": pri_total
    }

# Itilize fonksyon an
estatistik = analize_itilizasyon(repons)
```

---

## 6.5 Jere Konvèsasyon

### 6.5.1 Estrikti Konvèsasyon

```
┌─────────────────────────────────────────────────────────────┐
│              FLOU KONVÈSASYON                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Mesaj 1 (Itilizatè): "Bonjou!"                            │
│       │                                                     │
│       ▼                                                     │
│  Mesaj 2 (Asistan): "Bonjou! Kijan mwen ka ede ou?"        │
│       │                                                     │
│       ▼                                                     │
│  Mesaj 3 (Itilizatè): "Eksplike Python"                    │
│       │                                                     │
│       ▼                                                     │
│  Mesaj 4 (Asistan): "Python se yon langaj..."              │
│       │                                                     │
│       ▼                                                     │
│  [Kontinye konvèsasyon...]                                  │
│                                                             │
│  ENPÒTAN: Chak nouvo demann dwe enkli TOUT                 │
│           mesaj anvan yo pou kenbe kontèks la              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.5.2 Enplemante Konvèsasyon

```python
import anthropic

class AsistanKonvèsasyon:
    """
    Klas pou jere konvèsasyon ak Claude.
    """

    def __init__(self, enstriksyon_sistem=None):
        """
        Inisyalize asistan an.

        Agiman:
            enstriksyon_sistem: Enstriksyon pou defini konpòtman Claude
        """
        self.kliyan = anthropic.Anthropic()
        self.mesaj_yo = []
        self.enstriksyon_sistem = enstriksyon_sistem or "Ou se yon asistan itil ki reponn an Kreyòl Ayisyen."

    def voye_mesaj(self, kontni_itilizate):
        """
        Voye yon mesaj epi resevwa repons.

        Agiman:
            kontni_itilizate: Mesaj itilizatè a

        Retounen:
            Repons Claude
        """
        # Ajoute mesaj itilizatè a nan istwa a
        self.mesaj_yo.append({
            "role": "user",
            "content": kontni_itilizate
        })

        # Voye demann ak tout istwa a
        repons = self.kliyan.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            system=self.enstriksyon_sistem,
            messages=self.mesaj_yo
        )

        # Ekstrè tèks repons lan
        teks_repons = repons.content[0].text

        # Ajoute repons lan nan istwa a
        self.mesaj_yo.append({
            "role": "assistant",
            "content": teks_repons
        })

        return teks_repons

    def efase_istwa(self):
        """
        Efase tout istwa konvèsasyon an.
        """
        self.mesaj_yo = []

    def jwenn_istwa(self):
        """
        Retounen istwa konvèsasyon an.
        """
        return self.mesaj_yo


# Egzanp itilizasyon
asistan = AsistanKonvèsasyon(
    enstriksyon_sistem="Ou se yon pwofesè Python ki eksplike konsèp yo yon fason senp."
)

# Premye mesaj
repons1 = asistan.voye_mesaj("Kisa yon varyab ye nan Python?")
print(f"Claude: {repons1}\n")

# Dezyèm mesaj (Claude sonje kontèks la)
repons2 = asistan.voye_mesaj("Ka ou ban m yon egzanp?")
print(f"Claude: {repons2}\n")

# Twazyèm mesaj
repons3 = asistan.voye_mesaj("Kijan mwen ka chanje valè li?")
print(f"Claude: {repons3}\n")
```

### 6.5.3 Optimize Istwa Konvèsasyon

```python
class AsistanOptimize:
    """
    Asistan ki optimize istwa pou ekonomize jeton.
    """

    def __init__(self, limit_mesaj=10):
        """
        Inisyalize ak yon limit mesaj.

        Agiman:
            limit_mesaj: Kantite maksimòm mesaj pou kenbe
        """
        self.kliyan = anthropic.Anthropic()
        self.mesaj_yo = []
        self.limit_mesaj = limit_mesaj

    def voye_mesaj(self, kontni):
        """
        Voye mesaj ak jesyon limit.
        """
        self.mesaj_yo.append({
            "role": "user",
            "content": kontni
        })

        # Koupe istwa si li depase limit la
        if len(self.mesaj_yo) > self.limit_mesaj * 2:
            # Kenbe sèlman dènye mesaj yo
            self.mesaj_yo = self.mesaj_yo[-(self.limit_mesaj * 2):]

        repons = self.kliyan.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=self.mesaj_yo
        )

        teks = repons.content[0].text

        self.mesaj_yo.append({
            "role": "assistant",
            "content": teks
        })

        return teks
```

---

## 6.6 Bon Pratik pou Envit Entèlijans Atifisyèl

### 6.6.1 Teknik KLAPS

```
┌─────────────────────────────────────────────────────────────┐
│              TEKNIK KLAPS POU ENVIT                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  K - KONTÈKS                                               │
│      Bay enfòmasyon sou sitiyasyon an                      │
│      Egzanp: "Mwen ap travay sou yon aplikasyon wèb..."    │
│                                                             │
│  L - LIMIT                                                 │
│      Presize limit ak kontrènt                             │
│      Egzanp: "Repons la dwe gen mwens pase 100 mo"         │
│                                                             │
│  A - AKSYON                                                │
│      Di egzakteman kisa ou vle                             │
│      Egzanp: "Ekri yon fonksyon ki..."                     │
│                                                             │
│  P - PÈSONAJ                                               │
│      Defini wòl Claude                                     │
│      Egzanp: "Aji tankou yon ekspè sekirite..."            │
│                                                             │
│  S - SÒTI                                                  │
│      Presize fòma ou vle                                   │
│      Egzanp: "Bay repons nan fòma JSON"                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.6.2 Egzanp Teknik KLAPS

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "Ou se yon ekspè sekirite enfòmatik ki espesyalize nan Python.",
  "messages": [
    {
      "role": "user",
      "content": "KONTÈKS: Mwen ap devlope yon aplikasyon wèb ki aksepte done itilizatè.\n\nLIMIT: Solisyon an dwe senp epi fasil pou konprann.\n\nAKSYON: Ekri yon fonksyon Python ki valide yon adrès imèl.\n\nSÒTI: Bay kòd la ak eksplikasyon chak etap."
    }
  ]
}
```

### 6.6.3 Modèl Envit pou Travay Komen

**Modèl pou Analiz Done:**

```json
{
  "system": "Ou se yon analis done ki eksplike rezilta yo yon fason klè.",
  "messages": [
    {
      "role": "user",
      "content": "Analize done sa yo epi bay:\n1. Rezime estatistik\n2. Tandans prensipal\n3. Rekòmandasyon\n\nDone:\n[METE DONE ISIT]"
    }
  ]
}
```

**Modèl pou Revizyon Kòd:**

```json
{
  "system": "Ou se yon devlopè senior ki revize kòd pou kalite ak sekirite.",
  "messages": [
    {
      "role": "user",
      "content": "Revize kòd sa a epi bay:\n1. Pwoblèm ou jwenn\n2. Amelyorasyon ou sijere\n3. Bon pratik ki manke\n\nKòd:\n```python\n[METE KÒD ISIT]\n```"
    }
  ]
}
```

**Modèl pou Tradiksyon:**

```json
{
  "system": "Ou se yon tradiktè pwofesyonèl ki kenbe estil ak ton orijinal la.",
  "messages": [
    {
      "role": "user",
      "content": "Tradui tèks sa a nan [LANG SIBLE].\nKenbe fòma orijinal la.\nTèks:\n[METE TÈKS ISIT]"
    }
  ]
}
```

---

## 6.7 Limit Tarif ak Jesyon Erè

### 6.7.1 Konprann Limit Tarif

```
┌─────────────────────────────────────────────────────────────┐
│              LIMIT TARIF ANTHROPIC                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Nivo         Demann/Minit    Jeton/Minit    Jeton/Jou     │
│  ─────────────────────────────────────────────────────      │
│  Nivo 1       50              40,000         1,000,000      │
│  Nivo 2       100             80,000         2,500,000      │
│  Nivo 3       200             160,000        5,000,000      │
│  Nivo 4       400             400,000        10,000,000     │
│                                                             │
│  REMAK: Limit yo ogmante otomatikman ak itilizasyon        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.7.2 Kòd Erè Komen

| Kòd | Non | Deskripsyon | Solisyon |
|-----|-----|-------------|----------|
| 400 | Demann Pa Valid | Demann lan mal fòme | Tcheke JSON ou |
| 401 | Pa Otorize | Kle API pa valid | Verifye kle ou |
| 403 | Entèdi | Pa gen aksè | Tcheke pèmisyon |
| 404 | Pa Jwenn | Resous pa egziste | Tcheke pwen tèminal |
| 429 | Twòp Demann | Depase limit tarif | Tann epi re-eseye |
| 500 | Erè Sèvè | Pwoblèm Anthropic | Re-eseye pita |
| 529 | Sèvè Surchaje | Twòp trafik | Tann ak re-eseye |

### 6.7.3 Enplemante Jesyon Erè

```python
import anthropic
import time
from anthropic import APIError, RateLimitError, APIConnectionError

class KliyanAvèkJesyonErè:
    """
    Kliyan ki jere erè yo byen.
    """

    def __init__(self, max_tantativ=3, delè_baz=1):
        """
        Inisyalize kliyan an.

        Agiman:
            max_tantativ: Kantite maksimòm tantativ
            delè_baz: Delè inisyal ant tantativ (segonn)
        """
        self.kliyan = anthropic.Anthropic()
        self.max_tantativ = max_tantativ
        self.dele_baz = delè_baz

    def voye_ak_retantativ(self, **paramèt_yo):
        """
        Voye demann ak retantativ otomatik.
        """
        tantativ = 0
        dènye_erè = None

        while tantativ < self.max_tantativ:
            try:
                repons = self.kliyan.messages.create(**paramèt_yo)
                return repons

            except RateLimitError as e:
                tantativ += 1
                dènye_erè = e
                delè = self.dele_baz * (2 ** tantativ)  # Delè eksponansyèl
                print(f"Limit tarif ateyen. Ap tann {delè} segonn...")
                time.sleep(delè)

            except APIConnectionError as e:
                tantativ += 1
                dènye_erè = e
                delè = self.dele_baz * tantativ
                print(f"Erè koneksyon. Ap re-eseye nan {delè} segonn...")
                time.sleep(delè)

            except APIError as e:
                # Pa re-eseye pou erè 400 (demann pa valid)
                if e.status_code == 400:
                    raise e

                tantativ += 1
                dènye_erè = e
                print(f"Erè API: {e.message}")
                time.sleep(self.dele_baz)

        # Si tout tantativ echwe
        raise Exception(f"Echwe apre {self.max_tantativ} tantativ. Dènye erè: {dènye_erè}")


# Egzanp itilizasyon
kliyan = KliyanAvèkJesyonErè(max_tantativ=3, delè_baz=2)

try:
    repons = kliyan.voye_ak_retantativ(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": "Bonjou!"}
        ]
    )
    print(repons.content[0].text)

except Exception as e:
    print(f"Erè final: {e}")
```

### 6.7.4 Jesyon Erè ak cURL

```bash
#!/bin/bash

# Fonksyon pou voye demann ak retantativ
voye_demann() {
    local max_tantativ=3
    local tantativ=0
    local delè=1

    while [ $tantativ -lt $max_tantativ ]; do
        repons=$(curl -s -w "\n%{http_code}" \
            https://api.anthropic.com/v1/messages \
            --header "x-api-key: $ANTHROPIC_API_KEY" \
            --header "anthropic-version: 2023-06-01" \
            --header "content-type: application/json" \
            --data '{
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [
                    {"role": "user", "content": "Bonjou!"}
                ]
            }')

        # Separe kò repons ak kòd estati
        ko=$(echo "$repons" | head -n -1)
        kod_estati=$(echo "$repons" | tail -n 1)

        if [ "$kod_estati" -eq 200 ]; then
            echo "$ko"
            return 0
        elif [ "$kod_estati" -eq 429 ]; then
            echo "Limit tarif ateyen. Ap tann $delè segonn..."
            sleep $delè
            delè=$((delè * 2))
        else
            echo "Erè: Kòd estati $kod_estati"
            echo "$ko"
            return 1
        fi

        tantativ=$((tantativ + 1))
    done

    echo "Echwe apre $max_tantativ tantativ"
    return 1
}

# Rele fonksyon an
voye_demann
```

---

## 6.8 Egzèsis Pratik

### Egzèsis 6.1: Premye Apèl API

**Objektif:** Fè premye apèl API ou ak cURL.

**Etap:**
1. Asire ou gen kle API ou nan varyab anviwònman
2. Ekri kòmand cURL pou mande Claude eksplike kisa Python ye
3. Egzekite kòmand lan
4. Analize repons lan

**Solisyon:**
```bash
curl https://api.anthropic.com/v1/messages \
  --header "x-api-key: $ANTHROPIC_API_KEY" \
  --header "anthropic-version: 2023-06-01" \
  --header "content-type: application/json" \
  --data '{
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 500,
    "messages": [
      {
        "role": "user",
        "content": "Eksplike kisa Python ye an 3 fraz."
      }
    ]
  }'
```

### Egzèsis 6.2: Kreye Asistan Konvèsasyon

**Objektif:** Kreye yon asistan ki kenbe konvèsasyon.

**Egzijans:**
1. Asistan an dwe sonje mesaj anvan yo
2. Li dwe gen yon pèsonalite defini (pa egzanp: yon chèf kizin)
3. Enplemante fonksyon pou efase istwa

**Solisyon:**
```python
import anthropic

class ChèfKizinAsistan:
    """
    Asistan ki aji tankou yon chèf kizin Ayisyen.
    """

    def __init__(self):
        self.kliyan = anthropic.Anthropic()
        self.mesaj_yo = []
        self.sistem = """Ou se Chèf Pyè, yon chèf kizin Ayisyen ki gen 30 an eksperyans.
        Ou renmen pataje resèt tradisyonèl Ayisyen.
        Ou toujou reponn ak antouzyasm ak pasyon pou manje.
        Ou bay konsèy pratik ak engrediyan ki fasil jwenn."""

    def mande(self, kesyon):
        self.mesaj_yo.append({"role": "user", "content": kesyon})

        repons = self.kliyan.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            system=self.sistem,
            messages=self.mesaj_yo
        )

        teks = repons.content[0].text
        self.mesaj_yo.append({"role": "assistant", "content": teks})

        return teks

    def efase(self):
        self.mesaj_yo = []
        return "Istwa efase. Kòmanse nouvo konvèsasyon!"


# Teste asistan an
chef = ChèfKizinAsistan()

print(chef.mande("Bonjou Chef! Kijan pou m fè diri ak pwa nwa?"))
print("\n---\n")
print(chef.mande("Ak ki vyann li pi bon?"))
print("\n---\n")
print(chef.mande("Mèsi! Gen yon desè ou rekòmande?"))
```

### Egzèsis 6.3: Enplemante Jesyon Erè

**Objektif:** Kreye yon kliyan ki jere erè kòrèkteman.

**Egzijans:**
1. Jere erè limit tarif (429)
2. Jere erè koneksyon
3. Itilize delè eksponansyèl
4. Limite kantite tantativ

**Solisyon:** Gade seksyon 6.7.3 pi wo a pou yon egzanp konplè.

### Egzèsis 6.4: Optimize Envit

**Objektif:** Amelyore envit sa yo pou jwenn pi bon repons.

**Envit Orijinal:**
1. "Ekri kòd"
2. "Eksplike baz done"
3. "Fè tradiksyon"

**Envit Amelyore:**
1. "Ekri yon fonksyon JavaScript ki valide yon nimewo telefòn Ayisyen (fòma: +509 XXXX-XXXX). Enkli jesyon erè ak tès."

2. "Eksplike konsèp baz done relasyonèl pou yon etidyan ki fèk kòmanse aprann pwogramasyon. Itilize yon analoji ak yon bibliyotèk. Bay 2 egzanp tablo ak relasyon ant yo."

3. "Tradui tèks sa a nan Kreyòl Ayisyen. Kenbe ton fòmèl la. Adapte ekspresyon kiltirèl yo. Tèks: [METE TÈKS ISIT]"

---

## 6.9 Rezime

Nan chapit sa a, nou te aprann:

| Sijè | Pwen Kle |
|------|----------|
| Estrikti API | Pwen tèminal, antèt, kò demann |
| Premye Apèl | cURL, Python, JavaScript |
| Envit Efikas | Klète, kontèks, espesifik |
| Jere Repons | Tèks, blòk, jeton |
| Konvèsasyon | Kenbe istwa, optimize |
| Bon Pratik | Teknik KLAPS |
| Jesyon Erè | Kòd erè, retantativ, delè |

---

## 6.10 Kesyon Revizyon

1. Ki pwen tèminal API Claude itilize?
2. Ki antèt obligatwa pou demann API?
3. Ki diferans ant "system" ak "messages" nan demann lan?
4. Kijan ou jere limit tarif (erè 429)?
5. Ki teknik ou ka itilize pou ekri envit efikas?
6. Kijan ou kenbe kontèks nan yon konvèsasyon?

---

## 6.11 Pwochen Etap

Nan pwochen pati a, nou pral aprann sou Supabase:
- Kisa Supabase ye
- Konfigire baz done
- Otantifikasyon
- Fonksyon an tan reyèl
- Entegre ak n8n ak Claude

---

*Kontinye nan Pati 4: Supabase*
