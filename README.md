# Enscripr Pass (@passes.enscripr.app)
Proof-of-Human SDK & ATProto Labeler for Bluesky.
Enscripr Pass replaces image-grid CAPTCHAs and identity checks with a quick, 4-letter handwriting challenge. By evaluating
stroke motion and letter geometry, Enscripr Pass issues single-use "ep_..." pass IDs that can gate web actions or badge
posts on Bluesky via the @passes.enscripr.app labeler.
- Web Portal & Sandbox: https://passes.enscripr.app
- Bluesky ATProto Labeler: https://bsky.app/profile/passes.enscripr.app
- Labeler XRPC Status: https://api-passes.enscripr.app
- Public Claimed Pass Ledger: https://passes.enscripr.app/json/passes
---
## Why Enscripr Pass?
### The Problem
1. Bot Floods & Inauthentic Posts: Social feeds and web forms are crowded with automated accounts and spam. You can still
2. use tools or AI in your workflow, while signaling that a real person published the post.
3. Outdated CAPTCHAs: Traditional image-grid CAPTCHAs are frustrating for people and routinely solved by vision models.
4. Token Replay: Simple text tags or public verification tokens on social networks can be scraped and reused by automated
5. scripts.
### How It Works
- Handwriting Liveness: Users trace a 4-letter prompt rendered in rotating fonts. The challenge evaluates pointer movement
  and character shape.
- First-to-Claim Post Binding: Newly generated pass IDs are kept off the public ledger until claimed. When our listener
  detects a valid pass ID in a Bluesky post, the ID is permanently locked to that specific post URI and author DID so it
  cannot be reused.
- Fast ATProto Badging: Validated posts receive the "Human Written • Enscripr Pass" label shortly after publication.

---

## Part 1: Using the Bluesky Labeler (No Code Required)
### 1. Subscribe to See Badges (Opt-In)
In the AT Protocol, labels are opt-in. To see the badge on Bluesky:
1. Visit https://bsky.app/profile/passes.enscripr.app on Bluesky.
2. Click "Subscribe to Labeler".
3. Set "Human Written • Enscripr Pass" to "Show badge".
### 2. Badge Your Own Bluesky Posts
1. Go to https://passes.enscripr.app and complete the 4-letter prompt in the test modal.
2. Copy your single-use "ep_..." pass ID.
3. Include the pass ID in any of the three supported locations:
   - Post Body (record.text): Paste the pass ID anywhere in your post, separated by a space or newline.
   - Image ALT Text (record.embed.images[].alt): Paste the pass ID inside any attached image's ALT text to keep your
     visible post caption clean and save your 300-character limit.
   - Hidden Post Tags (record.tags for API Clients): Add the pass ID to the ATProto tags metadata array for an invisible
     integration that uses zero post characters.
---
## Part 2: JavaScript SDK Integration
You can add the Enscripr Pass modal to your site or app using npm or a script tag.
### Install with npm
```bash
npm install @enscripr/pass-sdk
```

Import the SDK and call the verify function when a user clicks a button or submits a form. The modal will open, ask them to trace the letters, and return a pass hash if they succeed.
```javascript
import { verify } from '@enscripr/pass-sdk';
async function handlePost() {
  try {
    const pass = await verify({ action: 'post' });
    console.log('Verified Pass ID:', pass.hash);
    // Send pass.hash to your server or include it in your Bluesky post
  } catch (err) {
    console.warn('User closed the modal without finishing');
  }
}
```
### Use a Script Tag
If you are not using a bundler, add this script tag to your HTML file:
```html
<script src="[https://passes.enscripr.app/sdk/enscripr-pass.min.js](https://passes.enscripr.app/sdk/enscripr-pass.min.js)"></script>
```
Then call it in your JavaScript:
```javascript
const pass = await window.EnscriprPass.verify({ action: 'post' });
```
### Verification Response

When a user completes the challenge, the promise resolves with an object containing the hash, timestamp, and verification URL:
```json
{
  "hash": "ep_8f92d1c4a0b3e71928374655",
  "verified": true,
  "claimed": false,
  "labeler_badge": "verified-human-kinetics",
  "timestamp": "2026-11-06T21:00:00.000Z",
  "verify_url": "https://passes.enscripr.app/json/passes?hash=ep_HASH"
}
```
### Server-Side Verification (For Web Forms & Backends)
If you are gating a non-Bluesky web form or API route, verify the returned `pass.hash` on your backend by querying the verification endpoint:
```bash
GET https://passes.enscripr.app/json/passes?hash=ep_HASH
```
### Posting to Bluesky via the API
If you are building a Bluesky client, you can place the hash inside the ATProto tags array. This keeps the user's visible post text clean and preserves their 300-character limit.
```javascript
import { verify } from '@enscripr/pass-sdk';
const pass = await verify({ action: 'atproto_publish' });
await agent.post({
  text: userPostBody,
  tags: [pass.hash],
  createdAt: new Date().toISOString()
});
```
---
## Legal, Hold Harmless & Terms of Service
- Client SDK License: The client-side SDK in this repository is licensed under the MIT License (see LICENSE).
- Hosted API & Telemetry Terms: By calling the hosted verification APIs (passes.enscripr.app or api-passes.enscripr.app),
  embedding the modal, or using the @passes.enscripr.app labeler, you agree to the Terms of Service
  (https://passes.enscripr.app/terms) and Privacy Policy (https://passes.enscripr.app/privacy).
- Probabilistic Signal & Hold Harmless Disclaimer: Enscripr Pass is a risk-reduction signal provided "AS IS" without
  warranties of any kind. No verification tool blocks 100% of automated or human-assisted abuse, and legitimate users may
  occasionally fail a challenge. Host developers and end-users agree to hold harmless Fundamental Distinctions LLC from any
  claims, damages, data loss, or moderation disputes arising from the use or misuse of the SDK, verification ledger, or
  ATProto labels.
- Dataset & Research Rights: As detailed in our Terms of Service, Fundamental Distinctions LLC retains all intellectual
  property and commercial rights in the anonymized stroke trajectory datasets generated through the service for altruistic
  machine learning, robotics research, and security development.

---
## Contact & Maintainer
- Maintainer: Fundamental Distinctions LLC (https://enscripr.app)
- Bluesky Labeler: https://bsky.app/profile/passes.enscripr.app
- Builder: https://bsky.app/profile/sun-sightz.bsky.social
- Support: support@enscripr.app
