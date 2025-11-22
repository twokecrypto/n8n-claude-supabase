# Kominote TWOKECRYPTO
# Chapit 8: Baz Done ak Otantifikasyon

## Rezime Chapit

Nan chapit sa a, nou pral aprann kijan pou kreye ak jere baz done nan Supabase. Nou pral kouvri kreye tab, tip done, relasyon, sekirite nivo ranje (RLS), otantifikasyon itilizatè, depo fichye, ak abònman tan reyèl.

---

## 8.1 Kreye Tab

### 8.1.1 Itilize Entèfas Grafik

```
┌─────────────────────────────────────────────────────────────────────┐
│  SUPABASE  │  Tab Editè                                             │
├────────────┼────────────────────────────────────────────────────────┤
│            │                                                        │
│  📊 Tab    │   ┌──────────────────────────────────────────────┐    │
│  Editè     │   │          KREYE NOUVO TAB                     │    │
│            │   ├──────────────────────────────────────────────┤    │
│  public    │   │                                              │    │
│  ├─ (vid)  │   │  Non Tab: [itilizate____________________]   │    │
│            │   │                                              │    │
│            │   │  Deskripsyon: [Tab pou estoke itilizatè]    │    │
│            │   │                                              │    │
│            │   │  ☑ Aktive Sekirite Nivo Ranje (RLS)        │    │
│            │   │                                              │    │
│            │   │  [  Anile  ]     [  Kreye Tab  ]            │    │
│            │   │                                              │    │
│            │   └──────────────────────────────────────────────┘    │
│            │                                                        │
└────────────┴────────────────────────────────────────────────────────┘
```

### 8.1.2 Kreye Tab ak SQL

```sql
-- Kreye yon tab senp pou itilizatè
CREATE TABLE itilizate (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    imel TEXT UNIQUE NOT NULL,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- Ajoute kòmantè sou tab la
COMMENT ON TABLE itilizate IS 'Tab pou estoke enfòmasyon itilizatè';
```

### 8.1.3 Egzanp Tab Konplè

```sql
-- Tab pou yon sistèm magazen an liy

-- 1. Tab Kategori
CREATE TABLE kategori (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    deskripsyon TEXT,
    imaj_url TEXT,
    aktif BOOLEAN DEFAULT true,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- 2. Tab Pwodui
CREATE TABLE pwodui (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    deskripsyon TEXT,
    pri DECIMAL(10,2) NOT NULL CHECK (pri >= 0),
    stok INTEGER DEFAULT 0 CHECK (stok >= 0),
    kategori_id UUID REFERENCES kategori(id),
    imaj_url TEXT,
    aktif BOOLEAN DEFAULT true,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()),
    modifye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- 3. Tab Kliyan
CREATE TABLE kliyan (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    itilizate_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
    non_konple TEXT NOT NULL,
    telefòn TEXT,
    adrès TEXT,
    vil TEXT,
    peyi TEXT DEFAULT 'Ayiti',
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- 4. Tab Kòmand
CREATE TABLE komand (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    kliyan_id UUID REFERENCES kliyan(id),
    total DECIMAL(10,2) NOT NULL,
    estati TEXT DEFAULT 'annatant' CHECK (estati IN ('annatant', 'konfime', 'livre', 'anile')),
    adrès_livrezon TEXT,
    nòt TEXT,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- 5. Tab Atik Kòmand
CREATE TABLE atik_komand (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    komand_id UUID REFERENCES komand(id) ON DELETE CASCADE,
    pwodui_id UUID REFERENCES pwodui(id),
    kantite INTEGER NOT NULL CHECK (kantite > 0),
    pri_inite DECIMAL(10,2) NOT NULL,
    sou_total DECIMAL(10,2) GENERATED ALWAYS AS (kantite * pri_inite) STORED
);
```

### 8.1.4 Dyagram Relasyon Tab

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DYAGRAM RELASYON BAZ DONE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────────┐                                                     │
│   │  auth.users  │                                                     │
│   │  (Supabase)  │                                                     │
│   └──────┬───────┘                                                     │
│          │ 1                                                            │
│          │                                                              │
│          │ ▼                                                            │
│   ┌──────────────┐         ┌──────────────┐         ┌──────────────┐   │
│   │   kliyan     │         │   komand     │         │ atik_komand  │   │
│   ├──────────────┤    1    ├──────────────┤    1    ├──────────────┤   │
│   │ id           │◄────────│ kliyan_id    │◄────────│ komand_id    │   │
│   │ itilizate_id │    *    │ id           │    *    │ id           │   │
│   │ non_konple   │         │ total        │         │ pwodui_id    │───┤
│   │ telefòn      │         │ estati       │         │ kantite      │   │
│   │ adrès        │         │ kreye_nan    │         │ pri_inite    │   │
│   └──────────────┘         └──────────────┘         └──────────────┘   │
│                                                              │          │
│                                                              │          │
│                                                              ▼          │
│   ┌──────────────┐         ┌──────────────┐                            │
│   │  kategori    │    1    │   pwodui     │                            │
│   ├──────────────┤◄────────├──────────────┤                            │
│   │ id           │    *    │ id           │                            │
│   │ non          │         │ kategori_id  │                            │
│   │ deskripsyon  │         │ non          │                            │
│   └──────────────┘         │ pri          │                            │
│                            │ stok         │                            │
│                            └──────────────┘                            │
│                                                                         │
│   LEJAND:                                                              │
│   ─────────                                                            │
│   1 ──► * : Yon-a-Plizyè (One-to-Many)                                │
│   ◄────── : Referans kle etranjè                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8.2 Tip Done

### 8.2.1 Tab Tip Done Prensipal

```
┌───────────────────────────────────────────────────────────────────────┐
│                       TIP DONE POSTGRESQL                             │
├───────────────────┬───────────────────┬───────────────────────────────┤
│       TIP         │      EGZANP       │         DESKRIPSYON           │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ TEXT              │ 'Bonjou'          │ Tèks san limit longè          │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ VARCHAR(n)        │ 'Ayiti'           │ Tèks ak longè maksimòm        │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ INTEGER           │ 42                │ Nimewo antye                  │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ BIGINT            │ 9223372036854     │ Gwo nimewo antye              │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ DECIMAL(p,s)      │ 199.99            │ Nimewo ak desimal presiz      │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ BOOLEAN           │ true / false      │ Vrè oswa Fo                   │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ DATE              │ '2024-01-15'      │ Dat sèlman                    │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ TIME              │ '14:30:00'        │ Lè sèlman                     │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ TIMESTAMP         │ '2024-01-15       │ Dat ak lè                     │
│                   │  14:30:00'        │                               │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ UUID              │ 'a0eebc99-9c0b-   │ Idantifyan inik inivèsèl      │
│                   │  4ef8-bb6d-...'   │                               │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ JSON              │ {"kle": "valè"}   │ Done JSON                     │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ JSONB             │ {"kle": "valè"}   │ Done JSON binè (pi rapid)     │
├───────────────────┼───────────────────┼───────────────────────────────┤
│ ARRAY             │ {1, 2, 3}         │ Lis valè                      │
└───────────────────┴───────────────────┴───────────────────────────────┘
```

### 8.2.2 Egzanp Itilizasyon Tip Done

```sql
-- Kreye yon tab ak divès tip done
CREATE TABLE egzanp_tip_done (
    -- Idantifyan
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Tèks
    non TEXT NOT NULL,
    kòd_postal VARCHAR(10),
    deskripsyon TEXT,

    -- Nimewo
    laj INTEGER CHECK (laj >= 0 AND laj <= 150),
    salè DECIMAL(12,2),
    kantite_atik BIGINT DEFAULT 0,

    -- Booleyen
    aktif BOOLEAN DEFAULT true,
    verifye BOOLEAN DEFAULT false,

    -- Dat ak Lè
    dat_nesans DATE,
    lè_randevou TIME,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ekspire_nan TIMESTAMP WITH TIME ZONE,

    -- JSON
    metadata JSONB DEFAULT '{}',
    preferans JSON,

    -- Lis (Array)
    etikèt TEXT[] DEFAULT '{}',
    nòt_yo INTEGER[]
);

-- Ajoute done egzanp
INSERT INTO egzanp_tip_done (
    non,
    kòd_postal,
    laj,
    salè,
    dat_nesans,
    metadata,
    etikèt
) VALUES (
    'Jan Pyè',
    'HT6120',
    35,
    75000.00,
    '1989-05-20',
    '{"lang": "kreyòl", "nivo": "avanse"}',
    ARRAY['kliyan', 'VIP', 'fidèl']
);
```

### 8.2.3 Kontrènt Done

```sql
-- Diferan tip kontrènt

CREATE TABLE egzanp_kontrent (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- NOT NULL: Valè obligatwa
    non TEXT NOT NULL,

    -- UNIQUE: Valè inik
    imel TEXT UNIQUE NOT NULL,

    -- CHECK: Kondisyon validasyon
    laj INTEGER CHECK (laj >= 18),
    pri DECIMAL(10,2) CHECK (pri > 0),

    -- DEFAULT: Valè pa defo
    estati TEXT DEFAULT 'aktif',
    kreye_nan TIMESTAMP DEFAULT NOW(),

    -- REFERENCES: Kle etranjè
    kategori_id UUID REFERENCES kategori(id),

    -- Kontrènt konbine
    CONSTRAINT pri_valid CHECK (pri >= 0 AND pri <= 1000000),
    CONSTRAINT estati_valid CHECK (estati IN ('aktif', 'inaktif', 'annatant'))
);
```

---

## 8.3 Relasyon Ant Tab

### 8.3.1 Tip Relasyon

```
┌─────────────────────────────────────────────────────────────────────┐
│                       TIP RELASYON                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. YON-A-YON (One-to-One)                                         │
│  ───────────────────────────                                       │
│  Chak ranje nan yon tab konekte ak yon sèl ranje nan lòt tab la    │
│                                                                     │
│  ┌──────────────┐         ┌──────────────┐                         │
│  │  itilizate   │    1    │    profil    │                         │
│  │  ──────────  │◄───────►│  ──────────  │                         │
│  │  id          │    1    │  itilizate_id│                         │
│  └──────────────┘         └──────────────┘                         │
│                                                                     │
│  2. YON-A-PLIZYÈ (One-to-Many)                                     │
│  ───────────────────────────                                       │
│  Yon ranje nan yon tab konekte ak plizyè ranje nan lòt tab la      │
│                                                                     │
│  ┌──────────────┐         ┌──────────────┐                         │
│  │  kategori    │    1    │   pwodui     │                         │
│  │  ──────────  │◄────────│  ──────────  │                         │
│  │  id          │    *    │  kategori_id │                         │
│  └──────────────┘         └──────────────┘                         │
│                                                                     │
│  3. PLIZYÈ-A-PLIZYÈ (Many-to-Many)                                 │
│  ───────────────────────────                                       │
│  Plizyè ranje nan yon tab konekte ak plizyè ranje nan lòt tab la   │
│  (Bezwen yon tab entèmedyè)                                        │
│                                                                     │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │   etidyan    │  *  │ etidyan_kou  │  *  │     kou      │        │
│  │  ──────────  │◄────│  ──────────  │────►│  ──────────  │        │
│  │  id          │  1  │  etidyan_id  │  1  │  id          │        │
│  └──────────────┘     │  kou_id      │     └──────────────┘        │
│                       └──────────────┘                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.3.2 Egzanp Relasyon Yon-a-Yon

```sql
-- Tab itilizatè
CREATE TABLE itilizate (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    imel TEXT UNIQUE NOT NULL,
    kreye_nan TIMESTAMP DEFAULT NOW()
);

-- Tab profil (yon-a-yon ak itilizatè)
CREATE TABLE profil (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    itilizate_id UUID UNIQUE REFERENCES itilizate(id) ON DELETE CASCADE,
    non_konple TEXT,
    foto_url TEXT,
    byografi TEXT,
    dat_nesans DATE
);

-- Rechèch ak JOIN
SELECT
    i.imel,
    p.non_konple,
    p.byografi
FROM itilizate i
JOIN profil p ON i.id = p.itilizate_id;
```

### 8.3.3 Egzanp Relasyon Yon-a-Plizyè

```sql
-- Tab kategori (yon)
CREATE TABLE kategori (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    deskripsyon TEXT
);

-- Tab pwodui (plizyè)
CREATE TABLE pwodui (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    kategori_id UUID REFERENCES kategori(id) ON DELETE SET NULL,
    non TEXT NOT NULL,
    pri DECIMAL(10,2) NOT NULL
);

-- Ajoute done
INSERT INTO kategori (non) VALUES ('Elektwonik'), ('Rad'), ('Manje');

INSERT INTO pwodui (kategori_id, non, pri)
SELECT id, 'Telefòn', 500.00 FROM kategori WHERE non = 'Elektwonik';

-- Rechèch ak JOIN
SELECT
    k.non AS kategori,
    p.non AS pwodui,
    p.pri
FROM pwodui p
JOIN kategori k ON p.kategori_id = k.id
ORDER BY k.non, p.non;
```

### 8.3.4 Egzanp Relasyon Plizyè-a-Plizyè

```sql
-- Tab etidyan
CREATE TABLE etidyan (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    imel TEXT UNIQUE NOT NULL
);

-- Tab kou
CREATE TABLE kou (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    pwofesè TEXT,
    kredi INTEGER DEFAULT 3
);

-- Tab entèmedyè (plizyè-a-plizyè)
CREATE TABLE enskripsyon (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    etidyan_id UUID REFERENCES etidyan(id) ON DELETE CASCADE,
    kou_id UUID REFERENCES kou(id) ON DELETE CASCADE,
    dat_enskripsyon DATE DEFAULT CURRENT_DATE,
    nòt DECIMAL(4,2),
    UNIQUE(etidyan_id, kou_id)  -- Yon etidyan pa ka enskri de fwa nan menm kou
);

-- Rechèch: Tout kou yon etidyan
SELECT
    e.non AS etidyan,
    k.non AS kou,
    en.nòt
FROM etidyan e
JOIN enskripsyon en ON e.id = en.etidyan_id
JOIN kou k ON en.kou_id = k.id
WHERE e.non = 'Mari Jozèf';

-- Rechèch: Tout etidyan nan yon kou
SELECT
    k.non AS kou,
    e.non AS etidyan,
    en.dat_enskripsyon
FROM kou k
JOIN enskripsyon en ON k.id = en.kou_id
JOIN etidyan e ON en.etidyan_id = e.id
WHERE k.non = 'Matematik Avanse';
```

---

## 8.4 Sekirite Nivo Ranje (RLS)

### 8.4.1 Kisa RLS Ye?

```
┌─────────────────────────────────────────────────────────────────────┐
│              SEKIRITE NIVO RANJE (RLS)                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  RLS = Row Level Security = Sekirite Nivo Ranje                    │
│                                                                     │
│  RLS pèmèt ou kontwole ki ranje chak itilizatè ka:                 │
│  - WÈ (SELECT)                                                     │
│  - AJOUTE (INSERT)                                                 │
│  - MODIFYE (UPDATE)                                                │
│  - EFASE (DELETE)                                                  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    TAB: dokiman                             │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ id │ itilizate_id │ non              │ kontni               │   │
│  ├────┼──────────────┼──────────────────┼──────────────────────┤   │
│  │ 1  │ user_A       │ Dokiman A1       │ ...                  │ ◄─┤ Itilizatè A ka wè
│  │ 2  │ user_A       │ Dokiman A2       │ ...                  │ ◄─┤
│  │ 3  │ user_B       │ Dokiman B1       │ ...                  │ ◄─┤ Itilizatè B ka wè
│  │ 4  │ user_C       │ Dokiman C1       │ ...                  │ ◄─┤ Itilizatè C ka wè
│  └────┴──────────────┴──────────────────┴──────────────────────┘   │
│                                                                     │
│  Chak itilizatè wè SÈLMAN pwòp done pa yo!                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.4.2 Aktive RLS

```sql
-- Kreye yon tab
CREATE TABLE dokiman (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    itilizate_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
    non TEXT NOT NULL,
    kontni TEXT,
    kreye_nan TIMESTAMP DEFAULT NOW()
);

-- ETAP 1: Aktive RLS sou tab la
ALTER TABLE dokiman ENABLE ROW LEVEL SECURITY;

-- ETAP 2: Kreye règ pou SELECT (wè)
CREATE POLICY "Itilizatè ka wè pwòp dokiman yo"
ON dokiman
FOR SELECT
USING (auth.uid() = itilizate_id);

-- ETAP 3: Kreye règ pou INSERT (ajoute)
CREATE POLICY "Itilizatè ka kreye dokiman"
ON dokiman
FOR INSERT
WITH CHECK (auth.uid() = itilizate_id);

-- ETAP 4: Kreye règ pou UPDATE (modifye)
CREATE POLICY "Itilizatè ka modifye pwòp dokiman yo"
ON dokiman
FOR UPDATE
USING (auth.uid() = itilizate_id)
WITH CHECK (auth.uid() = itilizate_id);

-- ETAP 5: Kreye règ pou DELETE (efase)
CREATE POLICY "Itilizatè ka efase pwòp dokiman yo"
ON dokiman
FOR DELETE
USING (auth.uid() = itilizate_id);
```

### 8.4.3 Egzanp Règ RLS Avanse

```sql
-- Egzanp 1: Piblik ka wè, sèlman pwopriyetè ka modifye
CREATE TABLE atik (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    itilizate_id UUID REFERENCES auth.users(id),
    tit TEXT NOT NULL,
    kontni TEXT,
    piblik BOOLEAN DEFAULT false
);

ALTER TABLE atik ENABLE ROW LEVEL SECURITY;

-- Tout moun ka wè atik piblik OSWA pwòp atik yo
CREATE POLICY "Wè atik piblik oswa pwòp atik"
ON atik FOR SELECT
USING (piblik = true OR auth.uid() = itilizate_id);

-- Sèlman pwopriyetè ka modifye
CREATE POLICY "Modifye pwòp atik"
ON atik FOR UPDATE
USING (auth.uid() = itilizate_id);

-----------------------------------------------------------

-- Egzanp 2: Sistèm ak wòl (admin, editè, lektè)
CREATE TABLE kontni (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    tit TEXT NOT NULL,
    kò TEXT,
    kreye_pa UUID REFERENCES auth.users(id)
);

-- Tab pou wòl itilizatè
CREATE TABLE wol_itilizate (
    itilizate_id UUID REFERENCES auth.users(id) PRIMARY KEY,
    wol TEXT CHECK (wol IN ('admin', 'edite', 'lekti'))
);

ALTER TABLE kontni ENABLE ROW LEVEL SECURITY;

-- Fonksyon pou tcheke wòl
CREATE OR REPLACE FUNCTION get_wol_itilizate()
RETURNS TEXT AS $$
    SELECT wol FROM wol_itilizate WHERE itilizate_id = auth.uid();
$$ LANGUAGE SQL SECURITY DEFINER;

-- Admin ka wè tout
CREATE POLICY "Admin wè tout"
ON kontni FOR SELECT
USING (get_wol_itilizate() = 'admin');

-- Editè ka wè ak modifye
CREATE POLICY "Editè wè ak modifye"
ON kontni FOR ALL
USING (get_wol_itilizate() IN ('admin', 'edite'));

-- Lektè ka wè sèlman
CREATE POLICY "Lektè wè sèlman"
ON kontni FOR SELECT
USING (get_wol_itilizate() = 'lekti');
```

### 8.4.4 Dyagram RLS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLOU SEKIRITE RLS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ITILIZATÈ A                                                      │
│       │                                                             │
│       ▼                                                             │
│   ┌────────────────┐                                               │
│   │ Demann SELECT  │                                               │
│   │ FROM dokiman   │                                               │
│   └───────┬────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│   ┌────────────────────────────────────────┐                       │
│   │         SUPABASE API                   │                       │
│   │                                        │                       │
│   │  1. Verifye token JWT                  │                       │
│   │  2. Ekstrè auth.uid() = user_A         │                       │
│   └───────────────┬────────────────────────┘                       │
│                   │                                                 │
│                   ▼                                                 │
│   ┌────────────────────────────────────────┐                       │
│   │         POSTGRESQL                     │                       │
│   │                                        │                       │
│   │  3. Aplike règ RLS                     │                       │
│   │     USING (auth.uid() = itilizate_id) │                       │
│   │                                        │                       │
│   │  4. SELECT * FROM dokiman              │                       │
│   │     WHERE itilizate_id = 'user_A'      │                       │
│   └───────────────┬────────────────────────┘                       │
│                   │                                                 │
│                   ▼                                                 │
│   ┌────────────────────────────────────────┐                       │
│   │  REZILTA: Sèlman dokiman itilizatè A   │                       │
│   │  ┌─────────────────────────────────┐   │                       │
│   │  │ id=1, non="Dokiman A1"          │   │                       │
│   │  │ id=2, non="Dokiman A2"          │   │                       │
│   │  └─────────────────────────────────┘   │                       │
│   └────────────────────────────────────────┘                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8.5 Konfigirasyon Otantifikasyon Itilizatè

### 8.5.1 Metòd Otantifikasyon Disponib

```
┌─────────────────────────────────────────────────────────────────────┐
│              METÒD OTANTIFIKASYON SUPABASE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  📧 IMÈL AK MODPAS                                          │   │
│  │  └── Metòd tradisyonèl                                      │   │
│  │  └── Konfirmasyon imèl opsyonèl                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  🔗 LYEN MAJIK                                              │   │
│  │  └── Koneksyon san modpas                                   │   │
│  │  └── Voye lyen pa imèl                                      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  📱 TELEFÒN (SMS)                                           │   │
│  │  └── Verifye ak kòd SMS                                     │   │
│  │  └── Bezwen konfigirasyon Twilio                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  🌐 FOUNISÈ SOSYAL                                          │   │
│  │  └── Google, Facebook, GitHub, Twitter                      │   │
│  │  └── Apple, Discord, Slack, ak plis                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.5.2 Konfigirasyon Otantifikasyon nan Tablo Bò

```
┌─────────────────────────────────────────────────────────────────────┐
│  SUPABASE │ Otantifikasyon > Founisè                                │
├───────────┼─────────────────────────────────────────────────────────┤
│           │                                                         │
│  Founisè  │   FOUNISÈ OTANTIFIKASYON                               │
│           │                                                         │
│  ─────────│   ┌─────────────────────────────────────────────────┐  │
│           │   │  ✓ Imèl                           [Aktive]      │  │
│  Modèl    │   │    ☑ Konfirme imèl                              │  │
│           │   │    ☑ Pèmèt enskripsyon nouvo                    │  │
│  ─────────│   └─────────────────────────────────────────────────┘  │
│           │                                                         │
│  Itiliza- │   ┌─────────────────────────────────────────────────┐  │
│  tè       │   │  ○ Telefòn                        [Dezaktive]   │  │
│           │   └─────────────────────────────────────────────────┘  │
│  ─────────│                                                         │
│           │   ┌─────────────────────────────────────────────────┐  │
│  Paramèt  │   │  ○ Google                         [Konfigire]   │  │
│           │   │    ID Kliyan: [____________________]            │  │
│           │   │    Sekrè: [____________________]                │  │
│           │   └─────────────────────────────────────────────────┘  │
│           │                                                         │
│           │   ┌─────────────────────────────────────────────────┐  │
│           │   │  ○ GitHub                         [Konfigire]   │  │
│           │   └─────────────────────────────────────────────────┘  │
│           │                                                         │
└───────────┴─────────────────────────────────────────────────────────┘
```

### 8.5.3 Egzanp Kòd Otantifikasyon

```javascript
// Enpòte bibliyotèk Supabase
const { createClient } = require('@supabase/supabase-js')

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_ANON_KEY
)

// ==========================================
// ENSKRIPSYON NOUVO ITILIZATÈ
// ==========================================

async function enskri(imel, modpas) {
    const { data, error } = await supabase.auth.signUp({
        email: imel,
        password: modpas,
    })

    if (error) {
        console.error('Erè enskripsyon:', error.message)
        return null
    }

    console.log('Itilizatè kreye:', data.user.email)
    return data
}

// ==========================================
// KONEKSYON
// ==========================================

async function konekte(imel, modpas) {
    const { data, error } = await supabase.auth.signInWithPassword({
        email: imel,
        password: modpas,
    })

    if (error) {
        console.error('Erè koneksyon:', error.message)
        return null
    }

    console.log('Koneksyon reyisi:', data.user.email)
    return data
}

// ==========================================
// KONEKSYON AK LYEN MAJIK
// ==========================================

async function konekteAvekLyenMajik(imel) {
    const { data, error } = await supabase.auth.signInWithOtp({
        email: imel,
    })

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    console.log('Lyen majik voye nan:', imel)
    return data
}

// ==========================================
// KONEKSYON AK FOUNISÈ SOSYAL
// ==========================================

async function konekteAvekGoogle() {
    const { data, error } = await supabase.auth.signInWithOAuth({
        provider: 'google',
    })

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data
}

// ==========================================
// DEKONEKSYON
// ==========================================

async function dekonekte() {
    const { error } = await supabase.auth.signOut()

    if (error) {
        console.error('Erè dekoneksyon:', error.message)
        return false
    }

    console.log('Dekoneksyon reyisi')
    return true
}

// ==========================================
// JWENN ITILIZATÈ AKTYÈL
// ==========================================

async function jwennItilizateAktyel() {
    const { data: { user } } = await supabase.auth.getUser()
    return user
}

// ==========================================
// EKOUT CHANJMAN ESTATI OTANTIFIKASYON
// ==========================================

supabase.auth.onAuthStateChange((evènman, sesyon) => {
    console.log('Evènman otantifikasyon:', evènman)

    if (evènman === 'SIGNED_IN') {
        console.log('Itilizatè konekte:', sesyon.user.email)
    } else if (evènman === 'SIGNED_OUT') {
        console.log('Itilizatè dekonekte')
    }
})
```

---

## 8.6 Jere Itilizatè

### 8.6.1 Tablo Itilizatè nan Tablo Bò

```
┌─────────────────────────────────────────────────────────────────────┐
│  SUPABASE │ Otantifikasyon > Itilizatè                              │
├───────────┼─────────────────────────────────────────────────────────┤
│           │                                                         │
│  Founisè  │   ITILIZATÈ (25 total)                                 │
│           │                                                         │
│  ─────────│   [Rechèch...]              [+ Ajoute Itilizatè]       │
│           │                                                         │
│  Modèl    │   ┌─────────────────────────────────────────────────┐  │
│           │   │ ☐ │ Imèl              │ Kreye       │ Dènye    │  │
│  ─────────│   ├───┼───────────────────┼─────────────┼──────────┤  │
│           │   │ ☐ │ jan@egzanp.com    │ 2 jou pase  │ 1 è pase │  │
│  Itiliza- │   │ ☐ │ mari@egzanp.com   │ 5 jou pase  │ 3 jou    │  │
│  tè       │   │ ☐ │ pyè@egzanp.com    │ 1 semèn     │ 2 jou    │  │
│           │   │ ☐ │ jak@egzanp.com    │ 2 semèn     │ 1 semèn  │  │
│  ─────────│   └─────────────────────────────────────────────────┘  │
│           │                                                         │
│  Paramèt  │   Paj 1 nan 3    [<] [1] [2] [3] [>]                   │
│           │                                                         │
└───────────┴─────────────────────────────────────────────────────────┘
```

### 8.6.2 Tab Profil Pèsonalize

```sql
-- Kreye tab profil pou enfòmasyon siplemantè
CREATE TABLE profil (
    id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
    non_konple TEXT,
    non_itilizate TEXT UNIQUE,
    avatar_url TEXT,
    byografi TEXT,
    sitwèb TEXT,
    lokasyon TEXT,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()),
    modifye_nan TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW())
);

-- Aktive RLS
ALTER TABLE profil ENABLE ROW LEVEL SECURITY;

-- Règ: Tout moun ka wè profil
CREATE POLICY "Profil piblik pou tout moun"
ON profil FOR SELECT
USING (true);

-- Règ: Sèlman pwopriyetè ka modifye
CREATE POLICY "Itilizatè ka modifye pwòp profil yo"
ON profil FOR UPDATE
USING (auth.uid() = id);

-- Fonksyon pou kreye profil otomatikman lè nouvo itilizatè enskri
CREATE OR REPLACE FUNCTION kreye_profil_pou_nouvo_itilizate()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO public.profil (id, non_konple)
    VALUES (
        NEW.id,
        NEW.raw_user_meta_data->>'non_konple'
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Deklanchè pou rele fonksyon an lè nouvo itilizatè kreye
CREATE TRIGGER on_auth_user_created
    AFTER INSERT ON auth.users
    FOR EACH ROW EXECUTE FUNCTION kreye_profil_pou_nouvo_itilizate();
```

### 8.6.3 Operasyon Admin sou Itilizatè

```javascript
// Kòd pou sèvè sèlman (itilize kle sèvis)
const { createClient } = require('@supabase/supabase-js')

const supabaseAdmin = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_SERVICE_KEY  // KLE SÈVIS - PA KLE ANONIM!
)

// ==========================================
// JWENN LIS ITILIZATÈ
// ==========================================

async function jwennToutItilizate() {
    const { data: { users }, error } = await supabaseAdmin.auth.admin.listUsers()

    if (error) {
        console.error('Erè:', error.message)
        return []
    }

    return users
}

// ==========================================
// KREYE ITILIZATÈ (ADMIN)
// ==========================================

async function kreYeItilizate(imel, modpas, metadata = {}) {
    const { data, error } = await supabaseAdmin.auth.admin.createUser({
        email: imel,
        password: modpas,
        email_confirm: true,  // Konfime imèl otomatikman
        user_metadata: metadata
    })

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data.user
}

// ==========================================
// MODIFYE ITILIZATÈ
// ==========================================

async function modifyeItilizate(itilizateId, donneYo) {
    const { data, error } = await supabaseAdmin.auth.admin.updateUserById(
        itilizateId,
        donneYo
    )

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data.user
}

// ==========================================
// EFASE ITILIZATÈ
// ==========================================

async function efaseItilizate(itilizateId) {
    const { error } = await supabaseAdmin.auth.admin.deleteUser(itilizateId)

    if (error) {
        console.error('Erè:', error.message)
        return false
    }

    return true
}
```

---

## 8.7 Depo pou Fichye

### 8.7.1 Kisa Depo Supabase Ye?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEPO SUPABASE                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Depo Supabase pèmèt ou estoke fichye tankou:                      │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │    IMAJ      │  │    VIDEYO    │  │   DOKIMAN    │              │
│  │  .jpg .png   │  │  .mp4 .mov   │  │  .pdf .doc   │              │
│  │  .gif .webp  │  │  .avi .webm  │  │  .xls .csv   │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                     │
│  ESTRIKTI DEPO:                                                    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  PWOJÈ                                                      │   │
│  │  └── BOKIT (avatars)                                        │   │
│  │      └── DOSYE (itilizate_123/)                            │   │
│  │          └── FICHYE (foto.jpg)                              │   │
│  │  └── BOKIT (dokiman)                                        │   │
│  │      └── DOSYE (rapò/)                                      │   │
│  │          └── FICHYE (rapò_2024.pdf)                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.7.2 Kreye Bokit nan Tablo Bò

```
┌─────────────────────────────────────────────────────────────────────┐
│  SUPABASE │ Depo > Bokit                                            │
├───────────┼─────────────────────────────────────────────────────────┤
│           │                                                         │
│  Bokit    │   KREYE NOUVO BOKIT                                    │
│           │                                                         │
│  ─────────│   ┌─────────────────────────────────────────────────┐  │
│           │   │                                                 │  │
│  avatars  │   │  Non Bokit: [avatars____________________]      │  │
│           │   │                                                 │  │
│  dokiman  │   │  Aksè Piblik:                                  │  │
│           │   │  ○ Prive (defò)                                │  │
│  imaj     │   │  ○ Piblik                                      │  │
│           │   │                                                 │  │
│           │   │  Limit Gwosè Fichye:                           │  │
│           │   │  [50________] MB                               │  │
│           │   │                                                 │  │
│           │   │  Tip Fichye Pèmi:                              │  │
│           │   │  [image/png, image/jpeg, image/gif]            │  │
│           │   │                                                 │  │
│           │   │  [  Anile  ]     [  Kreye Bokit  ]             │  │
│           │   │                                                 │  │
│           │   └─────────────────────────────────────────────────┘  │
│           │                                                         │
└───────────┴─────────────────────────────────────────────────────────┘
```

### 8.7.3 Konfigirasyon Bokit ak SQL

```sql
-- Kreye yon bokit piblik pou imaj
INSERT INTO storage.buckets (id, name, public)
VALUES ('imaj-piblik', 'imaj-piblik', true);

-- Kreye yon bokit prive pou dokiman
INSERT INTO storage.buckets (id, name, public)
VALUES ('dokiman-prive', 'dokiman-prive', false);

-- Règ RLS pou bokit piblik
-- Tout moun ka wè
CREATE POLICY "Imaj piblik aksesib pou tout moun"
ON storage.objects FOR SELECT
USING (bucket_id = 'imaj-piblik');

-- Sèlman itilizatè otantifye ka telechaje
CREATE POLICY "Itilizatè otantifye ka telechaje"
ON storage.objects FOR INSERT
WITH CHECK (
    bucket_id = 'imaj-piblik'
    AND auth.role() = 'authenticated'
);

-- Règ RLS pou bokit prive
-- Itilizatè wè sèlman pwòp fichye yo
CREATE POLICY "Itilizatè wè pwòp fichye yo"
ON storage.objects FOR SELECT
USING (
    bucket_id = 'dokiman-prive'
    AND auth.uid()::text = (storage.foldername(name))[1]
);

-- Itilizatè ka telechaje nan pwòp dosye yo
CREATE POLICY "Itilizatè telechaje nan pwòp dosye yo"
ON storage.objects FOR INSERT
WITH CHECK (
    bucket_id = 'dokiman-prive'
    AND auth.uid()::text = (storage.foldername(name))[1]
);
```

### 8.7.4 Operasyon Fichye nan Kòd

```javascript
const { createClient } = require('@supabase/supabase-js')

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_ANON_KEY
)

// ==========================================
// TELECHAJE FICHYE (UPLOAD)
// ==========================================

async function telechajeFichye(bokit, chemen, fichye) {
    const { data, error } = await supabase.storage
        .from(bokit)
        .upload(chemen, fichye, {
            cacheControl: '3600',
            upsert: false  // Mete true pou ranplase si egziste
        })

    if (error) {
        console.error('Erè telechajman:', error.message)
        return null
    }

    console.log('Fichye telechaje:', data.path)
    return data
}

// Egzanp itilizasyon
// await telechajeFichye('avatars', 'itilizate_123/foto.jpg', fichyeImaj)

// ==========================================
// JWENN URL PIBLIK
// ==========================================

function jwennUrlPiblik(bokit, chemen) {
    const { data } = supabase.storage
        .from(bokit)
        .getPublicUrl(chemen)

    return data.publicUrl
}

// Egzanp
// const url = jwennUrlPiblik('imaj-piblik', 'pwodui/telefon.jpg')

// ==========================================
// JWENN URL SIYE (POU FICHYE PRIVE)
// ==========================================

async function jwennUrlSiye(bokit, chemen, dire = 60) {
    const { data, error } = await supabase.storage
        .from(bokit)
        .createSignedUrl(chemen, dire)  // dire an segond

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data.signedUrl
}

// Egzanp: URL ki ekspire nan 60 segond
// const url = await jwennUrlSiye('dokiman-prive', 'rapò/kontra.pdf', 60)

// ==========================================
// TELECHAJE FICHYE (DOWNLOAD)
// ==========================================

async function telechajeFichyeDesann(bokit, chemen) {
    const { data, error } = await supabase.storage
        .from(bokit)
        .download(chemen)

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data  // Retounen Blob
}

// ==========================================
// LIS FICHYE NAN DOSYE
// ==========================================

async function lisFichye(bokit, dosye = '') {
    const { data, error } = await supabase.storage
        .from(bokit)
        .list(dosye, {
            limit: 100,
            offset: 0,
            sortBy: { column: 'name', order: 'asc' }
        })

    if (error) {
        console.error('Erè:', error.message)
        return []
    }

    return data
}

// ==========================================
// EFASE FICHYE
// ==========================================

async function efaseFichye(bokit, chemen) {
    const { error } = await supabase.storage
        .from(bokit)
        .remove([chemen])  // Lis fichye pou efase

    if (error) {
        console.error('Erè:', error.message)
        return false
    }

    return true
}

// ==========================================
// DEPLASE/RENOME FICHYE
// ==========================================

async function deplasFichye(bokit, ansyenChemen, nouvoChemen) {
    const { data, error } = await supabase.storage
        .from(bokit)
        .move(ansyenChemen, nouvoChemen)

    if (error) {
        console.error('Erè:', error.message)
        return null
    }

    return data
}
```

---

## 8.8 Abònman Tan Reyèl

### 8.8.1 Kisa Tan Reyèl Ye?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TAN REYÈL SUPABASE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Tan Reyèl pèmèt aplikasyon ou resevwa chanjman nan baz done       │
│  IMEDYATMAN san ou pa bezwen fè demann repete.                     │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                             │   │
│  │   ITILIZATÈ A          ITILIZATÈ B          ITILIZATÈ C    │   │
│  │       │                    │                    │          │   │
│  │       │                    │                    │          │   │
│  │       ▼                    ▼                    ▼          │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │              SUPABASE TAN REYÈL                     │   │   │
│  │  │                                                     │   │   │
│  │  │  Lè done chanje nan baz done a,                     │   │   │
│  │  │  Supabase voye notifikasyon bay tout                │   │   │
│  │  │  kliyan ki abòne.                                   │   │   │
│  │  │                                                     │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │                         │                                   │   │
│  │                         ▼                                   │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │               BAZ DONE POSTGRESQL                   │   │   │
│  │  │  ┌─────────────────────────────────────────────┐   │   │   │
│  │  │  │ INSERT, UPDATE, DELETE → Notifikasyon       │   │   │   │
│  │  │  └─────────────────────────────────────────────┘   │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │                                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.8.2 Aktive Tan Reyèl sou Tab

```sql
-- Aktive replikasyon pou yon tab
ALTER PUBLICATION supabase_realtime ADD TABLE mesaj;

-- Oswa nan SQL Editor nan tablo bò:
-- Ale nan Tab Editè > Chwazi Tab > Klike sou "Aktive Tan Reyèl"
```

### 8.8.3 Egzanp Kòd Abònman

```javascript
const { createClient } = require('@supabase/supabase-js')

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_ANON_KEY
)

// ==========================================
// ABÒNMAN BAZ (TOUT CHANJMAN NAN TAB)
// ==========================================

const abònman = supabase
    .channel('chanjman-mesaj')
    .on(
        'postgres_changes',
        {
            event: '*',  // Tout evènman: INSERT, UPDATE, DELETE
            schema: 'public',
            table: 'mesaj'
        },
        (payload) => {
            console.log('Chanjman resevwa:', payload)
            console.log('Tip evènman:', payload.eventType)
            console.log('Nouvo done:', payload.new)
            console.log('Ansyen done:', payload.old)
        }
    )
    .subscribe()

// ==========================================
// ABÒNMAN POU SÈLMAN INSERT
// ==========================================

const abònmanInsert = supabase
    .channel('nouvo-mesaj')
    .on(
        'postgres_changes',
        {
            event: 'INSERT',
            schema: 'public',
            table: 'mesaj'
        },
        (payload) => {
            console.log('Nouvo mesaj:', payload.new)
            // Trete nouvo mesaj la
            montreMesaj(payload.new)
        }
    )
    .subscribe()

// ==========================================
// ABÒNMAN POU SÈLMAN UPDATE
// ==========================================

const abònmanUpdate = supabase
    .channel('mizajou-mesaj')
    .on(
        'postgres_changes',
        {
            event: 'UPDATE',
            schema: 'public',
            table: 'mesaj'
        },
        (payload) => {
            console.log('Mesaj modifye')
            console.log('Ansyen:', payload.old)
            console.log('Nouvo:', payload.new)
        }
    )
    .subscribe()

// ==========================================
// ABÒNMAN AK FILT
// ==========================================

const abònmanFiltre = supabase
    .channel('mesaj-sal')
    .on(
        'postgres_changes',
        {
            event: '*',
            schema: 'public',
            table: 'mesaj',
            filter: 'sal_id=eq.123'  // Sèlman mesaj nan sal 123
        },
        (payload) => {
            console.log('Mesaj nan sal 123:', payload.new)
        }
    )
    .subscribe()

// ==========================================
// ANILE ABÒNMAN
// ==========================================

async function anileAbònman() {
    await supabase.removeChannel(abònman)
    console.log('Abònman anile')
}

// Oswa anile tout kanal
async function anileToutAbònman() {
    await supabase.removeAllChannels()
    console.log('Tout abònman anile')
}
```

### 8.8.4 Egzanp Aplikasyon Chat Tan Reyèl

```javascript
// Aplikasyon Chat Senp ak Tan Reyèl

const { createClient } = require('@supabase/supabase-js')

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_ANON_KEY
)

// ==========================================
// ESTRIKTI TAB MESAJ
// ==========================================

/*
CREATE TABLE mesaj (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    sal_id UUID NOT NULL,
    itilizate_id UUID REFERENCES auth.users(id),
    kontni TEXT NOT NULL,
    kreye_nan TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE mesaj ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Tout moun ka wè mesaj"
ON mesaj FOR SELECT USING (true);

CREATE POLICY "Itilizatè otantifye ka voye mesaj"
ON mesaj FOR INSERT
WITH CHECK (auth.uid() = itilizate_id);

-- Aktive tan reyèl
ALTER PUBLICATION supabase_realtime ADD TABLE mesaj;
*/

// ==========================================
// FONKSYON CHAT
// ==========================================

class SalChat {
    constructor(salId) {
        this.salId = salId
        this.abònman = null
    }

    // Kòmanse koute mesaj yo
    async kòmanse(fonksyonMesaj) {
        // Chaje mesaj egzistan yo
        const { data: mesajYo } = await supabase
            .from('mesaj')
            .select('*, profil:itilizate_id(non_konple)')
            .eq('sal_id', this.salId)
            .order('kreye_nan', { ascending: true })
            .limit(50)

        // Montre mesaj egzistan yo
        mesajYo.forEach(m => fonksyonMesaj(m))

        // Abòne pou nouvo mesaj
        this.abònman = supabase
            .channel(`sal-${this.salId}`)
            .on(
                'postgres_changes',
                {
                    event: 'INSERT',
                    schema: 'public',
                    table: 'mesaj',
                    filter: `sal_id=eq.${this.salId}`
                },
                (payload) => {
                    fonksyonMesaj(payload.new)
                }
            )
            .subscribe()
    }

    // Voye yon mesaj
    async voyeMesaj(kontni) {
        const { data: { user } } = await supabase.auth.getUser()

        if (!user) {
            console.error('Ou dwe konekte pou voye mesaj')
            return null
        }

        const { data, error } = await supabase
            .from('mesaj')
            .insert({
                sal_id: this.salId,
                itilizate_id: user.id,
                kontni: kontni
            })
            .select()
            .single()

        if (error) {
            console.error('Erè voye mesaj:', error.message)
            return null
        }

        return data
    }

    // Sispann koute
    async sispann() {
        if (this.abònman) {
            await supabase.removeChannel(this.abònman)
            this.abònman = null
        }
    }
}

// ==========================================
// ITILIZASYON
// ==========================================

async function egzanpItilizasyon() {
    const chat = new SalChat('123e4567-e89b-12d3-a456-426614174000')

    // Kòmanse koute mesaj
    await chat.kòmanse((mesaj) => {
        console.log(`[${mesaj.kreye_nan}] ${mesaj.kontni}`)
    })

    // Voye yon mesaj
    await chat.voyeMesaj('Bonjou tout moun!')

    // Pita... sispann koute
    // await chat.sispann()
}
```

### 8.8.5 Dyagram Flou Tan Reyèl

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLOU TAN REYÈL                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   KLIYAN A                    KLIYAN B                    KLIYAN C  │
│   (Voye Mesaj)               (Resevwa)                  (Resevwa)   │
│       │                          │                          │       │
│       │  1. INSERT mesaj         │                          │       │
│       │─────────────────────────►│                          │       │
│       │                          │                          │       │
│       ▼                          │                          │       │
│   ┌────────────────────────────────────────────────────────────┐   │
│   │                      SUPABASE                              │   │
│   │                                                            │   │
│   │  2. ┌──────────────────────────────────────────────────┐  │   │
│   │     │              POSTGRESQL                          │  │   │
│   │     │  INSERT INTO mesaj VALUES (...)                  │  │   │
│   │     │                    │                             │  │   │
│   │     │                    ▼                             │  │   │
│   │     │  3. TRIGGER: Notifye Tan Reyèl                   │  │   │
│   │     └──────────────────────────────────────────────────┘  │   │
│   │                          │                                 │   │
│   │                          ▼                                 │   │
│   │  4. ┌──────────────────────────────────────────────────┐  │   │
│   │     │            SÈVÈ TAN REYÈL                        │  │   │
│   │     │  Voye notifikasyon bay tout abòne                │  │   │
│   │     └──────────────────────────────────────────────────┘  │   │
│   │                          │                                 │   │
│   └──────────────────────────┼─────────────────────────────────┘   │
│                              │                                      │
│              ┌───────────────┼───────────────┐                     │
│              │               │               │                     │
│              ▼               ▼               ▼                     │
│   5. Resevwa          5. Resevwa      5. Resevwa                   │
│      notifikasyon        notifikasyon    notifikasyon              │
│              │               │               │                     │
│              ▼               ▼               ▼                     │
│       KLIYAN A         KLIYAN B        KLIYAN C                    │
│   (Konfime voye)    (Montre mesaj)  (Montre mesaj)                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8.9 Egzèsis Pratik

### Egzèsis 8.1: Kreye Tab ak Relasyon

```
┌─────────────────────────────────────────────────────────────────────┐
│                         EGZÈSIS 8.1                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  OBJEKTIF: Kreye yon sistèm baz done pou yon bibliyotèk            │
│                                                                     │
│  ETAP:                                                              │
│                                                                     │
│  1. Kreye tab "otè" ak kolòn:                                      │
│     - id (UUID, kle prensipal)                                     │
│     - non (TEXT, obligatwa)                                        │
│     - peyi (TEXT)                                                  │
│     - dat_nesans (DATE)                                            │
│                                                                     │
│  2. Kreye tab "liv" ak kolòn:                                      │
│     - id (UUID, kle prensipal)                                     │
│     - tit (TEXT, obligatwa)                                        │
│     - ote_id (UUID, referans otè)                                  │
│     - ane_piblikasyon (INTEGER)                                    │
│     - kantite_paj (INTEGER)                                        │
│                                                                     │
│  3. Kreye tab "prete" pou suiv ki moun prete ki liv                │
│     - id (UUID, kle prensipal)                                     │
│     - liv_id (UUID, referans liv)                                  │
│     - itilizate_id (UUID, referans auth.users)                     │
│     - dat_prete (DATE)                                             │
│     - dat_retounen (DATE)                                          │
│                                                                     │
│  4. Aktive RLS sou tout tab                                        │
│                                                                     │
│  5. Ajoute kèk done egzanp                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Solisyon:**

```sql
-- 1. Tab otè
CREATE TABLE ote (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non TEXT NOT NULL,
    peyi TEXT,
    dat_nesans DATE,
    kreye_nan TIMESTAMP DEFAULT NOW()
);

-- 2. Tab liv
CREATE TABLE liv (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    tit TEXT NOT NULL,
    ote_id UUID REFERENCES ote(id) ON DELETE SET NULL,
    ane_piblikasyon INTEGER CHECK (ane_piblikasyon > 0),
    kantite_paj INTEGER CHECK (kantite_paj > 0),
    kreye_nan TIMESTAMP DEFAULT NOW()
);

-- 3. Tab prete
CREATE TABLE prete (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    liv_id UUID REFERENCES liv(id) ON DELETE CASCADE,
    itilizate_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
    dat_prete DATE DEFAULT CURRENT_DATE,
    dat_retounen DATE,
    CONSTRAINT prete_valid CHECK (dat_retounen IS NULL OR dat_retounen >= dat_prete)
);

-- 4. Aktive RLS
ALTER TABLE ote ENABLE ROW LEVEL SECURITY;
ALTER TABLE liv ENABLE ROW LEVEL SECURITY;
ALTER TABLE prete ENABLE ROW LEVEL SECURITY;

-- Règ: Tout moun ka wè otè ak liv
CREATE POLICY "Otè piblik" ON ote FOR SELECT USING (true);
CREATE POLICY "Liv piblik" ON liv FOR SELECT USING (true);

-- Règ: Itilizatè wè sèlman pwòp prete yo
CREATE POLICY "Wè pwòp prete"
ON prete FOR SELECT
USING (auth.uid() = itilizate_id);

-- 5. Done egzanp
INSERT INTO ote (non, peyi, dat_nesans) VALUES
    ('Frankétienne', 'Ayiti', '1936-04-12'),
    ('Edwidge Danticat', 'Ayiti', '1969-01-19'),
    ('Jacques Roumain', 'Ayiti', '1907-06-04');

INSERT INTO liv (tit, ote_id, ane_piblikasyon, kantite_paj)
SELECT 'Dezafi', id, 1975, 272 FROM ote WHERE non = 'Frankétienne'
UNION ALL
SELECT 'Breath, Eyes, Memory', id, 1994, 234 FROM ote WHERE non = 'Edwidge Danticat'
UNION ALL
SELECT 'Gouverneurs de la Rosée', id, 1944, 192 FROM ote WHERE non = 'Jacques Roumain';
```

### Egzèsis 8.2: Konfigire Otantifikasyon ak Depo

```
┌─────────────────────────────────────────────────────────────────────┐
│                         EGZÈSIS 8.2                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  OBJEKTIF: Mete kanpe otantifikasyon ak depo pou foto profil       │
│                                                                     │
│  ETAP:                                                              │
│                                                                     │
│  1. Konfigire otantifikasyon ak imèl nan tablo bò                  │
│                                                                     │
│  2. Kreye tab "profil" ki lye ak auth.users                        │
│                                                                     │
│  3. Kreye bokit "avatars" pou foto profil                          │
│                                                                     │
│  4. Kreye règ RLS pou:                                             │
│     - Tout moun ka wè profil                                       │
│     - Sèlman pwopriyetè ka modifye profil yo                       │
│     - Sèlman pwopriyetè ka telechaje nan pwòp dosye yo             │
│                                                                     │
│  5. Kreye fonksyon deklanchè pou kreye profil otomatikman          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8.10 Rezime

```
┌─────────────────────────────────────────────────────────────────────┐
│                      REZIME CHAPIT 8                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✓ KREYE TAB                                                       │
│    └── Itilize entèfas grafik oswa SQL                             │
│    └── Defini kolòn ak kontrènt                                    │
│                                                                     │
│  ✓ TIP DONE                                                        │
│    └── TEXT, INTEGER, DECIMAL, BOOLEAN                             │
│    └── DATE, TIMESTAMP, UUID, JSON/JSONB                           │
│                                                                     │
│  ✓ RELASYON ANT TAB                                                │
│    └── Yon-a-Yon: UNIQUE REFERENCES                                │
│    └── Yon-a-Plizyè: REFERENCES                                    │
│    └── Plizyè-a-Plizyè: Tab entèmedyè                              │
│                                                                     │
│  ✓ SEKIRITE NIVO RANJE (RLS)                                       │
│    └── Kontwole aksè pou chak ranje                                │
│    └── Itilize auth.uid() pou idantifye itilizatè                  │
│    └── Kreye règ pou SELECT, INSERT, UPDATE, DELETE                │
│                                                                     │
│  ✓ OTANTIFIKASYON                                                  │
│    └── Imèl/Modpas, Lyen Majik, Telefòn                            │
│    └── Founisè sosyal (Google, GitHub, elatriye)                   │
│    └── Jesyon sesyon ak token                                      │
│                                                                     │
│  ✓ JERE ITILIZATÈ                                                  │
│    └── Tab profil pèsonalize                                       │
│    └── Fonksyon admin (kle sèvis)                                  │
│    └── Deklanchè pou otomatizasyon                                 │
│                                                                     │
│  ✓ DEPO FICHYE                                                     │
│    └── Kreye bokit (piblik oswa prive)                             │
│    └── Telechaje, telechaje desann, efase fichye                   │
│    └── URL piblik ak URL siye                                      │
│                                                                     │
│  ✓ TAN REYÈL                                                       │
│    └── Abònman pou chanjman nan baz done                           │
│    └── Resevwa INSERT, UPDATE, DELETE imedyatman                   │
│    └── Filtre pou evènman espesifik                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Pwochen Etap

Nan pwochen chapit yo, nou pral aprann kijan pou:
- Konekte Supabase ak n8n pou otomatizasyon
- Itilize Claude AI pou trete done nan Supabase
- Kreye flou travay konplè ak tout twa zouti yo

---

*Kontinye nan Chapit 9: Entegrasyon n8n ak Supabase →*
