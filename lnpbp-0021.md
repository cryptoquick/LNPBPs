```
LNPBP: 0021
Vertical: Smart contracts
Title: RGB non-fungible assets interface for collectibles (RGB-21)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Hunter Trujillo,
         Federico Tenga,
         Zoe Faltibà,
         Carlos Roldan,
         Olga Ukolova,
         Giacomo Zucco,
         Armando Dutra
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
Updated: 2023-07-24
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract


## Background


## Motivation


## Design

- [x] Media as URI (which can be attachments distributed with Storm or usual URLs)
- [x] Small media ("previews" if given together with large media)
- [x] Fraction of assets
- [x] Engravings
- [x] Reserves
- [x] Possible decentralized issue


## Specification

Interface specification is the following Contractum code:

```haskell
-- Defined by LNPBP-31 standard in `RGBContract.sty` file
import urn:ubideco:stl:6vbr9ZrtsD9aBjo5qRQ36QEZPVucqvRRjKCPqE8yPeJr#choice-little-boxer as RGBContract

-- # Defining main data structures

data ItemsCount :: U32

-- each collectible item is unique and must have an id
data TokenIndex :: U32

data OwnedFraction :: U64

-- allocation of a single token or its fraction to some transaction output
data Allocation :: TokenIndex, OwnedFraction

data EngravingData ::
    appliedTo TokenIndex,
    content EmbeddedMedia

data EmbeddedMedia ::
    type RGBContract.MediaType,
    data [Byte]

data TokenData ::
    index TokenIndex,
    ticker RGBContract.Ticker?,
    name RGBContract.Name?,
    details RGBContract.Details?,
    -- always-embedded preview media < 64kb
    preview EmbeddedMedia?,
    -- external media which is the main media for the token
    media RGBContract.Attachment?,
    attachments { U8 -> ^ ..20 RGBContract.Attachment } -- auxiliary attachments by type (up to 20 attachments)
    -- output containing locked bitcoins; how reserves are proved is a matter
    -- of a specific schema implementation
    reserves RGBContract.ProofOfReserves?

 -- each attachment type is a mapping from attachment id
 -- (used as `Token.attachments` keys) to a short Ascii string
 -- verbally explaining the type of the attachment for the UI
 -- (like "sample" etc).
data AttachmentType ::
    id U8,
    name AttachmentName

data AttachmentName :: [Std.AsciiPrintable ^ 1..20]

interface RGB21
    -- Asset specification containing ticker, name, precision etc.
    global spec :: RGBContract.DivisibleAssetSpec

    -- Contract text and creation date is separated from the spec since it must
    -- not be changeable by the issuer.
    global terms :: RGBContract.RicardianContract
    global created :: RGBContract.Timestamp

    -- Data for all issued tokens
    global tokens* :: TokenData
    global engravings* :: EngravingData
    global attachmentTypes* :: AttachmentType

    -- Right to do a secondary (post-genesis) issue
    public inflationAllowance* :: ItemsCount
    -- Right to update asset name
    public updateRight?

    -- Ownership right over assets
    private assetOwner* :: Allocation

    genesis        -> spec
                    , terms,
                    , created,
                    , tokens*
                    , assetOwner*
                    , inflationAllowance*
                    , updateRight?
                    , attachmentTypes*
                    , reserves {RGBContract.ProofOfReserves ^ 0..0xFFFF}
                   !! invalidProof
                    -- this error happens when amount of token > 1
                    | fractionOverflow
                    | insufficientReserves
                    | invalidAttachmentType

    op Transfer    :: previous assetOwner+,
                   -> beneficiaries assetOwner+
                   !! -- options for operation failure:
                      nonEqualValue
                    | fractionOverflow
                    | nonFractionalToken

    op? Engrave    :: previous assetOwner+
                    , engravings
                   -> beneficiaries assetOwner+
                   !! -- options for operation failure:
                      nonEqualValue
                    | nonFractionalToken
                    | nonEngravableToken

    op? Issue      :: used inflationAllowance
                    , newTokens tokens*
                    , newAttachmentTypes attachmentTypes*
                    , reserves {RGBContract.ProofOfReserves ^ 0..0xFFFF}
                   -> future inflationAllowance
                    , beneficiaries assetOwners+
                   !! invalidProof
                    | fractionOverflow
                    | insufficientReserves
                    | issueExceedsAllowance
                    | invalidAttachmentType

    op? Rename     :: used updateRight
                   -> future updateRight?
                    , new spec
```

## Compatibility


## Rationale


## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
