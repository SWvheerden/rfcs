# RFC-0155/TariAddress

## TariAddress specification

![status: stable](theme/images/status-stable.svg)

**Maintainer(s)**:[SW van Heerden](https://github.com/swvheerden)

# Licence

[ The 3-Clause BSD Licence](https://opensource.org/licenses/BSD-3-Clause).

Copyright 2024. The Tari Development Community

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the
following conditions are met:

1. Redistributions of this document must retain the above copyright notice, this list of conditions and the following
   disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following
   disclaimer in the documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products
   derived from this software without specific prior written permission.

THIS DOCUMENT IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS", AND ANY EXPRESS OR IMPLIED WARRANTIES,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be interpreted as described in
[BCP 14](https://tools.ietf.org/html/bcp14) (covering RFC2119 and RFC8174) when, and only when, they appear in all capitals, as
shown here.

## Disclaimer

This document and its content are intended for information purposes only and may be subject to change or update
without notice.

This document may include preliminary concepts that may or may not be in the process of being developed by the Tari
community. The release of this document is intended solely for review and discussion by the community regarding the
technological merits of the potential system outlined herein.

## Goals

This document describes the specification for Tari Address. Tari Addresses are encoded wallet addresses used to verify wallet addresses, features and networks.
This address should have human readable network and features identification. It should contain all information required to send transactions to a wallet owning an address. 

## Related Requests for Comment

None

## Description

Wallet addresses should give all the information necessary to send transactions to the wallet who's address it is. To start of a wallet needs to advertise its features that it supports as
well as what network it on. To keep this human readable, we need independent identifiers for both the features and the network. 

For normal interactive MW transactions it would need a public key to communicate to. The private key of this MUST be kept hidden by the node at all times to ensure that a node cannot be
spoofed. Because this needs to be kept hidden we can use this as a spend key to derive the actual spend key for the UTXO. 

As soon as we want to send non-interactive transactions, things get a bit more complicated. If the receiving wallet is a normal wallet it has access to all the nessasary
information required to spend the transaction. But if the receiving wallet contains a Hardware device such as a ledger, the wallet cannot know the spending information,
this introduces the need for a secondary view key. The wallet can give out the private key of this to another party to view the UTXOs. But this key cannot be used
to spend the UTXO. 

We can also enclude a checksum to detect errors when we encode this as bytes. This address can also be very easily encoded with emojis if we assign an emoji to each u8. 


Tari [Communication Node]s are identified on the 
network via their [Node ID]; which in turn are derived from the node's public key. Both the node id and public key are simple large integer numbers. The communication private key should be kept
private by the node at all times to ensure that a node cannot be spoofed. 



The most common practice for human beings to copy large numbers in cryptocurrency software is scanning a QR code or copying and pasting a value from one application to another. These numbers are typically encoded using hexadecimal or Base58
encoding. The user will then typically scan (parts) of the string by eye to ensure that the value was transferred
correctly.

For Tari, we propose encoding values, the node ID in particular and masking the network identifier, for Tari, using emojis. The advantages of this approach are:

* Emoji are more easily identifiable; and, if selected carefully, less prone to identification errors (e.g., mistaking an
  O for a 0).
* The alphabet can be considerably larger than hexadecimal (16) or Base58 (58), resulting in shorter character sequences
  in the encoding.
* Should be be able to detect if the address used belongs to the correct network. 
## The specification

### Address

Each address is divided into 4 parts: View key, Spend key, Network and Features.

#### View key
This is the key used by node to provide view access to its transactions. An entity possessing the private key of this key pair SHOULD be able to view all transactions
a wallet has made. 

#### Spend key
This is the key used to calculate the spend key of a UTXO and communicate to the node over the network. This private key of this key pair SHOULD be kept
secure and hidden at all costs.

#### Features
This is used to indicate features that the wallet supports or not supports. At current this is only used to indicate if the wallet supports interactive and or one-sided transactions.
This is an encoded u8 with each bit indicating a feature. 

#### Network
This is the Tari network the wallet is on, for example: Esmeralda, Nextnet etc. 

#### Checksum
We only include the checksum when encoding the address as bytes, hex or emojis. For the checksum, we use: [DammSum](https://github.com/cypherstack/dammsum) algorithm with `k = 8` and `m = 32` to compute an 8-bit checksum.

### Encoding 

#### Bytes
When creating a byte representation of the wallet, the following is used:
[0] - Network encoded into u8
[1] - Features raw u8
[2..33] - Public view key encoded as u8
[35..65] - Public spend key encoded as u8
[66] - DammSum checksum

For nodes that do not have a view key, the view key and spend key is treated as being the same, their addresses can be encoded as follows:
[0] - Network encoded into u8
[1] - Features raw u8
[2..33] - Public spend key encoded as u8
[34] - DammSum checksum

#### Hex
Encode each byte in the byte representation as two hex characters.

#### Emoji encoding
An emoji alphabet of 256 characters is selected. Each emoji is assigned a unique index from 0 to 255 inclusive. The
list of selected emojis is:

| | | | | | | | | | | | | | | | |
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
|🦋|📟|🌈|🌊|🎯|🐋|🌙|🤔|🌕|⭐|🎋|🌰|🌴|🌵|🌲|🌸|
|🌹|🌻|🌽|🍀|🍁|🍄|🥑|🍆|🍇|🍈|🍉|🍊|🍋|🍌|🍍|🍎|
|🍐|🍑|🍒|🍓|🍔|🍕|🍗|🍚|🍞|🍟|🥝|🍣|🍦|🍩|🍪|🍫|
|🍬|🍭|🍯|🥐|🍳|🥄|🍵|🍶|🍷|🍸|🍾|🍺|🍼|🎀|🎁|🎂|
|🎃|🤖|🎈|🎉|🎒|🎓|🎠|🎡|🎢|🎣|🎤|🎥|🎧|🎨|🎩|🎪|
|🎬|🎭|🎮|🎰|🎱|🎲|🎳|🎵|🎷|🎸|🎹|🎺|🎻|🎼|🎽|🎾|
|🎿|🏀|🏁|🏆|🏈|⚽|🏠|🏥|🏦|🏭|🏰|🐀|🐉|🐊|🐌|🐍|
|🦁|🐐|🐑|🐔|🙈|🐗|🐘|🐙|🐚|🐛|🐜|🐝|🐞|🐢|🐣|🐨|
|🦀|🐪|🐬|🐭|🐮|🐯|🐰|🦆|🦂|🐴|🐵|🐶|🐷|🐸|🐺|🐻|
|🐼|🐽|🐾|👀|👅|👑|👒|🧢|💅|👕|👖|👗|👘|👙|💃|👛|
|👞|👟|👠|🥊|👢|👣|🤡|👻|👽|👾|🤠|👃|💄|💈|💉|💊|
|💋|👂|💍|💎|💐|💔|🔒|🧩|💡|💣|💤|💦|💨|💩|➕|💯|
|💰|💳|💵|💺|💻|💼|📈|📜|📌|📎|📖|📿|📡|⏰|📱|📷|
|🔋|🔌|🚰|🔑|🔔|🔥|🔦|🔧|🔨|🔩|🔪|🔫|🔬|🔭|🔮|🔱|
|🗽|😂|😇|😈|🤑|😍|😎|😱|😷|🤢|👍|👶|🚀|🚁|🚂|🚚|
|🚑|🚒|🚓|🛵|🚗|🚜|🚢|🚦|🚧|🚨|🚪|🚫|🚲|🚽|🚿|🧲|



The emoji have been selected such that:
* Similar-looking emoji are excluded from the map. For example, neither 😁 or 😄 should be included. Similarly, the Irish and
  Côte d'Ivoire flags look very similar, and both should be excluded.
* Modified emoji (skin tones, gender modifiers) are excluded. Only the "base" emoji are considered.

Encode each byte as emoji listed in the corresponding index


## Change Log

| Date         | Change                   | Author        |
|:-------------|:-------------------------|:--------------|
| 2024-05-31   | Initial stable           | SWvHeerden    |

[Communication Node]: Glossary.md#communication-node
[Node ID]: Glossary.md#node-id