# Kannon.email — Un Mail Sender open-source per Kubernetes

Kannon è un servizio **open-source** per l'invio massivo di email, pensato per essere **cloud-native** e integrato facilmente in ambienti **Kubernetes**.

È stato sviluppato da zero con un obiettivo chiaro: **inviare milioni di email senza finire nello spam**, mantenendo **controllo completo sull'infrastruttura**, e rispettando le esigenze di chi lavora in Europa (GDPR, self-hosted, ecc).

> ⚠️ Non è una black-box americana. Nessun tracciamento nascosto, nessun vendor lock-in.

## ✨ Caratteristiche principali

- ✅ Architettura nativa per Kubernetes (Helm chart incluso)
- ✅ API HTTP e client `kannon.js` disponibili
- ✅ Integrazione DKIM/SPF/CNAME/MX semplice tramite interfaccia web
- ✅ Supporta template MJML e tracciamento apertura/click
- ✅ Progettato e sviluppato in 🇮🇹, pensato per 🇪🇺

###
---
Stai costruendo qualcosa di interessante? Fammelo sapere se vorresti utilizzare Kannon in una [sessione uno a uno](https://bit.ly/parla-con-ludo) con me.
---
###

## Getting Started

### Configurazione del dominio e DNS

Partendo dalla dashboard di Kannon, creiamo un nuovo dominio, ad esempio `k.miodominio.it`.

#### Scegli un sottodominio

Per evitare conflitti con il server email predefinito, solitamente preferisco scegliere un sottodominio per inviare le email da Kannon. Il mio standard è utilizzare un sottodominio nella forma `k.<il-tuo-dominio-principale>`.

Una volta creato un dominio, possiamo accedere alla pagina di Configurazione DNS, che ci aiuta a configurare il DNS per inviare correttamente le email da Kannon. Devi configurare due record DNS TXT (necessari per aggiungere l'autenticazione e autorizzazione SPF e DKIM al mittente email di Kannon), e un CNAME e un server MX necessari per tracciare le statistiche.

Dobbiamo aspettare che il DNS si propaghi correttamente prima di iniziare a inviare email. Per verificarlo, uso uno dei diversi servizi di tracciamento DNS disponibili su internet.

### Iniziare a scrivere codice per inviare email in Kannon con Node.js

Una volta che i DNS sono propagati, possiamo iniziare a scrivere il nostro client Node.js con l'aiuto di `kannon.js`, una semplice libreria Node per iniziare a utilizzare il servizio.

Nella pagina del codice della piattaforma, troverai un semplice esempio di codice per iniziare che puoi utilizzare per implementare il client. Devi usare la tua chiave API disponibile dalla console.

Possiamo testare se tutto funziona scrivendo un semplice script in TypeScript che invia una semplice email:

```typescript
import { KannonCli } from "kannon.js";

const kannon = new KannonCli(
  "k.miodominio.it",
  process.env.KANNON_KEY as string,
  {
    email: "info@miodominio.it",
    alias: "Il mio servizio",
  },
  {
    host: "grpc.kannon.email:443",
  }
);

const html = `<html>
<body>
	<h1>Prima email Kannon!</h1>
	<p>Questa è un'email di test da <a href="https://www.kannon.email">Kannon</a></p>
</body>
</html>`;

async function main() {
  return kannon.sendHtml(
    [
      {
        email: "utente@esempio.it",
        fields: {},
      },
    ],
    "📧 Prima Mail da Kannon 💣!",
    html
  );
}

main().then((res) => console.log(res));
```

Eseguendo questo script, dovresti essere in grado di ricevere l'email sul tuo client in pochi secondi. Inoltre, dovresti essere in grado di tracciare le statistiche (consegnate, aperture, ecc.) dalla dashboard di Kannon nella pagina delle statistiche del tuo dominio.

### Creare un template email in MJML

Oltre all'invio di email direttamente da HTML, Kannon ti permette di creare template email utilizzando la sintassi MJML. In questo modo, puoi facilmente utilizzare l'ID del template email nel codice per inviare l'email, e gestire e aggiornare il template dalla piattaforma.

Ecco un esempio di template MJML:

```xml
<mjml>
  <mj-head>
    <mj-title>Benvenuto nel nostro servizio</mj-title>
    <mj-preview>🚀 Benvenuto nel nostro servizio!</mj-preview>
    <mj-attributes>
      <mj-all
        font-family="'Helvetica Neue', Helvetica, Arial, sans-serif"
      ></mj-all>
      <mj-text
        font-weight="400"
        font-size="16px"
        color="#000000"
        line-height="24px"
        font-family="'Helvetica Neue', Helvetica, Arial, sans-serif"
      ></mj-text>
    </mj-attributes>
    <mj-style inline="inline">
      .body-section {
        -webkit-box-shadow: 1px 4px 11px 0px rgba(0, 0, 0, 0.15);
        -moz-box-shadow: 1px 4px 11px 0px rgba(0, 0, 0, 0.15);
        box-shadow: 1px 4px 11px 0px rgba(0, 0, 0, 0.15);
      }
    </mj-style>
    <mj-style inline="inline">.text-link { color: #5e6ebf }</mj-style>
    <mj-style inline="inline">.footer-link { color: #888888 }</mj-style>
  </mj-head>
  <mj-body background-color="#E7E7E7" width="600px">
    <mj-section full-width="full-width" background-color="white">
      <mj-column width="100px">
        <mj-image
          src="https://via.placeholder.com/100"
          alt="Logo"
          align="center"
          width="100px"
        ></mj-image>
      </mj-column>
    </mj-section>
    <mj-section
      full-width="full-width"
      padding-top="20px"
      background-color="#009900"
    >
    </mj-section>
    <mj-wrapper padding-top="0" padding-bottom="0px" css-class="body-section">
      <mj-section
        background-color="#ffffff"
        padding-left="15px"
        padding-right="15px"
      >
        <mj-column width="100%">
          <mj-text color="#637381" font-size="16px">
            <p>Ciao {{ name }},</p>
            <p>
              Grazie per la tua registrazione alla nostra piattaforma!
            </p>
            <p>
              Per accedere a tutte le funzionalità, dovrai completare la
              verifica del tuo account. Se hai domande, rispondi pure a
              questa email.
            </p>
            <p>Cordiali saluti,</p>
            <p>Il team di <strong>MioServizio</strong></p>
          </mj-text>
        </mj-column>
      </mj-section>
    </mj-wrapper>
    <mj-wrapper full-width="full-width">
      <mj-section>
        <mj-column width="100%" padding="0px">
          <mj-text
            color="#445566"
            font-size="11px"
            align="center"
            line-height="16px"
          >&copy; MioServizio, Tutti i diritti riservati.</mj-text>
        </mj-column>
      </mj-section>
    </mj-wrapper>
  </mj-body>
</mjml>
```

Ora dobbiamo creare un nuovo template nella sezione Template della dashboard.

Una volta creato, l'anteprima del Template è visibile dalla pagina del template.

#### Utilizzare campi personalizzati

Nota che ho utilizzato un campo personalizzato `{{ name }}` nel template email. Questi campi possono essere popolati quando l'email viene inviata per personalizzare l'email per ogni cliente.

Ora possiamo utilizzare il `template_id` per inviare email usando il nostro script personalizzato, con solo una piccola modifica nella funzione main:

```typescript
async function main() {
  return kannon.sendTemplate(
    [
      {
        email: "utente@esempio.it",
        fields: {
          name: "Mario",
        },
      },
    ],
    "Benvenuto nel nostro servizio 🔨👨‍💼!",
    "<TEMPLATE_ID>"
  );
}
```

### Integrare Kannon in Next.js e NextAuth

Quello che vogliamo fare ora è inviare automaticamente email quando un utente si registra alla piattaforma. Per farlo, possiamo sfruttare le funzionalità degli eventi di NextAuth, in particolare possiamo ascoltare gli eventi `createUser`. Ma prima di tutto dobbiamo creare una classe per gestire le email.

Suggerisco di wrappare il client kannon con una classe per gestire facilmente tutte le email in un unico posto, come nel codice seguente:

```typescript
// email-sender.ts

import type { KannonCli } from "kannon.js";

const templates = {
  welcomeUser: {
    id: "<TEMPLATE_ID>",
    subject: "Benvenuto nel nostro servizio 🔨👨‍💼!",
  },
} as const;

export class EmailSender {
  constructor(private readonly kannonCli: KannonCli) {}

  sendWelcomeUserEmail(email: string, name: string) {
    return this.kannonCli.sendTemplate(
      [{ email, fields: { name } }],
      templates.welcomeUser.subject,
      templates.welcomeUser.id
    );
  }
}
```

Ora possiamo utilizzare l'oggetto EmailSender per inviare email di benvenuto quando un utente viene creato in NextAuth:

```typescript
const kannon = new KannonCli(
  env.KANNON_DOMAIN,
  env.KANNON_KEY,
  {
    alias: env.KANNON_ALIAS,
    email: env.KANNON_EMAIL,
  },
  {
    host: env.KANNON_HOST,
  }
);
const sender = new EmailSender(kannon);

export const authOptions: NextAuthOptions = {
  // ...
  events: {
    createUser: async ({ user }) => {
      await sender.sendWelcomeUserEmail(user.email, user.name);
    },
  },
};
```

Ora, ogni volta che un utente viene creato nella piattaforma, riceverà questa email di benvenuto.

## Prossimi passi

Integrare Kannon con un servizio Next.js è super facile e super veloce. Da ora in poi, ho configurato la struttura per aggiungere più email transazionali, come per aggiornamenti di progetti, richieste di pubblicazione di nuovi progetti, ecc.

Stai costruendo qualcosa di interessante? Fammelo sapere se vorresti utilizzare Kannon in una [sessione uno a uno](https://bit.ly/parla-con-ludo) con me.
