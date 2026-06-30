# memengine MVP spec

## Product summary

A subscription service for people who want humor without social media. Subscribers pick a genre mix and a delivery cadence; memengine sends a small, curated pack of memes via email or text. Every meme lives on a branded mini page with a subscribe CTA at the bottom. Sharing keeps the brand attached.

---

## MVP scope

**In scope**
- Subscriber signup (email + SMS, genre preferences, 5-day or 7-day cadence)
- Manual curation queue (admin picks and tags memes, no AI curation yet)
- Branded meme page (one meme per URL, mobile-first, CTA at bottom)
- Email delivery (link to branded page, optional inline embed)
- SMS delivery (short link to branded page)
- Basic referral tracking (who shared, what converted)

**Out of scope for v1**
- Paid subscriptions / Stripe billing
- Algorithmic curation or scoring
- Video memes / YouTube Shorts mirror
- User-facing genre editing after signup

---

## Data model

```
Subscriber
  id, email, phone, genres[], cadence (5|7), status, referral_code, referred_by, created_at

Meme
  id, url, title, genre, format (image|gif|video), curator_notes, status (draft|approved|archived), created_at

Package
  id, subscriber_id, memes[] → Meme.id, sent_at, delivery_channel (email|sms)

DeliveryEvent
  id, package_id, channel, status (queued|sent|opened|clicked), ts

MiniPage
  id → Meme.id (1:1), slug, view_count, share_count
```

---

## Delivery channels

| Channel | What ships | Why first |
|---------|-----------|-----------|
| Email | Link + optional inline embed | Richer layout, easy unsubscribe, no per-message cost |
| SMS | Short link only | Immediate, personal, high open rate |
| Video (later) | YouTube Shorts mirror | Skip until core loop is proven |

---

## Build order

1. **Data model + admin queue** — Meme, Subscriber, Package tables; simple admin UI to approve memes and assign them to a package.
2. **Mini page** — `/m/:slug` route, meme at top, CTA at bottom, share button.
3. **Email pipe** — trigger on `Package.sent_at`, render template, send via Resend/Postmark.
4. **SMS pipe** — same trigger, send short link via Twilio.
5. **Signup flow** — public form: email, phone, genres, cadence → creates Subscriber.
6. **Referral + tracking** — append `?ref=CODE` on share links, record `DeliveryEvent` opens and clicks.

---

## First engineering step

Create the database schema and a minimal admin page that lets you:

1. paste a meme URL and pick its genre → saves a `Meme` row
2. select approved memes and assign them to a subscriber → saves a `Package` row
3. click "send" → queues the delivery event (stub the actual send for now)

Everything else builds on top of this content pipeline.
