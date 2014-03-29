Zerocoin
=====================

TODO:

* Update zc_ufo* to use the four ~3800 bit UFOs

* init.cpp#AppInit2: start Zerocoin thread pool (and document in coding.md)


None of the following scripts are actually executed; instead, the ZC opcodes
are recognized and so that input or output is ignored by the regular Anoncoin
transaction processing (this is why they are prefix rather than postfix). This
transaction is marked for Zerocoin processing.

New input type:

* ZC spend: can have a maximum of one in a transaction, to reduce
  implementation complexity.
  Non-rate-limited script format: ZCSPEND <version: VarInt> <isRateLimited: 0> <serialNum: VarInt> <spendRootHash: VarInt>
  Rate-limited script format: ZCSPEND <version: VarInt> <isRateLimited: 1> <serialNum: VarInt> <firstHalfTxHash: VarInt> <firstHalfTxOutIdx: VarInt>

New output types:

* ZC checkpoint: always has an amount of zero.
  Script format: RETURN ZCCHECKPT <version: VarInt> <denom: VarInt> <accum0value: VarInt> .. <accumNvalue: VarInt>.
  These can only occur in the coinbase transaction. A checkpoint output can
  occur at most once for any given denomination, and is required if any ZC mints
  using that denomination occur in the block.

* ZC mint: amount must be one of the denomination amounts.
  Script format: RETURN ZCMINT <version: VarInt> <coinCommitment: VarInt>

* ZC first half: always has an amount of zero.
  Script format: RETURN ZCFIRSTHALF <version: VarInt> <firstHalfHash: VarInt>
  At the time that this transaction is verified, only the length of firstHalfHash
  is checked, since the information required to verify more is not yet
  available.
  firstHalfHash consists of a hash of <spendTxHash> <commitmentToCoinUnderSerialParams> <t1-20hash> <t21-40hash>.
  <t1-20hash> and <t21-40hash> should match the "snSoK t" hashes described in
  the spend root section (below). spendTxHash is produced by taking the binary
  representation of the ZC spend transaction as found in "tx" network messages,
  and replacing the input's spendRootHash with all zero bytes.



### Format of Zerocoin spend roots (hashes are denoted spendRootHash, above)

Contents:

* the block hash containing the accumulator checkpoint

* 4 hashes of accPoKs (one for each UFO)

* hashes (4 if non-rate-limited, 2 if rate-limited) of snSoK s, s': each part
  corresponds to a separate 20 bits of the challenge hash, and thus are
  independently verifiable.

* snSoK t hashes (4 if non-rate-limited, 2 if rate-limited); each part corresponds
  to a separate 20 bits of the challenge hash.

* commitmentPoK

* commitmentToCoinUnderSerialParams

* commitmentToCoinUnderAccParams


### Producing the challenge hash

If rate-limited, the challenge hash is the hash of <block1hash> <block2hash> <firstHalfHash>,
where block1hash is the hash of the block containing firstHalfHash, and
block2hash is the hash of the block following that block. Thus, the ZC spend
txn must occur in block 3 or later.

If non-rate-limited, the challenge hash is the hash of <spendTxHash> <commitmentToCoinUnderSerialParams> <t1-20hash> <t21-40hash> <t41-60hash> <t61-80hash>.