# Kominote TWOKECRYPTO
# Chapit 14: Pwojè 3 - Aplikasyon Jesyon Done

## Entwodiksyon

Nan chapit sa a, nou pral konstwi yon aplikasyon konplè pou jesyon done ki pèmèt biznis yo jere done kliyan yo, analize done ak entèlijans atifisyèl, epi jenere rapò otomatik. Sistèm sa a konbine Supabase pou estokaj done, Claude pou analiz entèlijan, ak n8n pou otomatizasyon flodtravay.

## 14.1 Apèsi Jeneral Pwojè a

### 14.1.1 Objektif Pwojè a

```
+------------------------------------------------------------------------+
|                   APLIKASYON JESYON DONE                                |
+------------------------------------------------------------------------+
|                                                                         |
|   +-----------+      +------------+      +-----------+      +--------+  |
|   |  Done     | ---> |  Estoke    | ---> |  Analize  | ---> | Rapò   |  |
|   |  Antre    |      |  Supabase  |      |  Claude   |      | Tablo  |  |
|   +-----------+      +------------+      +-----------+      +--------+  |
|                             |                  |                        |
|                             v                  v                        |
|                      +------------+     +------------+                  |
|                      | Notifikasyon|    | Vizualizasyon|                |
|                      +------------+     +------------+                  |
|                                                                         |
+------------------------------------------------------------------------+
```

### 14.1.2 Fonksyonalite Prensipal

Aplikasyon jesyon done nou an pral gen kapasite sa yo:

1. **Jesyon Kliyan** - Kenbe enfòmasyon konplè sou kliyan yo
2. **Swiv Tranzaksyon** - Anrejistre ak swiv tout tranzaksyon yo
3. **Analiz Entèlijan** - Itilize Claude pou konprann done yo
4. **Rapò Otomatik** - Jenere rapò peryodik otomatikman
5. **Notifikasyon Entèlijan** - Avèti sou evènman enpòtan
6. **Tablo Bò Done** - Prepare done pou vizualizasyon

### 14.1.3 Achitekti Sistèm nan

```
+------------------------------------------------------------------------+
|                         ACHITEKTI KONPLÈ                                |
+------------------------------------------------------------------------+
|                                                                         |
|  ANTRE DONE             TRETMAN                SÒTI                     |
|  +-------------+      +-------------+      +-------------+              |
|  | Kliyan      |      |             |      | Rapò        |              |
|  | Tranzaksyon | ---> | Validasyon  | ---> | Analiz      |              |
|  | Pwodui      |      | + Estokaj   |      | Notifikasyon|              |
|  +-------------+      +-------------+      +-------------+              |
|        |                    |                    |                      |
|        v                    v                    v                      |
|  +-------------------------------------------------------------+       |
|  |                       SUPABASE                               |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  |  | kliyan   | |tranzaksyon| | pwodui  | | rapò     |        |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  |  | kategori | | analiz   | |notifikasyon| | metrik  |        |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  +-------------------------------------------------------------+       |
|                                                                         |
+------------------------------------------------------------------------+
```

---

## 14.2 Estrikti Baz Done Konplè

### 14.2.1 Tab Kliyan

```sql
-- =====================================================
-- TAB KLIYAN
-- =====================================================
-- Tab prensipal ki kenbe tout enfòmasyon sou kliyan yo

CREATE TABLE kliyan (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    kòd_kliyan VARCHAR(50) UNIQUE,

    -- Enfòmasyon debaz
    tip VARCHAR(20) NOT NULL DEFAULT 'endividyèl'
        CHECK (tip IN ('endividyèl', 'biznis', 'gouvènman', 'lòt')),
    non_konplè VARCHAR(255) NOT NULL,
    non_biznis VARCHAR(255),

    -- Kontak
    imèl VARCHAR(255),
    telefòn VARCHAR(50),
    telefòn_altènatif VARCHAR(50),

    -- Adrès
    adrès_ri TEXT,
    vil VARCHAR(100),
    depatman VARCHAR(100),
    peyi VARCHAR(100) DEFAULT 'Ayiti',
    kòd_postal VARCHAR(20),

    -- Enfòmasyon biznis (si aplikab)
    nimero_fiskal VARCHAR(50),
    sektè_aktivite VARCHAR(100),

    -- Klasifikasyon
    kategori_id UUID,
    nivo_kliyan VARCHAR(20) DEFAULT 'estanda'
        CHECK (nivo_kliyan IN ('estanda', 'ajan', 'vip', 'platin')),
    sou_kliyan_de UUID REFERENCES kliyan(id),

    -- Estati
    estati VARCHAR(20) DEFAULT 'aktif'
        CHECK (estati IN ('aktif', 'enaktif', 'sispann', 'pwospè')),

    -- Preferans
    lang_prefere VARCHAR(10) DEFAULT 'ht',
    metòd_kontak_prefere VARCHAR(20) DEFAULT 'imèl',

    -- Nòt
    nòt TEXT,
    etikèt TEXT[],

    -- Dat yo
    dat_premye_kontak DATE,
    dat_dènye_acha DATE,
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_kliyan_kòd ON kliyan(kòd_kliyan);
CREATE INDEX idx_kliyan_tip ON kliyan(tip);
CREATE INDEX idx_kliyan_estati ON kliyan(estati);
CREATE INDEX idx_kliyan_nivo ON kliyan(nivo_kliyan);
CREATE INDEX idx_kliyan_kategori ON kliyan(kategori_id);
CREATE INDEX idx_kliyan_imèl ON kliyan(imèl);
CREATE INDEX idx_kliyan_dat_kreye ON kliyan(dat_kreye DESC);

-- Rechèch tèks konplè
CREATE INDEX idx_kliyan_rechèch ON kliyan
    USING gin(to_tsvector('french', non_konplè || ' ' || COALESCE(non_biznis, '') || ' ' || COALESCE(imèl, '')));

-- Komentè
COMMENT ON TABLE kliyan IS 'Tab prensipal pou enfòmasyon kliyan yo';
COMMENT ON COLUMN kliyan.nivo_kliyan IS 'Nivo kliyan selon valè yo pote';
```

### 14.2.2 Tab Kategori Kliyan

```sql
-- =====================================================
-- TAB KATEGORI KLIYAN
-- =====================================================

CREATE TABLE kategori_kliyan (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non VARCHAR(100) NOT NULL UNIQUE,
    deskripsyon TEXT,
    koulè VARCHAR(7) DEFAULT '#3B82F6',
    paran_id UUID REFERENCES kategori_kliyan(id),
    lòd INTEGER DEFAULT 0,
    aktif BOOLEAN DEFAULT TRUE,
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Ajoute FK nan tab kliyan
ALTER TABLE kliyan ADD CONSTRAINT fk_kliyan_kategori
    FOREIGN KEY (kategori_id) REFERENCES kategori_kliyan(id);

-- Done inisyal
INSERT INTO kategori_kliyan (non, deskripsyon, koulè) VALUES
    ('Detay', 'Kliyan ki achte an detay', '#3B82F6'),
    ('Angwo', 'Kliyan ki achte an gwo', '#10B981'),
    ('Distribitè', 'Distribitè pwodui yo', '#F59E0B'),
    ('Patnè', 'Patnè biznis', '#8B5CF6');
```

### 14.2.3 Tab Pwodui ak Sèvis

```sql
-- =====================================================
-- TAB PWODUI AK SÈVIS
-- =====================================================

CREATE TABLE pwodui (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    kòd_pwodui VARCHAR(50) UNIQUE NOT NULL,

    -- Enfòmasyon debaz
    non VARCHAR(255) NOT NULL,
    deskripsyon TEXT,
    tip VARCHAR(20) NOT NULL CHECK (tip IN ('pwodui', 'sèvis')),

    -- Kategori
    kategori_pwodui_id UUID,

    -- Pri ak envantè
    pri_acha DECIMAL(15, 2),
    pri_vant DECIMAL(15, 2) NOT NULL,
    kantite_an_stòk INTEGER DEFAULT 0,
    nivo_rekòmand INTEGER DEFAULT 10,

    -- Metrik
    kantite_vann INTEGER DEFAULT 0,
    revni_total DECIMAL(15, 2) DEFAULT 0,

    -- Estati
    aktif BOOLEAN DEFAULT TRUE,
    disponib BOOLEAN DEFAULT TRUE,

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tab kategori pwodui
CREATE TABLE kategori_pwodui (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    non VARCHAR(100) NOT NULL UNIQUE,
    deskripsyon TEXT,
    paran_id UUID REFERENCES kategori_pwodui(id),
    aktif BOOLEAN DEFAULT TRUE,
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Ajoute FK
ALTER TABLE pwodui ADD CONSTRAINT fk_pwodui_kategori
    FOREIGN KEY (kategori_pwodui_id) REFERENCES kategori_pwodui(id);

-- Endèks
CREATE INDEX idx_pwodui_kòd ON pwodui(kòd_pwodui);
CREATE INDEX idx_pwodui_tip ON pwodui(tip);
CREATE INDEX idx_pwodui_kategori ON pwodui(kategori_pwodui_id);
CREATE INDEX idx_pwodui_aktif ON pwodui(aktif);

-- Komentè
COMMENT ON TABLE pwodui IS 'Katalòg pwodui ak sèvis';
```

### 14.2.4 Tab Tranzaksyon

```sql
-- =====================================================
-- TAB TRANZAKSYON
-- =====================================================
-- Tab ki kenbe tout tranzaksyon biznis yo

CREATE TABLE tranzaksyon (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    nimero_tranzaksyon VARCHAR(50) UNIQUE NOT NULL,

    -- Relasyon
    kliyan_id UUID NOT NULL REFERENCES kliyan(id),
    anplwaye_id UUID,

    -- Tip tranzaksyon
    tip VARCHAR(30) NOT NULL CHECK (tip IN (
        'vant', 'acha', 'retou', 'ranbousman',
        'peman', 'kredi', 'ajisteman', 'lòt'
    )),

    -- Montant
    sou_total DECIMAL(15, 2) NOT NULL,
    taks DECIMAL(15, 2) DEFAULT 0,
    rabè DECIMAL(15, 2) DEFAULT 0,
    total DECIMAL(15, 2) NOT NULL,

    -- Peman
    metòd_peman VARCHAR(30) CHECK (metòd_peman IN (
        'kach', 'kat', 'transfè', 'chèk', 'kredi', 'moncash', 'lòt'
    )),
    estati_peman VARCHAR(20) DEFAULT 'annatant' CHECK (estati_peman IN (
        'annatant', 'pasyèl', 'konplè', 'anreta', 'anile'
    )),
    montant_peye DECIMAL(15, 2) DEFAULT 0,
    balans DECIMAL(15, 2) DEFAULT 0,

    -- Estati tranzaksyon
    estati VARCHAR(20) DEFAULT 'ankou' CHECK (estati IN (
        'ankou', 'konplè', 'anile', 'ranbouse'
    )),

    -- Nòt
    nòt TEXT,
    referans_ekstèn VARCHAR(100),

    -- Metadata
    metadata JSONB DEFAULT '{}',

    -- Dat yo
    dat_tranzaksyon TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_echeyans DATE,
    dat_konplete TIMESTAMP WITH TIME ZONE,
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_tranzaksyon_nimero ON tranzaksyon(nimero_tranzaksyon);
CREATE INDEX idx_tranzaksyon_kliyan ON tranzaksyon(kliyan_id);
CREATE INDEX idx_tranzaksyon_tip ON tranzaksyon(tip);
CREATE INDEX idx_tranzaksyon_estati ON tranzaksyon(estati);
CREATE INDEX idx_tranzaksyon_dat ON tranzaksyon(dat_tranzaksyon DESC);
CREATE INDEX idx_tranzaksyon_estati_peman ON tranzaksyon(estati_peman);

-- Komentè
COMMENT ON TABLE tranzaksyon IS 'Tout tranzaksyon biznis yo';
```

### 14.2.5 Tab Detay Tranzaksyon

```sql
-- =====================================================
-- TAB DETAY TRANZAKSYON
-- =====================================================

CREATE TABLE detay_tranzaksyon (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Relasyon
    tranzaksyon_id UUID NOT NULL REFERENCES tranzaksyon(id) ON DELETE CASCADE,
    pwodui_id UUID NOT NULL REFERENCES pwodui(id),

    -- Detay
    kantite INTEGER NOT NULL DEFAULT 1,
    pri_inite DECIMAL(15, 2) NOT NULL,
    rabè DECIMAL(15, 2) DEFAULT 0,
    taks DECIMAL(15, 2) DEFAULT 0,
    total DECIMAL(15, 2) NOT NULL,

    -- Nòt
    nòt TEXT,

    -- Dat
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_detay_tranzaksyon ON detay_tranzaksyon(tranzaksyon_id);
CREATE INDEX idx_detay_pwodui ON detay_tranzaksyon(pwodui_id);

-- Komentè
COMMENT ON TABLE detay_tranzaksyon IS 'Detay chak tranzaksyon';
```

### 14.2.6 Tab Analiz ak Rapò

```sql
-- =====================================================
-- TAB ANALIZ DONE
-- =====================================================
-- Tab ki kenbe rezilta analiz Claude yo

CREATE TABLE analiz_done (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Tip analiz
    tip VARCHAR(50) NOT NULL CHECK (tip IN (
        'rezime_kliyan', 'tandans_vant', 'prediksyon',
        'segmantasyon', 'anomali', 'rekòmandasyon', 'lòt'
    )),

    -- Sijè analiz
    sijè VARCHAR(255) NOT NULL,
    deskripsyon TEXT,

    -- Done antre
    kèry_done TEXT,
    done_antre JSONB,
    paramèt JSONB DEFAULT '{}',

    -- Rezilta
    rezilta_analiz TEXT NOT NULL,
    rezime TEXT,
    rekòmandasyon TEXT[],
    metrik JSONB DEFAULT '{}',

    -- Token
    token_itilize INTEGER DEFAULT 0,

    -- Dat yo
    dat_analiz TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_ekspire DATE
);

-- Endèks
CREATE INDEX idx_analiz_tip ON analiz_done(tip);
CREATE INDEX idx_analiz_dat ON analiz_done(dat_analiz DESC);

-- Tab rapò otomatik
CREATE TABLE rapò (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Enfòmasyon rapò
    non VARCHAR(255) NOT NULL,
    tip VARCHAR(50) NOT NULL CHECK (tip IN (
        'jounal', 'semenn', 'mwa', 'trimès', 'ane', 'pèsonalize'
    )),
    deskripsyon TEXT,

    -- Peryòd
    dat_kòmanse DATE NOT NULL,
    dat_fen DATE NOT NULL,

    -- Kontni
    rezime TEXT,
    kontni_konplè JSONB NOT NULL,
    metrik_kle JSONB DEFAULT '{}',

    -- Analiz Claude
    analiz_id UUID REFERENCES analiz_done(id),
    rekòmandasyon TEXT[],

    -- Estati
    estati VARCHAR(20) DEFAULT 'jenere' CHECK (estati IN (
        'ankou', 'jenere', 'voye', 'achive'
    )),

    -- Distribisyon
    voye_pa_imèl BOOLEAN DEFAULT FALSE,
    destinatè TEXT[],

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_rapò_tip ON rapò(tip);
CREATE INDEX idx_rapò_peryòd ON rapò(dat_kòmanse, dat_fen);
CREATE INDEX idx_rapò_estati ON rapò(estati);
```

### 14.2.7 Tab Notifikasyon

```sql
-- =====================================================
-- TAB NOTIFIKASYON
-- =====================================================

CREATE TABLE notifikasyon (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Tip notifikasyon
    tip VARCHAR(50) NOT NULL CHECK (tip IN (
        'alèt', 'rapèl', 'enfòmasyon', 'avètisman', 'erè'
    )),
    priyorite VARCHAR(20) DEFAULT 'nòmal' CHECK (priyorite IN (
        'ba', 'nòmal', 'wo', 'ijan'
    )),

    -- Kontni
    tit VARCHAR(255) NOT NULL,
    mesaj TEXT NOT NULL,
    done_siplemantè JSONB DEFAULT '{}',

    -- Relasyon (opsyonèl)
    kliyan_id UUID REFERENCES kliyan(id),
    tranzaksyon_id UUID REFERENCES tranzaksyon(id),

    -- Destinatè
    destinatè_tip VARCHAR(20) CHECK (destinatè_tip IN (
        'sistèm', 'itilizatè', 'ekip', 'tout'
    )),
    destinatè_id UUID,

    -- Estati
    li BOOLEAN DEFAULT FALSE,
    trete BOOLEAN DEFAULT FALSE,

    -- Aksyon
    url_aksyon TEXT,
    bouton_aksyon VARCHAR(50),

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_li TIMESTAMP WITH TIME ZONE,
    dat_ekspire TIMESTAMP WITH TIME ZONE
);

-- Endèks
CREATE INDEX idx_notifikasyon_tip ON notifikasyon(tip);
CREATE INDEX idx_notifikasyon_priyorite ON notifikasyon(priyorite);
CREATE INDEX idx_notifikasyon_li ON notifikasyon(li);
CREATE INDEX idx_notifikasyon_dat ON notifikasyon(dat_kreye DESC);
```

### 14.2.8 Fonksyon ak Trigè

```sql
-- =====================================================
-- FONKSYON AK TRIGÈ
-- =====================================================

-- Fonksyon pou mete ajou dat modifikasyon
CREATE OR REPLACE FUNCTION mete_ajo_dat_modifye_done()
RETURNS TRIGGER AS $$
BEGIN
    NEW.dat_modifye = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigè yo
CREATE TRIGGER trigè_modifye_kliyan
    BEFORE UPDATE ON kliyan FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_done();

CREATE TRIGGER trigè_modifye_pwodui
    BEFORE UPDATE ON pwodui FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_done();

CREATE TRIGGER trigè_modifye_tranzaksyon
    BEFORE UPDATE ON tranzaksyon FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_done();

-- Fonksyon pou jenere nimero tranzaksyon
CREATE OR REPLACE FUNCTION jenere_nimero_tranzaksyon()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.nimero_tranzaksyon IS NULL THEN
        NEW.nimero_tranzaksyon = 'TRX-' ||
            TO_CHAR(NOW(), 'YYYYMMDD') || '-' ||
            LPAD(CAST(FLOOR(RANDOM() * 10000) AS TEXT), 4, '0');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigè_nimero_tranzaksyon
    BEFORE INSERT ON tranzaksyon FOR EACH ROW
    EXECUTE FUNCTION jenere_nimero_tranzaksyon();

-- Fonksyon pou kalkile balans tranzaksyon
CREATE OR REPLACE FUNCTION kalkile_balans()
RETURNS TRIGGER AS $$
BEGIN
    NEW.balans = NEW.total - NEW.montant_peye;

    IF NEW.balans <= 0 THEN
        NEW.estati_peman = 'konplè';
        NEW.balans = 0;
    ELSIF NEW.montant_peye > 0 THEN
        NEW.estati_peman = 'pasyèl';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigè_kalkile_balans
    BEFORE INSERT OR UPDATE ON tranzaksyon FOR EACH ROW
    EXECUTE FUNCTION kalkile_balans();

-- Fonksyon pou mete ajou envantè
CREATE OR REPLACE FUNCTION mete_ajo_envantè()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE pwodui
        SET
            kantite_an_stòk = kantite_an_stòk - NEW.kantite,
            kantite_vann = kantite_vann + NEW.kantite,
            revni_total = revni_total + NEW.total
        WHERE id = NEW.pwodui_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigè_mete_ajo_envantè
    AFTER INSERT ON detay_tranzaksyon FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_envantè();

-- Fonksyon pou kreye notifikasyon otomatik
CREATE OR REPLACE FUNCTION kreye_notifikasyon_envantè()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.kantite_an_stòk <= NEW.nivo_rekòmand AND
       OLD.kantite_an_stòk > OLD.nivo_rekòmand THEN
        INSERT INTO notifikasyon (tip, priyorite, tit, mesaj, done_siplemantè)
        VALUES (
            'avètisman',
            'wo',
            'Envantè ba pou ' || NEW.non,
            'Pwodui ' || NEW.non || ' (' || NEW.kòd_pwodui || ') gen sèlman ' ||
            NEW.kantite_an_stòk || ' inite. Nivo rekòmand: ' || NEW.nivo_rekòmand,
            jsonb_build_object('pwodui_id', NEW.id, 'kantite_aktyèl', NEW.kantite_an_stòk)
        );
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigè_notifikasyon_envantè
    AFTER UPDATE ON pwodui FOR EACH ROW
    EXECUTE FUNCTION kreye_notifikasyon_envantè();
```

### 14.2.9 Fonksyon Analiz ak Rapò

```sql
-- =====================================================
-- FONKSYON ANALIZ AK RAPÒ
-- =====================================================

-- Fonksyon pou jwenn rezime vant
CREATE OR REPLACE FUNCTION jwenn_rezime_vant(
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '30 days',
    p_dat_fen DATE DEFAULT CURRENT_DATE
)
RETURNS TABLE (
    total_tranzaksyon BIGINT,
    total_vant DECIMAL,
    mwayèn_pa_tranzaksyon DECIMAL,
    kliyan_inik BIGINT,
    pwodui_vann BIGINT,
    pi_gwo_vant DECIMAL,
    pi_piti_vant DECIMAL
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        COUNT(*)::BIGINT as total_tranzaksyon,
        COALESCE(SUM(total), 0) as total_vant,
        ROUND(COALESCE(AVG(total), 0), 2) as mwayèn_pa_tranzaksyon,
        COUNT(DISTINCT kliyan_id)::BIGINT as kliyan_inik,
        (SELECT COUNT(DISTINCT pwodui_id) FROM detay_tranzaksyon dt
         JOIN tranzaksyon t ON dt.tranzaksyon_id = t.id
         WHERE t.dat_tranzaksyon::DATE BETWEEN p_dat_kòmanse AND p_dat_fen)::BIGINT,
        COALESCE(MAX(total), 0) as pi_gwo_vant,
        COALESCE(MIN(total), 0) as pi_piti_vant
    FROM tranzaksyon
    WHERE tip = 'vant'
      AND estati = 'konplè'
      AND dat_tranzaksyon::DATE BETWEEN p_dat_kòmanse AND p_dat_fen;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou jwenn pi bon kliyan yo
CREATE OR REPLACE FUNCTION jwenn_pi_bon_kliyan(
    p_limit INTEGER DEFAULT 10,
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '365 days'
)
RETURNS TABLE (
    kliyan_id UUID,
    non_kliyan VARCHAR,
    total_acha DECIMAL,
    kantite_tranzaksyon BIGINT,
    mwayèn_acha DECIMAL,
    dènye_acha DATE
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        k.id,
        k.non_konplè,
        COALESCE(SUM(t.total), 0) as total_acha,
        COUNT(t.id)::BIGINT as kantite_tranzaksyon,
        ROUND(COALESCE(AVG(t.total), 0), 2) as mwayèn_acha,
        MAX(t.dat_tranzaksyon::DATE) as dènye_acha
    FROM kliyan k
    LEFT JOIN tranzaksyon t ON k.id = t.kliyan_id
        AND t.tip = 'vant'
        AND t.estati = 'konplè'
        AND t.dat_tranzaksyon >= p_dat_kòmanse
    GROUP BY k.id, k.non_konplè
    ORDER BY total_acha DESC
    LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou jwenn pwodui pi popilè yo
CREATE OR REPLACE FUNCTION jwenn_pwodui_popilè(
    p_limit INTEGER DEFAULT 10,
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '30 days'
)
RETURNS TABLE (
    pwodui_id UUID,
    non_pwodui VARCHAR,
    kantite_vann BIGINT,
    revni DECIMAL,
    kantite_tranzaksyon BIGINT
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        p.id,
        p.non,
        COALESCE(SUM(dt.kantite), 0)::BIGINT as kantite_vann,
        COALESCE(SUM(dt.total), 0) as revni,
        COUNT(DISTINCT dt.tranzaksyon_id)::BIGINT as kantite_tranzaksyon
    FROM pwodui p
    LEFT JOIN detay_tranzaksyon dt ON p.id = dt.pwodui_id
    LEFT JOIN tranzaksyon t ON dt.tranzaksyon_id = t.id
        AND t.dat_tranzaksyon::DATE >= p_dat_kòmanse
    GROUP BY p.id, p.non
    ORDER BY kantite_vann DESC
    LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou jwenn tandans vant pa jou
CREATE OR REPLACE FUNCTION jwenn_tandans_vant_pa_jou(
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '30 days',
    p_dat_fen DATE DEFAULT CURRENT_DATE
)
RETURNS TABLE (
    jou DATE,
    kantite_tranzaksyon BIGINT,
    total_vant DECIMAL,
    kantite_kliyan BIGINT
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        dat_tranzaksyon::DATE as jou,
        COUNT(*)::BIGINT as kantite_tranzaksyon,
        COALESCE(SUM(total), 0) as total_vant,
        COUNT(DISTINCT kliyan_id)::BIGINT as kantite_kliyan
    FROM tranzaksyon
    WHERE tip = 'vant'
      AND estati = 'konplè'
      AND dat_tranzaksyon::DATE BETWEEN p_dat_kòmanse AND p_dat_fen
    GROUP BY dat_tranzaksyon::DATE
    ORDER BY jou;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou prepare done pou tablo bò done
CREATE OR REPLACE FUNCTION jwenn_done_tablo_bò(
    p_peryòd VARCHAR DEFAULT 'mwa'
)
RETURNS JSONB AS $$
DECLARE
    dat_kòmanse DATE;
    rezilta JSONB;
BEGIN
    -- Detèmine dat kòmanse selon peryòd
    CASE p_peryòd
        WHEN 'jou' THEN dat_kòmanse := CURRENT_DATE;
        WHEN 'semèn' THEN dat_kòmanse := CURRENT_DATE - INTERVAL '7 days';
        WHEN 'mwa' THEN dat_kòmanse := CURRENT_DATE - INTERVAL '30 days';
        WHEN 'trimès' THEN dat_kòmanse := CURRENT_DATE - INTERVAL '90 days';
        WHEN 'ane' THEN dat_kòmanse := CURRENT_DATE - INTERVAL '365 days';
        ELSE dat_kòmanse := CURRENT_DATE - INTERVAL '30 days';
    END CASE;

    -- Konstwi rezilta
    SELECT jsonb_build_object(
        'peryòd', p_peryòd,
        'dat_kòmanse', dat_kòmanse,
        'dat_fen', CURRENT_DATE,
        'rezime_vant', (SELECT row_to_json(r) FROM jwenn_rezime_vant(dat_kòmanse) r),
        'pi_bon_kliyan', (SELECT jsonb_agg(row_to_json(r)) FROM jwenn_pi_bon_kliyan(5, dat_kòmanse) r),
        'pwodui_popilè', (SELECT jsonb_agg(row_to_json(r)) FROM jwenn_pwodui_popilè(5, dat_kòmanse) r),
        'tandans_jounalye', (SELECT jsonb_agg(row_to_json(r)) FROM jwenn_tandans_vant_pa_jou(dat_kòmanse) r),
        'total_kliyan', (SELECT COUNT(*) FROM kliyan WHERE estati = 'aktif'),
        'total_pwodui', (SELECT COUNT(*) FROM pwodui WHERE aktif = TRUE),
        'notifikasyon_pa_li', (SELECT COUNT(*) FROM notifikasyon WHERE li = FALSE)
    ) INTO rezilta;

    RETURN rezilta;
END;
$$ LANGUAGE plpgsql;
```

---

## 14.3 Analiz Done ak Claude

### 14.3.1 Konfigirasyon Analizè

```javascript
// =====================================================
// KONFIGIRASYON ANALIZÈ DONE
// =====================================================

const konfigAnalizè = {
    // Modèl Claude
    modèl: 'claude-sonnet-4-5-20250929',

    // Paramèt
    tanperati: 0.3, // Pi ba pou analiz presiz
    maksimumToken: 2000,

    // Enstriksyon sistèm
    enstriksyonSistèm: `Ou se yon analis done ekspè ki travay pou yon biznis Ayisyen.

RESPONSABLITE OU:
1. Analize done biznis yo ak presizyon
2. Idantifye tandans ak modèl yo
3. Bay rekòmandasyon pratik ak aksyonab
4. Ekri tout analiz an Kreyòl Ayisyen klè

FÒMA ANALIZ:
1. Rezime Egzekitif (2-3 fraz)
2. Dekouvèt Prensipal (lis)
3. Analiz Detaye
4. Rekòmandasyon Aksyonab
5. Metrik Kle

RÈG:
- Toujou bay kontèks pou chif yo
- Konpare ak peryòd anvan si posib
- Idantifye risk ak opòtinite
- Bay priyorite pou rekòmandasyon yo`,

    // Tip analiz sipòte
    tipAnaliz: {
        rezime_kliyan: {
            non: 'Rezime Konpòtman Kliyan',
            enstriksyon: 'Analize konpòtman acha kliyan yo epi idantifye segman ak opòtinite.'
        },
        tandans_vant: {
            non: 'Analiz Tandans Vant',
            enstriksyon: 'Analize tandans vant yo, idantifye sezon ak faktè ki afekte pèfòmans.'
        },
        prediksyon: {
            non: 'Prediksyon Biznis',
            enstriksyon: 'Baze sou done istorik yo, predi tandans pwochen mwa a.'
        },
        anomali: {
            non: 'Deteksyon Anomali',
            enstriksyon: 'Idantifye tout anomali oswa chanjman enpòtan nan done yo.'
        },
        rekòmandasyon: {
            non: 'Rekòmandasyon Estratejik',
            enstriksyon: 'Bay rekòmandasyon estratejik pou amelyore pèfòmans biznis la.'
        }
    }
};

module.exports = konfigAnalizè;
```

### 14.3.2 Fonksyon Analiz ak Claude

```javascript
// =====================================================
// FONKSYON ANALIZ AK CLAUDE
// =====================================================

/**
 * Egzekite yon analiz done ak Claude
 * @param {string} tipAnaliz - Tip analiz la
 * @param {Object} done - Done pou analize
 * @param {Object} paramèt - Paramèt siplemantè
 * @returns {Object} Rezilta analiz la
 */
async function egzekiteAnaliz(tipAnaliz, done, paramèt = {}) {
    // Verifye tip analiz la
    const konfig = konfigAnalizè.tipAnaliz[tipAnaliz];
    if (!konfig) {
        throw new Error(`Tip analiz "${tipAnaliz}" pa sipòte`);
    }

    // Prepare enstriksyon
    const enstriksyon = `${konfig.enstriksyon}

DONE POU ANALIZE:
${JSON.stringify(done, null, 2)}

PARAMÈT ADISYONÈL:
${JSON.stringify(paramèt, null, 2)}

Tanpri bay yon analiz konplè ak rekòmandasyon praktik.`;

    // Voye demann bay Claude
    const tanKòmanse = Date.now();

    const repons = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'x-api-key': process.env.CLE_API_CLAUDE,
            'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
            model: konfigAnalizè.modèl,
            max_tokens: konfigAnalizè.maksimumToken,
            temperature: konfigAnalizè.tanperati,
            system: konfigAnalizè.enstriksyonSistèm,
            messages: [
                { role: 'user', content: enstriksyon }
            ]
        })
    });

    if (!repons.ok) {
        const erè = await repons.text();
        throw new Error(`Erè Claude: ${erè}`);
    }

    const reponsDone = await repons.json();
    const analiz = reponsDone.content[0].text;

    // Ekstrè seksyon yo
    const rezime = ekstèSeksyon(analiz, 'Rezime Egzekitif');
    const rekòmandasyon = ekstèLis(analiz, 'Rekòmandasyon');

    return {
        tip: tipAnaliz,
        sijè: konfig.non,
        rezilta_analiz: analiz,
        rezime: rezime,
        rekòmandasyon: rekòmandasyon,
        token_itilize: reponsDone.usage.input_tokens + reponsDone.usage.output_tokens,
        tan_analiz_ms: Date.now() - tanKòmanse,
        modèl: reponsDone.model
    };
}

/**
 * Ekstrè yon seksyon nan tèks la
 */
function ekstèSeksyon(tèks, titSeksyon) {
    const regex = new RegExp(`##?\\s*${titSeksyon}[:\\s]*([\\s\\S]*?)(?=##|$)`, 'i');
    const match = tèks.match(regex);
    return match ? match[1].trim() : null;
}

/**
 * Ekstrè yon lis nan tèks la
 */
function ekstèLis(tèks, titSeksyon) {
    const seksyon = ekstèSeksyon(tèks, titSeksyon);
    if (!seksyon) return [];

    const lis = seksyon.match(/[-*]\s*(.+)/g);
    if (!lis) return [];

    return lis.map(l => l.replace(/^[-*]\s*/, '').trim());
}

/**
 * Jenere rapò konplè ak analiz
 */
async function jenereRapòKonplè(peryòd = 'mwa') {
    // Jwenn done nan baz done
    const { data: doneTablo, error } = await supabase
        .rpc('jwenn_done_tablo_bò', { p_peryòd: peryòd });

    if (error) {
        throw new Error('Erè pou jwenn done: ' + error.message);
    }

    // Egzekite analiz
    const analiz = await egzekiteAnaliz('tandans_vant', doneTablo, {
        peryòd: peryòd,
        objektif: 'Jenere rapò konplè pou biznis la'
    });

    // Estoke analiz la
    const { data: analizEstoke, error: erèEstoke } = await supabase
        .from('analiz_done')
        .insert({
            tip: 'tandans_vant',
            sijè: `Rapò ${peryòd} - ${new Date().toLocaleDateString('fr-HT')}`,
            done_antre: doneTablo,
            rezilta_analiz: analiz.rezilta_analiz,
            rezime: analiz.rezime,
            rekòmandasyon: analiz.rekòmandasyon,
            token_itilize: analiz.token_itilize
        })
        .select()
        .single();

    if (erèEstoke) {
        console.error('Erè estokaj analiz:', erèEstoke);
    }

    // Kreye rapò
    const { data: rapò, error: erèRapò } = await supabase
        .from('rapò')
        .insert({
            non: `Rapò ${peryòd} - ${new Date().toLocaleDateString('fr-HT')}`,
            tip: peryòd === 'jou' ? 'jounal' : peryòd === 'semèn' ? 'semèn' : 'mwa',
            dat_kòmanse: doneTablo.dat_kòmanse,
            dat_fen: doneTablo.dat_fen,
            rezime: analiz.rezime,
            kontni_konplè: doneTablo,
            analiz_id: analizEstoke?.id,
            rekòmandasyon: analiz.rekòmandasyon,
            estati: 'jenere'
        })
        .select()
        .single();

    return {
        rapò_id: rapò?.id,
        analiz_id: analizEstoke?.id,
        rezime: analiz.rezime,
        rekòmandasyon: analiz.rekòmandasyon,
        done: doneTablo
    };
}
```

---

## 14.4 Flodtravay n8n

### 14.4.1 Achitekti Flodtravay yo

```
+------------------------------------------------------------------------+
|                    FLODTRAVAY JESYON DONE                               |
+------------------------------------------------------------------------+
|                                                                         |
|  FLODTRAVAY 1: JESYON KLIYAN AK TRANZAKSYON                            |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Demann   |--->| Valide |--->| Estoke |--->| Repons   |              |
|  | (Webhook)|    | Done   |    | Supabase|   |          |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
|  FLODTRAVAY 2: ANALIZ OTOMATIK                                         |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Chak Jou |--->| Jwenn  |--->| Claude |--->| Estoke   |              |
|  | (Cron)   |    | Done   |    | Analiz |    | Rapò     |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
|  FLODTRAVAY 3: NOTIFIKASYON ENTÈLIJAN                                  |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Evènman  |--->| Evalye |--->| Claude |--->| Voye     |              |
|  | (Trigè)  |    | Enpòtans|   | Mesaj  |    | Alèt     |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
|  FLODTRAVAY 4: DONE TABLO BÒ                                           |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Demann   |--->| Agreje |--->| Fòmate |--->| Retounen |              |
|  | (Webhook)|    | Done   |    | JSON   |    | Done     |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
+------------------------------------------------------------------------+
```

### 14.4.2 Flodtravay Jesyon Kliyan

```json
{
  "name": "Jesyon Kliyan ak Tranzaksyon",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "kliyan",
        "responseMode": "responseNode"
      },
      "id": "webhook-kliyan",
      "name": "Webhook Kliyan",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "// Valide done kliyan\nconst done = $input.all()[0].json;\nconst aksyon = done.aksyon || 'kreye';\n\n// Validasyon selon aksyon\nif (aksyon === 'kreye') {\n  if (!done.non_konplè) {\n    throw new Error('Non kliyan obligatwa');\n  }\n  if (!done.tip || !['endividyèl', 'biznis', 'gouvènman', 'lòt'].includes(done.tip)) {\n    done.tip = 'endividyèl';\n  }\n}\n\nif (aksyon === 'modifye' || aksyon === 'efase') {\n  if (!done.id) {\n    throw new Error('ID kliyan obligatwa pou ' + aksyon);\n  }\n}\n\n// Jenere kòd kliyan si pa genyen\nif (aksyon === 'kreye' && !done.kòd_kliyan) {\n  done.kòd_kliyan = 'KLY-' + Date.now().toString(36).toUpperCase();\n}\n\nreturn [{ json: { aksyon, ...done } }];"
      },
      "id": "valide-kliyan",
      "name": "Valide Done Kliyan",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [450, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.aksyon }}",
              "value2": "kreye"
            }
          ]
        }
      },
      "id": "chwa-aksyon",
      "name": "Ki Aksyon?",
      "type": "n8n-nodes-base.switch",
      "typeVersion": 3,
      "position": [650, 300],
      "parameters": {
        "rules": {
          "rules": [
            {
              "outputKey": "kreye",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.aksyon }}",
                    "rightValue": "kreye",
                    "operator": { "type": "string", "operation": "equals" }
                  }
                ]
              }
            },
            {
              "outputKey": "modifye",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.aksyon }}",
                    "rightValue": "modifye",
                    "operator": { "type": "string", "operation": "equals" }
                  }
                ]
              }
            },
            {
              "outputKey": "jwenn",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.aksyon }}",
                    "rightValue": "jwenn",
                    "operator": { "type": "string", "operation": "equals" }
                  }
                ]
              }
            }
          ]
        }
      }
    },
    {
      "parameters": {
        "operation": "insert",
        "tableId": "kliyan",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            { "fieldId": "kòd_kliyan", "fieldValue": "={{ $json.kòd_kliyan }}" },
            { "fieldId": "tip", "fieldValue": "={{ $json.tip }}" },
            { "fieldId": "non_konplè", "fieldValue": "={{ $json.non_konplè }}" },
            { "fieldId": "non_biznis", "fieldValue": "={{ $json.non_biznis }}" },
            { "fieldId": "imèl", "fieldValue": "={{ $json.imèl }}" },
            { "fieldId": "telefòn", "fieldValue": "={{ $json.telefòn }}" },
            { "fieldId": "adrès_ri", "fieldValue": "={{ $json.adrès_ri }}" },
            { "fieldId": "vil", "fieldValue": "={{ $json.vil }}" },
            { "fieldId": "kategori_id", "fieldValue": "={{ $json.kategori_id }}" },
            { "fieldId": "nòt", "fieldValue": "={{ $json.nòt }}" }
          ]
        }
      },
      "id": "kreye-kliyan",
      "name": "Kreye Kliyan",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [850, 200],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "kliyan",
        "filterType": "string",
        "filterString": "=id=eq.{{ $json.id }}",
        "fieldsToSend": "autoMapInputData"
      },
      "id": "modifye-kliyan",
      "name": "Modifye Kliyan",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [850, 300],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "operation": "select",
        "tableId": "kliyan",
        "filterType": "string",
        "filterString": "={{ $json.id ? `id=eq.${$json.id}` : ($json.imèl ? `imèl=eq.${$json.imèl}` : `kòd_kliyan=eq.${$json.kòd_kliyan}`) }}"
      },
      "id": "jwenn-kliyan",
      "name": "Jwenn Kliyan",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [850, 400],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={\n  \"siksè\": true,\n  \"aksyon\": \"{{ $('Valide Done Kliyan').item.json.aksyon }}\",\n  \"kliyan\": {{ JSON.stringify($json) }}\n}"
      },
      "id": "retounen-repons",
      "name": "Retounen Repons",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [1050, 300]
    }
  ],
  "connections": {
    "Webhook Kliyan": { "main": [[{ "node": "Valide Done Kliyan" }]] },
    "Valide Done Kliyan": { "main": [[{ "node": "Ki Aksyon?" }]] },
    "Ki Aksyon?": {
      "main": [
        [{ "node": "Kreye Kliyan" }],
        [{ "node": "Modifye Kliyan" }],
        [{ "node": "Jwenn Kliyan" }]
      ]
    },
    "Kreye Kliyan": { "main": [[{ "node": "Retounen Repons" }]] },
    "Modifye Kliyan": { "main": [[{ "node": "Retounen Repons" }]] },
    "Jwenn Kliyan": { "main": [[{ "node": "Retounen Repons" }]] }
  }
}
```

### 14.4.3 Flodtravay Analiz Otomatik

```json
{
  "name": "Analiz Otomatik Chak Jou",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "hours",
              "triggerAtHour": 6
            }
          ]
        }
      },
      "id": "cron-analiz",
      "name": "Chak Jou a 6è AM",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "operation": "executeFunction",
        "functionName": "jwenn_done_tablo_bò",
        "functionParameters": "p_peryòd=jou"
      },
      "id": "jwenn-done",
      "name": "Jwenn Done Yè",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [450, 300],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "jsCode": "// Prepare done pou analiz\nconst done = $input.all()[0].json;\n\n// Verifye si gen done\nif (!done || !done.rezime_vant) {\n  return [{ json: { pa_gen_done: true } }];\n}\n\n// Fòmate done pou Claude\nconst doneFòmate = {\n  peryòd: 'Yè (' + new Date(Date.now() - 86400000).toLocaleDateString('fr-HT') + ')',\n  rezime: done.rezime_vant,\n  pi_bon_kliyan: done.pi_bon_kliyan,\n  pwodui_popilè: done.pwodui_popilè,\n  tandans: done.tandans_jounalye,\n  metrik_jeneral: {\n    total_kliyan: done.total_kliyan,\n    total_pwodui: done.total_pwodui,\n    notifikasyon_pa_li: done.notifikasyon_pa_li\n  }\n};\n\nreturn [{ json: doneFòmate }];"
      },
      "id": "prepare-done",
      "name": "Prepare Done pou Analiz",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [650, 300]
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ !$json.pa_gen_done }}",
              "value2": true
            }
          ]
        }
      },
      "id": "gen-done",
      "name": "Gen Done?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [850, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.anthropic.com/v1/messages",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            { "name": "anthropic-version", "value": "2023-06-01" },
            { "name": "content-type", "value": "application/json" }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"model\": \"claude-sonnet-4-5-20250929\",\n  \"max_tokens\": 1500,\n  \"temperature\": 0.3,\n  \"system\": \"Ou se yon analis biznis ekspè ki ekri an Kreyòl Ayisyen. Analize done yo epi bay yon rezime kout ak rekòmandasyon pratik.\",\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": \"Analize done biznis jounalye sa yo epi bay:\\n1. Rezime kout (3-4 fraz)\\n2. Obsèvasyon enpòtan (3 pwen)\\n3. Rekòmandasyon pou jodi a (2-3 aksyon)\\n\\nDONE:\\n\" + {{ JSON.stringify(JSON.stringify($json)) }}\n    }\n  ]\n}"
      },
      "id": "analiz-claude",
      "name": "Analiz ak Claude",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [1050, 200],
      "credentials": { "httpHeaderAuth": { "id": "2", "name": "Claude API Kle" } }
    },
    {
      "parameters": {
        "jsCode": "// Ekstrè rezilta analiz\nconst repons = $input.all()[0].json;\nconst doneOrijinal = $('Prepare Done pou Analiz').item.json;\n\nif (!repons.content || !repons.content[0]) {\n  throw new Error('Repons Claude pa valid');\n}\n\nconst analiz = repons.content[0].text;\n\n// Ekstrè rekòmandasyon\nconst rekòmandasyon = [];\nconst matchRekòmandasyon = analiz.match(/[-*]\\s*(.+)/g);\nif (matchRekòmandasyon) {\n  rekòmandasyon.push(...matchRekòmandasyon.slice(0, 5).map(r => r.replace(/^[-*]\\s*/, '')));\n}\n\nreturn [{\n  json: {\n    analiz_konplè: analiz,\n    rekòmandasyon,\n    done_analize: doneOrijinal,\n    token_itilize: repons.usage.input_tokens + repons.usage.output_tokens,\n    dat: new Date().toISOString()\n  }\n}];"
      },
      "id": "ekstrè-analiz",
      "name": "Ekstrè Rezilta Analiz",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1250, 200]
    },
    {
      "parameters": {
        "operation": "insert",
        "tableId": "analiz_done",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            { "fieldId": "tip", "fieldValue": "tandans_vant" },
            { "fieldId": "sijè", "fieldValue": "=Analiz Jounalye - {{ new Date().toLocaleDateString('fr-HT') }}" },
            { "fieldId": "done_antre", "fieldValue": "={{ JSON.stringify($json.done_analize) }}" },
            { "fieldId": "rezilta_analiz", "fieldValue": "={{ $json.analiz_konplè }}" },
            { "fieldId": "rekòmandasyon", "fieldValue": "={{ $json.rekòmandasyon }}" },
            { "fieldId": "token_itilize", "fieldValue": "={{ $json.token_itilize }}" }
          ]
        }
      },
      "id": "estoke-analiz",
      "name": "Estoke Analiz",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1450, 200],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "operation": "insert",
        "tableId": "notifikasyon",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            { "fieldId": "tip", "fieldValue": "enfòmasyon" },
            { "fieldId": "priyorite", "fieldValue": "nòmal" },
            { "fieldId": "tit", "fieldValue": "=Analiz Jounalye Disponib - {{ new Date().toLocaleDateString('fr-HT') }}" },
            { "fieldId": "mesaj", "fieldValue": "={{ $('Ekstrè Rezilta Analiz').item.json.analiz_konplè.substring(0, 500) }}..." },
            { "fieldId": "destinatè_tip", "fieldValue": "tout" }
          ]
        }
      },
      "id": "kreye-notifikasyon",
      "name": "Kreye Notifikasyon",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1650, 200],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    }
  ],
  "connections": {
    "Chak Jou a 6è AM": { "main": [[{ "node": "Jwenn Done Yè" }]] },
    "Jwenn Done Yè": { "main": [[{ "node": "Prepare Done pou Analiz" }]] },
    "Prepare Done pou Analiz": { "main": [[{ "node": "Gen Done?" }]] },
    "Gen Done?": { "main": [[{ "node": "Analiz ak Claude" }], []] },
    "Analiz ak Claude": { "main": [[{ "node": "Ekstrè Rezilta Analiz" }]] },
    "Ekstrè Rezilta Analiz": { "main": [[{ "node": "Estoke Analiz" }]] },
    "Estoke Analiz": { "main": [[{ "node": "Kreye Notifikasyon" }]] }
  }
}
```

### 14.4.4 Flodtravay Done Tablo Bò

```json
{
  "name": "Done Tablo Bò Done",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "GET",
        "path": "tablo-bò",
        "responseMode": "responseNode"
      },
      "id": "webhook-tablo",
      "name": "Webhook Tablo Bò",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "// Jwenn paramèt\nconst query = $input.all()[0].json.query || {};\nconst peryòd = query.peryòd || 'mwa';\n\n// Valide peryòd\nconst peryòdValid = ['jou', 'semèn', 'mwa', 'trimès', 'ane'];\nif (!peryòdValid.includes(peryòd)) {\n  throw new Error('Peryòd pa valid. Chwazi: ' + peryòdValid.join(', '));\n}\n\nreturn [{ json: { peryòd } }];"
      },
      "id": "valide-paramèt",
      "name": "Valide Paramèt",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [450, 300]
    },
    {
      "parameters": {
        "operation": "executeFunction",
        "functionName": "jwenn_done_tablo_bò",
        "functionParameters": "=p_peryòd={{ $json.peryòd }}"
      },
      "id": "jwenn-done-tablo",
      "name": "Jwenn Done Tablo Bò",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [650, 300],
      "credentials": { "supabaseApi": { "id": "1", "name": "Kont Supabase" } }
    },
    {
      "parameters": {
        "jsCode": "// Fòmate done pou tablo bò done\nconst done = $input.all()[0].json;\n\n// Kalkile metrik adisyonèl\nconst rezimeVant = done.rezime_vant || {};\nconst tandans = done.tandans_jounalye || [];\n\n// Kalkile kwasans\nlet kwasans = 0;\nif (tandans.length >= 2) {\n  const premyèMwatye = tandans.slice(0, Math.floor(tandans.length / 2));\n  const dezyèmMwatye = tandans.slice(Math.floor(tandans.length / 2));\n  \n  const totalPremye = premyèMwatye.reduce((a, b) => a + (b.total_vant || 0), 0);\n  const totalDezyèm = dezyèmMwatye.reduce((a, b) => a + (b.total_vant || 0), 0);\n  \n  if (totalPremye > 0) {\n    kwasans = ((totalDezyèm - totalPremye) / totalPremye) * 100;\n  }\n}\n\n// Fòmate pou vizualizasyon\nconst doneFòmate = {\n  rezime: {\n    total_vant: rezimeVant.total_vant || 0,\n    total_tranzaksyon: rezimeVant.total_tranzaksyon || 0,\n    mwayèn_pa_tranzaksyon: rezimeVant.mwayèn_pa_tranzaksyon || 0,\n    kliyan_inik: rezimeVant.kliyan_inik || 0,\n    kwasans_pousantaj: Math.round(kwasans * 100) / 100\n  },\n  grafik: {\n    vant_pa_jou: tandans.map(t => ({\n      dat: t.jou,\n      vant: t.total_vant,\n      tranzaksyon: t.kantite_tranzaksyon\n    })),\n    pi_bon_kliyan: (done.pi_bon_kliyan || []).map(k => ({\n      non: k.non_kliyan,\n      valè: k.total_acha\n    })),\n    pwodui_popilè: (done.pwodui_popilè || []).map(p => ({\n      non: p.non_pwodui,\n      kantite: p.kantite_vann,\n      revni: p.revni\n    }))\n  },\n  metrik: {\n    total_kliyan: done.total_kliyan || 0,\n    total_pwodui: done.total_pwodui || 0,\n    notifikasyon: done.notifikasyon_pa_li || 0\n  },\n  peryòd: done.peryòd,\n  dat_kòmanse: done.dat_kòmanse,\n  dat_fen: done.dat_fen,\n  dat_jenere: new Date().toISOString()\n};\n\nreturn [{ json: doneFòmate }];"
      },
      "id": "fòmate-done",
      "name": "Fòmate Done pou Tablo",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [850, 300]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ JSON.stringify($json) }}",
        "options": {
          "responseHeaders": {
            "entries": [
              { "name": "Content-Type", "value": "application/json" },
              { "name": "Access-Control-Allow-Origin", "value": "*" }
            ]
          }
        }
      },
      "id": "retounen-done",
      "name": "Retounen Done",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [1050, 300]
    }
  ],
  "connections": {
    "Webhook Tablo Bò": { "main": [[{ "node": "Valide Paramèt" }]] },
    "Valide Paramèt": { "main": [[{ "node": "Jwenn Done Tablo Bò" }]] },
    "Jwenn Done Tablo Bò": { "main": [[{ "node": "Fòmate Done pou Tablo" }]] },
    "Fòmate Done pou Tablo": { "main": [[{ "node": "Retounen Done" }]] }
  }
}
```

---

## 14.5 Tès ak Enstriksyon

### 14.5.1 Fichye Tès Konplè

```javascript
// =====================================================
// FICHYE TÈS: test-jesyon-done.js
// =====================================================

const konfigTès = {
    urlKliyan: 'http://localhost:5678/webhook/kliyan',
    urlTabloBò: 'http://localhost:5678/webhook/tablo-bò',
    atant: 2000
};

/**
 * Tès 1: Kreye nouvo kliyan
 */
async function tèsKreyeKliyan() {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 1: Kreye Nouvo Kliyan');
    console.log('='.repeat(60));

    const kliyan = {
        aksyon: 'kreye',
        tip: 'endividyèl',
        non_konplè: 'Jan Pyè Baptiste',
        imèl: 'jan.pye@egzanp.ht',
        telefòn: '+509 3456 7890',
        vil: 'Pòtoprens',
        nòt: 'Kliyan tès'
    };

    try {
        const repons = await fetch(konfigTès.urlKliyan, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(kliyan)
        });

        const done = await repons.json();

        if (done.siksè) {
            console.log('SIKSÈ!');
            console.log(`- ID: ${done.kliyan.id}`);
            console.log(`- Kòd: ${done.kliyan.kòd_kliyan}`);
            console.log(`- Non: ${done.kliyan.non_konplè}`);
            return done.kliyan;
        }
        return null;
    } catch (erè) {
        console.error('ERÈE:', erè.message);
        return null;
    }
}

/**
 * Tès 2: Jwenn kliyan
 */
async function tèsJwennKliyan(imèl) {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 2: Jwenn Kliyan');
    console.log('='.repeat(60));

    try {
        const repons = await fetch(konfigTès.urlKliyan, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ aksyon: 'jwenn', imèl })
        });

        const done = await repons.json();

        if (done.siksè) {
            console.log('SIKSÈ! Kliyan jwenn.');
            return true;
        }
        return false;
    } catch (erè) {
        console.error('ERÈE:', erè.message);
        return false;
    }
}

/**
 * Tès 3: Done tablo bò done
 */
async function tèsTabloBò() {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 3: Done Tablo Bò Done');
    console.log('='.repeat(60));

    try {
        const repons = await fetch(`${konfigTès.urlTabloBò}?peryòd=mwa`);
        const done = await repons.json();

        if (done.rezime) {
            console.log('SIKSÈ!');
            console.log('Rezime:');
            console.log(`- Total vant: ${done.rezime.total_vant}`);
            console.log(`- Tranzaksyon: ${done.rezime.total_tranzaksyon}`);
            console.log(`- Kliyan inik: ${done.rezime.kliyan_inik}`);
            console.log(`- Kwasans: ${done.rezime.kwasans_pousantaj}%`);
            return true;
        }
        return false;
    } catch (erè) {
        console.error('ERÈE:', erè.message);
        return false;
    }
}

/**
 * Egzekite tout tès yo
 */
async function egzekiteTès() {
    console.log('\n' + '#'.repeat(70));
    console.log('  KÒMANSE TÈS APLIKASYON JESYON DONE');
    console.log('#'.repeat(70));

    const rezilta = { total: 3, pase: 0, echwe: 0 };

    // Tès 1
    const kliyan = await tèsKreyeKliyan();
    if (kliyan) rezilta.pase++; else rezilta.echwe++;
    await new Promise(r => setTimeout(r, konfigTès.atant));

    // Tès 2
    if (kliyan && await tèsJwennKliyan(kliyan.imèl)) {
        rezilta.pase++;
    } else {
        rezilta.echwe++;
    }
    await new Promise(r => setTimeout(r, konfigTès.atant));

    // Tès 3
    if (await tèsTabloBò()) rezilta.pase++; else rezilta.echwe++;

    // Rezime
    console.log('\n' + '='.repeat(70));
    console.log('REZIME TÈS');
    console.log('='.repeat(70));
    console.log(`Total: ${rezilta.total} | Pase: ${rezilta.pase} | Echwe: ${rezilta.echwe}`);
    console.log(`Pousantaj: ${((rezilta.pase / rezilta.total) * 100).toFixed(1)}%`);
}

egzekiteTès();
```

### 14.5.2 Verifikasyon Baz Done

```sql
-- Verifye kliyan yo
SELECT
    id,
    kòd_kliyan,
    non_konplè,
    tip,
    estati,
    dat_kreye
FROM kliyan
ORDER BY dat_kreye DESC
LIMIT 10;

-- Verifye tranzaksyon yo
SELECT
    t.nimero_tranzaksyon,
    k.non_konplè as kliyan,
    t.tip,
    t.total,
    t.estati_peman,
    t.dat_tranzaksyon
FROM tranzaksyon t
JOIN kliyan k ON t.kliyan_id = k.id
ORDER BY t.dat_tranzaksyon DESC
LIMIT 10;

-- Verifye analiz yo
SELECT
    id,
    tip,
    sijè,
    token_itilize,
    dat_analiz
FROM analiz_done
ORDER BY dat_analiz DESC
LIMIT 5;

-- Verifye notifikasyon yo
SELECT
    tip,
    priyorite,
    tit,
    li,
    dat_kreye
FROM notifikasyon
ORDER BY dat_kreye DESC
LIMIT 10;

-- Estatistik jeneral
SELECT * FROM jwenn_rezime_vant();
SELECT * FROM jwenn_pi_bon_kliyan(5);
SELECT * FROM jwenn_pwodui_popilè(5);
```

---

## 14.6 Rezime ak Konklizyon

### 14.6.1 Sa Nou Te Konstwi

Nan chapit sa a, nou te konstwi yon aplikasyon jesyon done konplè ki:

1. **Jere kliyan** - Kreye, modifye, ak swiv enfòmasyon kliyan
2. **Swiv tranzaksyon** - Anrejistre ak jere tout tranzaksyon biznis
3. **Analize ak entèlijans atifisyèl** - Itilize Claude pou analiz done
4. **Jenere rapò otomatik** - Kreye rapò peryodik otomatikman
5. **Notifye sou evènman enpòtan** - Voye alèt entèlijan
6. **Prepare done pou vizualizasyon** - Done pare pou tablo bò done

### 14.6.2 Estrikti Pwojè Konplè

```
jesyon-done/
├── baz-done/
│   ├── 01-kliyan.sql
│   ├── 02-pwodui.sql
│   ├── 03-tranzaksyon.sql
│   ├── 04-analiz-rapò.sql
│   ├── 05-notifikasyon.sql
│   └── 06-fonksyon.sql
├── n8n/
│   ├── jesyon-kliyan.json
│   ├── analiz-otomatik.json
│   ├── notifikasyon-entèlijan.json
│   └── tablo-bò-done.json
├── kòd/
│   ├── konfigirasyon.js
│   └── analizè-done.js
└── tès/
    ├── test-jesyon-done.js
    └── verifye-baz-done.sql
```

### 14.6.3 Rezime Pati 6

Nan Pati 6, nou te konstwi twa pwojè pratik konplè:

| Pwojè | Chapit | Fonksyonalite Prensipal |
|-------|--------|-------------------------|
| Robo Chat Entèlijan | 12 | Konvèsasyon entèlijan ak memwa |
| Otomatizasyon Kontni | 13 | Jenerasyon ak piblikasyon otomatik |
| Jesyon Done | 14 | Analiz ak rapò entèlijan |

Chak pwojè demontre kijan pou konbine **n8n**, **Claude**, ak **Supabase** pou kreye aplikasyon reyèl ki pote valè.

### 14.6.4 Pwochen Etap

Pou kontinye devlopman ou:

1. **Pèsonalize pwojè yo** - Adapte yo pou bezwen espesifik ou
2. **Ajoute fonksyonalite** - Elaji kapasite yo
3. **Optimize pèfòmans** - Amelyore vitès ak efikasite
4. **Deploye an pwodiksyon** - Mete aplikasyon yo an liy
5. **Monitore ak amelyore** - Swiv metrik ak fè amelyorasyon

---

## Kesyon pou Pratik

1. Kijan ou ta ajoute sipò pou plizyè lajan (HTG, USD, EUR)?
2. Ki jan ou ta enplimante yon sistèm peman entegre?
3. Kijan ou ta ajoute fonksyonalite ekspòtasyon done?
4. Ki metrik ou ta ajoute pou mezire sante biznis la?
5. Kijan ou ta fè pou entegre ak sistèm kontablite?

---

## Felisitasyon!

Ou fini Pati 6 liv la! Kounye a ou gen kapasite pou:

- Konstwi aplikasyon konplè ak n8n, Claude, ak Supabase
- Kreye flodtravay otomatik sofistike
- Itilize entèlijans atifisyèl pou analiz done
- Jenere rapò ak notifikasyon entèlijan
- Prepare done pou vizualizasyon

Kontinye pratike ak eksperimante pou vin pi fò!
