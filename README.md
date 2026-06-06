# Settet — treningsplan-demo

En liten, deployerbar demo: bruker svarer på 14 spørsmål → får en individuell treningsplan som PDF.
Planen genereres av en streng systemprompt (fysioterapeut + PT-dømmekraft), med en sikkerhetsgaffel
som ruter røde flagg til en samtale i stedet for å lage plan.

```
index.html            Hele frontend (skjema, API-kall, PDF). Statisk — kan ligge på GitHub Pages.
api/plan.js           Serverless-funksjon. Holder API-nøkkel + systemprompt server-side. Kaller modell-API.
prompt/systemprompt.md  Lesbar kopi av systemprompten (samme tekst som er bakt inn i api/plan.js).
```

## Kjør lokalt — føl flyten med en gang (ingen nøkkel nødvendig)

Åpne `index.html` rett i nettleseren, eller server mappa:

```bash
npx serve .
```

Uten backend faller appen tilbake til **demo-modus** og viser en ferdig eksempelplan, slik at du kan
kjenne på skjema, layout og PDF-eksport umiddelbart. En liten merkelapp viser at det er demo-modus.

## Få ekte generering (med din egen API-nøkkel)

Frontend kaller `POST /api/plan`. Den ruta krever en backend. Enklest: **Vercel** (gratis-tier holder).

```bash
npm i -g vercel
vercel            # følg promptene, koble til repoet
vercel env add ANTHROPIC_API_KEY      # lim inn nøkkelen din
vercel --prod
```

Eller test lokalt med ekte kall:

```bash
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
vercel dev        # serverer index.html + /api/plan på localhost
```

Når `/api/plan` svarer, slutter appen å bruke demo-planen og viser ekte, generert innhold.

## Hvorfor arkitekturen ser slik ut

- **Nøkkelen ligger aldri i nettleseren.** Den bor som miljøvariabel i serverless-funksjonen.
  Et klient-side kall ville eksponert nøkkelen for hvem som helst som åpner Nettverk-fanen.
- **Systemprompten ligger server-side.** Den er IP-en i dette produktet. Hadde den ligget i
  `index.html`, kunne hvem som helst kopiert den og kjørt sin egen versjon gratis.

## Justeringer

- **Modell:** `api/plan.js` bruker `claude-sonnet-4-6` (god kvalitet, fornuftig kost for et 99-kr-produkt).
  Bytt til `claude-opus-4-8` for høyere kvalitet, eller `claude-haiku-4-5` for lavere kost.
- **CORS:** står på `*` for demo. Lås den til ditt eget domene før lansering (se kommentar i `api/plan.js`).
- **Prompt:** rediger `prompt/systemprompt.md` for lesing, men husk at det er kopien i `api/plan.js`
  som faktisk kjører. Hold dem i sync (eller refaktorer til at funksjonen leser .md-fila).

## Før du tar imot ekte brukere og betaling

Dette er en demo, ikke et ferdig produkt. Det som mangler:

1. **Betaling** — Vipps for 99-kr-kjøpet (krever org.nr + KYC, tar dager).
2. **Booking** — Cal.com → Google Meet for de ukentlige oppfølgingene (to uker gratis, så 299/uke).
3. **Personvern (GDPR).** Spørsmålene samler **helseopplysninger** — særlige kategorier etter art. 9.
   Du trenger uttrykkelig samtykke og en ryddig personvernerklæring fra dag én. Demoen lagrer ingenting;
   ikke begynn å lagre svar uten samtykke og et rettslig grunnlag.
4. **Tittelvern.** "Fysioterapeut" er beskyttet tittel. Hold kommunikasjonen tydelig på *trening*, ikke
   *behandling*, og la PT-en være PT — ikke antyd at PT-en driver fysioterapi.

## Fotnote

Treningsveiledning, ikke medisinsk behandling.
