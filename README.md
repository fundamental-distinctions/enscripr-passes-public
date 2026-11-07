# Enscripr Pass (@passes.enscripr.app)

Proof-of-Human SDK & ATProto Labeler for Bluesky.

Enscripr Pass replaces image-grid CAPTCHAs and identity checks with a quick, 4-letter handwriting challenge. By evaluating stroke motion and letter geometry, Enscripr Pass issues single-use "ep_..." pass IDs that can gate web actions or badge posts on Bluesky via the @passes.enscripr.app labeler.

- Web Portal & Sandbox: https://passes.enscripr.app
- Bluesky ATProto Labeler: https://bsky.app/profile/passes.enscripr.app
- Labeler XRPC Status: https://api-passes.enscripr.app
- Public Claimed Pass Ledger: https://passes.enscripr.app/json/passes

---

## Why Enscripr Pass?

### The Problem
1. Bot Floods & Inauthentic Posts: Social feeds and web forms are crowded with automated accounts and spam. You can still use tools or AI in your workflow, while signaling that a real person published the post.
2. Outdated CAPTCHAs: Traditional image-grid CAPTCHAs are frustrating for people and routinely solved by vision models.
3. Token Replay: Simple text tags or public verification tokens on social networks can be scraped and reused by automated scripts.

### How It Works
- Handwriting Liveness: Users trace a 4-letter prompt rendered in rotating fonts. The challenge evaluates pointer movement and character shape.
- First-to-Claim Post Binding (Firehose Indexing): Newly generated pass IDs are kept off the public ledger until claimed. The moment our real-time Bluesky firehose listener detects a valid pass ID in a new post, the ID is permanently locked to that specific post URI and author DID so it cannot be reused.
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
   - Image ALT Text (record.embed.images[].alt): Paste the pass ID inside any attached image's ALT text to keep your visible post caption clean and save your 300-character limit.
   - Hidden Post Tags (record.tags for API Clients): Add the pass ID to the ATProto tags metadata array for an invisible integration that uses zero post characters.

---

## Part 2: JavaScript SDK Integration

You can add the Enscripr Pass modal to your site or ATProto client using npm or a script tag.

### Install with npm

```bash
npm install @enscripr/pass-sdk
```

Import the SDK and call `verify()` before publishing or submitting a form:

```javascript
import { verify } from '@enscripr/pass-sdk';

async function handlePublish() {
  try {
    const pass = await verify({ action: 'post' });
    console.log('Verified Pass ID:', pass.hash);
    
    // Attach pass.hash to your Bluesky post or send it to your backend
  } catch (err) {
    console.warn('User closed the modal without finishing:', err.message);
  }
}
```

### Use a Script Tag

If you are not using a bundler, add the script tag to your HTML file:

```html
<script src="[https://passes.enscripr.app/sdk/enscripr-pass.min.js](https://passes.enscripr.app/sdk/enscripr-pass.min.js)"></script>
```

Then call it in your JavaScript:

```javascript
const pass = await window.EnscriprPass.verify({ action: 'post' });
```

### Verification Response

When a user completes the challenge, `verify()` resolves with an unclaimed pass object in the user's browser:

```json
{
  "hash": "ep_78cc1c834a0873231a39edf1",
  "verified": true,
  "claimed": false,
  "labeler_badge": "verified-human-kinetics",
  "timestamp": "2026-11-07T03:17:09.026Z",
  "verify_url": "[https://passes.enscripr.app/json/passes?hash=ep_78cc1c834a0873231a39edf1](https://passes.enscripr.app/json/passes?hash=ep_78cc1c834a0873231a39edf1)"
}
```

### Workflow A: Bluesky & ATProto Clients (Zero Backend Required)

If you are building a Bluesky client or publishing tool, you do not need to write any backend verification or anti-replay logic. Simply include `pass.hash` in the post's `tags` array (or `alt` text / post body).

Our real-time Jetstream firehose listener automatically detects the `ep_...` hash as soon as the post is published on Bluesky, verifies that it is unclaimed (`claimed: false`), locks it permanently to that post URI (`claimed: true`), and broadcasts the signed ATProto label.

```javascript
import { verify } from '@enscripr/pass-sdk';

const pass = await verify({ action: 'atproto_publish' });

await agent.post({
  text: userPostBody,
  tags: [pass.hash],
  createdAt: new Date().toISOString()
});
```

### Workflow B: Non-Bluesky Web Forms (Server-Side Check)

If you are gating a standard web form or API route outside of Bluesky, send `pass.hash` to your backend and query the lookup endpoint:

```bash
GET [https://passes.enscripr.app/json/passes?hash=ep_78cc1c834a0873231a39edf1](https://passes.enscripr.app/json/passes?hash=ep_78cc1c834a0873231a39edf1)
```

Confirm that `"verified": true` in the JSON response and record the `hash` in your own database so the same pass ID cannot be submitted twice to your form.

### SDK Configuration & Technical Notes

- No API Keys or Domain Whitelisting Required: The current v1 release is open-access and does not require registering a site key or client ID. Abuse and quota limits are managed automatically via edge rate-limiting.
- Mounting Behavior: Calling `verify()` automatically mounts an isolated, closed Shadow DOM full-screen modal overlay (`#enscripr-pass-host`, `z-index: 2147483647`) directly to `document.body` and removes it upon completion. Inline `<div>` mounting is not supported in v1.
- Fixed Light Theme: The current implementation does not support custom theming, dark mode overrides, or custom CSS injection. The canvas uses a fixed high-contrast light surface to keep guide letters consistent across devices.
- Error & Rejection Handling: Low accuracy scores (`<80%` per letter) and network retries are handled interactively inside the modal UI without rejecting the promise. The `verify()` promise only rejects (`new Error('Verification cancelled by user')`) if the user explicitly closes the modal before passing.
- Version Pinning vs. CDN Script: The hosted `<script>` tag at `passes.enscripr.app/sdk/enscripr-pass.min.js` always serves the latest v1 build. Applications requiring strict version pinning and lockfile integrity should install `@enscripr/pass-sdk` via npm.

---

## Legal, Hold Harmless & Terms of Service

- Client SDK License: The client-side SDK in this repository is licensed under the MIT License (see LICENSE).
- Hosted API & Telemetry Terms: By calling the hosted verification APIs (passes.enscripr.app or api-passes.enscripr.app), embedding the modal, or using the @passes.enscripr.app labeler, you agree to the Terms of Service (https://passes.enscripr.app/terms) and Privacy Policy (https://passes.enscripr.app/privacy).
- Probabilistic Signal & Hold Harmless Disclaimer: Enscripr Pass is a risk-reduction signal provided "AS IS" without warranties of any kind. No verification tool blocks 100% of automated or human-assisted abuse, and legitimate users may occasionally fail a challenge. Host developers and end-users agree to hold harmless Fundamental Distinctions LLC from any claims, damages, data loss, or moderation disputes arising from the use or misuse of the SDK, verification ledger, or ATProto labels.
- Dataset & Research Rights: As detailed in our Terms of Service, Fundamental Distinctions LLC retains all intellectual property and commercial rights in the anonymized stroke trajectory datasets generated through the service for machine learning, robotics research, and security development.

---

## Contact & Maintainer

- Maintainer: Fundamental Distinctions LLC (https://enscripr.app)
- Bluesky Labeler: https://bsky.app/profile/passes.enscripr.app
- Builder: https://bsky.app/profile/sun-sightz.bsky.social
- Support: support@enscripr.app

