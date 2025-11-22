# Kominote TWOKECRYPTO
# Chapit 9: Konekte N8N ak Claude

## Entwodiksyon

Nan chapit sa a, nou pral aprann kijan pou konekte N8N ak Claude AI pou kreye flodtravay entèlijan. Entegrasyon sa a pèmèt ou otomatize travay ki bezwen entèlijans atifisyèl.

---

## 9.1 Konfigire Akreditasyon Claude nan N8N

### Etap 1: Jwenn Kle API Claude

Pou kòmanse, ou bezwen yon kle API nan men Anthropic:

1. Ale sou sitwèb Anthropic
2. Kreye yon kont oswa konekte
3. Ale nan seksyon "Kle API"
4. Klike sou "Kreye Nouvo Kle"
5. Kopye kle a epi konsève li an sekirite

### Etap 2: Ajoute Akreditasyon nan N8N

```
┌─────────────────────────────────────────────────────┐
│                    N8N                              │
│  ┌─────────────────────────────────────────────┐   │
│  │         Paramèt Akreditasyon                │   │
│  │                                             │   │
│  │  Non: Claude API                            │   │
│  │  Tip: Otorizasyon Antèt                     │   │
│  │  Non Antèt: x-api-key                       │   │
│  │  Valè: [KLE_API_OU_A]                       │   │
│  │                                             │   │
│  │  [Sove]  [Teste Koneksyon]                  │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Enstriksyon Detaye

1. Ouvri N8N nan navigatè ou
2. Klike sou **Paramèt** nan meni prensipal la
3. Chwazi **Akreditasyon**
4. Klike sou **Ajoute Akreditasyon**
5. Chwazi **Otorizasyon Antèt**
6. Ranpli enfòmasyon yo:
   - **Non**: `Claude API`
   - **Non Antèt**: `x-api-key`
   - **Valè**: Kle API ou a

---

## 9.2 Nod Demann HTTP pou API Claude

### Konprann API Claude

API Claude aksepte demann HTTP POST ak fòma JSON. Men estrikti debaz la:

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "Mesaj ou a isit la"
    }
  ]
}
```

### Konfigirasyon Nod Demann HTTP

```
┌─────────────────────────────────────────────────────┐
│              Nod Demann HTTP                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Metòd: POST                                        │
│                                                     │
│  Adrès: https://api.anthropic.com/v1/messages      │
│                                                     │
│  Otentifikasyon: Akreditasyon Predefinisyon        │
│  Tip Akreditasyon: Otorizasyon Antèt               │
│  Akreditasyon: Claude API                          │
│                                                     │
│  Antèt Siplemantè:                                 │
│    - anthropic-version: 2023-06-01                 │
│    - content-type: application/json                │
│                                                     │
│  Kò Demann: JSON                                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Egzanp Konfigirasyon JSON Konplè

```json
{
  "nodes": [
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.anthropic.com/v1/messages",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "anthropic-version",
              "value": "2023-06-01"
            },
            {
              "name": "content-type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "{\n  \"model\": \"claude-sonnet-4-5-20250929\",\n  \"max_tokens\": 1024,\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"{{ $json.mesaj }}\"\n    }\n  ]\n}"
      },
      "name": "Demann Claude",
      "type": "n8n-nodes-base.httpRequest",
      "position": [450, 300]
    }
  ]
}
```

---

## 9.3 Kreye Flodtravay Entèlijan

### Achitekti Flodtravay AI

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Deklanchè  │───▶│  Prepare     │───▶│  Demann      │───▶│  Trete       │
│   (Kwòk Wèb) │    │  Done        │    │  Claude      │    │  Repons      │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                                                    │
                                                                    ▼
                                                            ┌──────────────┐
                                                            │  Rezilta     │
                                                            │  Final       │
                                                            └──────────────┘
```

### Egzanp: Flodtravay Analiz Tèks

Sa a se yon flodtravay ki analize santiman nan tèks:

```json
{
  "name": "Analiz Santiman",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "analize-santiman",
        "responseMode": "responseNode"
      },
      "name": "Kwòk Wèb Resevwa",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "const tèks = $input.first().json.teks;\nconst envwa = `Analize santiman tèks sa a epi reponn sèlman ak: POZITIF, NEGATIF, oswa NET. Tèks: ${tèks}`;\nreturn [{ json: { envwa } }];"
      },
      "name": "Prepare Envwa",
      "type": "n8n-nodes-base.code",
      "position": [450, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.anthropic.com/v1/messages",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "anthropic-version",
              "value": "2023-06-01"
            },
            {
              "name": "content-type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "{\n  \"model\": \"claude-sonnet-4-5-20250929\",\n  \"max_tokens\": 100,\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"{{ $json.envwa }}\"\n    }\n  ]\n}"
      },
      "name": "Demann Claude",
      "type": "n8n-nodes-base.httpRequest",
      "position": [650, 300]
    }
  ]
}
```

---

## 9.4 Trete Repons AI

### Estrikti Repons Claude

Lè Claude reponn, ou resevwa yon objè JSON tankou sa:

```json
{
  "id": "msg_123456789",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Men repons Claude a..."
    }
  ],
  "model": "claude-sonnet-4-5-20250929",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 25,
    "output_tokens": 150
  }
}
```

### Ekstrapole Repons la

Itilize yon nod Kòd pou ekstrapole tèks repons la:

```javascript
// Nod Kòd pou trete repons Claude
const repons = $input.first().json;

// Ekstrapole tèks prensipal la
const teksRepons = repons.content[0].text;

// Ekstrapole estatistik itilizasyon
const tokenAntre = repons.usage.input_tokens;
const tokenSòti = repons.usage.output_tokens;
const totalToken = tokenAntre + tokenSòti;

return [{
  json: {
    teks: teksRepons,
    estatistik: {
      tokenAntre: tokenAntre,
      tokenSòti: tokenSòti,
      total: totalToken
    },
    model: repons.model,
    id: repons.id
  }
}];
```

### Dyagram Tretman Repons

```
┌─────────────────────────────────────────────────────────────┐
│                    Repons Claude                            │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ {                                                      │ │
│  │   "content": [{"type": "text", "text": "..."}],       │ │
│  │   "usage": {"input_tokens": 25, "output_tokens": 150} │ │
│  │ }                                                      │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Nod Kòd                                  │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Ekstrapole: content[0].text                           │ │
│  │ Kalkile: total token                                  │ │
│  │ Fòmate: rezilta final                                 │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Rezilta Pwòp                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ { "teks": "...", "estatistik": {...} }                │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 9.5 Konstwi yon Flodtravay Robotkonvèsasyon Senp

### Achitekti Robotkonvèsasyon

```
                    ┌─────────────────────┐
                    │    Itilizatè        │
                    │    Voye Mesaj       │
                    └──────────┬──────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                         N8N                                 │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │ Kwòk Wèb │──▶│ Verifye  │──▶│ Demann   │──▶│ Fòmate   │ │
│  │ Resevwa  │   │ Done     │   │ Claude   │   │ Repons   │ │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘ │
│                                                      │      │
│                                                      ▼      │
│                                              ┌──────────┐   │
│                                              │ Voye     │   │
│                                              │ Repons   │   │
│                                              └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Itilizatè        │
                    │    Resevwa Repons   │
                    └─────────────────────┘
```

### Flodtravay Konplè Robotkonvèsasyon

```json
{
  "name": "Robotkonvèsasyon Claude",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "robotkonvèsasyon",
        "responseMode": "responseNode"
      },
      "name": "Resevwa Mesaj",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.mesaj }}",
              "operation": "isNotEmpty"
            }
          ]
        }
      },
      "name": "Verifye Mesaj",
      "type": "n8n-nodes-base.if",
      "position": [450, 300]
    },
    {
      "parameters": {
        "jsCode": "const mesaj = $input.first().json.mesaj;\nconst istwa = $input.first().json.istwa || [];\n\nconst mesajYo = istwa.map(m => ({\n  role: m.wòl,\n  content: m.kontni\n}));\n\nmesajYo.push({\n  role: 'user',\n  content: mesaj\n});\n\nreturn [{ json: { mesajYo } }];"
      },
      "name": "Prepare Konvèsasyon",
      "type": "n8n-nodes-base.code",
      "position": [650, 200]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.anthropic.com/v1/messages",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "anthropic-version",
              "value": "2023-06-01"
            },
            {
              "name": "content-type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "{\n  \"model\": \"claude-sonnet-4-5-20250929\",\n  \"max_tokens\": 1024,\n  \"system\": \"Ou se yon asistan itil ki reponn an Kreyòl Ayisyen.\",\n  \"messages\": {{ JSON.stringify($json.mesajYo) }}\n}"
      },
      "name": "Demann Claude",
      "type": "n8n-nodes-base.httpRequest",
      "position": [850, 200]
    },
    {
      "parameters": {
        "jsCode": "const repons = $input.first().json;\nconst teks = repons.content[0].text;\n\nreturn [{\n  json: {\n    siksè: true,\n    repons: teks,\n    timestamp: new Date().toISOString()\n  }\n}];"
      },
      "name": "Fòmate Repons",
      "type": "n8n-nodes-base.code",
      "position": [1050, 200]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ $json }}"
      },
      "name": "Voye Repons",
      "type": "n8n-nodes-base.respondToWebhook",
      "position": [1250, 200]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={\"siksè\": false, \"erè\": \"Mesaj vid pa aksepte\"}"
      },
      "name": "Repons Erè",
      "type": "n8n-nodes-base.respondToWebhook",
      "position": [650, 400]
    }
  ],
  "connections": {
    "Resevwa Mesaj": {
      "main": [[{"node": "Verifye Mesaj", "type": "main", "index": 0}]]
    },
    "Verifye Mesaj": {
      "main": [
        [{"node": "Prepare Konvèsasyon", "type": "main", "index": 0}],
        [{"node": "Repons Erè", "type": "main", "index": 0}]
      ]
    },
    "Prepare Konvèsasyon": {
      "main": [[{"node": "Demann Claude", "type": "main", "index": 0}]]
    },
    "Demann Claude": {
      "main": [[{"node": "Fòmate Repons", "type": "main", "index": 0}]]
    },
    "Fòmate Repons": {
      "main": [[{"node": "Voye Repons", "type": "main", "index": 0}]]
    }
  }
}
```

### Teste Robotkonvèsasyon an

Voye yon demann teste:

```json
{
  "mesaj": "Bonjou! Kijan ou ye?",
  "istwa": []
}
```

Repons espere:

```json
{
  "siksè": true,
  "repons": "Bonjou! Mwen byen, mèsi! Kijan mwen ka ede ou jodi a?",
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

---

## 9.6 Jere Erè

### Tip Erè API Claude

| Kòd Erè | Deskripsyon | Solisyon |
|---------|-------------|----------|
| 400 | Demann envalid | Verifye fòma JSON ou |
| 401 | Otentifikasyon echwe | Verifye kle API ou |
| 429 | Twòp demann | Tann epi eseye ankò |
| 500 | Erè sèvè | Kontakte sipò |

### Flodtravay ak Jesyon Erè

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Demann     │───▶│   Verifye    │───▶│   Siksè      │
│   Claude     │    │   Rezilta    │    │   Kontinye   │
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                           │ Erè
                           ▼
                    ┌──────────────┐    ┌──────────────┐
                    │   Analize    │───▶│   Eseye      │
                    │   Erè        │    │   Ankò       │
                    └──────────────┘    └──────────────┘
                           │
                           │ Echwe
                           ▼
                    ┌──────────────┐
                    │   Notifye    │
                    │   Admin      │
                    └──────────────┘
```

### Kòd Jesyon Erè

```javascript
// Nod Kòd pou jere erè
const repons = $input.first().json;
const kòdEta = $input.first().json.statusCode || 200;

// Verifye si demann lan reyisi
if (kòdEta >= 400) {
  const mesajErè = repons.error?.message || 'Erè enkoni';

  // Klasifye erè a
  let tipErè;
  let aksyon;

  switch (kòdEta) {
    case 400:
      tipErè = 'DEMANN_ENVALID';
      aksyon = 'Korije fòma demann lan';
      break;
    case 401:
      tipErè = 'OTENTIFIKASYON_ECHWE';
      aksyon = 'Verifye kle API';
      break;
    case 429:
      tipErè = 'LIMIT_DEPASE';
      aksyon = 'Tann 60 segonn epi eseye ankò';
      break;
    case 500:
    case 502:
    case 503:
      tipErè = 'ERÈ_SÈVÈ';
      aksyon = 'Eseye ankò pita';
      break;
    default:
      tipErè = 'ERÈ_ENKONI';
      aksyon = 'Kontakte sipò teknik';
  }

  return [{
    json: {
      siksè: false,
      kòdEta: kòdEta,
      tipErè: tipErè,
      mesaj: mesajErè,
      aksyon: aksyon,
      timestamp: new Date().toISOString()
    }
  }];
}

// Si siksè, kontinye
return [{
  json: {
    siksè: true,
    done: repons
  }
}];
```

### Mekanis Eseye Ankò

```json
{
  "name": "Eseye Ankò Claude",
  "nodes": [
    {
      "parameters": {
        "jsCode": "const maksimòmEsè = 3;\nlet esèAktyèl = $input.first().json.esèAktyèl || 0;\nesèAktyèl++;\n\nreturn [{\n  json: {\n    ....$input.first().json,\n    esèAktyèl: esèAktyèl,\n    kontinye: esèAktyèl <= maksimòmEsè\n  }\n}];"
      },
      "name": "Kontè Esè",
      "type": "n8n-nodes-base.code",
      "position": [450, 300]
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $json.kontinye }}",
              "value2": true
            }
          ]
        }
      },
      "name": "Verifye Limit",
      "type": "n8n-nodes-base.if",
      "position": [650, 300]
    },
    {
      "parameters": {
        "amount": 2,
        "unit": "seconds"
      },
      "name": "Tann",
      "type": "n8n-nodes-base.wait",
      "position": [850, 200]
    }
  ]
}
```

---

## Rezime Chapit

Nan chapit sa a, nou te aprann:

1. **Konfigire Akreditasyon**: Kijan pou ajoute kle API Claude nan N8N
2. **Nod Demann HTTP**: Kijan pou konfigure nod la pou API Claude
3. **Flodtravay AI**: Kijan pou kreye flodtravay ki itilize entèlijans atifisyèl
4. **Trete Repons**: Kijan pou ekstrapole epi fòmate repons Claude
5. **Robotkonvèsasyon**: Kijan pou konstwi yon sistèm konvèsasyon senp
6. **Jesyon Erè**: Kijan pou jere diferan erè API

### Pwen Kle

- Toujou konsève kle API ou an sekirite
- Itilize jesyon erè nan tout flodtravay ou yo
- Teste flodtravay ou yo avan ou mete yo an pwodiksyon
- Siveye itilizasyon token pou kontwole depans

### Pwochen Etap

Nan pwochen chapit la, nou pral aprann kijan pou konekte N8N ak Supabase pou estoke done yo.

---

## Egzèsis Pratik

### Egzèsis 1: Tradiktè Otomatik

Kreye yon flodtravay ki:
1. Resevwa tèks nan nenpòt lang
2. Voye li bay Claude pou tradiksyon
3. Retounen tèks tradwi an Kreyòl Ayisyen

### Egzèsis 2: Analizè Kontni

Kreye yon flodtravay ki:
1. Resevwa yon atik oswa dokiman
2. Mande Claude pou rezime li
3. Ekstrapole pwen prensipal yo
4. Retounen rezime estrikti

### Egzèsis 3: Sistèm Repons Otomatik

Kreye yon flodtravay ki:
1. Resevwa kesyon kliyan
2. Itilize Claude pou jenere repons
3. Klasifye kesyon an (teknik, faktirasyonjeneral)
4. Retounen repons pèsonalize

---

*Pwochen Chapit: Konekte N8N ak Supabase*
