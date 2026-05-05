# ERP System — Master Prompt v2

---

## 🧠 MASTER PROMPT
*(Diesen Prompt am Anfang jeder neuen KI-Session einfügen)*

```
Du hilfst mir beim Aufbau eines modularen, mandantenfähigen ERP-Systems als Web-App (SaaS).

ENTWICKLUNGSUMGEBUNG:
- Lokal auf MacBook (SQLite, npm run dev, kein Server nötig bis Phase 9)
- GitHub Repository: Versionskontrolle, CI/CD via GitHub Actions
- Branch-Strategie: main (production), develop (aktive Entwicklung)
- Deployment später: Vercel oder Coolify auf Hetzner

TECH-STACK:
- Next.js 14 (App Router), TypeScript strict, Prisma (SQLite → PostgreSQL)
- Tailwind CSS (Mobile-First: erst sm:, dann md:, dann lg:)
- NextAuth.js v5, Stripe Billing, Cloudflare R2, next-intl (i18n)
- PWA-fähig, Dark Mode, WCAG 2.1 AA

ARCHITEKTUR — DREI PFLICHT-CHECKS pro API-Route:
1. await requireTenant(req)           → Mandantentrennung (tenantId auf jeder Tabelle)
2. await requireModule(req, 'ID')     → Feature-Gate (TenantModule-Tabelle)
3. await requirePermission(req, '..') → Rollenberechtigung

DATENMODELL-GRUNDREGELN:
- Jede Tabelle: id, tenantId, createdAt, updatedAt
- Kein Hard-Delete auf Buchhaltungsdaten (isDeleted Flag)
- Gebuchte Belege: kein UPDATE → nur Storno-Beleg erstellen (GoBD)
- Belegnummer: aus NumberRange-Tabelle (tenantId + prefix + sequence)

MODULE (41 gesamt):
Core:       CORE, DS, STAM, LAGER, EINK, VERK, DMS, REP
             + POS (Add-on, in Phase 2 gebaut)
             + LABEL (Add-on, in Phase 2 gebaut)
Market:     SHOP (inkl. Baukasten-Editor), TERMIN, INTG, KONNEKT, CRM, MKT, HELP
             (TERMIN und KONNEKT: Add-ons, in Phase 3 gebaut)
Company:    FIBU, ZAHL, STEU, HR (inkl. Reisekosten), PROJ, ASSET, NOTIF, COMM
Operations: PROD (inkl. Kalkulation), RDEV, DISPO, QM, SCM
             (RDEV: Add-on, in Phase 5 gebaut)
Enterprise: KORE, ANLA, BI, COMP
Add-ons:    GAST, MEMBER, VERT, SOCI, IMPORT, FRAN (Phase 7)
Plattform:  PLAT — separates Admin-Interface, nicht im ERP sichtbar (Phase 10)

WICHTIG: Jedes Modul aus einem Abo-Plan kann auch einzeln gebucht werden.

PHASENPLAN (10 Phasen — eine Phase pro Abo-Stufe):
P1:  Technisches Fundament + GitHub    (2–3 Wo.)  — Setup, Auth, i18n, DS, Mobile-First
P2:  Core-Abo                          (8–10 Wo.) — CORE STAM LAGER EINK VERK POS LABEL DMS REP → v1.0
P3:  Market-Abo                        (8–10 Wo.) — SHOP TERMIN INTG KONNEKT CRM MKT HELP → v2.0
P4:  Company-Abo                      (10–12 Wo.) — FIBU ZAHL STEU HR PROJ ASSET NOTIF COMM → v3.0
P5:  Operations-Abo                   (10–12 Wo.) — PROD RDEV DISPO QM SCM → v4.0
P6:  Enterprise-Abo                   (10–12 Wo.) — KORE ANLA BI COMP → v5.0
P7:  Add-ons                          (10–14 Wo.) — GAST MEMBER VERT SOCI IMPORT FRAN
P8:  Cybersecurity-Check              ( 1–2 Wo.)  — OWASP Top 10, DSGVO, Security-Hardening
P9:  Deployment & Launch              ( 1–2 Wo.)  — PostgreSQL, CI/CD, Monitoring, PWA, Live
P10: SaaS-Plattform (PLAT)            ( 4–6 Wo.)  — Vermietsystem, Stripe Billing, Platform-Admin
     ⚠️ P10 erst nach 3 Monaten stabilem Betrieb!

WICHTIGE TECHNISCHE REGELN:
- GoBD: keine Updates auf Belegen → nur Storno, Belege 10 Jahre archivieren
- DSGVO: Tenant-Isolation, Daten-Export Art. 20, DSGVO-Offboarding 30 Tage
- Mobile: Touch-Targets min. 44×44px, Bottom Tab Bar auf Mobile
- i18n: t()-Wrapper in jeden UI-String (DE/EN/ES von Anfang an)
- Buchhaltung: SKR03/SKR04, doppelte Buchführung, DATEV-Export EXTF
- PDF: react-pdf, XRechnung XML, ZUGFeRD PDF/A-3
- Shop: Baukasten-Editor (Hero, Produktraster, Über uns, etc. per Drag & Drop)
- Security: bcrypt ≥12, JWT 15min, 2FA Pflicht für Admins, OWASP Top 10

Ich nenne dir jetzt das Modul oder die Aufgabe an der ich gerade arbeite.
Gib mir: (1) kurze Zusammenfassung was ich beachten muss, (2) spezifische To-Do-Liste.
```

---

## 📋 MODUL-REFERENZ KOMPAKT

| ID | Name | Abo | Abhängig von | Phase |
|---|---|---|---|---|
| CORE | Core System | Core | — | P2 |
| STAM | Stammdaten | Core | CORE | P2 |
| LAGER | Lager & Logistik | Core | STAM | P2 |
| EINK | Einkauf | Core | STAM, LAGER | P2 |
| VERK | Verkauf / Aufträge | Core | STAM | P2 |
| POS | Kassensystem | Add-on | VERK, LAGER | P2 |
| LABEL | Barcode & Labels | Add-on | STAM, LAGER | P2 |
| DMS | Dokumente & E-Rechnung | Core | CORE | P2 |
| REP | Reporting | Core | CORE | P2 |
| SHOP | Online-Shop + Baukasten | Market | STAM, LAGER, VERK | P3 |
| TERMIN | Terminbuchung | Add-on | STAM, VERK | P3 |
| INTG | API & Integrationen | Market | STAM | P3 |
| KONNEKT | Shop-Konnektoren | Add-on | INTG, STAM, LAGER | P3 |
| CRM | CRM & Verkaufschancen | Market | STAM, VERK | P3 |
| MKT | Marketing & Kampagnen | Market | CRM, STAM | P3 |
| HELP | Helpdesk & Support | Market | CRM, STAM | P3 |
| FIBU | Finanzbuchhaltung | Company | VERK, POS | P4 |
| ZAHL | Zahlungsverkehr | Company | FIBU | P4 |
| STEU | Steuer & Abschluss | Company | FIBU | P4 |
| HR | Personal (inkl. Reisekosten) | Company | CORE | P4 |
| PROJ | Projekte & Verträge | Company | VERK | P4 |
| ASSET | Asset Management | Company | CORE, FIBU | P4 |
| NOTIF | Workflows & Alerts | Company | CORE | P4 |
| COMM | Interne Kommunikation | Company | CORE, HR | P4 |
| PROD | Produktion + Kalkulation | Add-on | STAM, LAGER, EINK | P5 |
| RDEV | Produktentwicklung R&D | Add-on | STAM, PROD | P5 |
| DISPO | Absatzplanung | Operations | LAGER, VERK | P5 |
| QM | Qualitätsmanagement | Operations | PROD, EINK | P5 |
| SCM | Supply Chain | Operations | EINK, LAGER, PROD | P5 |
| KORE | Kostenrechnung | Enterprise | FIBU, PROD | P6 |
| ANLA | Anlagenbuchhaltung | Enterprise | FIBU | P6 |
| BI | Business Intelligence | Enterprise | REP | P6 |
| COMP | Compliance & Risiko | Enterprise | CORE, FIBU, HR | P6 |
| GAST | Gastro / Café / Bar | Add-on | POS, STAM | P7 |
| MEMBER | Abo & Mitgliedschaft | Add-on | STAM, VERK, ZAHL | P7 |
| VERT | Vertreter & Außendienst | Add-on | VERK, CRM | P7 |
| SOCI | Social Media | Add-on | MKT | P7 |
| IMPORT | Außenhandel | Add-on | EINK, STAM | P7 |
| FRAN | Franchise | Add-on | CORE, PLAT | P7 |
| PLAT | Plattform-Admin (SaaS) | Plattform | CORE | P10 |

---

## 🔢 ROADMAP ÜBERSICHT

```
P1  Fundament + GitHub      2–3 Wo.
P2  Core-Abo                8–10 Wo.   → v1.0 LAUNCH
P3  Market-Abo              8–10 Wo.   → v2.0
P4  Company-Abo            10–12 Wo.   → v3.0
P5  Operations-Abo         10–12 Wo.   → v4.0
P6  Enterprise-Abo         10–12 Wo.   → v5.0
P7  Add-ons                10–14 Wo.
P8  Cybersecurity           1–2 Wo.
P9  Deployment & Launch     1–2 Wo.    → LIVE
P10 SaaS-Plattform          4–6 Wo.    → nach 3 Monaten Betrieb
──────────────────────────────────────
    Gesamt                 ~15–20 Mo.
```

---

## ✅ VERWENDUNG

**Schritt 1** — Master Prompt in neue KI-Session einfügen

**Schritt 2** — Aufgabe nennen:
```
Ich arbeite gerade an: P2 — LAGER, Funktion: Inventur durchführen
```
```
Ich arbeite gerade an: P1 — GitHub Actions CI/CD einrichten
```
```
Ich arbeite gerade an: P3 — SHOP, Shop-Baukasten-Editor
```

**Schritt 3** — KI gibt:
1. Kurze Zusammenfassung (was beachten)
2. Spezifische To-Do-Liste für genau diese Funktion

---

## 📌 IMMER SICHTBAR LASSEN

```
PFLICHT-CHECKS jede API-Route:
  requireTenant() → requireModule() → requirePermission()

DATENBANK:
  tenantId auf jeder Tabelle
  kein Hard-Delete auf Buchhaltung
  Belege: kein UPDATE nach Finalisierung

MOBILE:
  min-h-[44px] min-w-[44px] auf Buttons
  sm: → md: → lg: Reihenfolge

GOBD:
  Belegnummer aus NumberRange
  Storno statt Löschen
  10 Jahre Archivierung

SECURITY:
  bcrypt ≥12, JWT 15min, 2FA für Admins
  Zod-Validation auf allen Inputs
  OWASP Top 10 vor Launch prüfen
```

---

*v2 — aktualisiert nach ERP_System_Planning_v14.html*
