# Kominote TWOKECRYPTO
# Chapit 13: Pwojè 2 - Sistèm Otomatizasyon Kontni

## Entwodiksyon

Nan chapit sa a, nou pral konstwi yon sistèm konplè pou otomatizasyon kontni. Sistèm sa a pral pèmèt ou kreye, jere, ak pibliye kontni otomatikman atravè entèlijans atifisyèl Claude, flodtravay n8n, ak baz done Supabase.

## 13.1 Apèsi Jeneral Pwojè a

### 13.1.1 Objektif Pwojè a

```
+------------------------------------------------------------------------+
|                   SISTÈM OTOMATIZASYON KONTNI                          |
+------------------------------------------------------------------------+
|                                                                        |
|   +-----------+      +------------+      +-----------+      +-------+  |
|   |  Sijè/    | ---> |   Claude   | ---> |  Estoke   | ---> |Pibliye|  |
|   |  Modèl    |      |  Jenere    |      | Supabase  |      | Rezo  |  |
|   +-----------+      +------------+      +-----------+      +-------+  |
|                             |                  |                       |
|                             v                  v                       |
|                      +------------+     +------------+                 |
|                      | Kalandriye |     |  Analitik  |                 |
|                      +------------+     +------------+                 |
|                                                                        |
+------------------------------------------------------------------------+
```

### 13.1.2 Fonksyonalite Prensipal

Sistèm otomatizasyon kontni nou an pral gen kapasite sa yo:

1. **Jenerasyon Kontni** - Kreye atik, pòs rezo sosyal, bilten otomatikman
2. **Kalandriye Piblikasyon** - Planifye kontni pou diferan dat ak lè
3. **Plizyè Platfòm** - Sipòte blòg, rezo sosyal, imèl
4. **Analitik Pèfòmans** - Swiv ki jan kontni yo pèfòme
5. **Modèl Kontni** - Kreye ak itilize modèl pou diferan tip kontni
6. **Apwobasyon Flodtravay** - Revize kontni anvan piblikasyon

### 13.1.3 Achitekti Sistèm nan

```
+------------------------------------------------------------------------+
|                         ACHITEKTI KONPLÈ                                |
+------------------------------------------------------------------------+
|                                                                         |
|  ANTRE KONTNI           TRETMAN              SÒTI                       |
|  +-------------+      +-------------+      +-------------+              |
|  | Sijè        |      |             |      | Blòg        |              |
|  | Modèl       | ---> |   n8n       | ---> | Rezo Sosyal |              |
|  | Kalandriye  |      | + Claude    |      | Imèl        |              |
|  +-------------+      +-------------+      +-------------+              |
|        |                    |                    |                      |
|        v                    v                    v                      |
|  +-------------------------------------------------------------+       |
|  |                       SUPABASE                               |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  |  | kontni   | | modèl    | | kalandriye| | analitik |        |       |
|  |  +----------+ +----------+ +----------+ +----------+        |       |
|  |  +----------+ +----------+ +----------+                     |       |
|  |  | kategori | | platfòm  | | itilizatè|                     |       |
|  |  +----------+ +----------+ +----------+                     |       |
|  +-------------------------------------------------------------+       |
|                                                                         |
+------------------------------------------------------------------------+
```

### 13.1.4 Ka Itilizasyon

1. **Blòg Otomatik** - Jenere atik blòg chak semèn sou sijè espesifik
2. **Rezo Sosyal** - Kreye piblikasyon pou diferan platfòm
3. **Bilten Imèl** - Prepare kontni pou bilten otomatik
4. **Dokimantasyon** - Jenere dokimantasyon teknik
5. **Maketing Kontni** - Kreye kontni pwomosyonèl

---

## 13.2 Estrikti Baz Done

### 13.2.1 Tab Kategori Kontni

```sql
-- =====================================================
-- TAB KATEGORI KONTNI
-- =====================================================
-- Tab sa a kenbe diferan kategori kontni yo

CREATE TABLE kategori_kontni (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Enfòmasyon kategori
    non VARCHAR(100) NOT NULL UNIQUE,
    deskripsyon TEXT,
    koulè VARCHAR(7) DEFAULT '#3B82F6',
    ikòn VARCHAR(50),

    -- Organizasyon
    paran_id UUID REFERENCES kategori_kontni(id),
    lòd INTEGER DEFAULT 0,

    -- Estati
    aktif BOOLEAN DEFAULT TRUE,

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_kategori_paran ON kategori_kontni(paran_id);
CREATE INDEX idx_kategori_aktif ON kategori_kontni(aktif);

-- Done inisyal
INSERT INTO kategori_kontni (non, deskripsyon, koulè) VALUES
    ('Teknoloji', 'Atik sou teknoloji ak inovasyon', '#3B82F6'),
    ('Biznis', 'Konsèy biznis ak antreprenarya', '#10B981'),
    ('Edikasyon', 'Kontni edikasyonèl', '#F59E0B'),
    ('Sante', 'Enfòmasyon sou sante ak byennèt', '#EF4444'),
    ('Kilti', 'Atik sou kilti ak atizana', '#8B5CF6');

-- Komentè
COMMENT ON TABLE kategori_kontni IS 'Kategori pou òganize kontni yo';
```

### 13.2.2 Tab Platfòm Piblikasyon

```sql
-- =====================================================
-- TAB PLATFÒM PIBLIKASYON
-- =====================================================
-- Tab sa a kenbe enfòmasyon sou diferan platfòm piblikasyon

CREATE TABLE platfòm_piblikasyon (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Enfòmasyon platfòm
    non VARCHAR(100) NOT NULL UNIQUE,
    tip VARCHAR(50) NOT NULL CHECK (tip IN ('blòg', 'rezo_sosyal', 'imèl', 'lòt')),
    deskripsyon TEXT,
    ikòn VARCHAR(100),

    -- Konfigirasyon API
    url_api TEXT,
    kle_api_chifre TEXT,
    konfigirasyon JSONB DEFAULT '{}',

    -- Limit
    limit_karaktè INTEGER,
    limit_imaj INTEGER,
    sipòte_videyo BOOLEAN DEFAULT FALSE,

    -- Estati
    aktif BOOLEAN DEFAULT TRUE,

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Done inisyal
INSERT INTO platfòm_piblikasyon (non, tip, limit_karaktè, sipòte_videyo) VALUES
    ('Blòg Prensipal', 'blòg', NULL, TRUE),
    ('Rezo Sosyal X', 'rezo_sosyal', 280, FALSE),
    ('Rezo Sosyal FB', 'rezo_sosyal', 63206, TRUE),
    ('Rezo Sosyal IG', 'rezo_sosyal', 2200, TRUE),
    ('Bilten Imèl', 'imèl', NULL, FALSE);

-- Komentè
COMMENT ON TABLE platfòm_piblikasyon IS 'Platfòm kote nou ka pibliye kontni';
COMMENT ON COLUMN platfòm_piblikasyon.kle_api_chifre IS 'Kle API chifre pou sekirite';
```

### 13.2.3 Tab Modèl Kontni

```sql
-- =====================================================
-- TAB MODÈL KONTNI
-- =====================================================
-- Tab sa a kenbe modèl pou jenere diferan tip kontni

CREATE TABLE modèl_kontni (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Enfòmasyon modèl
    non VARCHAR(200) NOT NULL,
    deskripsyon TEXT,

    -- Tip ak kategori
    tip VARCHAR(50) NOT NULL CHECK (tip IN (
        'atik_blòg', 'pòs_sosyal', 'bilten', 'deskripsyon_pwodui',
        'paj_vant', 'dokimantasyon', 'lòt'
    )),
    kategori_id UUID REFERENCES kategori_kontni(id),

    -- Modèl Claude
    enstriksyon_sistèm TEXT NOT NULL,
    modèl_kontni TEXT NOT NULL,
    egzanp_rezilta TEXT,

    -- Paramèt Claude
    tanperati DECIMAL(2,1) DEFAULT 0.7,
    maksimum_token INTEGER DEFAULT 2000,

    -- Varyab
    varyab JSONB DEFAULT '[]',

    -- Estati
    aktif BOOLEAN DEFAULT TRUE,

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_modèl_tip ON modèl_kontni(tip);
CREATE INDEX idx_modèl_kategori ON modèl_kontni(kategori_id);
CREATE INDEX idx_modèl_aktif ON modèl_kontni(aktif);

-- Egzanp modèl
INSERT INTO modèl_kontni (non, tip, enstriksyon_sistèm, modèl_kontni, varyab) VALUES
(
    'Atik Blòg Teknoloji',
    'atik_blòg',
    'Ou se yon redaktè ekspè ki ekri atik blòg an Kreyòl Ayisyen. Atik ou yo dwe:
- Enformè ak angaje lektè yo
- Itilize yon ton pwofesyonèl men aksesib
- Enkli egzanp konkrè
- Estrikti ak tit ak sou-tit klè',
    'Ekri yon atik blòg sou sijè sa a: {{sijè}}

Atik la dwe genyen:
1. Yon tit ki atire atansyon
2. Yon entwodiksyon ki akwoche lektè a
3. 3-5 seksyon prensipal ak sou-tit
4. Egzanp oswa etid ka
5. Yon konklizyon ak apèl a aksyon

Longè: {{longè}} mo anviwon
Ton: {{ton}}
Odyans: {{odyans}}',
    '[{"non": "sijè", "obligatwa": true}, {"non": "longè", "defò": "800"}, {"non": "ton", "defò": "pwofesyonèl"}, {"non": "odyans", "defò": "jeneral"}]'
),
(
    'Pòs Rezo Sosyal',
    'pòs_sosyal',
    'Ou se yon espesyalis maketing rezo sosyal ki kreye kontni an Kreyòl Ayisyen. Kontni ou dwe:
- Kout ak angajan
- Itilize emosyon apwopriye
- Ankouraje entèraksyon',
    'Kreye yon pòs rezo sosyal sou: {{sijè}}

Platfòm: {{platfòm}}
Limit karaktè: {{limit_karaktè}}
Ton: {{ton}}
Apèl a aksyon: {{apèl_aksyon}}

Enkli hashtag apwopriye.',
    '[{"non": "sijè", "obligatwa": true}, {"non": "platfòm", "defò": "jeneral"}, {"non": "limit_karaktè", "defò": "280"}, {"non": "ton", "defò": "angajan"}, {"non": "apèl_aksyon", "defò": "Pataje"}]'
),
(
    'Bilten Imèl Semèn',
    'bilten',
    'Ou se yon redaktè bilten ki ekri an Kreyòl Ayisyen. Bilten ou yo dwe:
- Gen yon ton zanmitay ak pèsonèl
- Bay valè reyèl bay lektè yo
- Klè ak byen òganize',
    'Ekri yon bilten imèl sou: {{sijè}}

Estrikti:
1. Salitasyon pèsonalize
2. Entwodiksyon kout sou kontèks la
3. Kontni prensipal ({{kantite_pwen}} pwen)
4. Rekòmandasyon oswa konsèy pratik
5. Kloti ak apèl a aksyon

Non antrepriz: {{non_antrepriz}}
Ton: {{ton}}',
    '[{"non": "sijè", "obligatwa": true}, {"non": "kantite_pwen", "defò": "3"}, {"non": "non_antrepriz", "defò": "Nou"}, {"non": "ton", "defò": "zanmitay"}]'
);

-- Komentè
COMMENT ON TABLE modèl_kontni IS 'Modèl pou jenere diferan tip kontni ak Claude';
```

### 13.2.4 Tab Kontni Jenere

```sql
-- =====================================================
-- TAB KONTNI JENERE
-- =====================================================
-- Tab prensipal ki kenbe tout kontni ki jenere yo

CREATE TABLE kontni (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Relasyon
    modèl_id UUID REFERENCES modèl_kontni(id),
    kategori_id UUID REFERENCES kategori_kontni(id),
    otè_id UUID,

    -- Enfòmasyon kontni
    tit VARCHAR(500) NOT NULL,
    rezime TEXT,
    kontni_konplè TEXT NOT NULL,
    kontni_orijinal TEXT,

    -- Metadata
    mo_kle TEXT[],
    etikèt TEXT[],
    imaj_prensipal TEXT,

    -- Paramèt jenerasyon
    paramèt_jenerasyon JSONB DEFAULT '{}',
    token_itilize INTEGER DEFAULT 0,

    -- Estati
    estati VARCHAR(30) DEFAULT 'bouyon' CHECK (estati IN (
        'bouyon', 'annatant_apwobasyon', 'apwouve',
        'planifye', 'pibliye', 'achive', 'efase'
    )),

    -- Dat yo
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_kontni_modèl ON kontni(modèl_id);
CREATE INDEX idx_kontni_kategori ON kontni(kategori_id);
CREATE INDEX idx_kontni_otè ON kontni(otè_id);
CREATE INDEX idx_kontni_estati ON kontni(estati);
CREATE INDEX idx_kontni_dat_kreye ON kontni(dat_kreye DESC);

-- Endèks pou rechèch tèks
CREATE INDEX idx_kontni_rechèch ON kontni
    USING gin(to_tsvector('french', tit || ' ' || COALESCE(kontni_konplè, '')));

-- Komentè
COMMENT ON TABLE kontni IS 'Kontni ki jenere pa sistèm nan';
COMMENT ON COLUMN kontni.estati IS 'Estati kontni: bouyon, annatant, apwouve, planifye, pibliye, achive, efase';
```

### 13.2.5 Tab Kalandriye Piblikasyon

```sql
-- =====================================================
-- TAB KALANDRIYE PIBLIKASYON
-- =====================================================
-- Tab sa a jere planifikasyon piblikasyon kontni yo

CREATE TABLE kalandriye_piblikasyon (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Relasyon
    kontni_id UUID NOT NULL REFERENCES kontni(id) ON DELETE CASCADE,
    platfòm_id UUID NOT NULL REFERENCES platfòm_piblikasyon(id),

    -- Planifikasyon
    dat_planifye TIMESTAMP WITH TIME ZONE NOT NULL,
    zòn_orè VARCHAR(50) DEFAULT 'Amerik/Pòtoprens',

    -- Kontni adapte pou platfòm
    tit_adapte VARCHAR(500),
    kontni_adapte TEXT,
    metadata_platfòm JSONB DEFAULT '{}',

    -- Estati piblikasyon
    estati VARCHAR(30) DEFAULT 'planifye' CHECK (estati IN (
        'planifye', 'ankou', 'pibliye', 'echwe', 'anile'
    )),

    -- Rezilta piblikasyon
    id_ekstèn VARCHAR(255),
    url_piblikasyon TEXT,
    erè_mesaj TEXT,

    -- Dat yo
    dat_pibliye TIMESTAMP WITH TIME ZONE,
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    dat_modifye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_kalandriye_kontni ON kalandriye_piblikasyon(kontni_id);
CREATE INDEX idx_kalandriye_platfòm ON kalandriye_piblikasyon(platfòm_id);
CREATE INDEX idx_kalandriye_dat ON kalandriye_piblikasyon(dat_planifye);
CREATE INDEX idx_kalandriye_estati ON kalandriye_piblikasyon(estati);

-- Komentè
COMMENT ON TABLE kalandriye_piblikasyon IS 'Kalandriye pou planifye piblikasyon kontni';
```

### 13.2.6 Tab Analitik Kontni

```sql
-- =====================================================
-- TAB ANALITIK KONTNI
-- =====================================================
-- Tab sa a kenbe estatistik sou pèfòmans kontni yo

CREATE TABLE analitik_kontni (
    -- Idantifikasyon
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,

    -- Relasyon
    kontni_id UUID NOT NULL REFERENCES kontni(id) ON DELETE CASCADE,
    platfòm_id UUID REFERENCES platfòm_piblikasyon(id),

    -- Peryòd
    dat_estatistik DATE NOT NULL,

    -- Metrik angajman
    vy INTEGER DEFAULT 0,
    klik INTEGER DEFAULT 0,
    pataj INTEGER DEFAULT 0,
    kòmantè INTEGER DEFAULT 0,
    renmen INTEGER DEFAULT 0,

    -- Metrik konvèsyon
    enskripsyon INTEGER DEFAULT 0,
    acha INTEGER DEFAULT 0,
    telechaje INTEGER DEFAULT 0,

    -- Metrik pèfòmans
    pousantaj_klik DECIMAL(5,2),
    pousantaj_konvèsyon DECIMAL(5,2),
    tan_mwayèn_lekti INTEGER,

    -- Sous trafik
    sous_trafik JSONB DEFAULT '{}',

    -- Dat
    dat_kreye TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Endèks
CREATE INDEX idx_analitik_kontni ON analitik_kontni(kontni_id);
CREATE INDEX idx_analitik_platfòm ON analitik_kontni(platfòm_id);
CREATE INDEX idx_analitik_dat ON analitik_kontni(dat_estatistik DESC);

-- Kontrènt inik pou evite diplika
CREATE UNIQUE INDEX idx_analitik_inik ON analitik_kontni(kontni_id, platfòm_id, dat_estatistik);

-- Komentè
COMMENT ON TABLE analitik_kontni IS 'Estatistik pèfòmans kontni yo';
```

### 13.2.7 Fonksyon ak Trigè

```sql
-- =====================================================
-- FONKSYON AK TRIGÈ
-- =====================================================

-- Fonksyon pou mete ajou dat modifikasyon
CREATE OR REPLACE FUNCTION mete_ajo_dat_modifye_kontni()
RETURNS TRIGGER AS $$
BEGIN
    NEW.dat_modifye = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigè yo
CREATE TRIGGER trigè_modifye_kategori
    BEFORE UPDATE ON kategori_kontni
    FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_kontni();

CREATE TRIGGER trigè_modifye_modèl
    BEFORE UPDATE ON modèl_kontni
    FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_kontni();

CREATE TRIGGER trigè_modifye_kontni
    BEFORE UPDATE ON kontni
    FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_kontni();

CREATE TRIGGER trigè_modifye_kalandriye
    BEFORE UPDATE ON kalandriye_piblikasyon
    FOR EACH ROW
    EXECUTE FUNCTION mete_ajo_dat_modifye_kontni();

-- Fonksyon pou jwenn kontni ki dwe pibliye
CREATE OR REPLACE FUNCTION jwenn_kontni_pou_pibliye(
    p_limit INTEGER DEFAULT 10
)
RETURNS TABLE (
    kalandriye_id UUID,
    kontni_id UUID,
    platfòm_id UUID,
    tit TEXT,
    kontni_adapte TEXT,
    dat_planifye TIMESTAMP WITH TIME ZONE,
    platfòm_non VARCHAR(100),
    platfòm_tip VARCHAR(50)
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        kp.id as kalandriye_id,
        kp.kontni_id,
        kp.platfòm_id,
        COALESCE(kp.tit_adapte, k.tit)::TEXT as tit,
        COALESCE(kp.kontni_adapte, k.kontni_konplè) as kontni_adapte,
        kp.dat_planifye,
        pp.non as platfòm_non,
        pp.tip as platfòm_tip
    FROM kalandriye_piblikasyon kp
    JOIN kontni k ON kp.kontni_id = k.id
    JOIN platfòm_piblikasyon pp ON kp.platfòm_id = pp.id
    WHERE kp.estati = 'planifye'
      AND kp.dat_planifye <= NOW()
      AND k.estati IN ('apwouve', 'planifye')
      AND pp.aktif = TRUE
    ORDER BY kp.dat_planifye ASC
    LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou jwenn estatistik kontni
CREATE OR REPLACE FUNCTION jwenn_estatistik_kontni(
    p_kontni_id UUID,
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '30 days',
    p_dat_fen DATE DEFAULT CURRENT_DATE
)
RETURNS TABLE (
    total_vy BIGINT,
    total_klik BIGINT,
    total_pataj BIGINT,
    total_renmen BIGINT,
    total_kòmantè BIGINT,
    mwayèn_pousantaj_klik DECIMAL,
    mwayèn_tan_lekti INTEGER
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        COALESCE(SUM(vy), 0) as total_vy,
        COALESCE(SUM(klik), 0) as total_klik,
        COALESCE(SUM(pataj), 0) as total_pataj,
        COALESCE(SUM(renmen), 0) as total_renmen,
        COALESCE(SUM(kòmantè), 0) as total_kòmantè,
        ROUND(AVG(pousantaj_klik), 2) as mwayèn_pousantaj_klik,
        ROUND(AVG(tan_mwayèn_lekti))::INTEGER as mwayèn_tan_lekti
    FROM analitik_kontni
    WHERE kontni_id = p_kontni_id
      AND dat_estatistik BETWEEN p_dat_kòmanse AND p_dat_fen;
END;
$$ LANGUAGE plpgsql;

-- Fonksyon pou rapò kontni pa peryòd
CREATE OR REPLACE FUNCTION jwenn_rapò_kontni(
    p_dat_kòmanse DATE DEFAULT CURRENT_DATE - INTERVAL '30 days',
    p_dat_fen DATE DEFAULT CURRENT_DATE
)
RETURNS TABLE (
    total_kontni_kreye BIGINT,
    total_kontni_pibliye BIGINT,
    total_token_itilize BIGINT,
    pi_popilè_kontni UUID,
    kategori_pi_popilè UUID
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        (SELECT COUNT(*) FROM kontni
         WHERE dat_kreye BETWEEN p_dat_kòmanse AND p_dat_fen + INTERVAL '1 day'),
        (SELECT COUNT(*) FROM kontni
         WHERE estati = 'pibliye'
         AND dat_modifye BETWEEN p_dat_kòmanse AND p_dat_fen + INTERVAL '1 day'),
        (SELECT COALESCE(SUM(token_itilize), 0) FROM kontni
         WHERE dat_kreye BETWEEN p_dat_kòmanse AND p_dat_fen + INTERVAL '1 day'),
        (SELECT kontni_id FROM analitik_kontni
         WHERE dat_estatistik BETWEEN p_dat_kòmanse AND p_dat_fen
         GROUP BY kontni_id ORDER BY SUM(vy) DESC LIMIT 1),
        (SELECT kategori_id FROM kontni
         WHERE dat_kreye BETWEEN p_dat_kòmanse AND p_dat_fen + INTERVAL '1 day'
         GROUP BY kategori_id ORDER BY COUNT(*) DESC LIMIT 1);
END;
$$ LANGUAGE plpgsql;
```

---

## 13.3 Enstriksyon Claude pou Jenerasyon Kontni

### 13.3.1 Konfigirasyon Jeneratè Kontni

```javascript
// =====================================================
// KONFIGIRASYON JENERATÈ KONTNI
// =====================================================

const konfigJeneratè = {
    // Modèl Claude
    modèl: 'claude-sonnet-4-5-20250929',

    // Paramèt defò
    tanperati: 0.7,
    maksimumToken: 2000,

    // Enstriksyon sistèm jeneral
    enstriksyonSistèmJeneral: `Ou se yon ekspè nan kreyasyon kontni pwofesyonèl an Kreyòl Ayisyen.

RÈG JENERAL POU TOUT KONTNI:

1. LANG AK STIL:
   - Ekri an Kreyòl Ayisyen natirèl ak pwofesyonèl
   - Evite anglisism lè gen mo Kreyòl ki ekivalan
   - Adapte ton an selon tip kontni an

2. KALITE KONTNI:
   - Kontni dwe orijinal ak valè ajoute
   - Toujou verifye presizyon enfòmasyon yo
   - Estrikti kontni yo byen ak tit ak sou-tit klè

3. OPTIMIZASYON:
   - Itilize mo kle natirèlman nan kontni an
   - Kreye tit ki atire atansyon
   - Enkli apèl a aksyon lè sa apwopriye

4. FÒMA:
   - Respekte limit longè ki mande yo
   - Adapte fòma pou platfòm sib la
   - Itilize lis ak paragraf pou lizibilite`,

    // Tip kontni sipòte
    tipKontni: {
        atik_blòg: {
            non: 'Atik Blòg',
            tanperati: 0.7,
            maksimumToken: 3000
        },
        pòs_sosyal: {
            non: 'Pòs Rezo Sosyal',
            tanperati: 0.8,
            maksimumToken: 500
        },
        bilten: {
            non: 'Bilten Imèl',
            tanperati: 0.6,
            maksimumToken: 1500
        },
        deskripsyon_pwodui: {
            non: 'Deskripsyon Pwodui',
            tanperati: 0.5,
            maksimumToken: 800
        }
    }
};

module.exports = konfigJeneratè;
```

### 13.3.2 Fonksyon Jenerasyon Kontni

```javascript
// =====================================================
// FONKSYON JENERASYON KONTNI
// =====================================================

/**
 * Jenere kontni ak Claude
 * @param {Object} paramèt - Paramèt pou jenerasyon
 * @returns {Object} Kontni jenere ak metadata
 */
async function jenereKontni(paramèt) {
    const {
        modèl_id,
        varyab,
        tipKontni = 'atik_blòg',
        kategori
    } = paramèt;

    // Jwenn modèl nan baz done
    const { data: modèl, error: erèModèl } = await supabase
        .from('modèl_kontni')
        .select('*')
        .eq('id', modèl_id)
        .single();

    if (erèModèl || !modèl) {
        throw new Error('Modèl kontni pa jwenn');
    }

    // Ranplase varyab yo nan modèl la
    let enstriksyon = modèl.modèl_kontni;
    let enstriksyonSistèm = modèl.enstriksyon_sistèm;

    // Ranplase chak varyab
    if (varyab && typeof varyab === 'object') {
        for (const [kle, valè] of Object.entries(varyab)) {
            const regex = new RegExp(`{{${kle}}}`, 'g');
            enstriksyon = enstriksyon.replace(regex, valè);
        }
    }

    // Ranplase varyab ki pa bay ak valè defò
    const varyabModèl = modèl.varyab || [];
    for (const v of varyabModèl) {
        if (v.defò) {
            const regex = new RegExp(`{{${v.non}}}`, 'g');
            enstriksyon = enstriksyon.replace(regex, v.defò);
        }
    }

    // Prepare demann pou Claude
    const tanKòmanse = Date.now();

    const repons = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'x-api-key': process.env.CLE_API_CLAUDE,
            'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
            model: konfigJeneratè.modèl,
            max_tokens: modèl.maksimum_token || konfigJeneratè.maksimumToken,
            temperature: modèl.tanperati || konfigJeneratè.tanperati,
            system: konfigJeneratè.enstriksyonSistèmJeneral + '\n\n' + enstriksyonSistèm,
            messages: [
                {
                    role: 'user',
                    content: enstriksyon
                }
            ]
        })
    });

    const tanRepons = Date.now() - tanKòmanse;

    if (!repons.ok) {
        const erè = await repons.text();
        throw new Error(`Erè Claude: ${erè}`);
    }

    const done = await repons.json();
    const kontniJenere = done.content[0].text;

    // Ekstrè tit la (premye liy ki gen #)
    let tit = varyab?.sijè || 'Kontni san tit';
    const matchTit = kontniJenere.match(/^#\s*(.+)$/m);
    if (matchTit) {
        tit = matchTit[1].trim();
    }

    // Kreye rezime (premye 200 karaktè)
    const rezime = kontniJenere
        .replace(/^#.+$/gm, '')
        .trim()
        .substring(0, 200) + '...';

    return {
        tit,
        rezime,
        kontni_konplè: kontniJenere,
        paramèt_jenerasyon: {
            modèl_id,
            varyab,
            enstriksyon_itilize: enstriksyon,
            tan_repons_ms: tanRepons
        },
        token_itilize: done.usage.input_tokens + done.usage.output_tokens,
        modèl_claude: done.model
    };
}

/**
 * Adapte kontni pou yon platfòm espesifik
 * @param {string} kontniOrijinal - Kontni orijinal la
 * @param {Object} platfòm - Enfòmasyon sou platfòm lan
 * @returns {Object} Kontni adapte
 */
async function adapteKontniPouPlatfòm(kontniOrijinal, platfòm) {
    const { non, tip, limit_karaktè } = platfòm;

    // Si pa gen limit, retounen kontni orijinal la
    if (!limit_karaktè) {
        return { kontni_adapte: kontniOrijinal };
    }

    // Prepare demann pou adapte kontni an
    const enstriksyon = `Adapte kontni sa a pou platfòm ${non} (${tip}).

LIMIT: ${limit_karaktè} karaktè maksimum
TON: Adapte pou rezo sosyal - kout, angajan, atiran

KONTNI ORIJINAL:
${kontniOrijinal.substring(0, 2000)}

ENSTRIKSYON:
1. Rezime kontni an nan ${limit_karaktè} karaktè oswa mwens
2. Kenbe mesaj prensipal la
3. Ajoute hashtag apwopriye pou ${non}
4. Fè l angajan ak apèl a aksyon`;

    const repons = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'x-api-key': process.env.CLE_API_CLAUDE,
            'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
            model: konfigJeneratè.modèl,
            max_tokens: 500,
            temperature: 0.8,
            messages: [
                {
                    role: 'user',
                    content: enstriksyon
                }
            ]
        })
    });

    if (!repons.ok) {
        throw new Error('Erè nan adaptasyon kontni');
    }

    const done = await repons.json();

    return {
        kontni_adapte: done.content[0].text,
        token_itilize: done.usage.input_tokens + done.usage.output_tokens
    };
}
```

---

## 13.4 Flodtravay n8n pou Otomatizasyon

### 13.4.1 Achitekti Flodtravay yo

```
+------------------------------------------------------------------------+
|                    FLODTRAVAY OTOMATIZASYON KONTNI                       |
+------------------------------------------------------------------------+
|                                                                         |
|  FLODTRAVAY 1: JENERASYON KONTNI                                       |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Demann   |--->| Claude |--->| Estoke |--->| Notifye  |              |
|  | (Webhook)|    | Jenere |    | Kontni |    | Itilizatè|              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
|  FLODTRAVAY 2: PIBLIKASYON PLANIFYE                                    |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Chak Lè  |--->| Jwenn  |--->| Pibliye|--->| Mete Ajo |              |
|  | (Cron)   |    | Kontni |    | Platfòm|    | Estati   |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
|  FLODTRAVAY 3: KOLEKSYON ANALITIK                                      |
|  +----------+    +--------+    +--------+    +----------+              |
|  | Chak Jou |--->| Jwenn  |--->| Kalkile|--->| Estoke   |              |
|  | (Cron)   |    | Done   |    | Metrik |    | Analitik |              |
|  +----------+    +--------+    +--------+    +----------+              |
|                                                                         |
+------------------------------------------------------------------------+
```

### 13.4.2 Flodtravay Jenerasyon Kontni

```json
{
  "name": "Jenerasyon Kontni Otomatik",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "jenere-kontni",
        "responseMode": "responseNode",
        "options": {}
      },
      "id": "webhook-jenere",
      "name": "Webhook Demann Jenerasyon",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "// Valide done antre yo\nconst done = $input.all()[0].json;\n\n// Verifye chan obligatwa\nif (!done.modèl_id) {\n  throw new Error('ID modèl obligatwa');\n}\n\nif (!done.varyab || !done.varyab.sijè) {\n  throw new Error('Sijè kontni obligatwa');\n}\n\n// Prepare done yo\nreturn [{\n  json: {\n    modèl_id: done.modèl_id,\n    kategori_id: done.kategori_id || null,\n    varyab: done.varyab,\n    platfòm_yo: done.platfòm_yo || [],\n    planifye: done.planifye || false,\n    dat_planifye: done.dat_planifye || null,\n    otè_id: done.otè_id || null\n  }\n}];"
      },
      "id": "valide-demann",
      "name": "Valide Demann",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [450, 300]
    },
    {
      "parameters": {
        "operation": "select",
        "tableId": "modèl_kontni",
        "filterType": "string",
        "filterString": "=id=eq.{{ $json.modèl_id }}"
      },
      "id": "jwenn-modèl",
      "name": "Jwenn Modèl Kontni",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [650, 300],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Prepare enstriksyon pou Claude\nconst modèl = $('Jwenn Modèl Kontni').item.json;\nconst demann = $('Valide Demann').item.json;\n\nif (!modèl || !modèl.id) {\n  throw new Error('Modèl pa jwenn');\n}\n\n// Ranplase varyab yo\nlet enstriksyon = modèl.modèl_kontni;\nconst varyab = demann.varyab;\n\nfor (const [kle, valè] of Object.entries(varyab)) {\n  const regex = new RegExp(`{{${kle}}}`, 'g');\n  enstriksyon = enstriksyon.replace(regex, valè);\n}\n\n// Ranplase varyab defò\nconst varyabModèl = modèl.varyab || [];\nfor (const v of varyabModèl) {\n  if (v.defò) {\n    const regex = new RegExp(`{{${v.non}}}`, 'g');\n    enstriksyon = enstriksyon.replace(regex, v.defò);\n  }\n}\n\nconst enstriksyonSistèm = `Ou se yon ekspè nan kreyasyon kontni pwofesyonèl an Kreyòl Ayisyen.\n\nRÈG JENERAL:\n1. Ekri an Kreyòl Ayisyen natirèl\n2. Kontni dwe orijinal ak valè ajoute\n3. Estrikti kontni yo byen ak tit ak sou-tit klè\n\n${modèl.enstriksyon_sistèm}`;\n\nreturn [{\n  json: {\n    enstriksyon,\n    enstriksyon_sistèm: enstriksyonSistèm,\n    tanperati: modèl.tanperati || 0.7,\n    maksimum_token: modèl.maksimum_token || 2000,\n    modèl_id: modèl.id,\n    kategori_id: demann.kategori_id || modèl.kategori_id,\n    varyab: demann.varyab,\n    platfòm_yo: demann.platfòm_yo,\n    planifye: demann.planifye,\n    dat_planifye: demann.dat_planifye,\n    otè_id: demann.otè_id\n  }\n}];"
      },
      "id": "prepare-enstriksyon",
      "name": "Prepare Enstriksyon Claude",
      "type": "n8n-nodes-base.code",
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
        "jsonBody": "={\n  \"model\": \"claude-sonnet-4-5-20250929\",\n  \"max_tokens\": {{ $json.maksimum_token }},\n  \"temperature\": {{ $json.tanperati }},\n  \"system\": {{ JSON.stringify($json.enstriksyon_sistèm) }},\n  \"messages\": [\n    {\n      \"role\": \"user\",\n      \"content\": {{ JSON.stringify($json.enstriksyon) }}\n    }\n  ]\n}",
        "options": {
          "timeout": 60000
        }
      },
      "id": "voye-claude",
      "name": "Jenere Kontni ak Claude",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [1050, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "2",
          "name": "Claude API Kle"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Ekstrè kontni jenere a\nconst reponsClaude = $input.all()[0].json;\nconst doneAnvan = $('Prepare Enstriksyon Claude').item.json;\n\nif (!reponsClaude.content || !reponsClaude.content[0]) {\n  throw new Error('Repons Claude pa valid');\n}\n\nconst kontniJenere = reponsClaude.content[0].text;\n\n// Ekstrè tit la\nlet tit = doneAnvan.varyab.sijè || 'Kontni san tit';\nconst matchTit = kontniJenere.match(/^#\\s*(.+)$/m);\nif (matchTit) {\n  tit = matchTit[1].trim();\n}\n\n// Kreye rezime\nconst rezime = kontniJenere\n  .replace(/^#.+$/gm, '')\n  .trim()\n  .substring(0, 200) + '...';\n\n// Ekstrè mo kle\nconst moCle = [];\nconst matchMoKle = kontniJenere.match(/#\\w+/g);\nif (matchMoKle) {\n  moCle.push(...matchMoKle.slice(0, 10));\n}\n\nreturn [{\n  json: {\n    tit,\n    rezime,\n    kontni_konplè: kontniJenere,\n    mo_kle: moCle,\n    modèl_id: doneAnvan.modèl_id,\n    kategori_id: doneAnvan.kategori_id,\n    otè_id: doneAnvan.otè_id,\n    paramèt_jenerasyon: {\n      varyab: doneAnvan.varyab,\n      enstriksyon: doneAnvan.enstriksyon\n    },\n    token_itilize: reponsClaude.usage.input_tokens + reponsClaude.usage.output_tokens,\n    platfòm_yo: doneAnvan.platfòm_yo,\n    planifye: doneAnvan.planifye,\n    dat_planifye: doneAnvan.dat_planifye\n  }\n}];"
      },
      "id": "ekstrè-kontni",
      "name": "Ekstrè Kontni Jenere",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1250, 300]
    },
    {
      "parameters": {
        "operation": "insert",
        "tableId": "kontni",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            {
              "fieldId": "tit",
              "fieldValue": "={{ $json.tit }}"
            },
            {
              "fieldId": "rezime",
              "fieldValue": "={{ $json.rezime }}"
            },
            {
              "fieldId": "kontni_konplè",
              "fieldValue": "={{ $json.kontni_konplè }}"
            },
            {
              "fieldId": "modèl_id",
              "fieldValue": "={{ $json.modèl_id }}"
            },
            {
              "fieldId": "kategori_id",
              "fieldValue": "={{ $json.kategori_id }}"
            },
            {
              "fieldId": "otè_id",
              "fieldValue": "={{ $json.otè_id }}"
            },
            {
              "fieldId": "mo_kle",
              "fieldValue": "={{ $json.mo_kle }}"
            },
            {
              "fieldId": "paramèt_jenerasyon",
              "fieldValue": "={{ JSON.stringify($json.paramèt_jenerasyon) }}"
            },
            {
              "fieldId": "token_itilize",
              "fieldValue": "={{ $json.token_itilize }}"
            },
            {
              "fieldId": "estati",
              "fieldValue": "bouyon"
            }
          ]
        }
      },
      "id": "estoke-kontni",
      "name": "Estoke Kontni nan Baz Done",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1450, 300],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $('Ekstrè Kontni Jenere').item.json.platfòm_yo.length > 0 }}",
              "value2": true
            }
          ]
        }
      },
      "id": "verifye-platfòm",
      "name": "Gen Platfòm pou Planifye?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [1650, 300]
    },
    {
      "parameters": {
        "jsCode": "// Prepare planifikasyon pou chak platfòm\nconst kontniId = $('Estoke Kontni nan Baz Done').item.json.id;\nconst doneAnvan = $('Ekstrè Kontni Jenere').item.json;\nconst platfòmYo = doneAnvan.platfòm_yo || [];\nconst datPlanifye = doneAnvan.dat_planifye || new Date().toISOString();\n\nconst planifikasyonYo = platfòmYo.map((platfòmId, indèks) => ({\n  kontni_id: kontniId,\n  platfòm_id: platfòmId,\n  dat_planifye: new Date(new Date(datPlanifye).getTime() + (indèks * 3600000)).toISOString(),\n  estati: doneAnvan.planifye ? 'planifye' : 'bouyon'\n}));\n\nreturn planifikasyonYo.map(p => ({ json: p }));"
      },
      "id": "prepare-planifikasyon",
      "name": "Prepare Planifikasyon",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1850, 200]
    },
    {
      "parameters": {
        "operation": "insert",
        "tableId": "kalandriye_piblikasyon",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            {
              "fieldId": "kontni_id",
              "fieldValue": "={{ $json.kontni_id }}"
            },
            {
              "fieldId": "platfòm_id",
              "fieldValue": "={{ $json.platfòm_id }}"
            },
            {
              "fieldId": "dat_planifye",
              "fieldValue": "={{ $json.dat_planifye }}"
            },
            {
              "fieldId": "estati",
              "fieldValue": "={{ $json.estati }}"
            }
          ]
        }
      },
      "id": "kreye-planifikasyon",
      "name": "Kreye Planifikasyon",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [2050, 200],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={\n  \"siksè\": true,\n  \"kontni_id\": \"{{ $('Estoke Kontni nan Baz Done').item.json.id }}\",\n  \"tit\": {{ JSON.stringify($('Ekstrè Kontni Jenere').item.json.tit) }},\n  \"token_itilize\": {{ $('Ekstrè Kontni Jenere').item.json.token_itilize }},\n  \"mesaj\": \"Kontni jenere ak siksè\"\n}"
      },
      "id": "retounen-repons",
      "name": "Retounen Repons",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [2250, 300]
    }
  ],
  "connections": {
    "Webhook Demann Jenerasyon": {
      "main": [[{"node": "Valide Demann"}]]
    },
    "Valide Demann": {
      "main": [[{"node": "Jwenn Modèl Kontni"}]]
    },
    "Jwenn Modèl Kontni": {
      "main": [[{"node": "Prepare Enstriksyon Claude"}]]
    },
    "Prepare Enstriksyon Claude": {
      "main": [[{"node": "Jenere Kontni ak Claude"}]]
    },
    "Jenere Kontni ak Claude": {
      "main": [[{"node": "Ekstrè Kontni Jenere"}]]
    },
    "Ekstrè Kontni Jenere": {
      "main": [[{"node": "Estoke Kontni nan Baz Done"}]]
    },
    "Estoke Kontni nan Baz Done": {
      "main": [[{"node": "Gen Platfòm pou Planifye?"}]]
    },
    "Gen Platfòm pou Planifye?": {
      "main": [
        [{"node": "Prepare Planifikasyon"}],
        [{"node": "Retounen Repons"}]
      ]
    },
    "Prepare Planifikasyon": {
      "main": [[{"node": "Kreye Planifikasyon"}]]
    },
    "Kreye Planifikasyon": {
      "main": [[{"node": "Retounen Repons"}]]
    }
  }
}
```

### 13.4.3 Flodtravay Piblikasyon Otomatik

```json
{
  "name": "Piblikasyon Kontni Otomatik",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "minutes",
              "minutesInterval": 15
            }
          ]
        }
      },
      "id": "cron-piblikasyon",
      "name": "Chak 15 Minit",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "operation": "executeFunction",
        "functionName": "jwenn_kontni_pou_pibliye",
        "functionParameters": "p_limit=10"
      },
      "id": "jwenn-kontni-planifye",
      "name": "Jwenn Kontni Pou Pibliye",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [450, 300],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $json.kalandriye_id }}",
              "operation": "exists"
            }
          ]
        }
      },
      "id": "gen-kontni",
      "name": "Gen Kontni Pou Pibliye?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [650, 300]
    },
    {
      "parameters": {
        "jsCode": "// Mete estati an kou\nconst kontni = $input.all().map(item => item.json);\n\nreturn kontni.map(k => ({\n  json: {\n    ...k,\n    aksyon: 'mete_ajo_estati',\n    nouvo_estati: 'ankou'\n  }\n}));"
      },
      "id": "prepare-piblikasyon",
      "name": "Prepare Piblikasyon",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [850, 200]
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "kalandriye_piblikasyon",
        "filterType": "string",
        "filterString": "=id=eq.{{ $json.kalandriye_id }}",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            {
              "fieldId": "estati",
              "fieldValue": "ankou"
            }
          ]
        }
      },
      "id": "mete-ajo-estati-ankou",
      "name": "Mete Estati Ankou",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1050, 200],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.platfòm_tip }}",
              "value2": "blòg"
            }
          ]
        }
      },
      "id": "chwa-platfòm",
      "name": "Ki Platfòm?",
      "type": "n8n-nodes-base.switch",
      "typeVersion": 3,
      "position": [1250, 200],
      "parameters": {
        "rules": {
          "rules": [
            {
              "outputKey": "blòg",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.platfòm_tip }}",
                    "rightValue": "blòg",
                    "operator": {
                      "type": "string",
                      "operation": "equals"
                    }
                  }
                ]
              }
            },
            {
              "outputKey": "rezo_sosyal",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.platfòm_tip }}",
                    "rightValue": "rezo_sosyal",
                    "operator": {
                      "type": "string",
                      "operation": "equals"
                    }
                  }
                ]
              }
            },
            {
              "outputKey": "imèl",
              "conditions": {
                "conditions": [
                  {
                    "leftValue": "={{ $json.platfòm_tip }}",
                    "rightValue": "imèl",
                    "operator": {
                      "type": "string",
                      "operation": "equals"
                    }
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
        "jsCode": "// Similasyon piblikasyon blòg\nconst kontni = $input.all()[0].json;\n\n// Nan reyalite, ou ta voye demann bay API blòg ou a isit la\n// Pa egzanp: WordPress REST API, Ghost API, elatriye\n\nconst siksè = true; // Simile siksè\n\nif (siksè) {\n  return [{\n    json: {\n      ...kontni,\n      pibliye_siksè: true,\n      url_piblikasyon: `https://blòg-ou.ht/atik/${kontni.kontni_id}`,\n      id_ekstèn: `blòg-${Date.now()}`\n    }\n  }];\n} else {\n  throw new Error('Echwe pibliye sou blòg');\n}"
      },
      "id": "pibliye-blòg",
      "name": "Pibliye sou Blòg",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1450, 100]
    },
    {
      "parameters": {
        "jsCode": "// Similasyon piblikasyon rezo sosyal\nconst kontni = $input.all()[0].json;\n\n// Nan reyalite, ou ta konekte ak API platfòm yo\n// Pa egzanp: X API, Facebook Graph API, elatriye\n\nconst siksè = true;\n\nif (siksè) {\n  return [{\n    json: {\n      ...kontni,\n      pibliye_siksè: true,\n      url_piblikasyon: `https://rezo-sosyal.ht/pòs/${Date.now()}`,\n      id_ekstèn: `sosyal-${Date.now()}`\n    }\n  }];\n} else {\n  throw new Error('Echwe pibliye sou rezo sosyal');\n}"
      },
      "id": "pibliye-rezo-sosyal",
      "name": "Pibliye sou Rezo Sosyal",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1450, 200]
    },
    {
      "parameters": {
        "jsCode": "// Similasyon voye imèl\nconst kontni = $input.all()[0].json;\n\n// Nan reyalite, ou ta itilize yon sèvis imèl\n// Pa egzanp: SendGrid, Mailchimp, elatriye\n\nconst siksè = true;\n\nif (siksè) {\n  return [{\n    json: {\n      ...kontni,\n      pibliye_siksè: true,\n      url_piblikasyon: null,\n      id_ekstèn: `imèl-${Date.now()}`\n    }\n  }];\n} else {\n  throw new Error('Echwe voye imèl');\n}"
      },
      "id": "voye-imèl",
      "name": "Voye Imèl",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [1450, 300]
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "kalandriye_piblikasyon",
        "filterType": "string",
        "filterString": "=id=eq.{{ $json.kalandriye_id }}",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            {
              "fieldId": "estati",
              "fieldValue": "pibliye"
            },
            {
              "fieldId": "url_piblikasyon",
              "fieldValue": "={{ $json.url_piblikasyon }}"
            },
            {
              "fieldId": "id_ekstèn",
              "fieldValue": "={{ $json.id_ekstèn }}"
            },
            {
              "fieldId": "dat_pibliye",
              "fieldValue": "={{ new Date().toISOString() }}"
            }
          ]
        }
      },
      "id": "mete-ajo-siksè",
      "name": "Mete Ajo Estati Siksè",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1650, 200],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "kontni",
        "filterType": "string",
        "filterString": "=id=eq.{{ $json.kontni_id }}",
        "fieldsToSend": "defineBelow",
        "fieldsList": {
          "values": [
            {
              "fieldId": "estati",
              "fieldValue": "pibliye"
            }
          ]
        }
      },
      "id": "mete-ajo-kontni",
      "name": "Mete Ajo Estati Kontni",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1850, 200],
      "credentials": {
        "supabaseApi": {
          "id": "1",
          "name": "Kont Supabase"
        }
      }
    }
  ],
  "connections": {
    "Chak 15 Minit": {
      "main": [[{"node": "Jwenn Kontni Pou Pibliye"}]]
    },
    "Jwenn Kontni Pou Pibliye": {
      "main": [[{"node": "Gen Kontni Pou Pibliye?"}]]
    },
    "Gen Kontni Pou Pibliye?": {
      "main": [
        [{"node": "Prepare Piblikasyon"}],
        []
      ]
    },
    "Prepare Piblikasyon": {
      "main": [[{"node": "Mete Estati Ankou"}]]
    },
    "Mete Estati Ankou": {
      "main": [[{"node": "Ki Platfòm?"}]]
    },
    "Ki Platfòm?": {
      "main": [
        [{"node": "Pibliye sou Blòg"}],
        [{"node": "Pibliye sou Rezo Sosyal"}],
        [{"node": "Voye Imèl"}]
      ]
    },
    "Pibliye sou Blòg": {
      "main": [[{"node": "Mete Ajo Estati Siksè"}]]
    },
    "Pibliye sou Rezo Sosyal": {
      "main": [[{"node": "Mete Ajo Estati Siksè"}]]
    },
    "Voye Imèl": {
      "main": [[{"node": "Mete Ajo Estati Siksè"}]]
    },
    "Mete Ajo Estati Siksè": {
      "main": [[{"node": "Mete Ajo Estati Kontni"}]]
    }
  }
}
```

---

## 13.5 Tès ak Enstriksyon Itilizasyon

### 13.5.1 Fichye Tès Konplè

```javascript
// =====================================================
// FICHYE TÈS: test-otomatizasyon-kontni.js
// =====================================================

const konfigTès = {
    urlJenerasyon: 'http://localhost:5678/webhook/jenere-kontni',
    atant: 3000
};

/**
 * Tès 1: Jenere atik blòg
 */
async function tèsJenereAtik() {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 1: Jenere Atik Blòg');
    console.log('='.repeat(60));

    const demann = {
        modèl_id: 'ID_MODÈL_ATIK_BLÒG', // Ranplase ak vrè ID
        varyab: {
            sijè: 'Kijan pou aprann pwogramasyon nan 2024',
            longè: '1000',
            ton: 'edikasyonèl',
            odyans: 'debitant'
        },
        kategori_id: 'ID_KATEGORI_TEKNOLOJI', // Ranplase ak vrè ID
        platfòm_yo: [],
        planifye: false
    };

    try {
        const repons = await fetch(konfigTès.urlJenerasyon, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(demann)
        });

        const done = await repons.json();

        if (done.siksè) {
            console.log('SIKSÈ!');
            console.log(`- Kontni ID: ${done.kontni_id}`);
            console.log(`- Tit: ${done.tit}`);
            console.log(`- Token itilize: ${done.token_itilize}`);
            return true;
        } else {
            console.log('ECHWE:', done);
            return false;
        }
    } catch (erè) {
        console.error('ERÈE:', erè.message);
        return false;
    }
}

/**
 * Tès 2: Jenere pòs rezo sosyal
 */
async function tèsJenerePòsSosyal() {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 2: Jenere Pòs Rezo Sosyal');
    console.log('='.repeat(60));

    const demann = {
        modèl_id: 'ID_MODÈL_PÒS_SOSYAL', // Ranplase ak vrè ID
        varyab: {
            sijè: 'Nouvo kous pwogramasyon disponib!',
            platfòm: 'X',
            limit_karaktè: '280',
            ton: 'angajan',
            apèl_aksyon: 'Enskri jodi a!'
        },
        platfòm_yo: ['ID_PLATFÒM_X'], // Ranplase ak vrè ID
        planifye: true,
        dat_planifye: new Date(Date.now() + 3600000).toISOString() // 1 èdtan pita
    };

    try {
        const repons = await fetch(konfigTès.urlJenerasyon, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(demann)
        });

        const done = await repons.json();

        if (done.siksè) {
            console.log('SIKSÈ!');
            console.log(`- Kontni ID: ${done.kontni_id}`);
            console.log(`- Tit: ${done.tit}`);
            return true;
        }
        return false;
    } catch (erè) {
        console.error('ERÈE:', erè.message);
        return false;
    }
}

/**
 * Tès 3: Jenere bilten imèl
 */
async function tèsJenereBilten() {
    console.log('\n' + '='.repeat(60));
    console.log('TÈS 3: Jenere Bilten Imèl');
    console.log('='.repeat(60));

    const demann = {
        modèl_id: 'ID_MODÈL_BILTEN', // Ranplase ak vrè ID
        varyab: {
            sijè: 'Nouvèl teknoloji semèn sa a',
            kantite_pwen: '5',
            non_antrepriz: 'TechHaiti',
            ton: 'enfòmatif'
        }
    };

    try {
        const repons = await fetch(konfigTès.urlJenerasyon, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(demann)
        });

        const done = await repons.json();

        if (done.siksè) {
            console.log('SIKSÈ!');
            console.log(`- Kontni ID: ${done.kontni_id}`);
            console.log(`- Tit: ${done.tit}`);
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
    console.log('  KÒMANSE TÈS SISTÈM OTOMATIZASYON KONTNI');
    console.log('#'.repeat(70));

    const rezilta = { total: 3, pase: 0, echwe: 0 };

    if (await tèsJenereAtik()) rezilta.pase++; else rezilta.echwe++;
    await new Promise(r => setTimeout(r, konfigTès.atant));

    if (await tèsJenerePòsSosyal()) rezilta.pase++; else rezilta.echwe++;
    await new Promise(r => setTimeout(r, konfigTès.atant));

    if (await tèsJenereBilten()) rezilta.pase++; else rezilta.echwe++;

    console.log('\n' + '='.repeat(70));
    console.log('REZIME TÈS');
    console.log('='.repeat(70));
    console.log(`Total: ${rezilta.total} | Pase: ${rezilta.pase} | Echwe: ${rezilta.echwe}`);
    console.log(`Pousantaj: ${((rezilta.pase / rezilta.total) * 100).toFixed(1)}%`);
}

egzekiteTès();
```

### 13.5.2 Verifikasyon Baz Done

```sql
-- Verifye modèl kontni yo
SELECT id, non, tip, aktif FROM modèl_kontni ORDER BY dat_kreye DESC;

-- Verifye kontni ki jenere yo
SELECT
    k.id,
    k.tit,
    k.estati,
    k.token_itilize,
    m.non as modèl,
    kat.non as kategori,
    k.dat_kreye
FROM kontni k
LEFT JOIN modèl_kontni m ON k.modèl_id = m.id
LEFT JOIN kategori_kontni kat ON k.kategori_id = kat.id
ORDER BY k.dat_kreye DESC
LIMIT 10;

-- Verifye kalandriye piblikasyon
SELECT
    kp.id,
    k.tit,
    pp.non as platfòm,
    kp.dat_planifye,
    kp.estati,
    kp.dat_pibliye
FROM kalandriye_piblikasyon kp
JOIN kontni k ON kp.kontni_id = k.id
JOIN platfòm_piblikasyon pp ON kp.platfòm_id = pp.id
ORDER BY kp.dat_planifye DESC
LIMIT 10;

-- Estatistik jeneral
SELECT * FROM jwenn_rapò_kontni();
```

---

## 13.6 Rezime ak Pwochen Etap

### 13.6.1 Sa Nou Te Konstwi

Nan chapit sa a, nou te konstwi yon sistèm otomatizasyon kontni konplè ki:

1. **Jenere kontni** - Itilize Claude pou kreye diferan tip kontni
2. **Jere modèl** - Kenbe modèl pou diferan tip kontni
3. **Planifye piblikasyon** - Orè kontni pou diferan platfòm
4. **Pibliye otomatikman** - Distribiye kontni selon kalandriye
5. **Swiv pèfòmans** - Kolekte ak analize metrik

### 13.6.2 Estrikti Pwojè

```
otomatizasyon-kontni/
├── baz-done/
│   ├── 01-kategori.sql
│   ├── 02-platfòm.sql
│   ├── 03-modèl.sql
│   ├── 04-kontni.sql
│   ├── 05-kalandriye.sql
│   ├── 06-analitik.sql
│   └── 07-fonksyon.sql
├── n8n/
│   ├── jenerasyon-kontni.json
│   ├── piblikasyon-otomatik.json
│   └── koleksyon-analitik.json
├── kòd/
│   ├── konfigirasyon.js
│   └── jeneratè-kontni.js
└── tès/
    ├── test-otomatizasyon.js
    └── verifye-baz-done.sql
```

### 13.6.3 Pwochen Etap

Nan **Chapit 14**, nou pral konstwi yon **Aplikasyon Jesyon Done** ki pral:
- Jere done kliyan ak biznis
- Analize done ak entèlijans atifisyèl
- Jenere rapò otomatik
- Kreye tablo bò done pou vizualizasyon

---

## Kesyon pou Pratik

1. Kijan ou ta ajoute sipò pou imaj nan kontni yo?
2. Ki jan ou ta enplimante yon sistèm apwobasyon anvan piblikasyon?
3. Kijan ou ta fè pou optimizasyon pou rechèch (SEO)?
4. Ki metrik ou ta ajoute pou mezire siksè kontni yo?
5. Kijan ou ta jere vèsyon diferan kontni yo?
