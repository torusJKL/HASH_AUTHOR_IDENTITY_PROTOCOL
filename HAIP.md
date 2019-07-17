# HASH AUTHOR IDENTITY PROTOCOL (HAIP)
> A simple and flexible method to sign arbitrary OP_RETURN data with Bitcoin ECDSA signatures.

> Based on the AUTHOR IDENTITY PROTOCOL (AIP) by Attila Aros and Satchmo (https://github.com/BitcoinFiles/AUTHOR_IDENTITY_PROTOCOL)

Author: Gal Buki (torusJKL)

Special thanks to the authors of AIP for creating a well designed protocol.
Inspired by techniques described by Monkeylord at https://github.com/monkeylord/bitcoin-ibe
Built on top and leveraging Bitcoin Data Protocol by Unwriter at https://b.bitdb.network/

> This document describes the second iteration of HAIP.
> The first iteration with prefix 1HA1Pcu6G5B2PZU75uG8Rxk2ZMNdQiGXhV is depricated

## Intro
The design goals:

1. A simple protocol to sign arbitrary OP_RETURN data in a single transaction
2. Decouple the signing with an address from the funding source address (ie: does not require any on-chain transactions from the signing identity address)
3. Allow multiple signatures to be layered on top to provide multi-party contracts.
4. Keep the data that needs to be signed small so that devices with limited capacity can be used (e.g. hardware wallets)

## Use Cases
- Prove ownership and authoring of any file
- Add multiple signatures to form agreements and contracts
- Decouple identity from funding addresses

The last point of being able to decouple identity from the funding source addresses means that we can now upload files and content and not have to expose our identity with an on-chain payment transaction.

An example is being able to upload a blog post and using Money Button to pay for the mining fees, yet never exposing your Identity key with an on-chain payment.

## Protocol
- The prefix for AUTHOR IDENTITY Protocol is `1HA1P2exomAwCUycZHr8WeyFoy5vuQASE3`

Here's an example of what a **B transactions with HAIP** look like:
```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut
  [Data]
  [Media Type]
  [Encoding]
  [Filename]
  |
  1HA1P2exomAwCUycZHr8WeyFoy5vuQASE3
  [Hashing Algorithm]
  [Signing Algorithm]
  [Signing Address]
  [Signature]
  [Index Unit Size] // 0x00 means implicit index and assumes that everything before HAIP is signed (including | as 0x7c)
                    // 0x01 means index fields are stored as 1 byte each
                    // 0x02 means index fields are stored as word (2 bytes) in little endian order
  [Field Index[]]  // Optional. 0 based index means the OP_RETURN (0x6a) is signed itself
                   // with 1 byte index size 0x00010a would describe that we hash data from field 0, 1, and 10
```

An example with signing [B:// Bitcoin Data](https://github.com/unwriter/B) is shown, however any arbitrary OP_RETURN content can be signed provided that the fields being signed are before the HASHED AUTHOR IDENTITY `1HA1P2exomAwCUycZHr8WeyFoy5vuQASE3` prefix.

We use the [Bitcom](https://bitcom.bitdb.network) convention to use the pipe '|' to indicate the protocol boundary.

Fields:
1. **Hashing Algorithm:** SHA256 - Hash of the hex message in ASM format.
2. **Signing Algorithm:** ECDSA - This is the default Bitcoin signing algorithm built into bsv.js. UTF-8 encoding.
3. **Signing Address:** Bitcoin Address that is used to sign the content. UTF-8 encoding.
4. **Signature:** The signature of the signed content with the Signing Address. Base64 encoding.
5. **Index Unit Size** The Size of each index. 0x00 means no index will follow and it is assumed that all fields to the left of the HASH AUTHOR IDENTITY prefix are signed (starting with the OP_RETURN 6a)
5. **Field Index[]:** (Optional if Index Unit Size is 0x00) The specific index (relative to Field Offset) that is covered by the Signature.  Non-negative integer hex encoding. When the index value more then 1 byte then the hex is in little endian order.

## Examples
### 1 signature, implicit field index
```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut  // B Prefix
  { "message": "Hello world!" }       // Content
  application/json                    // Content Type
  UTF-8                               // Encoding
  hello.json                          // File name (0x00 if empty/null)
  |                                   // Pipe to seperate protocols
  1HA1P2exomAwCUycZHr8WeyFoy5vuQASE3  // HASHED AUTHOR IDENTITY prefix
  SHA256                              // Hashing Algorithm
  BITCOIN_ECDSA                       // Signing Algorithm
  1Ghayxcf8askMqL9EV9V9QpExTR2j6afhv  // Signing Address
  H6Y5LXIZRaSQ0CJEt5eY1tbUhKTxII31MZwSpEYv5fqmZLzwuylAwrtHiI3lk3yCqf3Ib/Uv3LpAfCoNSKk68fY=  // Signature
  0x00
];

```

In raw hex form:
```
0x6a2231394878696756345179427633744870515663554551797131707a5a56646f4175741d7b20226d657373616765223a202248656c6c6f20776f726c642122207d106170706c69636174696f6e2f6a736f6e055554462d380a68656c6c6f2e6a736f6e017c2231484131503265786f6d4177435579635a487238576579466f793576755141534533065348413235360d424954434f494e5f45434453412231476861797863663861736b4d714c394556395639517045785452326a36616668764c58483659354c58495a5261535130434a457435655931746255684b5478494933314d5a7753704559763566716d5a4c7a7775796c41777274486949336c6b33794371663349622f5576334c704166436f4e534b6b363866593d0100
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
  '0x31484131503265786f6d4177435579635a487238576579466f793576755141534533',
  '0x534841323536',
  '0x424954434f494e5f4543445341',
  '0x31476861797863663861736b4d714c394556395639517045785452326a3661666876',
  '0x483659354c58495a5261535130434a457435655931746255684b5478494933314d5a7753704559763566716d5a4c7a7775796c41777274486949336c6b33794371663349622f5576334c704166436f4e534b6b363866593d',
  '0x00'
]
```

The hash is not part of the message and needs to be created from the raw hex string.
The calculation of the hash is as follow:
`SHA256(6a2231394878696756345179427633744870515663554551797131707a5a56646f4175741d7b20226d657373616765223a202248656c6c6f20776f726c642122207d106170706c69636174696f6e2f6a736f6e055554462d380a68656c6c6f2e6a736f6e017c)` which equals to `733c8e0d921a72e1e650c16820cbcca53aac42ba98c3b27635c2dc978e11fc7f`.

### 1 signature, explicit field index
```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut  // B Prefix
  { "message": "Hello world!" }       // Content
  application/json                    // Content Type
  UTF-8                               // Encoding
  hello.json                          // File name (0x00 if empty/null)
  |                                   // Pipe to seperate protocols
  1HA1P2exomAwCUycZHr8WeyFoy5vuQASE3  // HASHED AUTHOR IDENTITY prefix
  SHA256                              // Hashing Algorithm
  BITCOIN_ECDSA                       // Signing Algorithm
  1Ghayxcf8askMqL9EV9V9QpExTR2j6afhv  // Signing Address
  IFh8jXM4pKa5W0GFZ3aE1PYer8Wwynv76OIyqslBXXj+MYxG52nIV1mVfOSgR/5ozTKOCjHHhmVyx/6EZ7q+NBs=  // Signature
  0x01                                // 1 byte is used per index
  0x02030405                          // Explicit definition of fields 2 - 5 (including)
];

```

In raw hex form:
```
0x6a2231394878696756345179427633744870515663554551797131707a5a56646f4175741d7b20226d657373616765223a202248656c6c6f20776f726c642122207d106170706c69636174696f6e2f6a736f6e055554462d380a68656c6c6f2e6a736f6e017c2231484131503265786f6d4177435579635a487238576579466f793576755141534533065348413235360d424954434f494e5f45434453412231476861797863663861736b4d714c394556395639517045785452326a36616668764c58494668386a584d34704b6135573047465a3361453150596572385777796e7637364f497971736c4258586a2b4d59784735326e4956316d56664f5367522f356f7a544b4f436a4848686d5679782f36455a37712b4e42733d01010402030405
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
  '0x31484131503265786f6d4177435579635a487238576579466f793576755141534533',
  '0x534841323536',
  '0x424954434f494e5f4543445341',
  '0x31476861797863663861736b4d714c394556395639517045785452326a3661666876',
  '0x494668386a584d34704b6135573047465a3361453150596572385777796e7637364f497971736c4258586a2b4d59784735326e4956316d56664f5367522f356f7a544b4f436a4848686d5679782f36455a37712b4e42733d',
  '0x01'
  '0x02030405'
]
```

The hash is not part of the message and needs to be created from the raw hex string.
The calculation of the hash is as follow:
`SHA256(1d7b20226d657373616765223a202248656c6c6f20776f726c642122207d106170706c69636174696f6e2f6a736f6e055554462d380a68656c6c6f2e6a736f6e)` which equals to `5fd6576f1b935089fa7799dabc05a40288e5aa2ea9215bce94e9fec4b143c633`.

## Transaction Examples
### 1 signature
#### implicit definition of signed fields
Transaction:
https://whatsonchain.com/tx/acc01b0415557e55b84c516f258946ef6186a45b8ef073650f97b30dc413a0f2

File:
https://static.bitcoinfiles.org/acc01b0415557e55b84c516f258946ef6186a45b8ef073650f97b30dc413a0f2

#### signing of fields 2, 3, 4, 5
Transaction:
https://whatsonchain.com/tx/d500c4be90f6b66d37544553b86b75d9681e3529c648982c84e3e092e37c5be3

File:
https://static.bitcoinfiles.org/d500c4be90f6b66d37544553b86b75d9681e3529c648982c84e3e092e37c5be3
