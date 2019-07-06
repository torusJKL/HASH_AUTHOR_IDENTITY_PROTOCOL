# HASH AUTHOR IDENTITY PROTOCOL (HAIP)
> A simple and flexible method to sign arbitrary OP_RETURN data with Bitcoin ECDSA signatures.

> Based on the AUTHOR IDENTITY PROTOCOL (AIP) by Attila Aros and Satchmo (https://github.com/BitcoinFiles/AUTHOR_IDENTITY_PROTOCOL)

Author: Gal Buki (torusJKL)

Special thanks to the authors of AIP for creating a well designed protocol.
Inspired by techniques described by Monkeylord at https://github.com/monkeylord/bitcoin-ibe
Built on top and leveraging Bitcoin Data Protocol by Unwriter at https://b.bitdb.network/

# Intro

The design goals:

1. A simple protocol to sign arbitrary OP_RETURN data in a single transaction
2. Decouple the signing with an address from the funding source address (ie: does not require any on-chain transactions from the signing identity address)
3. Allow multiple signatures to be layered on top to provide multi-party contracts.
4. Keep the data that needs to be signed small so that devices with limited capacity can be used (e.g. hardware wallets)

# Use Cases

- Prove ownership and authoring of any file
- Add multiple signatures to form agreements and contracts
- Decouple identity from funding addresses

The last point of being able to decouple identity from the funding source addresses means that we can now upload files and content and not have to expose our identity with an on-chain payment transaction.

An example is being able to upload a blog post and using Money Button to pay for the mining fees, yet never exposing your Identity key with an on-chain payment.

# Protocol

- The prefix for AUTHOR IDENTITY Protocol is `1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV`

Here's an example of what a **B transactions with HAIP** look like:

```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut
  [Data]
  [Media Type]
  [Encoding]
  [Filename]
  |
  1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV
  [Hashing Algorithm]
  [Signing Algorithm]
  [Signing Address]
  [Signature]
  [Field Index 0] // Optional. 0 based index means the OP_RETURN (0x6a) is signed itself
  [Field Index 1] // Optional.
  ...             // If the Field Indexes are omitted, then it's assumed that all fields to the left of the AUTHOR_IDENTITY prefix are signed.
```

An example with signing [B:// Bitcoin Data](https://github.com/unwriter/B) is shown, however any arbitrary OP_RETURN content can be signed provided that the fields being signed are before the HASHED AUTHOR IDENTITY `1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV` prefix.

We use the [Bitcom](https://bitcom.bitdb.network) convention to use the pipe '|' to indicate the protocol boundary.

Fields:

1. **Signing Algorithm:** SHA256 - Hash of the hex message in ASM format. UTF-8 encoding.
2. **Signing Algorithm:** ECDSA - This is the default Bitcoin signing algorithm built into bsv.js. UTF-8 encoding.
3. **Signing Address:** Bitcoin Address that is used to sign the content. UTF-8 encoding.
4. **Signature:** The signature of the signed content with the Signing Address. Base64 encoding.
5. **Field Index <index>:** (Optional) The specific index (relative to Field Offset) that is covered by the Signature.  Non-negative integer hex encoding. When there are no indexes provided, it is assumed that all fields to the left of the AUTHOR IDENTITY prefix are signed (starting with the OP_RETURN 6a).

Example:

```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut  // B Prefix
  { "message": "Hello world!" }       // Content
  application/json                    // Content Type
  UTF-8                               // Encoding
  hello.json                          // File name (0x00 if empty/null)
  |                                   // Pipe to seperate protocols
  1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV  // HASHED AUTHOR IDENTITY prefix
  SHA256                              // Hashing Algorithm
  BITCOIN_ECDSA                       // Signing Algorithm
  1Ghayxcf8askMqL9EV9V9QpExTR2j6afhv  // Signing Address
  H776s+VMBJp9AIN7dg2cNEDCZRPTrpAG4tNTxE5Ol2JZFOz+xhmbIOSWvMyD+kJsF5YnP/xy0sZCqm5d03sniY0=  // Signature
  0,  // OP_RETURN 6a
  1,  // 19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut
  2,  // { "message": "Hello world!" }
  3,  // application/json
  4,  // UTF-8
  5,  // 0x00
  6   // |
];

```

In ASM form:
```
6a 31394878696756345179427633744870515663554551797131707a5a56646f417574 7b20226d657373616765223a202248656c6c6f20776f726c642122207d 6170706c69636174696f6e2f6a736f6e 5554462d38 68656c6c6f2e6a736f6e 7c 314841315063753647354232505a55373575473852786b325a4d4e64516947586856 534841323536 424954434f494e5f4543445341 31476861797863663861736b4d714c394556395639517045785452326a3661666876 48373736732b564d424a703941494e37646732634e4544435a5250547270414734744e547845354f6c324a5a464f7a2b78686d62494f5357764d79442b6b4a734635596e502f787930735a43716d35643033736e6959303d 00 01 02 03 04 05 06
```

In Hex form:
```
[
  '0x6a', /// OP_RETURN
  '0x31394878696756345179427633744870515663554551797131707a5a56646f417574',
  '0x7b20226d657373616765223a202248656c6c6f20776f726c642122207d',
  '0x6170706c69636174696f6e2f6a736f6e',
  '0x5554462d38',
  '0x68656c6c6f2e6a736f6e',
  '0x7c',
  '0x314841315063753647354232505a55373575473852786b325a4d4e64516947586856',
  '0x534841323536',
  '0x424954434f494e5f4543445341',
  '0x31476861797863663861736b4d714c394556395639517045785452326a3661666876',
  '0x48373736732b564d424a703941494e37646732634e4544435a5250547270414734744e547845354f6c324a5a464f7a2b78686d62494f5357764d79442b6b4a734635596e502f787930735a43716d35643033736e6959303d',
  '0x00',
  '0x01',
  '0x02',
  '0x03',
  '0x04',
  '0x05',
  '0x06'
]
```

The hash is not part of the message and needs to be created from the ASM string.
The calculation of the hash is as follow:
`SHA256(6a 31394878696756345179427633744870515663554551797131707a5a56646f417574 7b20226d657373616765223a202248656c6c6f20776f726c642122207d 6170706c69636174696f6e2f6a736f6e 5554462d38 68656c6c6f2e6a736f6e 7c)` which equals to `a0606320b3fd24053c6bfd890d9add4a3e722f0dd2c7ea270dea7666a8dbf01a`.

The bsv.js library can be used to create the ASM string.
Restrictions
- OP_RETURN needs to be replaced with 6a
- index in hex (eg 0x00 needs to be added manually)

``
bsv.Script.buildDataOut(['19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut','{ "message": "Hello world!" }', 'application/json', 'UTF-8', 'hello.json', '|', '1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV', 'SHA256', 'BITCOIN_ECDSA', '1Ghayxcf8askMqL9EV9V9QpExTR2j6afhv', 'H776s+VMBJp9AIN7dg2cNEDCZRPTrpAG4tNTxE5Ol2JZFOz+xhmbIOSWvMyD+kJsF5YnP/xy0sZCqm5d03sniY0=']).toASM()
``

# Transaction Examples

##### 1 signature

File:
https://static.bitcoinfiles.org/bc9ed5a2995df4d34bcbc187cb8fae209818c0e0d4d6eb005f74656eaaeabd72


Transaction:
https://whatsonchain.com/tx/bc9ed5a2995df4d34bcbc187cb8fae209818c0e0d4d6eb005f74656eaaeabd72
