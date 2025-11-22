# Kominote TWOKECRYPTO
# Chapit 13: Pwojè Otomatizasyon Kontni

## Entwodiksyon

Nan chapit sa a, nou pral konstwi yon sistèm otomatizasyon kontni konplè. Sistèm sa a pral otomatikman jenere atik blog, rezime, ak pòs medya sosyal lè li itilize pouvwa Claude ak n8n, epi li estoke tout bagay nan Supabase.

Sa a se yon pwojè pratik pou moun ki vle ekonomize tan nan kreyasyon kontni.

---

## 13.1 Apèsi Pwojè a

### 13.1.1 Objektif Pwojè a

```
╔══════════════════════════════════════════════════════════════════════╗
║              SISTÈM OTOMATIZASYON KONTNI                             ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║   ┌─────────────┐                                                    ║
║   │  DEKLANCHÈ  │                                                    ║
║   │ (Pwograme/  │                                                    ║
║   │  Webhook)   │                                                    ║
║   └──────┬──────┘                                                    ║
║          │                                                           ║
║          ▼                                                           ║
║   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             ║
║   │    N8N      │───▶│   CLAUDE    │───▶│  SUPABASE   │             ║
║   │  (Flwo)     │    │  (Jenere)   │    │  (Estoke)   │             ║
║   └─────────────┘    └─────────────┘    └─────────────┘             ║
║          │                                    │                      ║
║          ▼                                    ▼                      ║
║   ┌─────────────┐                      ┌─────────────┐              ║
║   │  PIBLIYE    │                      │  ACHIV      │              ║
║   │  (Blog/     │                      │  (Istwa)    │              ║
║   │  Sosyal)    │                      │             │              ║
║   └─────────────┘                      └─────────────┘              ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### 13.1.2 Ka Itilizasyon

| Ka Itilizasyon | Deskripsyon |
|----------------|-------------|
| Atik Blog | Jenere atik blog otomatikman chak semèn |
| Rezime Atik | Kreye rezime pou atik ki long |
| Pòs Sosyal | Jenere pòs pou rezo sosyal yo |
| Bilten Nouvèl | Kreye bilten nouvèl otomatik |
| Deskripsyon Pwodwi | Jenere deskripsyon pou pwodwi yo |

### 13.1.3 Achitekti Jeneral

```
╔═══════════════════════════════════════════════════════════════════════╗
║                   ACHITEKTI SISTÈM KONTNI                             ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │                      KOUCH DEKLANCHÈ                             │  ║
║  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  ║
║  │  │ Pwograme │  │ Webhook  │  │ Manyèl   │  │ Evenman  │        │  ║
║  │  │ (Kron)   │  │          │  │          │  │ (Supabase)│       │  ║
║  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │  ║
║  └───────┼─────────────┼─────────────┼─────────────┼───────────────┘  ║
║          │             │             │             │                  ║
║          ▼             ▼             ▼             ▼                  ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │                       KOUCH PWOSESIS                             │  ║
║  │                                                                  │  ║
║  │   Jwenn Sijè ──▶ Rechèch ──▶ Jenere ──▶ Revize ──▶ Pibliye     │  ║
║  │                                                                  │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                               │                                       ║
║          ┌────────────────────┼────────────────────┐                 ║
║          ▼                    ▼                    ▼                 ║
║  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐          ║
║  │   KONTNI    │      │   METADATA  │      │   MEDYA     │          ║
║  │   (Tèks)    │      │   (Done)    │      │   (Imaj)    │          ║
║  └─────────────┘      └─────────────┘      └─────────────┘          ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## 13.2 Konsepsyon Baz Done

### 13.2.1 Dyagram Relasyon Antite

```
╔═══════════════════════════════════════════════════════════════════════╗
║                    DYAGRAM BAZ DONE KONTNI                            ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌───────────────────┐         ┌───────────────────┐                ║
║   │     KATEGORI      │         │      KONTNI       │                ║
║   ├───────────────────┤         ├───────────────────┤                ║
║   │ id (KLE PRENSIPAL)│◀────────│ kategori_id (KLE  │                ║
║   │ non               │    1:N  │    ETRANJÈ)       │                ║
║   │ deskripsyon       │         │ id (KLE PRENSIPAL)│                ║
║   │ paramèt           │         │ tit               │                ║
║   └───────────────────┘         │ kò                │                ║
║                                 │ rezime            │                ║
║   ┌───────────────────┐         │ tip               │                ║
║   │    MODÈL_DEMANN   │         │ estati            │                ║
║   ├───────────────────┤         │ dat_kreye         │                ║
║   │ id (KLE PRENSIPAL)│◀────────│ modèl_id          │                ║
║   │ non               │    1:N  │ metadata          │                ║
║   │ demann            │         └─────────┬─────────┘                ║
║   │ paramèt           │                   │                          ║
║   └───────────────────┘                   │ 1:N                      ║
║                                           ▼                          ║
║   ┌───────────────────┐         ┌───────────────────┐                ║
║   │      TAG          │◀────────│   KONTNI_TAG      │                ║
║   ├───────────────────┤   N:M   ├───────────────────┤                ║
║   │ id (KLE PRENSIPAL)│         │ kontni_id         │                ║
║   │ non               │         │ tag_id            │                ║
║   │ koulè             │         └───────────────────┘                ║
║   └───────────────────┘                                              ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 13.2.2 Deskripsyon Tab yo

#### Tab Kategori

| Kolòn | Tip | Deskripsyon |
|-------|-----|-------------|
| id | IDANTIFYAN INIK | Idantifyan inik kategori a |
| non | TÈKS | Non kategori a |
| deskripsyon | TÈKS | Deskripsyon kategori a |
| paramèt | JSON | Paramèt espesifik kategori a |

#### Tab Kontni

| Kolòn | Tip | Deskripsyon |
|-------|-----|-------------|
| id | IDANTIFYAN INIK | Idantifyan inik kontni an |
| kategori_id | IDANTIFYAN INIK | Referans kategori a |
| tit | TÈKS | Tit kontni an |
| kò | TÈKS | Kò prensipal kontni an |
| rezime | TÈKS | Rezime kout kontni an |
| tip | TÈKS | Tip kontni (atik, pòs, rezime) |
| estati | TÈKS | Estati aktyèl (bouyon, revize, pibliye) |
| modèl_id | IDANTIFYAN INIK | Modèl demann itilize |
| metadata | JSON | Enfòmasyon siplemantè |

---

## 13.3 Konfigirasyon Tab Supabase

### 13.3.1 Kreye Tab Kategori

```sql
-- Kreye tab pou kategori kontni yo
KREYE TAB SI PA EGZISTE kategori (
    -- Idantifyan inik
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Enfòmasyon kategori a
    non TÈKS PA NIL INIK,
    deskripsyon TÈKS,

    -- Paramèt pou jenerasyon kontni
    paramèt JSONB DEFO '{
        "ton": "pwofesyonèl",
        "longè_min": 500,
        "longè_maks": 2000,
        "lang": "ht"
    }'::jsonb,

    -- Dat yo
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dat_mete_ajou HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A()
);

-- Ajoute kèk kategori pa defo
ENSERE NAN kategori (non, deskripsyon, paramèt)
VALÈ
    ('Teknoloji', 'Atik sou teknoloji ak inovasyon', '{
        "ton": "teknik",
        "longè_min": 800,
        "longè_maks": 2500
    }'::jsonb),
    ('Sante', 'Atik sou sante ak byennèt', '{
        "ton": "enfòmatif",
        "longè_min": 600,
        "longè_maks": 1500
    }'::jsonb),
    ('Biznis', 'Atik sou biznis ak antreprenaryat', '{
        "ton": "pwofesyonèl",
        "longè_min": 700,
        "longè_maks": 2000
    }'::jsonb),
    ('Edikasyon', 'Atik sou edikasyon ak aprantisaj', '{
        "ton": "edikatif",
        "longè_min": 500,
        "longè_maks": 1800
    }'::jsonb);

-- Kreye endèks
KREYE ENDÈKS endèks_kategori_non SOU kategori(non);
```

### 13.3.2 Kreye Tab Modèl Demann

```sql
-- Kreye tab pou modèl demann yo
KREYE TAB SI PA EGZISTE modèl_demann (
    -- Idantifyan inik
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Enfòmasyon modèl la
    non TÈKS PA NIL INIK,
    deskripsyon TÈKS,

    -- Demann lan menm
    demann TÈKS PA NIL,

    -- Paramèt Claude
    paramèt JSONB DEFO '{
        "tanperati": 0.7,
        "maks_token": 2048
    }'::jsonb,

    -- Tip kontni modèl la kreye
    tip_kontni TÈKS DEFO 'atik',

    -- Dat yo
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    aktif BOOLEYEN DEFO vré
);

-- Ajoute modèl pa defo yo
ENSERE NAN modèl_demann (non, deskripsyon, demann, tip_kontni)
VALÈ
    ('Atik Blog Konplè', 'Jenere yon atik blog konplè', '
Ou se yon ekriven ekspè nan Kreyòl Ayisyen. Ekri yon atik blog konplè sou sijè sa a:

Sijè: {{sijè}}
Kategori: {{kategori}}

Enstriksyon:
1. Kòmanse ak yon entwodiksyon ki atire atansyon
2. Divize kontni an an seksyon klè ak soutit
3. Itilize egzanp konkrè
4. Fini ak yon konklizyon
5. Ekri ant {{longè_min}} ak {{longè_maks}} mo

Ton: {{ton}}

Fòma repons ou:
## [Tit Atik la]

[Entwodiksyon]

### [Seksyon 1]
[Kontni]

### [Seksyon 2]
[Kontni]

[Konklizyon]
', 'atik'),

    ('Rezime Atik', 'Kreye yon rezime kout pou yon atik', '
Kreye yon rezime kout (3-5 fraz) pou atik sa a nan Kreyòl Ayisyen:

Tit: {{tit}}
Kontni: {{kontni}}

Rezime a dwe:
1. Kaptire pwen prensipal yo
2. Rete kout men enfòmatif
3. Atire lektè pou li atik konplè a
', 'rezime'),

    ('Pòs Medya Sosyal', 'Jenere pòs pou medya sosyal', '
Kreye yon pòs medya sosyal nan Kreyòl Ayisyen pou pwomote atik sa a:

Tit Atik: {{tit}}
Rezime: {{rezime}}
Platfòm: {{platfòm}}

Enstriksyon:
1. Rete kout (mwens pase 280 karaktè pou Twitter)
2. Itilize emosyon si apwopriye
3. Ajoute apèl a aksyon
4. Sijere 3-5 mo kle (hashtags)
', 'pòs_sosyal');

-- Kreye endèks
KREYE ENDÈKS endèks_modèl_non SOU modèl_demann(non);
KREYE ENDÈKS endèks_modèl_aktif SOU modèl_demann(aktif);
```

### 13.3.3 Kreye Tab Kontni

```sql
-- Kreye tab prensipal pou kontni yo
KREYE TAB SI PA EGZISTE kontni (
    -- Idantifyan inik
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),

    -- Referans yo
    kategori_id IDANTIFYAN_INIK REFERANS kategori(id),
    modèl_id IDANTIFYAN_INIK REFERANS modèl_demann(id),

    -- Kontni an
    tit TÈKS PA NIL,
    kò TÈKS PA NIL,
    rezime TÈKS,

    -- Tip ak estati
    tip TÈKS DEFO 'atik' TCHEKE (tip NAN ('atik', 'rezime', 'pòs_sosyal', 'bilten')),
    estati TÈKS DEFO 'bouyon' TCHEKE (estati NAN ('bouyon', 'revize', 'pibliye', 'achive')),

    -- Done jenerasyon
    sijè_orijinal TÈKS,
    demann_itilize TÈKS,

    -- Metadata
    metadata JSONB DEFO '{}'::jsonb,

    -- Estatistik
    kantite_mo ANTYE,
    tan_jenerasyon_ms ANTYE,
    token_itilize ANTYE,

    -- Dat yo
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dat_mete_ajou HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A(),
    dat_pibliye HORODATAJ AK ZÒN_LÈ
);

-- Kreye endèks yo
KREYE ENDÈKS endèks_kontni_kategori SOU kontni(kategori_id);
KREYE ENDÈKS endèks_kontni_tip SOU kontni(tip);
KREYE ENDÈKS endèks_kontni_estati SOU kontni(estati);
KREYE ENDÈKS endèks_kontni_dat SOU kontni(dat_kreye DESANDAN);

-- Endèks pou rechèch tèks konplè
KREYE ENDÈKS endèks_kontni_rechèch SOU kontni
ITILIZE GIN(to_tsvector('french', tit || ' ' || kò));
```

### 13.3.4 Kreye Tab Tag

```sql
-- Kreye tab pou tag yo
KREYE TAB SI PA EGZISTE tag (
    id IDANTIFYAN_INIK PRENSIPAL DEFO jenere_idantifyan_inik_v4(),
    non TÈKS PA NIL INIK,
    koulè TÈKS DEFO '#808080',
    dat_kreye HORODATAJ AK ZÒN_LÈ DEFO KOUNYE_A()
);

-- Kreye tab relasyon kontni-tag
KREYE TAB SI PA EGZISTE kontni_tag (
    kontni_id IDANTIFYAN_INIK REFERANS kontni(id) SOU EFASE KASKAD,
    tag_id IDANTIFYAN_INIK REFERANS tag(id) SOU EFASE KASKAD,
    KLE PRENSIPAL (kontni_id, tag_id)
);

-- Ajoute kèk tag pa defo
ENSERE NAN tag (non, koulè)
VALÈ
    ('teknoloji', '#3498db'),
    ('sante', '#2ecc71'),
    ('biznis', '#f39c12'),
    ('edikasyon', '#9b59b6'),
    ('kilti', '#e74c3c'),
    ('espò', '#1abc9c');

-- Kreye endèks
KREYE ENDÈKS endèks_kontni_tag_kontni SOU kontni_tag(kontni_id);
KREYE ENDÈKS endèks_kontni_tag_tag SOU kontni_tag(tag_id);
```

### 13.3.5 Kreye Fonksyon Itil

```sql
-- Fonksyon pou konte mo nan yon tèks
KREYE OSWA RANPLASE FONKSYON konte_mo(tèks TÈKS)
RETOUNEN ANTYE
LANG sql
KÒM $$
    CHWAZI array_length(regexp_split_to_array(trim(tèks), '\s+'), 1);
$$;

-- Fonksyon pou mete ajou dat ak konte mo lè kontni chanje
KREYE OSWA RANPLASE FONKSYON mete_ajou_kontni_metadata()
RETOUNEN DEKLANCHÈ
LANG plpgsql
KÒM $$
KÒMANSE
    NOUVO.dat_mete_ajou := KOUNYE_A();
    NOUVO.kantite_mo := konte_mo(NOUVO.kò);
    RETOUNEN NOUVO;
FEN;
$$;

-- Kreye deklanchè
KREYE DEKLANCHÈ deklanchè_mete_ajou_kontni
ANVAN ENSERE OSWA METE_AJOU SOU kontni
POU CHAK RANJE
EGZEKITE FONKSYON mete_ajou_kontni_metadata();

-- Fonksyon pou jwenn kontni pa kategori ak estati
KREYE OSWA RANPLASE FONKSYON jwenn_kontni_pa_kategori(
    p_kategori_non TÈKS,
    p_estati TÈKS DEFO 'pibliye',
    p_limit ANTYE DEFO 10
)
RETOUNEN TAB (
    id IDANTIFYAN_INIK,
    tit TÈKS,
    rezime TÈKS,
    dat_kreye HORODATAJ AK ZÒN_LÈ
)
LANG sql
KÒM $$
    CHWAZI k.id, k.tit, k.rezime, k.dat_kreye
    DEPI kontni k
    JWENN kategori kat SOU k.kategori_id = kat.id
    KOTE kat.non = p_kategori_non
    AK k.estati = p_estati
    ÒDONE PA k.dat_kreye DESANDAN
    LIMIT p_limit;
$$;

-- Fonksyon pou estatistik kontni
KREYE OSWA RANPLASE FONKSYON jwenn_estatistik_kontni()
RETOUNEN TAB (
    total_kontni BIGINT,
    total_pibliye BIGINT,
    total_bouyon BIGINT,
    mwayèn_mo DESIMAL,
    total_token_itilize BIGINT
)
LANG sql
KÒM $$
    CHWAZI
        KONTE(*) KÒM total_kontni,
        KONTE(*) FILTRE (KOTE estati = 'pibliye') KÒM total_pibliye,
        KONTE(*) FILTRE (KOTE estati = 'bouyon') KÒM total_bouyon,
        AVG(kantite_mo) KÒM mwayèn_mo,
        SUM(token_itilize) KÒM total_token_itilize
    DEPI kontni;
$$;
```

---

## 13.4 Kreye Flwo n8n

### 13.4.1 Apèsi Flwo Prensipal

```
╔═══════════════════════════════════════════════════════════════════════╗
║                   FLWO OTOMATIZASYON KONTNI                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   FLWO 1: JENERASYON PWOGRAME                                         ║
║   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          ║
║   │  KRON   │───▶│  JWENN  │───▶│ PREPARE │───▶│ JENERE  │          ║
║   │(Chak jou│    │  SIJÈ   │    │ DEMANN  │    │ (Claude)│          ║
║   └─────────┘    └─────────┘    └─────────┘    └─────────┘          ║
║                                                      │               ║
║                                                      ▼               ║
║   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          ║
║   │NOTIFYE  │◀───│  ESTOKE │◀───│ REVIZE  │◀───│ PWOSESIS│          ║
║   │         │    │(Supabase│    │ KALITE  │    │ REPONS  │          ║
║   └─────────┘    └─────────┘    └─────────┘    └─────────┘          ║
║                                                                       ║
║   FLWO 2: DEMANN MANYÈL                                               ║
║   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          ║
║   │ WEBHOOK │───▶│ VALIDE  │───▶│ JENERE  │───▶│  ESTOKE │          ║
║   │         │    │ DONE    │    │ KONTNI  │    │         │          ║
║   └─────────┘    └─────────┘    └─────────┘    └─────────┘          ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 13.4.2 Flwo Pwograme (Kron)

#### Ne Deklanchè Kron

```json
{
  "ne": "Deklanchè Pwograme",
  "tip": "n8n-nodes-base.scheduleTrigger",
  "pozisyon": [250, 300],
  "paramèt": {
    "règ": {
      "entèval": [
        {
          "champ": "jou",
          "valè_jou": ["lendi", "mèkredi", "vandredi"],
          "lè": 9,
          "minit": 0
        }
      ]
    }
  }
}
```

#### Ne Jwenn Sijè Pwochen

```json
{
  "ne": "Jwenn Sijè Pwochen",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [450, 300],
  "paramèt": {
    "operasyon": "select",
    "tablo": "sijè_planifye",
    "filt": {
      "kolòn": "estati",
      "valè": "ann_atant"
    },
    "limit": 1,
    "òdone": {
      "kolòn": "priyorite",
      "direksyon": "DESC"
    }
  }
}
```

### 13.4.3 Flwo Webhook

#### Ne Webhook Resevwa Demann

```json
{
  "ne": "Resevwa Demann Kontni",
  "tip": "n8n-nodes-base.webhook",
  "pozisyon": [250, 500],
  "paramèt": {
    "chemen": "jenere-kontni",
    "metòd": "POST",
    "otantifikasyon": "headerAuth",
    "opsyon": {
      "responseMode": "lastNode"
    }
  }
}
```

#### Ne Valide Done Antre

```json
{
  "ne": "Valide Done Antre",
  "tip": "n8n-nodes-base.code",
  "pozisyon": [450, 500],
  "paramèt": {
    "kòd": "// Jwenn done yo\nconst done = $json;\n\n// Verifye chan obligatwa yo\nconst chan_obligatwa = ['sijè', 'kategori', 'tip'];\nconst chan_manke = [];\n\nfor (const chan of chan_obligatwa) {\n  if (!done[chan]) {\n    chan_manke.push(chan);\n  }\n}\n\nif (chan_manke.length > 0) {\n  throw new Error(`Chan obligatwa manke: ${chan_manke.join(', ')}`);\n}\n\n// Valide tip kontni\nconst tip_valab = ['atik', 'rezime', 'pòs_sosyal', 'bilten'];\nif (!tip_valab.includes(done.tip)) {\n  throw new Error(`Tip kontni pa valab. Itilize youn nan: ${tip_valab.join(', ')}`);\n}\n\n// Retounen done valide\nreturn [{\n  json: {\n    ...done,\n    valide: true,\n    horodataj: new Date().toISOString()\n  }\n}];"
  }
}
```

### 13.4.4 Ne Jwenn Modèl Demann

```json
{
  "ne": "Jwenn Modèl Demann",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [650, 400],
  "paramèt": {
    "operasyon": "select",
    "tablo": "modèl_demann",
    "filt": {
      "kolòn": "tip_kontni",
      "valè": "={{ $json.tip }}"
    },
    "limit": 1
  }
}
```

### 13.4.5 Ne Jwenn Paramèt Kategori

```json
{
  "ne": "Jwenn Paramèt Kategori",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [650, 500],
  "paramèt": {
    "operasyon": "select",
    "tablo": "kategori",
    "filt": {
      "kolòn": "non",
      "valè": "={{ $json.kategori }}"
    },
    "limit": 1
  }
}
```

### 13.4.6 Ne Prepare Demann Claude

```json
{
  "ne": "Prepare Demann pou Claude",
  "tip": "n8n-nodes-base.code",
  "pozisyon": [850, 400],
  "paramèt": {
    "kòd": "// Jwenn done yo\nconst modèl = $('Jwenn Modèl Demann').item.json;\nconst kategori = $('Jwenn Paramèt Kategori').item.json;\nconst demand_orijinal = $('Valide Done Antre').item.json;\n\n// Jwenn paramèt yo\nconst paramèt = kategori.paramèt || {};\nconst ton = paramèt.ton || 'pwofesyonèl';\nconst longè_min = paramèt.longè_min || 500;\nconst longè_maks = paramèt.longè_maks || 2000;\n\n// Ranplase varyab nan modèl la\nlet demann = modèl.demann;\ndemann = demann.replace('{{sijè}}', demand_orijinal.sijè);\ndemann = demann.replace('{{kategori}}', demand_orijinal.kategori);\ndemann = demann.replace('{{ton}}', ton);\ndemann = demann.replace('{{longè_min}}', longè_min);\ndemann = demann.replace('{{longè_maks}}', longè_maks);\n\n// Kreye enstriksyon sistèm\nconst enstriksyon_sistèm = `Ou se yon ekriven ekspè nan Kreyòl Ayisyen.\nOu ekri kontni edikatif, enfòmatif, ak angajan.\nToujou ekri nan Kreyòl Ayisyen natirèl ak fasil pou li.\nSuiv enstriksyon yo egzakteman.`;\n\nreturn [{\n  json: {\n    demann: demann,\n    enstriksyon_sistèm: enstriksyon_sistèm,\n    modèl_id: modèl.id,\n    kategori_id: kategori.id,\n    sijè: demand_orijinal.sijè,\n    tip: demand_orijinal.tip,\n    paramèt_claude: modèl.paramèt,\n    tan_kòmanse: Date.now()\n  }\n}];"
  }
}
```

### 13.4.7 Ne Voye a Claude

```json
{
  "ne": "Jenere Kontni ak Claude",
  "tip": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
  "pozisyon": [1050, 400],
  "paramèt": {
    "modèl": "claude-sonnet-4-20250514",
    "mesaj": [
      {
        "role": "user",
        "content": "={{ $json.demann }}"
      }
    ],
    "opsyon": {
      "tanperati": "={{ $json.paramèt_claude.tanperati || 0.7 }}",
      "maksimòm_token": "={{ $json.paramèt_claude.maks_token || 2048 }}",
      "enstriksyon_sistèm": "={{ $json.enstriksyon_sistèm }}"
    }
  },
  "kalifikasyon": {
    "anthropicApi": {
      "id": "kalifikasyon-anthropic",
      "non": "Anthropic API"
    }
  }
}
```

### 13.4.8 Ne Pwosesis Repons

```json
{
  "ne": "Pwosesis Repons Claude",
  "tip": "n8n-nodes-base.code",
  "pozisyon": [1250, 400],
  "paramèt": {
    "kòd": "// Jwenn repons lan\nconst repons = $json.repons || $json.text || $json.content;\nconst done_anvan = $('Prepare Demann pou Claude').item.json;\n\n// Ekstrè tit la (premye liy ki kòmanse ak ##)\nlet tit = done_anvan.sijè;\nconst match_tit = repons.match(/^##\\s+(.+)$/m);\nif (match_tit) {\n  tit = match_tit[1].trim();\n}\n\n// Kreye rezime (premye paragraf apre tit la)\nlet rezime = '';\nconst paragraf = repons.split('\\n\\n');\nfor (const p of paragraf) {\n  if (p.trim() && !p.startsWith('#')) {\n    rezime = p.trim().substring(0, 300);\n    if (p.length > 300) rezime += '...';\n    break;\n  }\n}\n\n// Kalkile tan jenerasyon\nconst tan_jenerasyon = Date.now() - done_anvan.tan_kòmanse;\n\n// Konte mo\nconst konte_mo = repons.split(/\\s+/).filter(m => m.length > 0).length;\n\nreturn [{\n  json: {\n    tit: tit,\n    kò: repons,\n    rezime: rezime,\n    tip: done_anvan.tip,\n    kategori_id: done_anvan.kategori_id,\n    modèl_id: done_anvan.modèl_id,\n    sijè_orijinal: done_anvan.sijè,\n    demann_itilize: done_anvan.demann,\n    tan_jenerasyon_ms: tan_jenerasyon,\n    kantite_mo: konte_mo,\n    token_itilize: $json.itilizasyon?.total_token || 0,\n    estati: 'bouyon'\n  }\n}];"
  }
}
```

### 13.4.9 Ne Estoke nan Supabase

```json
{
  "ne": "Estoke Kontni nan Supabase",
  "tip": "n8n-nodes-base.supabase",
  "pozisyon": [1450, 400],
  "paramèt": {
    "operasyon": "insert",
    "tablo": "kontni",
    "done_mòd": "autoMapInputData",
    "opsyon": {
      "retounen": "representation"
    }
  }
}
```

### 13.4.10 Ne Retounen Repons

```json
{
  "ne": "Retounen Repons",
  "tip": "n8n-nodes-base.respondToWebhook",
  "pozisyon": [1650, 400],
  "paramèt": {
    "opsyon": {
      "kòd_repons": 201
    },
    "kò_repons": {
      "siksè": true,
      "kontni_id": "={{ $json.id }}",
      "tit": "={{ $json.tit }}",
      "rezime": "={{ $json.rezime }}",
      "estati": "={{ $json.estati }}",
      "kantite_mo": "={{ $json.kantite_mo }}",
      "horodataj": "={{ $now.toISO() }}"
    }
  }
}
```

---

## 13.5 Flwo Konplè JSON

### 13.5.1 Ekspòtasyon Flwo Konplè

```json
{
  "non": "Otomatizasyon Kontni Konplè",
  "ne": [
    {
      "paramèt": {
        "règ": {
          "entèval": [
            {
              "champ": "jou",
              "valè_jou": ["lendi", "mèkredi", "vandredi"],
              "lè": 9,
              "minit": 0
            }
          ]
        }
      },
      "id": "ne-kron",
      "non": "Deklanchè Pwograme",
      "tip": "n8n-nodes-base.scheduleTrigger",
      "tipVèsyon": 1,
      "pozisyon": [250, 200]
    },
    {
      "paramèt": {
        "chemen": "jenere-kontni",
        "metòd": "POST",
        "otantifikasyon": "headerAuth"
      },
      "id": "ne-webhook",
      "non": "Resevwa Demann Kontni",
      "tip": "n8n-nodes-base.webhook",
      "tipVèsyon": 1,
      "pozisyon": [250, 400]
    },
    {
      "paramèt": {
        "kòd": "// Validasyon done antre\nconst done = $json;\n\n// Verifye chan obligatwa\nif (!done.sijè) throw new Error('Sijè obligatwa');\nif (!done.kategori) throw new Error('Kategori obligatwa');\nif (!done.tip) done.tip = 'atik';\n\n// Valide tip\nconst tip_valab = ['atik', 'rezime', 'pòs_sosyal', 'bilten'];\nif (!tip_valab.includes(done.tip)) {\n  throw new Error('Tip pa valab');\n}\n\nreturn [{ json: { ...done, valide: true } }];"
      },
      "id": "ne-valide",
      "non": "Valide Done",
      "tip": "n8n-nodes-base.code",
      "tipVèsyon": 2,
      "pozisyon": [450, 400]
    },
    {
      "paramèt": {
        "operasyon": "select",
        "tablo": "modèl_demann",
        "filtre": "tip_kontni,eq,{{ $json.tip }}",
        "limit": 1
      },
      "id": "ne-jwenn-modèl",
      "non": "Jwenn Modèl",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [650, 300]
    },
    {
      "paramèt": {
        "operasyon": "select",
        "tablo": "kategori",
        "filtre": "non,eq,{{ $json.kategori }}",
        "limit": 1
      },
      "id": "ne-jwenn-kategori",
      "non": "Jwenn Kategori",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [650, 500]
    },
    {
      "paramèt": {
        "kòd": "// Prepare demann pou Claude\nconst modèl = $('Jwenn Modèl').item.json;\nconst kategori = $('Jwenn Kategori').item.json;\nconst demand = $('Valide Done').item.json;\n\nconst paramèt = kategori.paramèt || {};\n\nlet demann = modèl.demann;\ndemann = demann.replace('{{sijè}}', demand.sijè);\ndemann = demann.replace('{{kategori}}', demand.kategori);\ndemann = demann.replace('{{ton}}', paramèt.ton || 'pwofesyonèl');\ndemann = demann.replace('{{longè_min}}', paramèt.longè_min || 500);\ndemann = demann.replace('{{longè_maks}}', paramèt.longè_maks || 2000);\n\nconst enstriksyon = `Ou se yon ekriven ekspè nan Kreyòl Ayisyen.\nEkri kontni edikatif ak angajan.\nSuiv enstriksyon yo egzakteman.`;\n\nreturn [{\n  json: {\n    demann: demann,\n    enstriksyon_sistèm: enstriksyon,\n    modèl_id: modèl.id,\n    kategori_id: kategori.id,\n    sijè: demand.sijè,\n    tip: demand.tip,\n    tan_kòmanse: Date.now()\n  }\n}];"
      },
      "id": "ne-prepare-demann",
      "non": "Prepare Demann",
      "tip": "n8n-nodes-base.code",
      "tipVèsyon": 2,
      "pozisyon": [850, 400]
    },
    {
      "paramèt": {
        "modèl": "claude-sonnet-4-20250514",
        "mesaj": "={{ $json.demann }}",
        "opsyon": {
          "tanperati": 0.7,
          "maksimòm_token": 2048,
          "enstriksyon_sistèm": "={{ $json.enstriksyon_sistèm }}"
        }
      },
      "id": "ne-claude",
      "non": "Jenere ak Claude",
      "tip": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
      "tipVèsyon": 1,
      "pozisyon": [1050, 400],
      "kalifikasyon": {
        "anthropicApi": {
          "id": "kalifikasyon-anthropic",
          "non": "Anthropic API"
        }
      }
    },
    {
      "paramèt": {
        "kòd": "// Pwosesis repons Claude\nconst repons = $json.repons || $json.text;\nconst done = $('Prepare Demann').item.json;\n\n// Ekstrè tit\nlet tit = done.sijè;\nconst match = repons.match(/^##\\s+(.+)$/m);\nif (match) tit = match[1].trim();\n\n// Kreye rezime\nlet rezime = '';\nconst parag = repons.split('\\n\\n');\nfor (const p of parag) {\n  if (p.trim() && !p.startsWith('#')) {\n    rezime = p.substring(0, 300);\n    break;\n  }\n}\n\nconst tan = Date.now() - done.tan_kòmanse;\nconst mo = repons.split(/\\s+/).length;\n\nreturn [{\n  json: {\n    tit, kò: repons, rezime,\n    tip: done.tip,\n    kategori_id: done.kategori_id,\n    modèl_id: done.modèl_id,\n    sijè_orijinal: done.sijè,\n    demann_itilize: done.demann,\n    tan_jenerasyon_ms: tan,\n    kantite_mo: mo,\n    estati: 'bouyon'\n  }\n}];"
      },
      "id": "ne-pwosesis",
      "non": "Pwosesis Repons",
      "tip": "n8n-nodes-base.code",
      "tipVèsyon": 2,
      "pozisyon": [1250, 400]
    },
    {
      "paramèt": {
        "operasyon": "insert",
        "tablo": "kontni",
        "done_mòd": "autoMapInputData"
      },
      "id": "ne-estoke",
      "non": "Estoke Kontni",
      "tip": "n8n-nodes-base.supabase",
      "tipVèsyon": 1,
      "pozisyon": [1450, 400],
      "kalifikasyon": {
        "supabaseApi": {
          "id": "kalifikasyon-supabase",
          "non": "Supabase API"
        }
      }
    },
    {
      "paramèt": {
        "kò_repons": "={{ JSON.stringify({ siksè: true, id: $json.id, tit: $json.tit }) }}"
      },
      "id": "ne-repons",
      "non": "Retounen Repons",
      "tip": "n8n-nodes-base.respondToWebhook",
      "tipVèsyon": 1,
      "pozisyon": [1650, 400]
    }
  ],
  "koneksyon": {
    "Deklanchè Pwograme": {
      "main": [[{"ne": "Valide Done", "tip": "main", "endèks": 0}]]
    },
    "Resevwa Demann Kontni": {
      "main": [[{"ne": "Valide Done", "tip": "main", "endèks": 0}]]
    },
    "Valide Done": {
      "main": [[
        {"ne": "Jwenn Modèl", "tip": "main", "endèks": 0},
        {"ne": "Jwenn Kategori", "tip": "main", "endèks": 0}
      ]]
    },
    "Jwenn Modèl": {
      "main": [[{"ne": "Prepare Demann", "tip": "main", "endèks": 0}]]
    },
    "Jwenn Kategori": {
      "main": [[{"ne": "Prepare Demann", "tip": "main", "endèks": 0}]]
    },
    "Prepare Demann": {
      "main": [[{"ne": "Jenere ak Claude", "tip": "main", "endèks": 0}]]
    },
    "Jenere ak Claude": {
      "main": [[{"ne": "Pwosesis Repons", "tip": "main", "endèks": 0}]]
    },
    "Pwosesis Repons": {
      "main": [[{"ne": "Estoke Kontni", "tip": "main", "endèks": 0}]]
    },
    "Estoke Kontni": {
      "main": [[{"ne": "Retounen Repons", "tip": "main", "endèks": 0}]]
    }
  },
  "paramèt": {
    "saveExecutionProgress": true,
    "timezone": "America/Port-au-Prince"
  }
}
```

---

## 13.6 Tès ak Verifikasyon

### 13.6.1 Ekran Tès Webhook

```
╔═══════════════════════════════════════════════════════════════════════╗
║                    TÈS JENERASYON KONTNI                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │  DEMAND TÈS (POST)                                              │  ║
║  ├─────────────────────────────────────────────────────────────────┤  ║
║  │  Adrès: https://ou-n8n.com/webhook/jenere-kontni                │  ║
║  │                                                                 │  ║
║  │  Kò Demand:                                                     │  ║
║  │  ┌─────────────────────────────────────────────────────────┐    │  ║
║  │  │  {                                                      │    │  ║
║  │  │    "sijè": "Kijan pou aprann pwogramasyon",             │    │  ║
║  │  │    "kategori": "Teknoloji",                             │    │  ║
║  │  │    "tip": "atik"                                        │    │  ║
║  │  │  }                                                      │    │  ║
║  │  └─────────────────────────────────────────────────────────┘    │  ║
║  │                                                                 │  ║
║  │  [ VOYE DEMAND ]                                                │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────────┐  ║
║  │  REPONS (201 KREYE)                                             │  ║
║  ├─────────────────────────────────────────────────────────────────┤  ║
║  │  {                                                              │  ║
║  │    "siksè": true,                                               │  ║
║  │    "kontni_id": "abc123-def456-ghi789",                         │  ║
║  │    "tit": "Gid Konplè pou Aprann Pwogramasyon",                 │  ║
║  │    "rezime": "Nan atik sa a, nou pral eksplore...",             │  ║
║  │    "estati": "bouyon",                                          │  ║
║  │    "kantite_mo": 1247,                                          │  ║
║  │    "horodataj": "2025-01-15T10:30:00Z"                          │  ║
║  │  }                                                              │  ║
║  └─────────────────────────────────────────────────────────────────┘  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 13.6.2 Senaryo Tès

#### Tès 1: Jenere Atik Blog

```json
{
  "deskripsyon": "Tès jenerasyon yon atik blog konplè",
  "demand": {
    "metòd": "POST",
    "adrès": "/webhook/jenere-kontni",
    "kò": {
      "sijè": "Avantaj entelijan atifisyèl nan edikasyon",
      "kategori": "Edikasyon",
      "tip": "atik"
    }
  },
  "verifikasyon": [
    "Repons gen tit",
    "Kontni gen plis pase 500 mo",
    "Estoke nan tab kontni",
    "Estati se bouyon"
  ]
}
```

#### Tès 2: Jenere Rezime

```json
{
  "deskripsyon": "Tès jenerasyon rezime",
  "demand": {
    "metòd": "POST",
    "adrès": "/webhook/jenere-kontni",
    "kò": {
      "sijè": "Rezime atik sou sante",
      "kategori": "Sante",
      "tip": "rezime",
      "kontni_orijinal": "[Kontni atik orijinal la]"
    }
  },
  "verifikasyon": [
    "Rezime kout (3-5 fraz)",
    "Kaptire pwen prensipal yo"
  ]
}
```

---

## 13.7 Deplwaman

### 13.7.1 Lis Verifikasyon Deplwaman

```
╔═══════════════════════════════════════════════════════════════════════╗
║                    LIS VERIFIKASYON DEPLWAMAN                         ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  [ ] Konfigirasyon Baz Done                                          ║
║      [ ] Tout tab kreye                                              ║
║      [ ] Endèks yo an plas                                           ║
║      [ ] Fonksyon yo teste                                           ║
║      [ ] Done pa defo ajoute                                         ║
║                                                                       ║
║  [ ] Konfigirasyon n8n                                               ║
║      [ ] Kalifikasyon Supabase konfigire                             ║
║      [ ] Kalifikasyon Claude konfigire                               ║
║      [ ] Webhook teste                                               ║
║      [ ] Deklanchè kron konfigire                                    ║
║                                                                       ║
║  [ ] Sekirite                                                        ║
║      [ ] Otantifikasyon webhook aktive                               ║
║      [ ] Kle API an sekirite                                         ║
║      [ ] Politik RLS aktive                                          ║
║                                                                       ║
║  [ ] Siveyans                                                        ║
║      [ ] Alèt erè konfigire                                          ║
║      [ ] Jounal aktivite aktive                                      ║
║      [ ] Metrik pèfòmans swiv                                        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### 13.7.2 Konfigirasyon Pwodikasyon

```json
{
  "anviwònman": "pwodikasyon",
  "paramèt": {
    "n8n": {
      "webhook_baz_adrès": "https://ou-n8n.com",
      "egzekisyon_mòd": "queue",
      "maks_egzekisyon_palalèl": 5,
      "tan_limit_ms": 120000
    },
    "supabase": {
      "koneksyon_maks": 20,
      "delè_koneksyon_ms": 5000
    },
    "claude": {
      "modèl": "claude-sonnet-4-20250514",
      "maks_token": 2048,
      "tanperati": 0.7,
      "esè_ankò": 3
    }
  }
}
```

---

## 13.8 Konklizyon

### 13.8.1 Rezime Pwojè a

Nan chapit sa a, nou te konstwi yon sistèm otomatizasyon kontni konplè ki:

1. **Aksepte demann** atravè webhook oswa pwograme
2. **Itilize modèl demann** pou konsistans
3. **Jenere kontni** ak Claude nan Kreyòl Ayisyen
4. **Estoke tout bagay** nan Supabase
5. **Swiv estatistik** jenerasyon

### 13.8.2 Amelyorasyon Posib

| Amelyorasyon | Deskripsyon |
|--------------|-------------|
| Revizyon Otomatik | Ajoute etap revizyon ak Claude |
| Pibliye Otomatik | Konekte ak platfòm blog yo |
| Imaj AI | Jenere imaj pou atik yo |
| Tradiksyon | Tradui kontni nan lòt lang |

---

**Pwochen Chapit:** Pwojè Jesyon Done
