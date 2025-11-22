# Kominote TWOKECRYPTO
# Chapit 10: Konekte N8N ak Supabase

## Entwodiksyon

Nan chapit sa a, nou pral aprann kijan pou konekte N8N ak Supabase pou estoke, li, mete ajou, epi efase done. Entegrasyon sa a pèmèt ou kreye flodtravay ki jere baz done ou san pwoblèm.

---

## 10.1 Konfigire Akreditasyon Supabase

### Etap 1: Jwenn Enfòmasyon Koneksyon Supabase

Pou konekte N8N ak Supabase, ou bezwen:

1. **Adrès API**: Adrès pwojè Supabase ou a
2. **Kle API**: Kle sekrè pou aksè

```
┌─────────────────────────────────────────────────────────────┐
│                    Tablo Bò Supabase                        │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  Paramèt Pwojè                                        │ │
│  │                                                        │ │
│  │  Adrès API: https://xxxxx.supabase.co                 │ │
│  │                                                        │ │
│  │  Kle Anonim (anon): eyJhbGciOiJIUzI1NiIsInR5c...      │ │
│  │                                                        │ │
│  │  Kle Sèvis (service_role): eyJhbGciOiJIUzI1Ni...      │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Etap 2: Ajoute Akreditasyon nan N8N

1. Ouvri N8N
2. Ale nan **Paramèt** > **Akreditasyon**
3. Klike sou **Ajoute Akreditasyon**
4. Chwazi **Supabase API**
5. Ranpli enfòmasyon yo:

```
┌─────────────────────────────────────────────────────────────┐
│              Akreditasyon Supabase                          │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Non: Supabase Pwojè Prensipal                        │ │
│  │                                                        │ │
│  │  Lòt Supabase:                                        │ │
│  │  https://PWOJÈ_ID_OU.supabase.co                      │ │
│  │                                                        │ │
│  │  Kle API Sèvis:                                       │ │
│  │  [KLE_SÈVIS_OU_A]                                     │ │
│  │                                                        │ │
│  │  [Sove]  [Teste Koneksyon]                            │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Tip Kle API

| Tip Kle | Itilizasyon | Nivo Aksè |
|---------|-------------|-----------|
| Anonim (anon) | Pou kliyan piblik | Limite |
| Sèvis (service_role) | Pou sèvè | Konplè |

**Enpòtan**: Toujou itilize kle sèvis nan N8N paske li bezwen aksè konplè pou operasyon yo.

---

## 10.2 Nod Supabase nan N8N

### Prezantasyon Nod Supabase

N8N gen yon nod dedye pou Supabase ki sipòte operasyon sa yo:

```
┌─────────────────────────────────────────────────────────────┐
│                    Nod Supabase                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon Disponib:                                  │ │
│  │                                                        │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │
│  │  │   Kreye     │  │    Li       │  │  Mete Ajou  │   │ │
│  │  │   (Insert)  │  │  (Select)   │  │  (Update)   │   │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘   │ │
│  │                                                        │ │
│  │  ┌─────────────┐  ┌─────────────┐                     │ │
│  │  │   Efase     │  │   Ipsèt     │                     │ │
│  │  │  (Delete)   │  │  (Upsert)   │                     │ │
│  │  └─────────────┘  └─────────────┘                     │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Konfigirasyon Nod Debaz

```json
{
  "parameters": {
    "operation": "select",
    "tableId": "non_tab_ou",
    "returnAll": true
  },
  "name": "Li Done Supabase",
  "type": "n8n-nodes-base.supabase",
  "typeVersion": 1,
  "position": [450, 300],
  "credentials": {
    "supabaseApi": {
      "id": "1",
      "name": "Supabase Pwojè Prensipal"
    }
  }
}
```

---

## 10.3 Operasyon KLAD (Kreye, Li, Ajou, Delete)

### 10.3.1 Kreye (Insert)

Ajoute nouvo anrejistreman nan yon tab:

```
┌─────────────────────────────────────────────────────────────┐
│              Operasyon Kreye                                │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon: insert                                    │ │
│  │  Tab: itilizatè                                       │ │
│  │                                                        │ │
│  │  Done pou Ajoute:                                     │ │
│  │  {                                                     │ │
│  │    "non": "Jan Pyè",                                  │ │
│  │    "imèl": "jan@egzanp.com",                          │ │
│  │    "aktif": true                                      │ │
│  │  }                                                     │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Egzanp JSON Flodtravay:**

```json
{
  "nodes": [
    {
      "parameters": {
        "operation": "insert",
        "tableId": "itilizatè",
        "fieldsToSend": "defineBelow",
        "fieldProperties": {
          "fieldValues": [
            {
              "fieldName": "non",
              "fieldValue": "={{ $json.non }}"
            },
            {
              "fieldName": "imèl",
              "fieldValue": "={{ $json.imèl }}"
            },
            {
              "fieldName": "aktif",
              "fieldValue": true
            }
          ]
        }
      },
      "name": "Kreye Itilizatè",
      "type": "n8n-nodes-base.supabase",
      "position": [450, 300]
    }
  ]
}
```

### 10.3.2 Li (Select)

Li done nan yon tab:

```
┌─────────────────────────────────────────────────────────────┐
│              Operasyon Li                                   │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon: select                                    │ │
│  │  Tab: itilizatè                                       │ │
│  │                                                        │ │
│  │  Opsyon:                                              │ │
│  │  - Retounen Tout: Wi                                  │ │
│  │  - Filtre: aktif = true                               │ │
│  │  - Òd: kreye_a (desandan)                             │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Egzanp JSON Flodtravay:**

```json
{
  "nodes": [
    {
      "parameters": {
        "operation": "select",
        "tableId": "itilizatè",
        "returnAll": false,
        "limit": 50,
        "filterType": "string",
        "filterString": "aktif=eq.true",
        "options": {
          "orderBy": {
            "values": [
              {
                "column": "kreye_a",
                "direction": "desc"
              }
            ]
          }
        }
      },
      "name": "Li Itilizatè Aktif",
      "type": "n8n-nodes-base.supabase",
      "position": [450, 300]
    }
  ]
}
```

### Operatè Filtre Supabase

| Operatè | Deskripsyon | Egzanp |
|---------|-------------|--------|
| eq | Egal | `kolòn=eq.valè` |
| neq | Pa egal | `kolòn=neq.valè` |
| gt | Plis pase | `laj=gt.18` |
| lt | Mwens pase | `pri=lt.100` |
| gte | Plis oswa egal | `laj=gte.18` |
| lte | Mwens oswa egal | `pri=lte.100` |
| like | Sanble (pasyèl) | `non=like.*Jan*` |
| ilike | Sanble (san majiskil) | `non=ilike.*jan*` |
| is | Verite/Fo/Nil | `aktif=is.true` |
| in | Nan lis | `eta=in.(aktif,pandye)` |

### 10.3.3 Mete Ajou (Update)

Mete ajou anrejistreman ki egziste deja:

```
┌─────────────────────────────────────────────────────────────┐
│              Operasyon Mete Ajou                            │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon: update                                    │ │
│  │  Tab: itilizatè                                       │ │
│  │                                                        │ │
│  │  Filtre: id = 123                                     │ │
│  │                                                        │ │
│  │  Done pou Mete Ajou:                                  │ │
│  │  {                                                     │ │
│  │    "aktif": false,                                    │ │
│  │    "mete_ajou_a": "2024-01-15T10:00:00Z"              │ │
│  │  }                                                     │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Egzanp JSON Flodtravay:**

```json
{
  "nodes": [
    {
      "parameters": {
        "operation": "update",
        "tableId": "itilizatè",
        "filterType": "string",
        "filterString": "id=eq.{{ $json.id }}",
        "fieldsToSend": "defineBelow",
        "fieldProperties": {
          "fieldValues": [
            {
              "fieldName": "aktif",
              "fieldValue": false
            },
            {
              "fieldName": "mete_ajou_a",
              "fieldValue": "={{ $now.toISO() }}"
            }
          ]
        }
      },
      "name": "Dezaktive Itilizatè",
      "type": "n8n-nodes-base.supabase",
      "position": [450, 300]
    }
  ]
}
```

### 10.3.4 Efase (Delete)

Efase anrejistreman nan yon tab:

```
┌─────────────────────────────────────────────────────────────┐
│              Operasyon Efase                                │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon: delete                                    │ │
│  │  Tab: itilizatè                                       │ │
│  │                                                        │ │
│  │  Filtre: id = 123                                     │ │
│  │                                                        │ │
│  │  ⚠️ ATANSYON: Aksyon sa a pa ka defèt!               │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Egzanp JSON Flodtravay:**

```json
{
  "nodes": [
    {
      "parameters": {
        "operation": "delete",
        "tableId": "itilizatè",
        "filterType": "string",
        "filterString": "id=eq.{{ $json.id }}"
      },
      "name": "Efase Itilizatè",
      "type": "n8n-nodes-base.supabase",
      "position": [450, 300]
    }
  ]
}
```

### 10.3.5 Ipsèt (Upsert)

Kreye oswa mete ajou otomatikman:

```
┌─────────────────────────────────────────────────────────────┐
│              Operasyon Ipsèt                                │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │  Operasyon: upsert                                    │ │
│  │  Tab: itilizatè                                       │ │
│  │                                                        │ │
│  │  Konpòtman:                                           │ │
│  │  - Si anrejistreman egziste: Mete Ajou               │ │
│  │  - Si anrejistreman pa egziste: Kreye                │ │
│  │                                                        │ │
│  │  Kolòn Konfli: id                                     │ │
│  │                                                        │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Egzanp JSON Flodtravay:**

```json
{
  "nodes": [
    {
      "parameters": {
        "operation": "upsert",
        "tableId": "itilizatè",
        "fieldsToSend": "defineBelow",
        "fieldProperties": {
          "fieldValues": [
            {
              "fieldName": "id",
              "fieldValue": "={{ $json.id }}"
            },
            {
              "fieldName": "non",
              "fieldValue": "={{ $json.non }}"
            },
            {
              "fieldName": "imèl",
              "fieldValue": "={{ $json.imèl }}"
            }
          ]
        },
        "options": {
          "onConflict": "id"
        }
      },
      "name": "Ipsèt Itilizatè",
      "type": "n8n-nodes-base.supabase",
      "position": [450, 300]
    }
  ]
}
```

---

## 10.4 Deklanche Flodtravay Depi Supabase

### Itilize Kwòk Wèb Baz Done

Supabase ka voye notifikasyon lè done chanje. Men kijan pou konfigire li:

```
┌─────────────────────────────────────────────────────────────┐
│                    Achitekti Kwòk Wèb                       │
│                                                             │
│  ┌─────────────┐         ┌─────────────┐         ┌───────┐ │
│  │  Supabase   │  Kwòk   │   Fonksyon  │  Demann │  N8N  │ │
│  │  Tab       │────────▶│   Edge      │────────▶│ Kwòk  │ │
│  │  (Chanjman) │  Baz    │             │  HTTP   │ Wèb   │ │
│  └─────────────┘  Done   └─────────────┘         └───────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Etap 1: Kreye Fonksyon Edge nan Supabase

```sql
-- Kreye fonksyon pou voye notifikasyon
CREATE OR REPLACE FUNCTION notifye_n8n()
RETURNS TRIGGER AS $$
DECLARE
  kò_demann JSONB;
BEGIN
  kò_demann := jsonb_build_object(
    'tab', TG_TABLE_NAME,
    'operasyon', TG_OP,
    'ansyen_done', CASE WHEN TG_OP = 'DELETE' THEN row_to_json(OLD) ELSE NULL END,
    'nouvo_done', CASE WHEN TG_OP != 'DELETE' THEN row_to_json(NEW) ELSE NULL END,
    'timestamp', NOW()
  );

  PERFORM net.http_post(
    url := 'https://n8n-ou.com/kwok-web/supabase-chanjman',
    body := kò_demann::TEXT,
    headers := '{"Content-Type": "application/json"}'::JSONB
  );

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### Etap 2: Kreye Deklanchè sou Tab

```sql
-- Deklanchè pou tab itilizatè
CREATE TRIGGER deklanchè_itilizatè_n8n
AFTER INSERT OR UPDATE OR DELETE ON itilizatè
FOR EACH ROW
EXECUTE FUNCTION notifye_n8n();
```

### Etap 3: Konfigire Kwòk Wèb nan N8N

```json
{
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "supabase-chanjman",
        "responseMode": "onReceived",
        "options": {}
      },
      "name": "Resevwa Chanjman Supabase",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.operasyon }}",
              "operation": "equals",
              "value2": "INSERT"
            }
          ]
        }
      },
      "name": "Verifye Operasyon",
      "type": "n8n-nodes-base.switch",
      "position": [450, 300]
    }
  ]
}
```

### Dyagram Konplè

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SUPABASE                                    │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐                │
│  │   Tab      │───▶│ Deklanchè  │───▶│  Fonksyon  │                │
│  │ itilizatè  │    │            │    │ notifye_n8n│                │
│  └────────────┘    └────────────┘    └────────────┘                │
│                                             │                       │
└─────────────────────────────────────────────│───────────────────────┘
                                              │
                                              │ HTTP POST
                                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                            N8N                                      │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌─────────┐ │
│  │  Kwòk Wèb  │───▶│  Switch    │───▶│  Trete     │───▶│ Aksyon  │ │
│  │  Resevwa   │    │ Operasyon  │    │  Done      │    │ Final   │ │
│  └────────────┘    └────────────┘    └────────────┘    └─────────┘ │
│                          │                                          │
│                          ├──▶ INSERT ──▶ Voye Imèl Byenveni        │
│                          ├──▶ UPDATE ──▶ Notifye Admin             │
│                          └──▶ DELETE ──▶ Achive Done               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.5 Entegrasyon Tan Reyèl

### Itilize Supabase Realtime ak N8N

Supabase ofri fonksyonalite tan reyèl pou koute chanjman nan baz done. Men kijan pou entegre li ak N8N:

### Achitekti Tan Reyèl

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Achitekti Tan Reyèl                              │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                      Supabase                                 │  │
│  │  ┌─────────────┐         ┌─────────────────────────────────┐ │  │
│  │  │ Baz Done    │────────▶│ Kanal Tan Reyèl                 │ │  │
│  │  │ PostgreSQL  │         │ (WebSocket)                      │ │  │
│  │  └─────────────┘         └─────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                    │                                │
│                                    │ Evènman                        │
│                                    ▼                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    Sèvè Entèmedyè                            │  │
│  │  ┌─────────────┐         ┌─────────────────────────────────┐ │  │
│  │  │ Koute       │────────▶│ Voye bay N8N                    │ │  │
│  │  │ Evènman     │         │ (HTTP POST)                      │ │  │
│  │  └─────────────┘         └─────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                    │                                │
│                                    ▼                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                         N8N                                   │  │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │  │
│  │  │ Kwòk Wèb    │───▶│ Trete       │───▶│ Aksyon      │      │  │
│  │  └─────────────┘    └─────────────┘    └─────────────┘      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Kòd Sèvè Entèmedyè (JavaScript)

```javascript
// sèvè-tan-reyèl.js
const { createClient } = require('@supabase/supabase-js');
const axios = require('axios');

const supabaseUrl = 'https://PWOJÈ_ID_OU.supabase.co';
const supabaseKey = 'KLE_SÈVIS_OU';
const n8nWebhookUrl = 'https://n8n-ou.com/kwok-web/tan-reyèl';

const supabase = createClient(supabaseUrl, supabaseKey);

// Koute chanjman nan tab itilizatè
const kanal = supabase
  .channel('chanjman-itilizatè')
  .on(
    'postgres_changes',
    {
      event: '*',
      schema: 'public',
      table: 'itilizatè'
    },
    async (payload) => {
      console.log('Chanjman resevwa:', payload);

      // Voye bay N8N
      try {
        await axios.post(n8nWebhookUrl, {
          tip: payload.eventType,
          tab: payload.table,
          ansyen: payload.old,
          nouvo: payload.new,
          timestamp: new Date().toISOString()
        });
        console.log('Notifikasyon voye bay N8N');
      } catch (erè) {
        console.error('Erè voye notifikasyon:', erè.message);
      }
    }
  )
  .subscribe();

console.log('Koute chanjman tan reyèl...');
```

### Flodtravay N8N pou Tan Reyèl

```json
{
  "name": "Trete Evènman Tan Reyèl",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "tan-reyèl",
        "responseMode": "onReceived"
      },
      "name": "Resevwa Evènman",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "rules": {
          "rules": [
            {
              "operation": "equal",
              "value1": "={{ $json.tip }}",
              "value2": "INSERT"
            },
            {
              "operation": "equal",
              "value1": "={{ $json.tip }}",
              "value2": "UPDATE"
            },
            {
              "operation": "equal",
              "value1": "={{ $json.tip }}",
              "value2": "DELETE"
            }
          ]
        }
      },
      "name": "Wout pa Tip",
      "type": "n8n-nodes-base.switch",
      "position": [450, 300]
    },
    {
      "parameters": {
        "jsCode": "const done = $input.first().json;\n\n// Trete nouvo itilizatè\nconst mesaj = `Nouvo itilizatè kreye: ${done.nouvo.non} (${done.nouvo.imèl})`;\n\nreturn [{\n  json: {\n    mesaj: mesaj,\n    itilizatè: done.nouvo\n  }\n}];"
      },
      "name": "Trete Nouvo Itilizatè",
      "type": "n8n-nodes-base.code",
      "position": [650, 200]
    },
    {
      "parameters": {
        "jsCode": "const done = $input.first().json;\n\n// Trete mizajou\nconst chanjman = {};\nfor (const kle in done.nouvo) {\n  if (done.ansyen[kle] !== done.nouvo[kle]) {\n    chanjman[kle] = {\n      ansyen: done.ansyen[kle],\n      nouvo: done.nouvo[kle]\n    };\n  }\n}\n\nreturn [{\n  json: {\n    mesaj: 'Itilizatè mete ajou',\n    chanjman: chanjman\n  }\n}];"
      },
      "name": "Trete Mizajou",
      "type": "n8n-nodes-base.code",
      "position": [650, 300]
    },
    {
      "parameters": {
        "jsCode": "const done = $input.first().json;\n\n// Trete sipresyon\nconst mesaj = `Itilizatè efase: ${done.ansyen.non}`;\n\nreturn [{\n  json: {\n    mesaj: mesaj,\n    itilizatè_efase: done.ansyen\n  }\n}];"
      },
      "name": "Trete Sipresyon",
      "type": "n8n-nodes-base.code",
      "position": [650, 400]
    }
  ],
  "connections": {
    "Resevwa Evènman": {
      "main": [[{"node": "Wout pa Tip", "type": "main", "index": 0}]]
    },
    "Wout pa Tip": {
      "main": [
        [{"node": "Trete Nouvo Itilizatè", "type": "main", "index": 0}],
        [{"node": "Trete Mizajou", "type": "main", "index": 0}],
        [{"node": "Trete Sipresyon", "type": "main", "index": 0}]
      ]
    }
  }
}
```

---

## Rezime Chapit

Nan chapit sa a, nou te aprann:

1. **Konfigire Akreditasyon**: Kijan pou konekte N8N ak Supabase
2. **Nod Supabase**: Tout operasyon disponib nan nod la
3. **Operasyon KLAD**: Kreye, Li, Mete Ajou, epi Efase done
4. **Deklanche Flodtravay**: Kijan pou deklanche flodtravay lè done chanje
5. **Entegrasyon Tan Reyèl**: Kijan pou koute chanjman an tan reyèl

### Tablo Rezime Operasyon

| Operasyon | Itilizasyon | Nod N8N |
|-----------|-------------|---------|
| Kreye | Ajoute nouvo done | Supabase (insert) |
| Li | Rekipere done | Supabase (select) |
| Mete Ajou | Modifye done egzistan | Supabase (update) |
| Efase | Retire done | Supabase (delete) |
| Ipsèt | Kreye oswa mete ajou | Supabase (upsert) |

### Pwen Kle

- Itilize kle sèvis pou aksè konplè nan N8N
- Toujou mete filtre lè w ap mete ajou oswa efase
- Konsidere sekirite lè w ap ekspoze kwòk wèb
- Teste operasyon yo nan anviwònman devlopman anvan pwodiksyon

### Pwochen Etap

Nan pwochen chapit la, nou pral konbine tout sa nou aprann pou kreye yon entegrasyon konplè ant N8N, Claude, ak Supabase.

---

## Egzèsis Pratik

### Egzèsis 1: Sistèm Enskripsyon

Kreye yon flodtravay ki:
1. Resevwa done enskripsyon nan yon kwòk wèb
2. Verifye si imèl la pa deja egziste
3. Kreye nouvo itilizatè nan Supabase
4. Voye imèl konfirmasyon

### Egzèsis 2: Sinkronizasyon Done

Kreye yon flodtravay ki:
1. Koute chanjman nan yon tab Supabase
2. Sinkronize chanjman yo ak yon lòt sèvis
3. Kenbe yon jounal tout chanjman yo

### Egzèsis 3: Rapò Otomatik

Kreye yon flodtravay ki:
1. Fonksyone chak jou a 8è nan maten
2. Li estatistik nan Supabase
3. Jenere yon rapò
4. Voye rapò a pa imèl

---

*Pwochen Chapit: Entegrasyon Konplè*
