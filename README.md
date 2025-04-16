> organize [your own nfts](https://warpcast.com/hot/0x7455c2f6) or assemble collections [across different owners and authors](https://x.com/artignatyev/status/1803864822129529197)

# how to sync nft folders

use [ethereum attestation service](https://base.easscan.org/schema/view/0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b)

1 folder = 1 attestation

read and write folders in 5 steps:

### 1️⃣ upload `FolderSnapshot` json to ipfs
```swift
struct FolderSnapshot: Codable {
    let tokens: [Token]
    let description: String?
    let cover: String?
}

struct Token: Codable {
    let id: String
    let address: String
    let chainId: String
    let comment: String?
}
```

### 2️⃣ create an attestation with `FolderSnapshot` ipfs cid

```swift
let cid = "bafkreia4i7jwb5qq4upcdl5knvgtvvrqhjchsqsot6vozd26vcabgoed44"
let folderName = "zorbs"
let nameAndCid = folderName + " " + cid
let folderType: UInt8 = 42

// use 42 when you organize your own nfts
// use 69 when assembling custom boards

let schemaId = "0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b"
let arguments = String.paddedHexString(folderType, nameAndCid)

let template = "#template=::0:false:\(arguments)"
let url = "https://base.easscan.org/attestation/attestWithSchema/" + schemaId + template
```
see [example new attestation url](https://base.easscan.org/attestation/attestWithSchema/0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b#template=::0:true:0x000000000000000000000000000000000000000000000000000000000000002a000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000417a6f726273206261666b726569613469376a776235717134757063646c356b6e76677476767271686a63687371736f7436766f7a64323676636162676f6564343400000000000000000000000000000000000000000000000000000000000000)

> [!TIP]
> use `multiAttest` to batch multiple attestations into a single transaction

### 3️⃣ get folders
use [easscan graphql api](https://docs.attest.org/docs/developer-tools/api)

attestations with an empty `refUID` correspond to all created folders

attestations with a non-zero `refUID` correspond to folder edits

#### 📁 for your own nfts organized
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b" }, 
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" }, # owner address
            refUID: { equals: "0x0000000000000000000000000000000000000000000000000000000000000000" }, # created folders only
            revoked: { equals: false },
            data: { startsWith: "0x000000000000000000000000000000000000000000000000000000000000002a"} # corresponds to folderType 42
        }
    ) {
        decodedDataJson
        refUID
        id
    }
}
```
```sh
curl --request POST --header 'content-type: application/json' --url 'https://base.easscan.org/graphql' --data '{"query":"query Attestation { attestations(take: 20, skip: 0, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, refUID: { equals: \"0x0000000000000000000000000000000000000000000000000000000000000000\" }, revoked: { equals: false }, data: { startsWith: \"0x000000000000000000000000000000000000000000000000000000000000002a\" } } ) { decodedDataJson refUID id } }"}'
```

#### 🍒 for custom boards assembled by you
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b" }, 
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" }, # curator address
            refUID: { equals: "0x0000000000000000000000000000000000000000000000000000000000000000" }, # created folders only
            revoked: { equals: false },
            data: { startsWith: "0x0000000000000000000000000000000000000000000000000000000000000045"} # corresponds to folderType 69
        }
    ) {
        decodedDataJson
        refUID
        id
    }
}
```
```sh
curl --request POST --header 'content-type: application/json' --url 'https://base.easscan.org/graphql' --data '{"query":"query Attestation { attestations(take: 20, skip: 0, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, refUID: { equals: \"0x0000000000000000000000000000000000000000000000000000000000000000\" }, revoked: { equals: false }, data: { startsWith: \"0x0000000000000000000000000000000000000000000000000000000000000045\" } } ) { decodedDataJson refUID id } }"}'
```

#### 💦 latest edits
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b" }, 
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" },
            refUID: { notIn: "0x0000000000000000000000000000000000000000000000000000000000000000" }, # edits only
            revoked: { equals: false },
        },
        distinct: [refUID] # unique and latest
    ) {
        decodedDataJson
        refUID
        id
    }
}
```
```sh
curl --request POST --header 'content-type: application/json' --url 'https://base.easscan.org/graphql' --data '{"query":"query Attestation { attestations(take: 20, skip: 0, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0x24a31e6646f2d422a173165d76984a4ee1cfd2bea26be543ba15f7e9319bca4b\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, refUID: { notIn: \"0x0000000000000000000000000000000000000000000000000000000000000000\" }, revoked: { equals: false } }, distinct: [refUID] ) { decodedDataJson refUID id } }"}'

```

### 4️⃣ get `FolderSnapshot` jsons corresponding to the latest attestations
https://ipfs.decentralized-content.com/ipfs/bafkreia4i7jwb5qq4upcdl5knvgtvvrqhjchsqsot6vozd26vcabgoed44

### 5️⃣ get nfts from an api of your choice
use the latest `FolderSnapshot` values to display nfts in folders

---

### 📝 edit a folder
use [referenced attestations](https://docs.attest.org/docs/tutorials/referenced-attestations)

create a new attestation with the `refUID` of the initial attestation for that folder

the next edit for that folder should contain the same `refUID` of the initial attestation


### 🗑️ remove a folder
set `nameAndCid` value to an empty string

or [revoke](https://docs.attest.org/docs/core--concepts/revocation) the initial attestation for that folder

# projects syncing folders

[CANCELED]

# feedback
### [create an issue](https://github.com/lil-org/how-to-sync-nft-folders/issues) or [create a pull request](https://github.com/lil-org/how-to-sync-nft-folders/pulls)
