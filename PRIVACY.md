# Privacy & Encryption in ChartID

## Overview

Birth dates and times are personally identifiable information (PII). ChartID provides optional tools for privacy-conscious users. No privacy mechanism is required — but for sensitive use cases, this guide explains what's available.

---

## Visibility Flags

The `visibility` field on `space.chartid.natalChart` records signals intended access:

| Value | Meaning |
|-------|---------|
| `public` | Anyone on ATProto can read the chart |
| `private` | Intended for the DID owner only; encrypt `birthDatetime` as JWE for stronger protection |
| `friends` | Social graph access — future feature, reserved for now |

Note: ATProto records are technically readable by any PDS operator. `visibility: "private"` is a convention; **encryption is required for genuine confidentiality**.

---

## Encrypting Birth Times

For `visibility: "private"` charts, encrypt `birthDatetime` as a JSON Web Encryption (JWE) token and store it in `encryptedData`. Omit the plaintext `birthDatetime` field (or store only the date portion if the time is the sensitive part).

### Encryption (TypeScript / `jose`)

```typescript
import * as jose from 'jose';

// Encrypt birthDatetime with the user's public key (RSA-OAEP + A256GCM)
async function encryptBirthDatetime(
  birthDatetime: string,
  recipientPublicKeyPem: string
): Promise<string> {
  const publicKey = await jose.importSPKI(recipientPublicKeyPem, 'RSA-OAEP-256');

  const payload = JSON.stringify({ birthDatetime });

  const jwe = await new jose.CompactEncrypt(new TextEncoder().encode(payload))
    .setProtectedHeader({ alg: 'RSA-OAEP-256', enc: 'A256GCM' })
    .encrypt(publicKey);

  return jwe;
}

// Usage
const record = {
  $type: 'space.chartid.natalChart',
  schemaVersion: '1.0.0',
  traditions: ['jyotish'],
  visibility: 'private',
  encryptedData: await encryptBirthDatetime('1992-05-15T14:30:00Z', publicKeyPem),
  birthPlace: { placeName: 'Oakland, CA, USA' },
  createdAt: new Date().toISOString(),
};
```

### Decryption (DID owner only)

```typescript
import * as jose from 'jose';

async function decryptBirthDatetime(
  jwe: string,
  ownerPrivateKeyPem: string
): Promise<string> {
  const privateKey = await jose.importPKCS8(ownerPrivateKeyPem, 'RSA-OAEP-256');

  const { plaintext } = await jose.compactDecrypt(jwe, privateKey);
  const { birthDatetime } = JSON.parse(new TextDecoder().decode(plaintext));

  return birthDatetime;
}
```

### Key management notes

- Store private keys in secure enclave / OS keychain, never in your PDS record
- Consider using the user's ATProto signing key (if exposed) or a separate encryption keypair
- Apps reading `encryptedData` should gracefully handle decryption failure (wrong key, revoked access)

---

## Birth Time Trust vs. Encryption

These are separate concerns:

| Field | Purpose |
|-------|---------|
| `birthTimeTrust` | Epistemological — how confident is the birth time? |
| `timeConfidence` | Same as above, with `rectified` for derived times |
| `visibility` | Access control convention |
| `encryptedData` | Actual confidentiality via cryptography |

A chart can be `birthTimeTrust: "exact"` and `visibility: "private"` — exact time from a birth certificate that the user doesn't want to share publicly.

---

## Data Retention & Deletion

- You own your PDS — records exist only where you or your PDS host them
- Delete records anytime via `com.atproto.repo.deleteRecord`
- Records are cryptographically signed; once deleted from your repo, they are gone (no central server retains copies)
- AppViews and indexers may cache records — ATProto's firehose makes true erasure eventual, not instant

---

## Compliance Notes

| Regulation | Consideration |
|-----------|--------------|
| GDPR | Publicly posted astrological data is not inherently PII unless linked to a name or identifier. If `subjectDid` is set, that creates linkage. Users can delete records (right to erasure). |
| CCPA | Users control their own PDS; they are the data controller for self-posted records. |
| COPPA | Consider parental consent before posting birth data for minors. ChartID does not define age-gating — that's an app responsibility. |

---

## Reporting Privacy Issues

Open a GitHub issue at [schwentker/chartid](https://github.com/schwentker/chartid) with the label `privacy`. For sensitive disclosures, contact the steward directly via ATProto DM at `@schwentker.sandboxlabs.ai`.
