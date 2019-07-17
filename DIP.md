# DATA INTEGRITY PROTOCOL (DIP)
> A simple and flexible method to store a fingerprint with the data.

Author: Gal Buki (torusJKL)

Built on top and leveraging Bitcoin Data Protocol by Unwriter at https://b.bitdb.network/

## Intro
The design goals:

1. A simple protocol to create a fingerprint of arbitrary OP_RETURN data in a single transaction
2. allow to search for fingerprints when querying a planaria provider that does not calculate them on the server side
3. Allow multiple fingerprints to be layered on top to provide the possibility to choose security over speed

> This protocol does not guarentee that the fingerprint is correct.
> To ensure that the data and fingerprint can be trusted this protocol would need to be used together with HAIP

## Use Cases
- Provide a way to search for content by its hash
- Check data integrity

As an example we could store a hash and a CRC in order to search for it by hash and verify that the data we received is not corupted using the CRC which is faster than calculating a hash.

## Protocol
- The prefix for DATA INTEGRITY PROTOCOL is `1D1PdbxVxcjfovTATC3ginxjj4enTgxLyY`

Here's an example of what a **B transactions with FIP** looks like:
```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut
  [Data]
  [Media Type]
  [Encoding]
  [Filename]
  |
  1D1PdbxVxcjfovTATC3ginxjj4enTgxLyY
  [Fingerprint Algorithm]
  [Fingerprint]
  [Index Unit Size] // 0x00 means implicit index and assumes that everything before FIP is included (including | as 0x7c)
                    // 0x01 means index fields are stored as 1 byte each
                    // 0x02 means index fields are stored as word (2 bytes) in little endian order
  [Field Index[]]  // Optional. 0 based index means the OP_RETURN (0x6a) is signed itself
                   // with 1 byte index size 0x00010a would describe that we hash data from field 0, 1, and 10
```

An example with signing [B:// Bitcoin Data](https://github.com/unwriter/B) is shown, however any arbitrary OP_RETURN content can be signed provided that the fields being signed are before the DATA INTEGRITY PROTOCOL `1D1PdbxVxcjfovTATC3ginxjj4enTgxLyY` prefix.

We use the [Bitcom](https://bitcom.bitdb.network) convention to use the pipe '|' to indicate the protocol boundary.

Fields:
1. **Integrity Check Algorithm:** CRC16CCITT - Fingerprint of the concatenated data in raw hex format without the pushdata values. (could also be a hash like SHA256)
2. **Checksum:** The fingerprint of the signed content with the Signing Address. Base64 encoding.
5. **Index Unit Size** The Size of each index. 0x00 means no index will follow and it is assumed that all fields to the left of the DATA INTEGRITY PROTOCOL prefix are signed (starting with the OP_RETURN 6a)
5. **Field Index[]:** (Optional if Index Unit Size is 0x00) The specific index (relative to Field Offset) that is covered by the Signature.  Non-negative integer hex encoding. When the index value more then 1 byte then the hex is in little endian order.

## Examples
### explicit field index
If we want to only hash part of the message we can specify the segment.
In this example we will only hash the content.

```
OP_RETURN
  19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut  // B Prefix
  { "message": "Hello world!" }       // Content
  application/json                    // Content Type
  UTF-8                               // Encoding
  hello.json                          // File name (0x00 if empty/null)
  |                                   // Pipe to seperate protocols
  1D1PdbxVxcjfovTATC3ginxjj4enTgxLyY  // DATA INTEGRITY PROTOCOL prefix
  CRC16_KERMIT                        // Integrity Check Algorithm
  0x3ae4                              // Checksum
  0x01                                // 1 byte is used per index
  0x02                                // Explicit definition of field 2
];

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
  '0x314431506462785678636a666f76544154433367696e786a6a34656e5467784c7959',
  '0x43524331365f4b45524d4954',
  '0x3ae4',
  '0x01'
  '0x02'
]
```

## Transaction Examples
### 1 hash
#### explicit definition of field 2
Transaction:
https://whatsonchain.com/tx/3bb160858f20e83128b5f9aa58702ebb26361dbf76150ad0b88cd302e72e70f1

File:
https://static.bitcoinfiles.org/3bb160858f20e83128b5f9aa58702ebb26361dbf76150ad0b88cd302e72e70f1
