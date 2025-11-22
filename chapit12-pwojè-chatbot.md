# Kominote TWOKECRYPTO
# Chapit 12: Pwojè Chatbot Entèlijan

## Entwodiksyon

Nan chapit sa a, nou pral konstwi yon chatbot entèlijan konplè ki itilize pouvwa n8n, Claude, ak Supabase. Chatbot sa a pral kapab:

- Resevwa mesaj itilizatè yo
- Konprann kontèks konvèsasyon an
- Bay repons entèlijan ak Claude
- Estoke tout istwa konvèsasyon nan Supabase
- Aprann preferans itilizatè yo

Sa a se yon pwojè konplè pou debitant ki vle konprann kijan pou yo bati aplikasyon entèlijan.

---

## 12.1 Apèsi Pwojè a

### 12.1.1 Objektif Pwojè a

```
╔══════════════════════════════════════════════════════════════════╗
║                    CHATBOT ENTÈLIJAN                             ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         ║
║   │  ITILIZATÈ  │───▶│   N8N       │───▶│   CLAUDE    │         ║
║   │   (Mesaj)   │    │  (Flwo)     │    │   (Repons)  │         ║
║   └─────────────┘    └─────────────┘    └─────────────┘         ║
║         ▲                  │                   │                 ║
║         │                  ▼                   │                 ║
║         │            ┌─────────────┐           │                 ║
║         └────────────│  SUPABASE   │◀──────────┘                 ║
║                      │  (Done)     │                             ║
║                      └─────────────┘                             ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

### 12.1.2 Fonksyonalite Prensipal

| Fonksyonalite | Deskripsyon |
|---------------|-------------|
| Resepsyon Mesaj | Aksepte mesaj nan diferan fòma |
| Konpreyansyon Kontèks | Sonje konvèsasyon anvan yo |
| Repons Entèlijan | Itilize Claude pou repons natirèl |
| Estokaj Done | Konsève tout done nan Supabase |
| Jesyon Itilizatè | Swiv chak itilizatè separeman |

### 12.1.3 Achitekti Jeneral

```
╔═══════════════════════════════════════════════════════════════════════╗
║                      ACHITEKTI CHATBOT                                ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │                        KOUCH PREZANTASYON                        │  ║
║  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  ║
║  │  │  Sitwèb  │  │ Telegram │  │  Slack   │  │   API    │        │  ║
║  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │  ║
║  └───────┼─────────────┼─────────────┼─────────────┼───────────────┘  ║
║          │             │             │             │                  ║
║          ▼             ▼             ▼             ▼                  ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │                        KOUCH N8N                                 │  ║
║  │  ┌──────────────────────────────────────────────────────────┐   │  ║
║  │  │  Webhook ──▶ Pwosesis ──▶ Claude ──▶ Repons             │   │  ║
║  │  └──────────────────────────────────────────────────────────┘   │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                               │                                       ║
║                               ▼                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │                        KOUCH DONE                                │  ║
║  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  ║
║  │  │Itilizatè │  │Konvèsasyon│ │  Mesaj   │  │ Paramèt  │        │  ║
║  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## 12.2 Konsepsyon Baz Done

### 12.2.1 Dyagram Relasyon Antite

```
╔═══════════════════════════════════════════════════════════════════════╗
║                    DYAGRAM BAZ DONE                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌───────────────────┐         ┌───────────────────┐                ║
║   │    ITILIZATÈ      │         │   KONVÈSASYON     │                ║
║   ├───────────────────┤         ├───────────────────┤                ║
║   │ id (KLE PRENSIPAL)│◀────────│ itilizatè_id (KLE │                ║
║   │ non               │    1:N  │    ETRANJÈ)       │                ║
║   │ imèl              │         │ id (KLE PRENSIPAL)│                ║
║   │ dat_kreye         │         │ tit               │                ║
║   │ dènye_aktivite    │         │ estati            │                ║
║   │ paramèt           │         │ dat_kreye         │                ║
║   └───────────────────┘         │ dat_fini          │                ║
║                                 └─────────┬─────────┘                ║
║                                           │                          ║
║                                           │ 1:N                      ║
║                                           ▼                          ║
║                                 ┌───────────────────┐                ║
║                                 │      MESAJ        │                ║
║                                 ├───────────────────┤                ║
║                                 │ id (KLE PRENSIPAL)│                ║
║                                 │ konvèsasyon_id    │                ║
║                                 │ wòl               │                ║
║                                 │ kontni            │                ║
║                                 │ dat_kreye         │                ║
║                                 │ metadata          │                ║
║                                 └───────────────────┘                ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 12.2.2 Deskripsyon Tab yo

#### Tab Itilizatè

| Kolòn | Tip | Deskripsyon |
|-------|-----|-------------|
| id | IDANTIFYAN INIK | Idantifyan inik itilizatè a |
| non | TÈKS | Non konplè itilizatè a |
| imèl | TÈKS | Adrès imèl inik |
| dat_kreye | HORODATAJ | Dat kreye kont lan |
| dènye_aktivite | HORODATAJ | Dènye fwa itilizatè a te aktif |
| paramèt | JSON | Preferans itilizatè a |

#### Tab Konvèsasyon

| Kolòn | Tip | Deskripsyon |
|-------|-----|-------------|
| id | IDANTIFYAN INIK | Idantifyan inik konvèsasyon an |
| itilizatè_id | IDANTIFYAN INIK | Referans itilizatè a |
| tit | TÈKS | Tit konvèsasyon an |
| estati | TÈKS | Estati aktyèl (aktif, fèmen, ann atant) |
| dat_kreye | HORODATAJ | Dat konvèsasyon an te kòmanse |
| dat_fini | HORODATAJ | Dat konvèsasyon an te fini |

#### Tab Mesaj

| Kolòn | Tip | Deskripsyon |
|-------|-----|-------------|
| id | IDANTIFYAN INIK | Idantifyan inik mesaj la |
| konvèsasyon_id | IDANTIFYAN INIK | Referans konvèsasyon an |
| wòl | TÈKS | Ki moun ki voye mesaj la (itilizatè, asistan) |
| kontni | TÈKS | Kontni mesaj la |
| dat_kreye | HORODATAJ | Dat mesaj la te kreye |
| metadata | JSON | Enfòmasyon siplemantè |

---

## 12.3 Konfigirasyon Tab Supabase

### 12.3.1 Kreye Tab Itilizatè

```sql
-- Kreye tab pou estoke enfòmasyon itilizatè yo
KREYE TAB SI PA EGZISTE itilizatè (
    -- Idantifyan inik ki jenere otomatikman
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Enfòmasyon debaz itilizatè a
    non TÈKS PA NIL,
    imèl TÈKS INIK PA NIL,

    -- Dat yo
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dènye_aktivite HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),

    -- Paramèt an fòma JSON
    paramèt JSONB DEFO '{
        "lang": "ht",
        "notifikasyon": vré,
        "tèm": "klè"
    }'::jsonb,

    -- Kontrènt validasyon
    KONTRÈNT imèl_valab TCHEKE (imèl ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Kreye endèks pou rechèch rapid
KREYE ENDÈKS endèks_itilizatè_imèl SOU itilizatè(imèl);
KREYE ENDÈKS endèks_itilizatè_dat_kreye SOU itilizatè(dat_kreye);

-- Ajoute kòmantè
KÒMANTE SOU TAB itilizatè SE 'Tab ki estoke tout enfòmasyon itilizatè chatbot la';
```

### 12.3.2 Kreye Tab Konvèsasyon

```sql
-- Kreye tab pou estoke konvèsasyon yo
KREYE TAB SI PA EGZISTE konvèsasyon (
    -- Idantifyan inik
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Referans itilizatè a
    itilizatè_id IDANTIFYAN_INIK PA NIL REFERANS itilizatè(id)
        SOU EFASE KASKAD,

    -- Enfòmasyon konvèsasyon an
    tit TÈKS DEFO 'Nouvo Konvèsasyon',
    estati TÈKS DEFO 'aktif' TCHEKE (estati NAN ('aktif', 'fèmen', 'ann_atant')),

    -- Dat yo
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dat_fini HORODATAJ AK ZÒN_LÈ,

    -- Metadata siplemantè
    metadata JSONB DEFO '{}'::jsonb
);

-- Kreye endèks yo
KREYE ENDÈKS endèks_konvèsasyon_itilizatè SOU konvèsasyon(itilizatè_id);
KREYE ENDÈKS endèks_konvèsasyon_estati SOU konvèsasyon(estati);
KREYE ENDÈKS endèks_konvèsasyon_dat SOU konvèsasyon(dat_kreye DESANDAN);

-- Ajoute kòmantè
KÒMANTE SOU TAB konvèsasyon SE 'Tab ki estoke tout konvèsasyon chatbot la';
```

### 12.3.3 Kreye Tab Mesaj

```sql
-- Kreye tab pou estoke mesaj yo
KREYE TAB SI PA EGZISTE mesaj (
    -- Idantifyan inik
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Referans konvèsasyon an
    konvèsasyon_id IDANTIFYAN_INIK PA NIL REFERANS konvèsasyon(id)
        SOU EFASE KASKAD,

    -- Enfòmasyon mesaj la
    wòl TÈKS PA NIL TCHEKE (wòl NAN ('itilizatè', 'asistan', 'sistèm')),
    kontni TÈKS PA NIL,

    -- Dat
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),

    -- Metadata (kantite token, tan repons, elatriye)
    metadata JSONB DEFO '{}'::jsonb
);

-- Kreye endèks yo
KREYE ENDÈKS endèks_mesaj_konvèsasyon SOU mesaj(konvèsasyon_id);
KREYE ENDÈKS endèks_mesaj_wòl SOU mesaj(wòl);
KREYE ENDÈKS endèks_mesaj_dat SOU mesaj(dat_kreye);

-- Ajoute kòmantè
KÒMANTE SOU TAB mesaj SE 'Tab ki estoke tout mesaj nan konvèsasyon yo';
```

### 12.3.4 Kreye Fonksyon Itil

```sql
-- Fonksyon pou jwenn dènye mesaj nan yon konvèsasyon
KREYE OSWA RANPLASE FONKSYON jwenn_dènye_mesaj(
    p_konvèsasyon_id IDANTIFYAN_INIK,
    p_limit ANTYE DEFO 10
)
RETOUNEN TAB (
    id IDANTIFYAN_INIK,
    wòl TÈKS,
    kontni TÈKS,
    dat_kreye HORODATAJ AK ZÒN_LÈ
)
LANG sql
KÒM $$
    CHWAZI id, wòl, kontni, dat_kreye
    DEPI mesaj
    KOTE konvèsasyon_id = p_konvèsasyon_id
    ÒDONE PA dat_kreye DESANDAN
    LIMIT p_limit;
$$;

-- Fonksyon pou konte mesaj nan yon konvèsasyon
KREYE OSWA RANPLASE FONKSYON konte_mesaj_konvèsasyon(
    p_konvèsasyon_id IDANTIFYAN_INIK
)
RETOUNEN ANTYE
LANG sql
KÒM $$
    CHWAZI KONTE(*)
    DEPI mesaj
    KOTE konvèsasyon_id = p_konvèsasyon_id;
$$;

-- Fonksyon pou mete ajou dènye aktivite itilizatè a
KREYE OSWA RANPLASE FONKSYON mete_ajou_dènye_aktivite()
RETOUNEN DEKLANCHÈ
LANG plpgsql
KÒM $$
KÒMANSE
    METE_AJOU itilizatè
    ETABLI dènye_aktivite = KOUNYE_A()
    KOTE id = (
        CHWAZI itilizatè_id
        DEPI konvèsasyon
        KOTE id = NOUVO.konvèsasyon_id
    );
    RETOUNEN NOUVO;
FEN;
$$;

-- Kreye deklanchè pou mete ajou aktivite
KREYE DEKLANCHÈ deklanchè_mete_ajou_aktivite
APRE ENSERE SOU mesaj
POU CHAK RANJE
EGZEKITE FONKSYON mete_ajou_dènye_aktivite();
```

### 12.3.5 Konfigirasyon Sekirite

```sql
-- Aktive Sekirite Nivo Ranje
ALTERE TAB itilizatè AKTIVE SEKIRITE NIVO RANJE;
ALTERE TAB konvèsasyon AKTIVE SEKIRITE NIVO RANJE;
ALTERE TAB mesaj AKTIVE SEKIRITE NIVO RANJE;

-- Politik pou itilizatè otantifye
KREYE POLITIK itilizatè_politik SOU itilizatè
    POU TOUT
    ITILIZE (auth.uid() = id);

KREYE POLITIK konvèsasyon_politik SOU konvèsasyon
    POU TOUT
    ITILIZE (itilizatè_id = auth.uid());

KREYE POLITIK mesaj_politik SOU mesaj
    POU TOUT
    ITILIZE (
        konvèsasyon_id NAN (
            CHWAZI id DEPI konvèsasyon
            KOTE itilizatè_id = auth.uid()
        )
    );
```

---

## 12.4 Kreye Flwo n8n

### 12.4.1 Apèsi Flwo Prensipal

```
╔═══════════════════════════════════════════════════════════════════════╗
║                        FLWO CHATBOT N8N                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          ║
║   │ WEBHOOK │───▶│ VERIFYE │───▶│  JWENN  │───▶│ PREPARE │          ║
║   │  RESEVWA│    │ITILIZATÈ│    │ ISTWA   │    │ KONTÈKS │          ║
║   └─────────┘    └─────────┘    └─────────┘    └─────────┘          ║
║                                                      │               ║
║                                                      ▼               ║
║   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          ║
║   │ RETOUNEN│◀───│  ESTOKE │◀───│ VOYE A  │◀───│ KREYE   │          ║
║   │ REPONS  │    │  MESAJ  │    │  CLAUDE │    │ DEMAND  │          ║
║   └─────────┘    └─────────┘    └─────────┘    └─────────┘          ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 12.4.2 Konfigirasyon Webhook

```json
{
  "ne": "Webhook Resevwa Mesaj",
  "tip": "n8n-nodes-base.webhook",
  "pozisyon": [250, 300],
  "paramèt": {
    "chemen": "chatbot-mesaj",
    "metòd": "POST",
    "otantifikasyon": "headerAuth",
    "opsyon": {
      "responseMode": "lastNode",
      "responseData": "allEntries"
    }
  },
  "kalifikasyon": {
    "httpHeaderAuth": {
      "id": "kalifikasyon-otantifikasyon-antèt",
      "non": "Otantifikasyon Chatbot"
    }
  }
}
```

### 12.4.3 Ne Verifye Itilizatè

```json
{
  "ne": "Verifye oswa Kreye Itilizatè",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [450, 300],
  "paramèt": {
    "operasyon": "upsert",
    "tab": "itilizatè",
    "kolòn_konfli": "imèl",
    "done": {
      "imèl": "={{ $json.itilizatè_imèl }}",
      "non": "={{ $json.itilizatè_non }}",
      "dènye_aktivite": "={{ $now.toISO() }}"
    },
    "opsyon": {
      "retounen": "representation"
    }
  }
}
```

### 12.4.4 Ne Jwenn Istwa Konvèsasyon

```json
{
  "ne": "Jwenn Istwa Konvèsasyon",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [650, 300],
  "paramèt": {
    "operasyon": "select",
    "tab": "mesaj",
    "retounen_tout": false,
    "limit": 20,
    "filt": {
      "konvèsasyon_id": "={{ $json.konvèsasyon_id }}"
    },
    "òdone": {
      "kolòn": "dat_kreye",
      "direksyon": "ASC"
    }
  }
}
```

### 12.4.5 Ne Prepare Kontèks

```json
{
  "ne": "Prepare Kontèks pou Claude",
  "tip": "n8n-nodes-base.code",
  "pozisyon": [850, 300],
  "paramèt": {
    "kòd": "// Jwenn done yo\nconst istwa = $input.all();\nconst mesaj_nouvo = $('Webhook Resevwa Mesaj').item.json.mesaj;\n\n// Prepare mesaj yo nan fòma Claude\nconst mesaj_fòmate = istwa.map(m => ({\n  wòl: m.json.wòl === 'itilizatè' ? 'user' : 'assistant',\n  kontni: m.json.kontni\n}));\n\n// Ajoute nouvo mesaj la\nmesaj_fòmate.push({\n  wòl: 'user',\n  kontni: mesaj_nouvo\n});\n\n// Kreye enstriksyon sistèm nan\nconst enstriksyon_sistèm = `Ou se yon asistan entèlijan ki pale Kreyòl Ayisyen.\nOu ede itilizatè yo ak tout kesyon yo genyen.\nReponn toujou nan yon fason janti ak itil.\nSi ou pa konnen repons lan, di sa onetman.`;\n\nreturn [{\n  json: {\n    enstriksyon_sistèm: enstriksyon_sistèm,\n    mesaj: mesaj_fòmate,\n    mesaj_orijinal: mesaj_nouvo\n  }\n}];"
  }
}
```

### 12.4.6 Ne Entegrasyon Claude

```json
{
  "ne": "Voye a Claude",
  "tip": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
  "pozisyon": [1050, 300],
  "paramèt": {
    "modèl": "claude-sonnet-4-20250514",
    "opsyon": {
      "tanperati": 0.7,
      "max_token": 1024,
      "enstriksyon_sistèm": "={{ $json.enstriksyon_sistèm }}"
    },
    "mesaj": "={{ $json.mesaj }}"
  },
  "kalifikasyon": {
    "anthropicApi": {
      "id": "kalifikasyon-claude-api",
      "non": "Claude API"
    }
  }
}
```

### 12.4.7 Ne Estoke Mesaj Itilizatè

```json
{
  "ne": "Estoke Mesaj Itilizatè",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [1250, 200],
  "paramèt": {
    "operasyon": "insert",
    "tab": "mesaj",
    "done": {
      "konvèsasyon_id": "={{ $('Webhook Resevwa Mesaj').item.json.konvèsasyon_id }}",
      "wòl": "itilizatè",
      "kontni": "={{ $('Prepare Kontèks pou Claude').item.json.mesaj_orijinal }}",
      "metadata": {
        "sous": "webhook",
        "dat_pwosesis": "={{ $now.toISO() }}"
      }
    }
  }
}
```

### 12.4.8 Ne Estoke Repons Claude

```json
{
  "ne": "Estoke Repons Claude",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [1250, 400],
  "paramèt": {
    "operasyon": "insert",
    "tab": "mesaj",
    "done": {
      "konvèsasyon_id": "={{ $('Webhook Resevwa Mesaj').item.json.konvèsasyon_id }}",
      "wòl": "asistan",
      "kontni": "={{ $json.repons }}",
      "metadata": {
        "modèl": "claude-sonnet-4-20250514",
        "token_itilize": "={{ $json.itilizasyon.total_token }}",
        "dat_pwosesis": "={{ $now.toISO() }}"
      }
    }
  }
}
```

### 12.4.9 Ne Retounen Repons

```json
{
  "ne": "Retounen Repons a Itilizatè",
  "tip": "n8n-nodes-base.respondToWebhook",
  "pozisyon": [1450, 300],
  "paramèt": {
    "opsyon": {
      "kòd_repons": 200
    },
    "kò_repons": {
      "siksè": true,
      "repons": "={{ $('Voye a Claude').item.json.repons }}",
      "konvèsasyon_id": "={{ $('Webhook Resevwa Mesaj').item.json.konvèsasyon_id }}",
      "horodataj": "={{ $now.toISO() }}"
    }
  }
}
```

---

## 12.5 Flwo Konplè JSON

### 12.5.1 Ekspòtasyon Flwo Konplè

```json
{
  "non": "Chatbot Entèlijan Konplè",
  "ne": [
    {
      "paramèt": {
        "chemen": "chatbot-mesaj",
        "metòd": "POST",
        "otantifikasyon": "headerAuth",
        "opsyon": {
          "responseMode": "lastNode"
        }
      },
      "id": "ne-webhook",
      "non": "Resevwa Mesaj",
      "tip": "n8n-nodes-base.webhook",
      "tipVèsyon": 1,
      "pozisyon": [250, 300]
    },
    {
      "paramèt": {
        "operasyon": "upsert",
        "tablo": "itilizatè",
        "kolòn_konfli": "imèl",
        "done_mòd": "defineBelow",
        "done_definisyon": {
          "done": [
            {
              "kle": "imèl",
              "valè": "={{ $json.itilizatè_imèl }}"
            },
            {
              "kle": "non",
              "valè": "={{ $json.itilizatè_non }}"
            }
          ]
        }
      },
      "id": "ne-verifye-itilizatè",
      "non": "Verifye Itilizatè",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [450, 300],
      "kalifikasyon": {
        "supabaseApi": {
          "id": "kalifikasyon-supabase",
          "non": "Supabase API"
        }
      }
    },
    {
      "paramèt": {
        "operasyon": "select",
        "tablo": "konvèsasyon",
        "filt": {
          "kolòn": "itilizatè_id",
          "valè": "={{ $json.id }}"
        },
        "limit": 1,
        "òdone": {
          "kolòn": "dat_kreye",
          "direksyon": "DESC"
        }
      },
      "id": "ne-jwenn-konvèsasyon",
      "non": "Jwenn Dènye Konvèsasyon",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [650, 300]
    },
    {
      "paramèt": {
        "kondisyon": {
          "kondisyon": [
            {
              "valè1": "={{ $json.id }}",
              "operasyon": "isNotEmpty"
            }
          ]
        }
      },
      "id": "ne-si-konvèsasyon-egziste",
      "non": "Konvèsasyon Egziste?",
      "tip": "n8n-nodes-base.if",
      "tipVèsyon": 1,
      "pozisyon": [850, 300]
    },
    {
      "paramèt": {
        "operasyon": "insert",
        "tablo": "konvèsasyon",
        "done_mòd": "defineBelow",
        "done_definisyon": {
          "done": [
            {
              "kle": "itilizatè_id",
              "valè": "={{ $('Verifye Itilizatè').item.json.id }}"
            },
            {
              "kle": "tit",
              "valè": "Nouvo Konvèsasyon"
            },
            {
              "kle": "estati",
              "valè": "aktif"
            }
          ]
        }
      },
      "id": "ne-kreye-konvèsasyon",
      "non": "Kreye Nouvo Konvèsasyon",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [1050, 400]
    },
    {
      "paramèt": {
        "operasyon": "select",
        "tablo": "mesaj",
        "filt": {
          "kolòn": "konvèsasyon_id",
          "valè": "={{ $json.id }}"
        },
        "limit": 20,
        "òdone": {
          "kolòn": "dat_kreye",
          "direksyon": "ASC"
        }
      },
      "id": "ne-jwenn-istwa",
      "non": "Jwenn Istwa Mesaj",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [1250, 300]
    },
    {
      "paramèt": {
        "kòd": "// Fonksyon pou prepare kontèks konvèsasyon an\nconst istwa = $input.all();\nconst mesaj_nouvo = $('Resevwa Mesaj').item.json.mesaj;\nconst itilizatè_non = $('Verifye Itilizatè').item.json.non;\n\n// Kreye lis mesaj yo\nlet mesaj_lis = [];\n\n// Ajoute istwa a\nfor (const m of istwa) {\n  mesaj_lis.push({\n    role: m.json.wòl === 'itilizatè' ? 'user' : 'assistant',\n    content: m.json.kontni\n  });\n}\n\n// Ajoute nouvo mesaj la\nmesaj_lis.push({\n  role: 'user',\n  content: mesaj_nouvo\n});\n\n// Kreye enstriksyon sistèm nan\nconst enstriksyon = `Ou se yon asistan entèlijan ki rele 'Asistan Chatbot'.\nOu pale Kreyòl Ayisyen epi ou ede itilizatè yo ak tout kesyon yo.\n\nEnfòmasyon itilizatè aktyèl la:\n- Non: ${itilizatè_non}\n\nRèg ou dwe swiv:\n1. Toujou reponn nan Kreyòl Ayisyen\n2. Rete janti ak respektye\n3. Bay repons klè ak konplè\n4. Si ou pa konnen yon bagay, di sa onetman\n5. Pa bay enfòmasyon danjere oswa ilegal`;\n\nreturn [{\n  json: {\n    enstriksyon_sistèm: enstriksyon,\n    mesaj: mesaj_lis,\n    mesaj_orijinal: mesaj_nouvo,\n    konvèsasyon_id: $('Jwenn Dènye Konvèsasyon').item.json.id || $('Kreye Nouvo Konvèsasyon').item.json.id\n  }\n}];"
      },
      "id": "ne-prepare-kontèks",
      "non": "Prepare Kontèks",
      "tip": "n8n-nodes-base.code",
      "tipVèsyon": 2,
      "pozisyon": [1450, 300]
    },
    {
      "paramèt": {
        "modèl": "claude-sonnet-4-20250514",
        "mesaj": "={{ $json.mesaj }}",
        "opsyon": {
          "tanperati": 0.7,
          "maksimòm_token": 1024,
          "enstriksyon_sistèm": "={{ $json.enstriksyon_sistèm }}"
        }
      },
      "id": "ne-claude",
      "non": "Voye a Claude",
      "tip": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
      "tipVèsyon": 1,
      "pozisyon": [1650, 300],
      "kalifikasyon": {
        "anthropicApi": {
          "id": "kalifikasyon-anthropic",
          "non": "Anthropic API"
        }
      }
    },
    {
      "paramèt": {
        "operasyon": "insert",
        "tablo": "mesaj",
        "done_mòd": "defineBelow",
        "done_definisyon": {
          "done": [
            {
              "kle": "konvèsasyon_id",
              "valè": "={{ $('Prepare Kontèks').item.json.konvèsasyon_id }}"
            },
            {
              "kle": "wòl",
              "valè": "itilizatè"
            },
            {
              "kle": "kontni",
              "valè": "={{ $('Prepare Kontèks').item.json.mesaj_orijinal }}"
            }
          ]
        }
      },
      "id": "ne-estoke-mesaj-itilizatè",
      "non": "Estoke Mesaj Itilizatè",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [1850, 200]
    },
    {
      "paramèt": {
        "operasyon": "insert",
        "tablo": "mesaj",
        "done_mòd": "defineBelow",
        "done_definisyon": {
          "done": [
            {
              "kle": "konvèsasyon_id",
              "valè": "={{ $('Prepare Kontèks').item.json.konvèsasyon_id }}"
            },
            {
              "kle": "wòl",
              "valè": "asistan"
            },
            {
              "kle": "kontni",
              "valè": "={{ $json.repons }}"
            }
          ]
        }
      },
      "id": "ne-estoke-repons",
      "non": "Estoke Repons Claude",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [1850, 400]
    },
    {
      "paramèt": {
        "opsyon": {},
        "kò_repons": "={{ JSON.stringify({ siksè: true, repons: $('Voye a Claude').item.json.repons, konvèsasyon_id: $('Prepare Kontèks').item.json.konvèsasyon_id, horodataj: new Date().toISOString() }) }}"
      },
      "id": "ne-repons-webhook",
      "non": "Retounen Repons",
      "tip": "n8n-nodes-base.respondToWebhook",
      "tipVèsyon": 1,
      "pozisyon": [2050, 300]
    }
  ],
  "koneksyon": {
    "Resevwa Mesaj": {
      "main": [
        [
          {
            "ne": "Verifye Itilizatè",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Verifye Itilizatè": {
      "main": [
        [
          {
            "ne": "Jwenn Dènye Konvèsasyon",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Jwenn Dènye Konvèsasyon": {
      "main": [
        [
          {
            "ne": "Konvèsasyon Egziste?",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Konvèsasyon Egziste?": {
      "main": [
        [
          {
            "ne": "Jwenn Istwa Mesaj",
            "tip": "main",
            "endèks": 0
          }
        ],
        [
          {
            "ne": "Kreye Nouvo Konvèsasyon",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Kreye Nouvo Konvèsasyon": {
      "main": [
        [
          {
            "ne": "Jwenn Istwa Mesaj",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Jwenn Istwa Mesaj": {
      "main": [
        [
          {
            "ne": "Prepare Kontèks",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Prepare Kontèks": {
      "main": [
        [
          {
            "ne": "Voye a Claude",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Voye a Claude": {
      "main": [
        [
          {
            "ne": "Estoke Mesaj Itilizatè",
            "tip": "main",
            "endèks": 0
          },
          {
            "ne": "Estoke Repons Claude",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Estoke Mesaj Itilizatè": {
      "main": [
        [
          {
            "ne": "Retounen Repons",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    },
    "Estoke Repons Claude": {
      "main": [
        [
          {
            "ne": "Retounen Repons",
            "tip": "main",
            "endèks": 0
          }
        ]
      ]
    }
  },
  "paramèt": {
    "saveExecutionProgress": true,
    "saveManualExecutions": true,
    "timezone": "America/Port-au-Prince"
  }
}
```

---

## 12.6 Tès Chatbot la

### 12.6.1 Ekran Tès Webhook

```
╔═══════════════════════════════════════════════════════════════════════╗
║                        TÈS WEBHOOK CHATBOT                            ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │  DEMAND TÈS (POST)                                              │  ║
║  ├─────────────────────────────────────────────────────────────────┤  ║
║  │  Adrès: https://ou-n8n.com/webhook/chatbot-mesaj                │  ║
║  │                                                                 │  ║
║  │  Antèt:                                                         │  ║
║  │  ┌─────────────────────────────────────────────────────────┐    │  ║
║  │  │  Content-Type: application/json                         │    │  ║
║  │  │  Authorization: Bearer kle-sekrè-ou-a                   │    │  ║
║  │  └─────────────────────────────────────────────────────────┘    │  ║
║  │                                                                 │  ║
║  │  Kò Demand:                                                     │  ║
║  │  ┌─────────────────────────────────────────────────────────┐    │  ║
║  │  │  {                                                      │    │  ║
║  │  │    "itilizatè_imèl": "jan@egzanp.com",                  │    │  ║
║  │  │    "itilizatè_non": "Jan Pyè",                          │    │  ║
║  │  │    "mesaj": "Bonjou! Kijan ou ye?",                     │    │  ║
║  │  │    "konvèsasyon_id": null                               │    │  ║
║  │  │  }                                                      │    │  ║
║  │  └─────────────────────────────────────────────────────────┘    │  ║
║  │                                                                 │  ║
║  │  [ VOYE DEMAND ]                                                │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │  REPONS (200 OK)                                                │  ║
║  ├─────────────────────────────────────────────────────────────────┤  ║
║  │  {                                                              │  ║
║  │    "siksè": true,                                               │  ║
║  │    "repons": "Bonjou Jan Pyè! Mwen byen, mèsi pou               │  ║
║  │              mande. Kijan mwen ka ede ou jodi a?",              │  ║
║  │    "konvèsasyon_id": "abc123-def456-ghi789",                    │  ║
║  │    "horodataj": "2025-01-15T10:30:00Z"                          │  ║
║  │  }                                                              │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 12.6.2 Senaryo Tès

#### Tès 1: Premye Mesaj (Nouvo Itilizatè)

```json
{
  "deskripsyon": "Tès premye mesaj pou yon nouvo itilizatè",
  "demand": {
    "metòd": "POST",
    "adrès": "/webhook/chatbot-mesaj",
    "kò": {
      "itilizatè_imèl": "nouvo@egzanp.com",
      "itilizatè_non": "Mari Jozèf",
      "mesaj": "Bonjou! Mwen bezwen èd.",
      "konvèsasyon_id": null
    }
  },
  "repons_atann": {
    "siksè": true,
    "repons": "Kontni repons Claude"
  },
  "verifikasyon": [
    "Nouvo itilizatè kreye nan tab itilizatè",
    "Nouvo konvèsasyon kreye nan tab konvèsasyon",
    "De mesaj estoke (itilizatè + asistan)"
  ]
}
```

#### Tès 2: Kontinye Konvèsasyon

```json
{
  "deskripsyon": "Tès kontinye yon konvèsasyon egzistan",
  "demand": {
    "metòd": "POST",
    "adrès": "/webhook/chatbot-mesaj",
    "kò": {
      "itilizatè_imèl": "nouvo@egzanp.com",
      "itilizatè_non": "Mari Jozèf",
      "mesaj": "Mwen vle aprann pwogramasyon.",
      "konvèsasyon_id": "abc123-def456-ghi789"
    }
  },
  "repons_atann": {
    "siksè": true,
    "repons": "Kontni repons Claude ak kontèks"
  },
  "verifikasyon": [
    "Mesaj ajoute nan konvèsasyon egzistan",
    "Claude resevwa istwa konvèsasyon an",
    "Repons gen kontèks konvèsasyon anvan"
  ]
}
```

#### Tès 3: Erè Validasyon

```json
{
  "deskripsyon": "Tès erè lè done manke",
  "demand": {
    "metòd": "POST",
    "adrès": "/webhook/chatbot-mesaj",
    "kò": {
      "mesaj": "Bonjou!"
    }
  },
  "repons_atann": {
    "siksè": false,
    "erè": "Imèl itilizatè a obligatwa"
  }
}
```

### 12.6.3 Verifikasyon Done nan Supabase

```
╔═══════════════════════════════════════════════════════════════════════╗
║                  VERIFIKASYON DONE SUPABASE                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Tab Itilizatè:                                                       ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │ id          │ non         │ imèl              │ dat_kreye      │   ║
║  ├─────────────┼─────────────┼───────────────────┼────────────────┤   ║
║  │ abc-123...  │ Mari Jozèf  │ nouvo@egzanp.com  │ 2025-01-15     │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
║  Tab Konvèsasyon:                                                     ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │ id          │ itilizatè_id │ tit              │ estati         │   ║
║  ├─────────────┼──────────────┼──────────────────┼────────────────┤   ║
║  │ def-456...  │ abc-123...   │ Nouvo Konvèsasyon│ aktif          │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
║  Tab Mesaj:                                                           ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │ id          │ konvèsasyon_id │ wòl      │ kontni              │   ║
║  ├─────────────┼────────────────┼──────────┼─────────────────────┤   ║
║  │ ghi-789...  │ def-456...     │ itilizatè│ Bonjou! Mwen...     │   ║
║  │ jkl-012...  │ def-456...     │ asistan  │ Bonjou Mari...      │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## 12.7 Fonksyonalite Avanse

### 12.7.1 Ajoute Memwa Longdire

```sql
-- Tab pou estoke memwa itilizatè
KREYE TAB SI PA EGZISTE memwa_itilizatè (
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),
    itilizatè_id IDANTIFYAN_INIK PA NIL REFERANS itilizatè(id),
    kle TÈKS PA NIL,
    valè TÈKS PA NIL,
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dat_mete_ajou HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),

    KONTRÈNT inik_kle_itilizatè INIK (itilizatè_id, kle)
);

-- Egzanp done memwa
-- ENSERE NAN memwa_itilizatè (itilizatè_id, kle, valè)
-- VALÈ ('abc-123', 'lang_prefere', 'Kreyòl');
-- VALÈ ('abc-123', 'sijè_enterè', 'pwogramasyon, mizik, espò');
```

### 12.7.2 Flwo Memwa

```json
{
  "ne": "Jwenn Memwa Itilizatè",
  "tip": "n8n-nodes-base.supabase",
  "paramèt": {
    "operasyon": "select",
    "tablo": "memwa_itilizatè",
    "filt": {
      "kolòn": "itilizatè_id",
      "valè": "={{ $json.itilizatè_id }}"
    }
  }
}
```

### 12.7.3 Analiz Santiman

```json
{
  "ne": "Analize Santiman Mesaj",
  "tip": "n8n-nodes-base.code",
  "paramèt": {
    "kòd": "// Analiz santiman senp\nconst mesaj = $json.mesaj.toLowerCase();\n\n// Mo pozitif yo\nconst mo_pozitif = ['mèsi', 'byen', 'kontan', 'renmen', 'ekselan', 'bon', 'sipè'];\n// Mo negatif yo\nconst mo_negatif = ['pa', 'mal', 'tris', 'fache', 'pwoblèm', 'move', 'difisil'];\n\nlet skò_pozitif = 0;\nlet skò_negatif = 0;\n\nfor (const mo of mo_pozitif) {\n  if (mesaj.includes(mo)) skò_pozitif++;\n}\n\nfor (const mo of mo_negatif) {\n  if (mesaj.includes(mo)) skò_negatif++;\n}\n\nlet santiman = 'nèt';\nif (skò_pozitif > skò_negatif) santiman = 'pozitif';\nelse if (skò_negatif > skò_pozitif) santiman = 'negatif';\n\nreturn [{\n  json: {\n    ...$json,\n    santiman: santiman,\n    skò: {\n      pozitif: skò_pozitif,\n      negatif: skò_negatif\n    }\n  }\n}];"
  }
}
```

---

## 12.8 Konklizyon

### 12.8.1 Rezime Pwojè a

Nan chapit sa a, nou te konstwi yon chatbot entèlijan konplè ki:

1. **Resevwa mesaj** atravè webhook
2. **Jere itilizatè** ak konvèsasyon yo
3. **Estoke istwa** nan Supabase
4. **Itilize Claude** pou repons entèlijan
5. **Retounen repons** bay itilizatè a

### 12.8.2 Pwochen Etap

| Etap | Deskripsyon |
|------|-------------|
| Ajoute entèfas | Kreye yon sitwèb pou chatbot la |
| Entegre plis platfòm | Konekte ak Telegram, WhatsApp, elatriye |
| Amelyore memwa | Ajoute sistèm memwa pi avanse |
| Ajoute analiz | Swiv estatistik konvèsasyon yo |

### 12.8.3 Resous Siplemantè

- Dokimantasyon n8n: Gid konplè pou ne yo
- Dokimantasyon Claude: Gid pou optimizasyon demann yo
- Dokimantasyon Supabase: Gid pou fonksyon avanse yo

---

**Pwochen Chapit:** Pwojè Otomatizasyon Kontni
