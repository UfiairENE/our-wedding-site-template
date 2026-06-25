# Wedding Website

Built this for my own wedding at [omoniyi.co](https://omoniyi.co). It does everything — countdown, RSVP, seat assignment, guest emails, gift registry with crypto, travel guide, dress code. Forking should take you about 15 minutes.

---

## What's in it

| Section | Details |
|---|---|
| Hero + Countdown | Live ticker to the wedding date. Flips to "We're Married!" after |
| Event Details | Date, time, venue. Add to Calendar for Google and Outlook |
| RSVP | Physical or virtual, party size, dietary, media consent. Backed by Google Sheets. Closes on your deadline |
| Gift Registry | Bank transfer (NGN + USD) with masked account reveal. Crypto: BTC on-chain, Lightning, EVM (Base / BSC / Assetchain), Solana, Tron |
| Travel Guide | Flights, ride-hailing, hotels, restaurants, Google Maps embed |
| Color of Day | Dress code with colour swatches and outfit photos |
| Music Player | Background music, autoplay-aware, bottom-right corner |
| Dark / Light mode | Defaults dark |

---

## Stack

- React 18.3 + TypeScript 5.8
- Vite 5.4 (runs on port 3000)
- Tailwind CSS 3.4 — Playfair Display + Inter, CSS variable theming
- Framer Motion 12 — scroll animations
- shadcn/ui + Radix UI — accessible primitives
- React Hook Form 7 + Zod 3 — form validation
- next-themes — dark/light
- qrcode.react — QR codes in gift section
- TanStack React Query 5

---

## Setup

```bash
git clone https://github.com/UfiairENE/our-wedding-site-template.git
cd our-wedding-site-template

npm install

cp src/config.example.ts src/config.ts
cp .env.example .env

# fill in config.ts and .env (see below)

cp your-song.mp3 public/audio/background-music.mp3

npm run dev
# http://localhost:3000
```

---

## Configuration

Two files, both gitignored so your personal details never get committed.

### `src/config.ts`

Copy from `src/config.example.ts`. Full example:

```ts
export const config = {
  // Couple
  partner1: "Ada",
  partner2: "Emeka",
  conjunction: "et",  // "et", "&", "and" — shown between names

  // Dates — everything in UTC
  // WAT is UTC+1, so 1:00 PM WAT = "12:00:00Z"
  // Use https://www.timeanddate.com/worldclock/converter.html to convert
  eventStartUTC:   "2026-11-14T12:00:00Z",
  eventEndUTC:     "2026-11-14T20:00:00Z",
  seatingStartUTC: "2026-11-14T11:30:00Z",

  eventDateDisplay:    "November 14, 2026",
  eventDayDisplay:     "Saturday",
  seatingCopy:         "Please be seated · Ceremony begins at 1PM WAT",
  rsvpDeadlineDisplay: "October 31, 2026",

  // Set null to close the RSVP form automatically at eventStartUTC
  rsvpClosesAtUTC: "2026-10-31T23:59:59Z",

  postWeddingHeadline: "We're Married!",
  postWeddingSubtext:  "Thank you for celebrating with us",

  establishmentYear: "2026",

  // Venue
  venueDisplay: "Private Location in Lagos, Nigeria",
  venueSubtext: "Address sent upon RSVP approval",

  // Travel
  airport: {
    name: "Murtala Muhammed International Airport",
    iata: "LOS",
  },
  airlineBookingUrl: "https://www.arikair.com/",
  airlineName:       "ARIK AIR",
  rideshareCity:     "Lagos",
  transitCostRange:  "₦5,000 – ₦12,000",

  hotels: [
    "Eko Hotel & Suites",
    "Federal Palace Hotel",
    "Radisson Blu Anchorage",
  ],
  restaurants: [
    "Nok by Alara",
    "Nkoyo",
    "Bungalow Beach Bar",
  ],

  // Google Maps → Share → Embed a map → copy the src="..." value
  mapEmbedSrc: "https://www.google.com/maps/embed?pb=...",

  // {city} gets replaced with rideshareCity automatically
  travelVideos: [
    { label: "Interesting Places", url: "https://youtu.be/..." },
    { label: "Sights of {city}",   url: "https://youtu.be/..." },
  ],

  // Gifts
  bankAccounts: [
    { currency: "NGN (Naira)",  bank: "Moniepoint MFB", account: "0123456789", name: "Ada Okafor" },
    { currency: "USD (Dollar)", bank: "Providus Bank",   account: "9876543210", name: "Ada Okafor" },
  ],

  crypto: {
    btcOnchain:    "bc1q...",
    btcLightning:  "you@walletofsatoshi.com",
    evmAddress:    "0x...",  // used for Base, BSC, and Assetchain
    solanaAddress: "...",
    tronAddress:   "T...",
  },

  // Dress code
  colorOfDayDisclaimer: "Please dress in one of the colours below.",  // set "" to hide

  colorOfDay: [
    {
      name: "Dusty Rose",
      hex: "#C4849A",
      description: "A sophisticated mauve pink",
      images: [
        { src: "/src/assets/bride-pink.png", alt: "Bride in dusty rose" },
        { src: "/src/assets/groom-pink.png", alt: "Groom in dusty rose" },
      ],
    },
  ],

  musicSrc: "/audio/background-music.mp3",  // set "" to disable the player

  quote: "Though one may be overpowered, two can defend themselves.",
  quoteAttribution: "Ecclesiastes 4:12",

  siteTitle:       "Ada et Emeka | Marriage Ceremony",
  siteDescription: "Join us for our marriage ceremony.",
  twitterHandle:   "@yourhandle",
};
```

### `.env`

```bash
VITE_GOOGLE_SCRIPT_URL="https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec"
```

The script URL comes from the RSVP backend setup below.

---

## RSVP backend

The form POSTs to a Google Apps Script web app. The script lives in `wedding-script.gs` in this repo — it handles form submissions, seat assignment, and all guest emails (confirmation, reminder, final week, day-of).

### Step 1 — Create the spreadsheet

1. New spreadsheet at [sheets.google.com](https://sheets.google.com)
2. Rename the default tab to `Physical`, add a second tab called `Virtual`

### Step 2 — Add the script

1. Extensions → Apps Script
2. Delete everything and paste `wedding-script.gs`
3. Fill in the `CONFIG` block at the top:

```javascript
var CONFIG = {
  PARTNER_1_NAME   : "Ada",
  PARTNER_2_NAME   : "Emeka",
  EVENT_DATE       : "November 14, 2026",
  EVENT_TIME       : "1:00 PM WAT",
  CHECKIN_TIME     : "12:30 PM WAT",
  COLOR_OF_THE_DAY : "Sage Green & Ivory",

  VENUE_NAME       : "The Grand Hall, 12 Example Street, Lagos",
  VENUE_MAP_LINK   : "https://maps.google.com/?q=The+Grand+Hall+Lagos",

  YOUTUBE_LINK     : "https://www.youtube.com/live/YOUR_STREAM_ID",

  WEBSITE_URL      : "https://yourweddingsite.com",
  GIFT_PAGE_URL    : "https://yourweddingsite.com/gifts",

  SEATS_PER_TABLE  : 10,
  MAX_TABLES       : 10,

  GIFT_METHODS: [
    { label: "Bank Transfer", value: "Your Bank: Account Number (Your Name)" },
    { label: "PayPal",        value: "yourname@email.com" }
  ]
};
```

4. Save, then run **Wedding Admin → Fix Sheet Headers** to set up both tabs

### Step 3 — Deploy

1. Deploy → New deployment → Web app
2. Execute as: Me / Who has access: Anyone
3. Copy the URL into `.env` as `VITE_GOOGLE_SCRIPT_URL`

> If you edit the script later, deploy again as a new deployment — don't edit an existing one. The URL stays the same.

### Sheet columns

`Fix Sheet Headers` creates these automatically:

| Column | What it holds |
|--------|--------------|
| A — Timestamp | Set on submission |
| B — Attendance | `physical` or `virtual` |
| C — Name | Guest name |
| D — Email | Used for duplicate checking across both tabs |
| E — Phone | |
| F — Relationship | How they know you |
| G — Guests | Total party size |
| H — Guest Name | Additional guest names |
| I — Dietary | |
| J — Media Consent | |
| K — Message | |
| L — Seat Number | Auto-filled on approval |
| M — Status | **Set to `APPROVED` to trigger seat assignment + confirmation email** |
| N — Email Sent | `YES` / `NO` — updated automatically |

### Approving a guest

Change column M to `APPROVED`. That's it — seat gets assigned and confirmation email goes out immediately. To process a batch at once, use **Wedding Admin → Assign Seats & Send Emails**.

### Week and day-of emails

Run these manually from the Wedding Admin menu when you're ready:

- **Send Reminder** — run any time to send all confirmed guests their seat details and a countdown
- **Send Final Week Reminder** — ~7 days out
- **Send Day-Of Email** — morning of the wedding

Both only go to guests who are already confirmed (Status = APPROVED, Email Sent = YES), so no one gets emailed out of order or twice.

### Reserved seating

Want specific guests at a specific table? say VIP? Type their seat into column L before approving them. The script will skip auto-assignment and just use what you put there.

---

## Music

Drop an MP3 in `public/audio/` and update `musicSrc` in `config.ts`:

```ts
musicSrc: "/audio/your-song.mp3",
```

Autoplay runs at 30% volume. Most browsers block it on mobile until the first tap — the player handles that gracefully. Set `musicSrc: ""` to remove the player entirely.

---

## Dress code photos

Add images to `src/assets/` and reference them in `colorOfDay`. Two placeholder SVGs are already there so the section doesn't break before you swap them out.

---

## Deployment

`vercel.json` is already set up with SPA rewrites and security headers.

**Vercel** — connect your repo, add `VITE_GOOGLE_SCRIPT_URL` as an environment variable, deploy. Every push to main redeploys.

**Netlify** — build command `npm run build`, publish directory `dist`, add the env variable in site settings.

**Anywhere else** — run `npm run build`, upload `dist/`, configure the host to serve `index.html` for all routes.

---

## Routes

| URL | Section |
|---|---|
| `/` | Top |
| `/rsvp` | RSVP form |
| `/gifts` | Gift registry |
| `/details` | Event details |
| `/travel` | Travel guide |

---

## Project structure

```
.
├── public/
│   ├── audio/background-music.mp3
│   └── favicon.ico
├── src/
│   ├── assets/
│   ├── components/
│   │   ├── ColorOfDay.tsx
│   │   ├── Countdown.tsx
│   │   ├── EventDetails.tsx
│   │   ├── Footer.tsx
│   │   ├── GiftRegistry.tsx
│   │   ├── Hero.tsx
│   │   ├── MusicPlayer.tsx
│   │   ├── Navigation.tsx
│   │   ├── RSVPForm.tsx
│   │   ├── TravelGuide.tsx
│   │   └── ui/
│   ├── config.example.ts
│   ├── config.ts          ← gitignored
│   └── pages/
│       └── Index.tsx
├── wedding-script.gs      ← Google Apps Script backend
├── .env.example
├── .env                   ← gitignored
├── vercel.json
└── tailwind.config.ts
```

---

MIT. Use it, change it, ship it. A star is always appreciated.
