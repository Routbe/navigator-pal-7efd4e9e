# Eigen e-mails eerst, Brevo-templates als vangnet

## Wat verandert er voor jou
- Elke mail (inloggen, facturen, contact, betalingen) wordt door ROUT zelf opgemaakt in jouw donkere stijl met het konijnenlogo, in NL, EN, FR of DE.
- De inlogmail bevat een grote knop **en** een 6-cijferige code in grote letters. Op het inlogscherm verschijnen na "Check je inbox" zes vakjes om de code in te tikken, naast de knop "Open inloglink".
- Lukt het opmaken niet, dan stuurt ROUT automatisch het juiste Brevo-template. Bestaat er geen template in jouw taal (bv. Portugees), dan het Engelse.
- Brevo telt geen opens of klikken meer (geen trackingpixels).

## Template-nummers (vangnet)
```text
Inloggen            nl 93  en 13  fr 14  de 15
Admin subdomein     2
Contact             nl 22  en 23  fr 24  de 25
Betaalbevestiging   nl 32  en 33  fr 34  de 35
Betaling mislukt    nl 42  en 43  fr 44  de 45
Verlenging          nl 52 en 53 fr 54 de 55 es 56 it 57 pt 58 pl 59 zh 60, reserve 61
Factuur / geslaagd  nl 62  en 63  fr 64  de 65
Geannuleerd         nl 72  en 73  fr 74  de 75
Terugbetaling       nl 82  en 83  fr 84  de 85
Ontbrekende taal -> en, daarna nl
```

## Keuzes die ik maak bij tegenstrijdigheden in je briefing
- Factuur = 62–65 (zoals in de hoofdtabel); 82–85 blijft terugbetaling/creditnota.
- "generic_fallback 9 / 3 / 4 / 5" laat ik weg: die nummers vallen binnen andere blokken en zijn niet geverifieerd. Zeg het als ze toch echt bestaan.
- Inloggen blijft via ROUT's eigen inlogsysteem (geen extern systeem); de bestaande 6-cijferige code wordt nu juist zichtbaar gemaakt in plaats van verwijderd.
- Spaans/Italiaans/Pools/Chinees krijgen geen eigen tekst; die vallen terug op Engels.

## Technisch
- `bun add @react-email/components @react-email/render`.
- `src/emails/react/TemplateWrapper.tsx` (obsidian, logo uit `src/lib/brand/logo.ts`, inline styles, `Body` bg `#0b0b0c` met `color-scheme` meta zodat Gmail/Outlook niet inverteren) + `MagicLinkEmail`, `BillingEmail` (factuur/annulering/terugbetaling/verlenging/mislukt/bevestiging), `ContactConfirmationEmail`.
- `src/locales/emails/{nl,en,fr,de}.json` met `{{magic_link}}`, `{{otp_code}}`, `{{user_handle}}`; kleine `interpolate()` + `emailT(locale, key)` met fallback en → nl.
- `src/emails/template-ids.ts`: tabel herschrijven naar bovenstaande matrix; resolutie taal → en → nl → categorie-reserve; `subdomain_claim: 2` toevoegen. `src/lib/brevo/templates.ts` laten afleiden van dezelfde tabel (één bron).
- `send.server.ts` `sendMail`: volgorde omdraaien — eerst `render()` + `htmlContent`; bij render-fout of Brevo 4xx op HTML → `templateId` met `params`; payload altijd `headers: { "X-Mailin-Tag" }` en tracking uit (`trackClicks:false, trackOpens:false` / `X-Mailin-track*: 0`).
- `email.server.ts` `sendLocalizedEmail`: rendert React-template, blijft non-blocking en nooit gooien.
- Magic-link + code: bij versturen wordt één `email_code` (bestaande `auth_tokens`-purpose, db/05) samen met de link aangemaakt; inlogscherm toont `OtpInput` en valideert via bestaande code-verificatie.
- Tests: bijwerken `template-ids.test.ts` en `email-service.test.ts` (pt-factuur → 63, nl-terugbetaling → 82, verlenging pt → 58, onbekende taal → en, render-fout → templateId-pad, tracking uit in payload).
- `AGENTS.md`-regel over e-mailvolgorde vervangen (HTML eerst, template als vangnet).
