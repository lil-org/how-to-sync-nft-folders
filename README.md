> organize [your own nfts](https://warpcast.com/hot/0x7455c2f6) or assemble collections [across different owners and authors](https://x.com/artignatyev/status/1803864822129529197)

# how to sync nft folders

use [ethereum attestation service](https://docs.attest.org)

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
let cid = "bafkreiezuaccaqjwt6urnqzk4ko6ukugeutdrhrxgbbgwdyrexnjmim5uy"
let folderName = "zorbs"
let folderType: UInt32 = 4242424242

// use 4242424242 when you organize your own nfts
// use 69696969 when assembling custom boards

let schemaId = "0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a"
let arguments = String.paddedHexString(folderType, folderName, cid)

let recipient = "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE"

// recipient = nft owner address
// recipient can be empty when assembling custom boards

let url = "https://base.easscan.org/attestation/attestWithSchema/" + schemaId + "#template=\(recipient)::0:false:\(arguments)"
```
see [example new attestation url](https://base.easscan.org/attestation/attestWithSchema/0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a#template=0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE::0:true:0x00000000000000000000000000000000000000000000000000000000fcde41b2000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000057a6f726273000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003b6261666b726569657a7561636361716a77743675726e717a6b346b6f36756b75676575746472687278676262677764797265786e6a6d696d3575790000000000)

> [!TIP]
> use `multiAttest` to batch multiple attestations into a single transaction

### 3️⃣ get the latest attestations
use [easscan graphql api](https://docs.attest.org/docs/developer-tools/api)

#### 📁 for your own nfts organized
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a" }, 
            recipient: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" }, # owner address
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" }, # owner address
            data: { contains: "fcde41b2"} # corresponds to folderType 4242424242
        }
    ) {
        attester
        recipient
        decodedDataJson
        timeCreated
    }
}
```
try curl

```sh
curl --request POST \
     --header 'content-type: application/json' \
     --url 'https://base.easscan.org/graphql' \
     --data '{"query":"query Attestation { attestations(take: 20, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a\" }, recipient: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, data: { contains: \"fcde41b2\"} }) { attester recipient decodedDataJson timeCreated } }","variables":{}}'
```
#### 🍒 for custom boards assembled by you
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a" }, 
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" }, # assembler address
            data: { contains: "4277dc9"} # corresponds to folderType 69696969
        }
    ) {
        attester
        recipient
        decodedDataJson
        timeCreated
    }
}
```
try curl

```sh
curl --request POST \
     --header 'content-type: application/json' \
     --url 'https://base.easscan.org/graphql' \
     --data '{"query":"query Attestation { attestations(take: 20, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0x8c273fb082aea02208b56223fa76cea434c5eaa5dc3c2a5b3bbab474bae5019a\" }, recipient: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, data: { contains: \"4277dc9\"} }) { attester recipient decodedDataJson timeCreated } }","variables":{}}'
```

### 4️⃣ get `FolderSnapshot` jsons corresponding to the latest attestations
https://ipfs.decentralized-content.com/ipfs/bafkreiezuaccaqjwt6urnqzk4ko6ukugeutdrhrxgbbgwdyrexnjmim5uy

### 5️⃣ get nfts from an api of your choice
use the latest `FolderSnapshot` values to display nfts in folders

---

### 📝 edit a folder
use [referenced attestations](https://docs.attest.org/docs/tutorials/referenced-attestations)

create a new attestation with the `refUID` of the initial attestation for that folder

the next edit for that folder should contain the same `refUID` of the initial attestation


### 🗑️ remove a folder
set `cid` value to an empty string

or — [revoke](https://docs.attest.org/docs/core--concepts/revocation) the initial attestation for that folder

# projects syncing folders

[nft folder macos](https://github.com/lil-org/nft-folder) to organize your own nfts via folder type `4242424242`

[cherry](https://github.com/jordanpunzalann/cherry) to assemble custom boards via folder type `69696969`

add yours too!

# feedback
### [create an issue](https://github.com/lil-org/how-to-sync-nft-folders/issues) or [create a pull request](https://github.com/lil-org/how-to-sync-nft-folders/pulls)
