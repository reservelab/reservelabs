# Is Analizi: ReserveLabs
Tarih: 2026-03-23
Analist: Claude (Is Analisti Skill)
Onaylayan: ReserveLab

---

## 1. Kapsam

| Soru | Cevap |
|------|-------|
| **Ne** | AI-generated codebase'lerde design drift tespit eden skill suite |
| **Kime** | Solo vibecoder developer'lar (Claude Code, Codex, Cursor kullananlar) |
| **Neden** | 50+ AI prompt sonra UI tutarsızlaşıyor, hiçbir mevcut araç bunu yakalamıyor |
| **Nerede** | Claude Code skill + OpenAI Codex uyumu (developer tool) |
| **Kaç kişi** | N/A — lokal geliştirici aracı, sunucu yok |
| **Veri** | Sadece kaynak kod dosyalarını okur. Hassas veri yok |
| **Entegrasyon** | Claude Code SDK, Codex agents, tailwind.config/tema dosyaları |
| **Süre** | Phase 1 (skill): 1-2 gün / Phase 2 (CLI): belirsiz |
| **Bütçe** | Sıfır — açık kaynak, API yok, hosting yok |
| **Başarı** | GitHub 100+ star, "design drift" teriminin yayılması |
| **GitHub** | reservelab/reservelabs |
| **Tanıtım** | X (Twitter) |
| **Lisans** | MIT |

---

## 2. Gereksinimler

### Fonksiyonel

| # | İşlem | Girdi | İşlem | Çıktı |
|---|-------|-------|-------|-------|
| F1 | Planning checkpoint | Developer "X yapacağım" der | Checklist'ten gap analysis | Eksik tasarım kararları listesi |
| F2 | Implementation review | Developer "kontrol et" der + dosya yolları | Kaynak kodu okur, kurallara karşı tarar | Uyarı listesi + düzeltme önerileri |
| F3 | Pre-release review | Developer "bitti" veya "review" der | Tüm frontend dosyalarını tarar | Drift raporu + skor |
| F4 | Config okuma | .reservelabs.yml varsa okur | Override'ları default'lara uygular | Proje-spesifik kural seti |
| F5 | Context detection | tailwind.config / globals.css varsa okur | Tema token'larını çıkarır | Dinamik kural baseline'ı |
| F6 | Auto-fix | Uyarı + developer "Y" onayı | Dosyada değişiklik yapar | Düzeltilmiş kod |

### Fonksiyonel Olmayan

| # | Kategori | Gereksinim |
|---|----------|------------|
| NF1 | Performans | 50 dosyalık projede Stage 3 full review < 2 dakika |
| NF2 | Güvenilirlik | Aynı codebase'de benzer sonuçlar (kalitatif, yüzde değil) |
| NF3 | Taşınabilirlik | Claude Code + Codex, iki platformda çalışır |
| NF4 | Bakılabilirlik | Checklist'ler ayrı dosyalarda, kolayca güncellenebilir |
| NF5 | Kullanılabilirlik | Sıfır config ile hemen çalışır, kurulum 2 dakika |

### Edge Case'ler

| # | Ya ... olursa? | Çözüm |
|---|----------------|-------|
| E1 | Proje çok büyükse (200+ dosya) | Chunk stratejisi — src/components ve src/app tara, geri kalanı atla |
| E2 | Tailwind config yoksa | Layer 1 static default'lara düş |
| E3 | Proje Tailwind kullanmıyorsa | Genel CSS kurallarıyla devam, Tailwind-specific atla |
| E4 | Developer her uyarıyı reddederse | Logla, bloklamadan devam |
| E5 | Monorepo / workspace | En yakın config'i kullan (yukarı arama) |
| E6 | Context'e sığmazsa | Chunk: dosya listesi → grup analiz → birleştir |
| E7 | False positive (geçerli karar drift olarak işaretlenir) | 3 güven seviyesi (HIGH/MEDIUM/INFO) + "intentional?" sorusu |
| E8 | JavaScript (TypeScript değil) | Hem .tsx/.jsx hem .ts/.js tara |
| E9 | shadcn component'leri customize edilmişse | shadcn = base, kasıtlı customization drift sayılmaz |

---

## 3. Riskler

| # | Risk | Olasılık | Etki | Skor | Önlem |
|---|------|:---:|:---:|:---:|-------|
| 🔴 R1 | False positive — geçerli karar drift olarak işaretlenir | 4 | 5 | **20** | 3 güven seviyesi (HIGH/MEDIUM/INFO), "intentional?" sorusu |
| 🔴 R2 | Context window limiti — 100+ dosyada full scan sığmaz | 4 | 4 | **16** | Chunk stratejisi: dosya listesi → grup grup analiz → birleştir |
| 🟡 R3 | LLM tutarsızlığı — aynı codebase farklı sonuçlar | 4 | 3 | **12** | Kalitatif derecelendirme, deterministik checklist'ler |
| 🟡 R4 | "Design drift" terimi aranmıyor | 3 | 3 | **9** | README'de SEO: "UI consistency", "design lint", "vibecoding quality" |
| 🟡 R5 | Codex uyumu belirsiz | 3 | 3 | **9** | Claude Code öncelikli, Codex best-effort |
| 🟡 R6 | Tailwind v4 config değişikliği | 3 | 3 | **9** | Hem v3 (tailwind.config.ts) hem v4 (CSS-based) desteği |
| 🟢 R7 | Platform lock-in | 2 | 4 | **8** | Mantık checklist'lerde, platform-specific sadece skill.md |
| 🟢 R8 | Düşük adoption | 2 | 3 | **6** | Güçlü README + X duyurusu + dogfooding |
| 🟢 R9 | Scope creep | 3 | 2 | **6** | Sıkı v1/v2 sınırı |

---

## 4. Mimari

### Topoloji

```
[Developer] → "kontrol et" → [skill.md (Stage Router)]
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                  ▼
            [Stage 1]          [Stage 2]          [Stage 3]
            planning.md      implementation.md    prerelease.md
                    │                 │                  │
                    ▼                 ▼                  ▼
              Gap Analysis      Live Check        design-reviewer.md
                                                  (Full Scan Agent)
                    │                 │                  │
                    └────────┬────────┘                  │
                             ▼                           ▼
                    [Context Engine]              [Chunk Manager]
                    tailwind.config?              Dosya listesi →
                    globals.css?                  Grup grup analiz →
                    .reservelabs.yml?             Sonuç birleştir
                             │                           │
                             ▼                           ▼
                    [Kural Seti Oluştur]          [Drift Raporu]
                    Static + Dynamic              Skor + Öneriler
```

### Veri Akışı

1. Developer skill'i çağırır
2. skill.md stage belirler (keyword: yapacağım→S1, kontrol et→S2, bitti→S3)
3. İlgili checklist yüklenir
4. Context engine: .reservelabs.yml → tailwind.config → static default (sırasıyla)
5. Kaynak dosyalar taranır
6. Kurallarla karşılaştırılır
7. Çıktı: uyarılar (HIGH/MEDIUM/INFO) + düzeltme önerileri

### Dosya Yapısı

```
reservelabs/
├── skill.md                      # Stage router + context engine
├── agents/
│   └── design-reviewer.md        # Otonom full-scan agent
├── checklists/
│   ├── planning.md               # Stage 1
│   ├── implementation.md         # Stage 2
│   └── prerelease.md             # Stage 3
├── references/
│   ├── visual-rules.md           # Spacing + color + typography
│   ├── ux-patterns.md            # A11y + state coverage
│   └── responsive.md             # Breakpoints + responsive
├── supported-stacks/
│   └── react-nextjs-tailwind.md  # Stack-specific kurallar
├── .reservelabs.example.yml      # Config örneği
├── README.md                     # 🇹🇷🇬🇧
├── INSTALL.md
├── LICENSE
└── CHANGELOG.md
```

### Kritik Tasarım Kararları

| Karar | Seçim | Gerekçe |
|-------|-------|---------|
| Stage tespiti | Keyword-based (manuel) | Claude Code'da auto-trigger yok |
| Skor tipi | Kalitatif (iyi/orta/kötü) | LLM yüzde üretemez |
| Uyarı güveni | 3 seviye (HIGH/MEDIUM/INFO) | False positive'i azaltır |
| Büyük proje | Chunk stratejisi | Context window patlamaz |
| Config yoksa | Static default | Sıfır config vaadini tutar |

### Altyapı Etkisi

**Sıfır.** Spark'ta servis çalıştırmaz, port kullanmaz, sadece GitHub repo.

---

## 5. Uygulama Planı

### MVP

`skill.md` + `implementation.md` + `visual-rules.md` + README = Minimum değerli ürün.

### Fazlama

| Faz | Ne Yapılır | Bağımlılık | Öncelik | Kabul Kriteri |
|-----|-----------|------------|---------|---------------|
| F1 | Repo iskelet + README + LICENSE + INSTALL | — | P0 | Repo açılır, README net |
| F2 | skill.md — stage router + context engine | F1 | P0 | Stage tespiti + config okuma çalışır |
| F3 | checklists/ — 3 checklist | F2 | P0 | Her stage checklist'ini yükler |
| F4 | references/ — 3 referans | F2 | P0 | Kurallar yazılır |
| F5 | supported-stacks/ — React/Next/Tailwind | F4 | P1 | Stack-specific kontroller çalışır |
| F6 | agents/design-reviewer.md | F3+F4 | P1 | Full scan + drift raporu üretir |
| F7 | .reservelabs.example.yml + docs | F2 | P2 | Config çalışır |
| F8 | Dogfooding — gerçek projede test | F6 | P2 | 1 projede test, feedback |
| F9 | X duyurusu + GitHub public | F8 | P2 | Tweet + repo public |

### Test Senaryoları

| # | Senaryo | Nasıl Test Edilir |
|---|---------|-------------------|
| T1 | Stage 1 gap analysis | "Dashboard yapacağım" → eksikleri listeliyor mu? |
| T2 | Stage 2 spacing | gap-3 component → uyarı veriyor mu? |
| T3 | Stage 2 a11y | alt text'siz Image → yakalıyor mu? |
| T4 | Stage 2 state coverage | Error state'siz component → uyarıyor mu? |
| T5 | Stage 3 full review | 10+ component projede rapor mantıklı mı? |
| T6 | Config okuma | .reservelabs.yml override çalışıyor mu? |
| T7 | Tailwind context | tailwind.config renkleri okunuyor mu? |
| T8 | Sıfır config | Default'lara düşüyor mu? |
| T9 | False positive | Kasıtlı farklılık → "intentional?" soruyor mu? |

---

## 6. Karar

- [x] **ONAYLANDI** — Kodlamaya geçilsin
- [ ] REVİZYON GEREKLİ
- [ ] REDDEDİLDİ

**Not:** Spec review'dan gelen 6 bulgu (auto-trigger düzeltmesi, kalitatif skor, false positive riski, execution model açıklaması, roadmap checkmark'ları, drift score formülü) spec dokümanına yansıtılacak.
