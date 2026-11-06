# Love Market (Ọjà Ìfẹ́)

**A Nigeria-native dating app. Strategy and build plan.**

Working name: **Ọjà Ìfẹ́**, "love market." The market metaphor is load-bearing. A market is a place
with traders, reputations and witnesses. That is the product.

**One-line thesis:** Tinder sells access to strangers. Love Market sells *verified* strangers you
can safely meet, and it takes naira, on any phone, over any network.

---

## 1. Why Tinder is beatable here

Not the swipe. The swipe is fine. The gaps are structural and Tinder cannot fix them locally.

| Gap | What actually happens in Nigeria | Love Market's answer |
|---|---|---|
| **Trust** | Catfishing, romance fraud, sextortion, "runs girls." The local verdict on the current options is that both Tinder and Bumble "have been occupied by SW." | Verification is the gate, not a badge. Phone plus NIN plus liveness selfie before you appear in a deck. |
| **Safety** | Women carry real kidnapping and robbery risk at a first meet. Nobody local addresses it. | Women-first messaging, in-app voice and video, date check-in with a trusted contact, live location share. Never paywalled. |
| **Payments** | Cards are rare, fail on FX, and people distrust silent recurring charges. | Naira pricing, bank transfer, USSD, and airtime. Wallet credits so you never re-auth a card silently. |
| **Data** | Roughly 40% mobile internet penetration, metered data, 2G and 3G moments. | Low-data mode by default, aggressive photo compression, voice notes as a first-class medium. |
| **Culture** | Courtship is semi-public, "what will people say" is real, and hookup framing alienates a huge serious segment. | Intention tags up front. Voice-note openers, because that is how Nigerians actually start a conversation. |

### Market numbers

- Smartphone ownership rose from **64% to 75%** between 2023 and 2025 (KPMG and Orange).
- **Android holds about 63%** of that base; iOS is under 2%.
- Mobile internet penetration is still only around **40%**.
- Tinder's Nigerian usage is largely the Lagos and Abuja elite, often via Passport mode.

The second half of the market is arriving, not saturated. That is the opening.

---

## 2. The five decisions that define the product

1. **Verification is the product, not a feature.** If verification is optional you have built Tinder with a green tick. Make it the gate: unverified profiles are shadow-ranked and cannot be swiped right.
2. **Safety is never paywalled.** Every safety feature is free for everyone, forever. This is an ethics position and also the single best marketing claim in the category.
3. **Naira-native, low-ticket, non-recurring by default.** A ₦2,500 pass paid by transfer converts in Nigeria where a $9.99 card subscription does not.
4. **Density before breadth.** One saturated zone beats five empty ones. A dating app with 40 people within 10 km is dead regardless of the code quality.
5. **Android-first, data-light, offline-tolerant.** Assume a Tecno, Infinix or itel on 4G that drops to H.

---

## 3. Architecture

Built for speed, given a Node and JavaScript background.

| Layer | Choice | Why |
|---|---|---|
| Client | Expo React Native and TypeScript | One codebase for both stores, and OTA updates via EAS so you are not waiting on store review to fix a bug. |
| Server | Node and TypeScript (Fastify) | Familiar ground. |
| Data | PostgreSQL with PostGIS, plus Redis | Geo queries and the deck queue with rate limits. |
| Chat | Socket.IO or Supabase Realtime | Store messages yourself. Do not proxy them through a third party. |
| Media | Cloudflare R2, server-side compression, Opus for voice | Data cost is a feature, not an afterthought. |
| Identity | Dojah or Prembly through NINAuth | See section 4.1. |
| Payments | Paystack primary (cards, transfer, USSD, airtime), Flutterwave fallback | See section 5. |
| Push | Expo Notifications over FCM | Standard. |
| Moderation | NSFW classifier feeding a human review queue | Trust and Safety is headcount, not a feature flag. |

**Why distance reads "12 mins away" and not "4.2 km":** in Lagos, kilometres are meaningless and
traffic is everything. Compute drive time, not crow-flies. A small thing that makes the product
feel local instead of translated.

---

## 4. Trust architecture: three layers, not one

Device fingerprinting and identity verification are different products that people constantly
confuse, and the confusion is expensive. Love Market needs three layers, and only one of them
earns a public badge.

### Layer 1: Identity (who you are)

Phone OTP, NIN verification, liveness selfie, and a face match. This is the only layer that
produces the "Verified" badge users actually see. Automated, per head, at signup.

### Layer 2: Device (how many of you there are)

A persistent device identifier. **Fingerprint** (fingerprint.com) ships a React Native SDK; the
free open-source FingerprintJS covers web only. This catches the real local pattern: one handset
running twenty profiles, and a banned user back the same afternoon. Use it to rank down, to force
re-verification, and to flag for review.

Two caveats specific to this market:

- Fingerprinting assumes one person equals one device. In Nigeria, handsets get shared inside families and among friends, so a shared device is weak evidence on its own. Treat it as a signal, never a verdict, and never auto-ban on it.
- Disclose it in the privacy policy in plain language, and give users a "devices on this account" screen they can review and revoke. Being caught doing this silently is worse than the fraud you were preventing.

### Layer 3: Graph (is this a ring)

Reports, message patterns, photo reuse detection via perceptual hashes across profiles, shared
payment instruments, and OSINT lookups run by Trust and Safety analysts on accounts that have
already been reported. Back-office tooling, never a signup step.

### On tools like fingerprint.to

Careful, there are two different products sharing that name, and they are opposites.

**fingerprint.to** is not device fingerprinting. It is OSINT reverse lookup: given a username,
email or phone number it sweeps 700+ platforms and returns matching accounts. It is built for
private and fraud investigators. It is a real tool, and it does not belong in your signup funnel.

- **It does not verify anything.** Their own FAQ: "A search match is a research lead; it does not confirm a person's identity." You would be making trust decisions on a signal the vendor explicitly disclaims.
- **The signal is adversarially inverted.** The service has an opt-out. A scammer operating at scale removes himself; an ordinary user in Surulere has never heard of it. You would filter out honest users and keep the professionals.
- **It is disproportionate under NDPA 2023.** Sweeping a new user's cross-platform footprint to decide whether they are real is profiling far beyond the purpose, and you cannot map data provenance because the provider queries third parties. That is a DPIA you will lose.
- **It is the wrong story.** "Love Market looks up your whole internet history when you sign up" is a headline that ends a dating app whose whole pitch is women's safety.
- **The API is not live.** It is a waitlist.

Where the capability does belong: in your Trust and Safety back office, pointed at reported
accounts, run by a human analyst, with a documented lawful basis and an audit log. That is a
legitimate fraud-investigation use. Pointing it at every new signup is not.

**The rule to hold:** only Layer 1 earns a badge a user can see. Layers 2 and 3 change ranking,
trigger review, and quietly protect the market.

---

## 4.1 How you actually get NIN verification

You do not get NIMC access. Nobody outside the licensed circle does, and anyone promising a direct
NIMC integration is selling something they cannot deliver. The path is fixed and it runs through a
partner.

NIMC runs the consent-based service, **NINAuth**. You connect to it through an **approved
identity Verification Partner**, never directly. Their own wording: an enterprise is connected to
NIMC verification services "through an approved identity Verification Partner."

1. **Be a real Nigerian company.** CAC registration, TIN, and physical presence in Nigeria. NINAuth's access criteria require Nigerian presence and being subject to Nigerian law.
2. **Be NDPA compliant first.** Register with the NDPC as a data controller, appoint a DPO, publish a privacy policy. This is a stated eligibility requirement for NINAuth access, not a later chore. You cannot get access while non-compliant.
3. **Contract an approved NINAuth Verification Partner.** NIMC publishes the list; you pick one and sign, and they notify NIMC on your behalf. Expect a decision in 5 to 7 business days after contracting. Pricing is negotiated with the partner directly.
4. **Build the consent flow.** NINAuth is consent based: the user explicitly authorises the match. NIMC requires informed consent, purpose limitation, logging and monitoring, compliance reporting, and acceptance of independent audits of your use.
5. **Pair the NIN with a liveness check and a face match.** This is the step people skip and it is the one that matters. A NIN match proves a real Nigerian with that name exists. It does **not** prove the face in the profile photos belongs to them. Without liveness plus face match, a scammer uses a relative's NIN and a stolen photo set.
6. **Never store the raw NIN.** Keep a token or hash, a verified-at timestamp, and which partner ran the check. The NIN itself is not yours to keep.

Vendors already holding partner status: **Dojah, Prembly (IdentityPass), Smile ID, Youverify,
VerifyMe**. Published entry pricing in 2026 runs about $0.04 per check at the cheap end (Dojah) to
$0.12 (Prembly); Smile ID and Youverify are quote based.

### What it costs at scale

A NIN lookup is a cheap database query. Liveness and face match is the expensive layer. End to
end, budget roughly **$0.24 to $0.42 per verified user**, which at the CBN rate of about
**₦1,347 to the dollar** (2 November 2026) is **₦320 to ₦570 per verified user**:

| Verified users | One-time verification cost |
|---|---|
| 5,000 | $1,200 to $2,100 (₦1.6m to ₦2.8m) |
| 50,000 | $12,000 to $21,000 (₦16m to ₦28m) |
| 100,000 | $24,000 to $42,000 (₦32m to ₦57m) |

At 100,000 users that is roughly **three months of your entire subscription revenue, spent once,
before anyone has paid you anything.** This is why verification must be risk based: verify when a
user becomes swipeable or asks for the badge, not casually, and re-verify only when the device
layer flags something.

### The tailwind

The **NIMC Act 2026**, signed 26 June 2026, repeals the 2007 Act and makes the NIN the foundation
of Nigeria's digital identity, with Section 26 expanding mandatory NIN use across banking,
telecoms and other transactions. Nigerians are being pushed toward NIN verification everywhere.
Your check stops being an odd thing a dating app asks for.

---

## 4.2 Social handles: stage them, do not ban them

In Lagos, a person's social presence *is* their credibility, so refusing to let anyone check is
asking people to meet blind. But the exchange is also the single most documented scam mechanism in
the category.

The evidence is blunt:

- The FTC found **40% of romance scam losses started on social media**, and that scammers then "quickly moved the sweet talk to WhatsApp, Google Chat, or Telegram."
- **ICE** lists "requests to move conversations to a separate app" as a headline red flag.
- **Sextortion reports are up more than eightfold since 2019**, and the US and Nigeria are actively cooperating on sextortion and romance scam enforcement right now.

A Nigerian dating app that accelerates the off-platform handoff is building the on-ramp for the
exact crime its safety story promises to stop. So gate it on trust, do not forbid it.

- **No handles in the profile or bio.** That turns the deck into a follower farm, which is the most common complaint about Hinge's social links.
- **Unlock handle exchange after a trust threshold:** both parties verified, a minimum number of messages over a minimum number of days, and at least one completed in-app voice or video call. The call is the real gate, because it is the one thing a scammer working from stolen photos cannot do.
- **Do not design around OAuth linking.** Instagram Basic Display API, the thing that let Hinge prove a user owned their Instagram, was **permanently shut down on 4 December 2024**. There is no consumer Instagram API any more; the replacement requires the user to hold a professional account. X's API is paid and expensive. The "link your socials to prove you're real" feature that was easy in 2021 is not buildable for ordinary users in 2026.
- **If you accept a declared handle, run a proof-of-ownership challenge** (post a code in the bio or story, you check it) and label it **self-declared, not verified**. Never let a pasted handle earn the verified badge, or the badge means nothing.
- **When a handle is shared, log it** and show a one-line interstitial with the scam script warning. Keep report and block working on off-platform conduct.

---

## 4.3 What the back office does not do

The Trust and Safety back office **does not verify anyone**. It investigates the small share of
users who get reported, and it is reactive, human and slow on purpose.

At 100,000 monthly active users and a typical 2% monthly report rate you are looking at about
**2,000 cases a month**. A trained analyst working properly closes roughly 40 a day, so that is
about **three analysts plus a lead**. It cannot touch every user, and it must not try to.

Verification of everyone is Layer 1: automated, per head, cheap, at signup. The back office
protects the market; Layer 1 verifies the individual. Confuse the two and you will try to staff
your way to trust and go bankrupt doing it. The back office is a permanent cost centre with no
revenue attached. Budget for it like a product team and expect it to grow with MAU.

---

## 5. Monetisation

Revenue lines, in the order they turn on.

### Phase 1: individual premium

| Product | Price |
|---|---|
| Market Pass, one month | ₦2,500 |
| Market Pass, three months | ₦6,500 (save 13%) |
| Market Pass, one year | ₦19,000 (save 37%) |
| Spotlight boost, 30 minutes | ₦700 |
| Super Like | ₦200 each |
| Market Cash wallet | ₦1,000 → 1,100 credits · ₦5,000 → 6,000 · ₦20,000 → 26,000 |

Market Pass gives unlimited likes, see who liked you, rewind, five Super Likes a week, and
Passport to any city.

**The wallet is the real trick.** It decouples you from two Nigerian problems at once: the failed
card, and the fear of "why is this app charging me again."

**Safety is free, forever.** Verification, reporting, blocking, date check-in and live location
share are never paywalled. This is an ethics position and the best marketing claim in the category.

### Phase 2: Àṣà events (the moat)

Ticketed singles mixers, Lagos then Abuja then Port Harcourt. ₦15,000 to ₦40,000 per head, sponsor
funded. This is in-person, it delivers a real date, and a global app will never run it.

### Phase 3: Owambe marketplace (where the money actually is)

Nigerian courtship spending ends in a wedding, and that budget is enormous. Route matched,
long-term couples to vetted vendors (photographers, planners, aso-ebi, caterers) at **10 to 15%
commission**. The dating app becomes the customer-acquisition channel for the wedding economy.
This is the line that turns a small subscription business into a big one.

### Unit economics to hold yourself to

- Blended CAC under **₦1,500**.
- Market Pass conversion at **4% of MAU or better**.
- **LTV to CAC above 3** within the first two cities.
- Airtime and wallet payments clearing **30% or more** of transactions. If not, your pricing is wrong for the market. Revisit it.

---

## 6. Build sequence

Each phase ends with something a real person in Lagos can use.

### Phase 0, Foundations (week 1)

- CAC registration, TIN, corporate bank account. Apple and Google organisation accounts need a D-U-N-S number, so start early.
- **NDPA 2023 compliance:** register with the NDPC as a data controller, appoint a DPO, run a DPIA. Dating data (photos, location, identity, relationships) is about as sensitive as consumer data gets. Do this before you collect a single profile.
- Pick verification and payment vendors. Start the NINAuth partner conversation now, because it takes 5 to 7 business days just for the decision.

### Phase 1, The deck works (weeks 2 to 5)

- Auth by phone number with OTP, and a hard 18+ gate.
- Profile creation: four photos, three prompts, an intention tag, an area.
- Verification flow: NIN plus liveness plus face match, producing the badge.
- Swipe deck with a PostGIS radius, a Redis queue, and drive-time distance.
- Report and block on every surface. Moderation queue from day one.

### Phase 2, People can actually talk (weeks 6 to 9)

- Match to chat, with voice notes and photo messages.
- In-app audio and video calls over WebRTC. Keep people off WhatsApp until trust is established.
- Safety kit: date check-in, live location share to a trusted contact, "I'm safe" ping.
- Push notifications, batched and quiet-hours aware.

### Phase 3, Money (weeks 10 to 12)

- Paystack integration: cards, transfer, USSD, airtime.
- Wallet and credits ledger, double entry and idempotent webhooks.
- Market Pass, Spotlight, Super Likes.
- **Do not hold customer float yourself.** Use a partner so you stay out of PSSP-licence territory.

### Phase 4, Launch city #1 (weeks 13 to 16)

- Soft launch in **Lekki and Ajah plus Yaba and Surulere**. 5,000 verified humans, hand-seeded via campus ambassadors (UNILAG, Yabatech) and night-life partnerships.
- Waitlist with referral position and a "Founding 5,000" badge.
- Subsidise verified women with six months of free Market Pass until the ratio is healthy. If the ratio is bad, men churn out and you never recover.

### Phase 5, Scale cities and later revenue

- Abuja (Wuse and Gwarinpa), then Port Harcourt. Repeat the playbook.
- First Àṣà event. Vendor marketplace pilot.

---

## 7. Go to market

- **Distribution:** X and Twitter Nigeria are the cultural engine. TikTok skits. Instagram meme pages. Big Brother Naija season is the single biggest dating-adjacent moment of the year, so budget for it.
- **Message:** lead with *"Every profile is a real Nigerian."* Trust, not horniness. It is the only ad Tinder cannot run.
- **Referrals:** friend links with credits on both sides, shared as a WhatsApp-ready card. Nigeria runs on WhatsApp groups.
- **The founder story:** Ọjà Ìfẹ́ is a Lagos name. Lean on it. A local brand beats a translated one.

---

## 8. Risks and honest constraints

- **Same-sex matching is a legal minefield in Nigeria under SSMPA 2013.** This needs an explicit decision and a Nigerian lawyer, not an engineer. It also caps the Western app-store and investor narrative. Decide it on purpose.
- **Trust and Safety is a permanent cost centre.** Underfund it and one kidnapping story ends the company. Fund it like a product team.
- **Cold start is the real killer.** Code is the easy part. The first city's density is the whole game.
- **NDPA penalties are real**, and dating data is exactly the kind regulators care about.
- **Verification friction will cost you signups.** Accept it, measure it, keep it. Friction is the feature.
- **Verification cost is front-loaded.** ₦32m to ₦57m at 100,000 users, spent before revenue. It dictates a risk-based rollout.

---

## 9. Open decisions

These are calls, not tasks.

1. **Positioning:** serious relationships first (larger, safer, monetises via weddings) or casual (faster growth, worse ratio, Tinder's war)? This changes the entire design.
2. **Legal posture on same-sex matching** under SSMPA 2013. Needs your call and a lawyer's.
3. **First city and your access to it.** Do you have a way to seed Lagos density through a campus, event or friend network, or do we go where you do?
4. **Build or buy the app shell.** Expo from scratch is about 12 weeks to Phase 3. A whitelabel dating backend could cut that, but you would rent your differentiation.
5. **NIN gate position:** required to be swipeable (safest, costs signups, ₦320 to ₦570 per head you cannot afford on tyre-kickers) or required only to be verified and to unlock premium (cheaper, market self-selects). Recommended: start with the second, tighten to the first inside your launch zone only.

---

## 10. Prototype

A clickable HTML mockup of the core screens lives at `prototype/index.html`: onboarding, the
verification flow, a working swipe deck, the voice-note chat with the handle gate, and the Market
Pass paywall. It exists to make the positioning arguable before a line of app code is written.

---

## Sources

- KPMG and Orange, *Nigeria Smartphone Study 2025* (smartphone penetration 75%, Android share).
- Statista, mobile internet penetration Nigeria, 2025.
- NIMC, NINAuth for business and enterprise access criteria: https://ninauth.nimc.gov.ng/enterprises
- NIMC, licensed service providers: https://nimc.gov.ng/licensed-service-providers
- Manifield Solicitors, *NIMC Act 2026: 7 key changes for businesses*.
- Dojah, *Best Identity Verification API in Africa 2026* and *How much does KYC verification cost in Nigeria*: https://dojah.io/blog/kyc-pricing-nigeria
- Prembly pricing: https://prembly.com/pricing/
- Meta for Developers, *Update on Instagram Basic Display API* (shutdown 4 December 2024): https://developers.facebook.com/blog/post/2024/09/04/update-on-instagram-basic-display-api/
- FTC, *Romance scammers' favorite lies exposed*: https://www.ftc.gov/news-events/data-visualizations/data-spotlight/2023/02/romance-scammers-favorite-lies-exposed
- ICE, *Protect yourself against romance scams*.
- Central Bank of Nigeria exchange rates, 2 November 2026 (₦1,347 to the dollar).
