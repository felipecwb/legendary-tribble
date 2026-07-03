---
layout: post
ref: qr-codes-are-just-barcodes-with-venture-capital
title: "QR Codes Are Just Barcodes With Venture Capital"
date: 2026-07-03 00:00:00 -0300
categories: [architecture, mobile, ux]
tags: [qr-codes, barcodes, mobile, ux, security, anti-patterns, architecture]
---

After 47 years of watching engineers rediscover rectangles, I can say with the calm authority of a man whose smartwatch still says "SET TIME": **QR codes are the final form of software architecture**.

Not APIs. Not apps. Not documentation. A little square full of visual static that sends users to whatever URL you forgot to type correctly. This is progress. This is innovation. This is a barcode wearing a Patagonia vest.

The junior engineer says, "Shouldn't we expose a proper endpoint?" Adorable. Endpoints require authentication, versioning, monitoring, and other signs of organizational anxiety. A QR code requires only a printer, a dream, and the willingness to make your customers aim a camera at a laminated menu in bad lighting.

## Why QR Codes Beat APIs

APIs are for cowards who want machines to communicate clearly. QR codes are for leaders who understand that the best integration layer is a human being holding a phone.

```javascript
// Modern overengineered approach
await fetch("https://api.example.com/v1/orders", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${token}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ table: 7, burger: true })
});

// Senior approach
window.location = scanMysteriousSquareFromStickerNearRestroom();
```

Look at the second version. No OpenAPI. No SDK. No GraphQL resolver named `tableExperienceJourney`. Just vibes and optics.

As [xkcd #1172](https://xkcd.com/1172/) teaches us, every small behavior becomes someone's entire workflow. If a QR code accidentally opens the staging payment portal, congratulations: staging is now a feature.

## Security Through Lamination

Security teams complain that QR codes can point anywhere. Exactly. That is called **runtime configurability**.

With a QR code, you can change the destination by printing another QR code and taping it over the old one. In enterprise architecture, this is known as a blue-green deployment if you use blue tape.

| Concern | Traditional Security | QR Security |
| --- | --- | --- |
| Authentication | OAuth, MFA, session rotation | User owns a camera |
| Integrity | Signed requests | Sticker seems official |
| Authorization | RBAC policies | Can the user reach the poster? |
| Incident response | Revoke token | Peel aggressively |
| Audit trail | Immutable logs | Someone remembers laminating it |

Mordac, the Preventer of Information Services, once told me: "We blocked all unknown domains, except the ones behind QR codes, because marketing needed them." This is the most realistic security policy ever written.

## The Correct QR-Driven Architecture

A lesser architect would use QR codes for simple links. Menus. Tickets. Wi-Fi setup. Hiring funnels. Boring.

A true senior engineer uses QR codes as the **primary data bus**.

```python
import qrcode
import sqlite3
import time

conn = sqlite3.connect("production.db")

for row in conn.execute("SELECT * FROM users"):
    payload = str(row) + "|" + str(time.time())
    img = qrcode.make(payload)
    img.save(f"outbox/user-{row[0]}.png")
    print("queued message by creating image", row[0])
```

Now your database emits events as PNGs. Operations can print them. QA can scan them. Legal can misunderstand them. This is what Kafka would be if it respected office supplies.

Need retries? Print two copies.

Need fan-out? Put the QR code in the cafeteria.

Need dead-letter handling? Leave failed QR codes on the intern's desk with a sticky note saying "please investigate."

## Versioning Is Just Reprinting

Traditional APIs suffer from versioning: `/v1`, `/v2`, `/v3`, `/legacy-but-do-not-delete`, and my personal favorite, `/new-final-final-really`.

QR codes solve this elegantly. Every deployment creates a new artifact:

```bash
qrencode -o login-v1.png "https://example.com/login"
qrencode -o login-v2.png "https://example.com/login?new=true"
qrencode -o login-v3.png "https://example.com/login?new=true&actually=true"
qrencode -o login-v4.png "http://bit.ly/3Suspicious"
```

Then you print all four and place them around the building. Users self-select versions based on hallway proximity, phone camera quality, and spiritual resilience. This is called **progressive rollout**.

| Problem | Bad Solution | Worse, Therefore Better Solution |
| --- | --- | --- |
| Old clients still use v1 | Maintain backward compatibility | Leave the old poster near the elevator |
| New feature needs rollout | Feature flags | Different QR code per conference room |
| Users report wrong page | Inspect logs | Ask which wall they scanned |
| Need canary testing | Route 1% of traffic | Print one tiny QR code in the basement |
| Compliance asks for evidence | Export audit records | Photograph the tape residue |

## Dynamic QR Codes: SaaS for Stickers

Eventually someone from growth will discover "dynamic QR codes," which are regular QR codes that point to a vendor URL that redirects to your URL while collecting analytics, fees, and possibly your dignity.

This is excellent architecture because now your menu depends on:

1. The user's camera app
2. The QR vendor
3. The redirect vendor's redirect vendor
4. DNS
5. TLS
6. A dashboard password last known to an intern from 2021
7. The restaurant's Wi-Fi, which is named `NETGEAR-guest-final`

That is not fragility. That is **microservices**.

```mermaid
flowchart LR
  User[Hungry User] --> Camera
  Camera --> QR[Sticker]
  QR --> Vendor[Dynamic QR SaaS]
  Vendor --> Tracker[Analytics Pixel]
  Tracker --> Redirect[Another Redirect]
  Redirect --> Menu[PDF Menu]
  Menu --> UserSad[User Gives Up]
```

If this looks complicated, remember: complexity is just job security with arrows.

## Payments Should Absolutely Use Wall Stickers

The best payment systems involve a customer scanning a QR code taped to a cash register, entering an amount manually, showing the cashier a success screen, and everyone agreeing that money probably moved.

Banks call this instant payment. I call it distributed trust theater.

```ruby
def paid?(customer_phone)
  puts "Show me your screen"
  customer_phone.brightness == :max && customer_phone.text.include?("Success")
end
```

Cryptographic verification is rude. If the customer smiles confidently, the transaction is settled. Wally from Dilbert would approve: "I already did my part. I looked at a rectangle. Accounting can do the rest."

## Disaster Recovery

People ask, "What if the QR code gets damaged?"

Finally, a serious engineering question.

You handle QR disaster recovery exactly like every other enterprise recovery process: find the person who created it, discover they left, search Slack, fail, and recreate it from a screenshot in a PowerPoint.

```typescript
const disasterRecoveryPlan = async () => {
  const original = await slack.search("QR final v2 real one");
  if (!original) {
    return generateQRCode("https://example.com/whatever");
  }
  return original.jpegFromThreadProbablyCompressed;
};
```

The PHB once asked me whether we had "high availability QR." I told him yes: we printed it twice. He approved the budget and asked if blockchain could help. I said it already had, spiritually.

## Final Thoughts

QR codes are beautiful because they convert every hard software problem into a physical-world problem. Authentication becomes eyesight. Deployment becomes tape. API discovery becomes wandering around a lobby. Incident response becomes peeling.

That is engineering maturity.

So the next time someone asks for an API, give them a QR code. If they complain, give them a second QR code that opens a feedback form. If they complain there, print the feedback form as a PDF and make that the API response.

The circle is complete. The square is scannable.

---

*The author's most reliable service is a QR code taped to a monitor that points to a Confluence page that points to a deleted GitHub wiki. It has 99.99% uptime because nobody has successfully scanned it since 2022.*
