# keysplit

Simple, zero-dependency TypeScript implementation of [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) by Onoal.

Uses GF(2^8) and operates on `Uint8Array`s. Inspired by [hashicorp/vault](https://github.com/hashicorp/vault/tree/main/shamir).

## Install

```bash
npm install keysplit
```

## Quick start

```typescript
import {split, combine} from 'keysplit';

const encoder = new TextEncoder();
const secret = encoder.encode('hello world');

const [a, b, c] = await split(secret, 3, 2);
const reconstructed = await combine([a, c]);

console.log(Buffer.from(reconstructed).toString('hex'));
```

### Node (CommonJS)

```js
const {split, combine} = require('keysplit');
```

### Browser

- Modern browsers supported. Uses `crypto.getRandomValues` for CSPRNG.
- Bundle with your favorite tool (Vite, Webpack, Rollup). No polyfills required.

## API

This package exposes two async functions.

### split(secret, shares, threshold): Promise<Uint8Array[]>

```ts
declare function split(
  secret: Uint8Array,
  shares: number,
  threshold: number,
): Promise<Uint8Array[]>;
```

- `secret`: `Uint8Array` to split. Must be non-empty.
- `shares`: total number of shares, 2 ≤ n ≤ 255.
- `threshold`: minimum shares to reconstruct, 2 ≤ t ≤ 255 and t ≤ n.
- Returns: array of `shares` `Uint8Array`s. Each share is `secret.length + 1` bytes.

### combine(shares): Promise<Uint8Array>

```ts
declare function combine(shares: Uint8Array[]): Promise<Uint8Array>;
```

- `shares`: array of shares, length 2 ≤ m ≤ 255. All must be the same length, ≥ 2 bytes, and last byte of each must be unique.
- Returns: reconstructed `Uint8Array`.

## Examples

### Split browser input

```ts
import {split, combine} from 'keysplit';

const toBytes = (s: string) => new TextEncoder().encode(s);

const input = (document.querySelector('input#secret') as HTMLInputElement).value.normalize('NFKC');
const secret = toBytes(input);
const [s1, s2, s3] = await split(secret, 3, 2);
const back = await combine([s1, s3]);
```

### Split random entropy

```ts
const entropy = crypto.getRandomValues(new Uint8Array(16));
const [e1, e2, e3] = await split(entropy, 3, 2);
const back = await combine([e2, e3]);
```

### Split a symmetric key (Web Crypto)

```ts
const key = await crypto.subtle.generateKey({name: 'AES-GCM', length: 256}, true, [
  'encrypt',
  'decrypt',
]);
const buf = new Uint8Array(await crypto.subtle.exportKey('raw', key));
const [k1, k2, k3] = await split(buf, 3, 2);
const back = await combine([k2, k1]);
```

## Environment notes

- Node: uses `node:crypto` for CSPRNG.
- Browser: uses `window.crypto.getRandomValues`.
- ESM and CJS builds are provided via package exports.

## Security considerations

The underlying approach has been independently audited in prior work (see reports below). This fork preserves the core algorithm and constraints.

1. Side-channel resistance: JavaScript environments cannot guarantee true constant time. We aim for algorithmic constant time where possible.
2. Integrity: Incorrect or corrupted shares will yield incorrect reconstruction. Verify integrity at a higher layer.
3. Entropy: Prefer uniformly random secrets. If not, encrypt first and split the key.

## Errors

- `TypeError('secret must be a Uint8Array')`
- `Error('secret cannot be empty')`
- `TypeError('shares must be a number')`
- `RangeError('shares must be at least 2 and at most 255')`
- `TypeError('threshold must be a number')`
- `RangeError('threshold must be at least 2 and at most 255')`
- `Error('shares cannot be less than threshold')`
- `TypeError('shares must be an Array')`
- `TypeError('shares must have at least 2 and at most 255 elements')`
- `TypeError('each share must be a Uint8Array')`
- `TypeError('each share must be at least 2 bytes')`
- `TypeError('all shares must have the same byte length')`
- `Error('shares must contain unique values but a duplicate was found')`

## Contributing

Please [open an issue](https://github.com/onoal/Onoal-KeySplit/issues/new) with as much detail as possible for bugs and feature requests.

## License

Apache-2.0. See the [license file](LICENSE).

Made with ❤️ by Onoal.
