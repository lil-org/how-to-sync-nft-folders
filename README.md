# syncing custom nft folders

organize nfts into folders and see the same custom folders on zora, opensea, rainbow, etc.

what could be a good standard way to sync these folders?

# wip approach using [eas](https://docs.attest.org)

read and write folders in 5 steps

### 1️⃣ upload `Snapshot` json to ipfs
```swift
struct Snapshot: Codable {
    let folders: [Folder]
    let uuid: String
}

struct Folder: Codable {
    let name: String
    let tokens: [Token]
}

struct Token: Codable {
    let id: String
    let address: String
    let chainId: String
}
```

### 2️⃣ create an attestation with `Snapshot` ipfs cid
```swift
let cid = "bafkreig6jexnmx5nvfsx4qwao3zggfhkufetrva2kek353zgjz6i6hmzsa"

let recipient = "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" // owner
let folderType: UInt32 = 0 // 0 for now
let formatVersion: UInt32 = 0 // 0 for now
let arguments = String.paddedHexString(cid: cid, folderType: folderType, formatVersion: formatVersion)
let schemaId = "0xb7cc934d4a5b37542520bfc80184538e568529d5fba5b13abe89109a23620cb6"

let url = "https://base.easscan.org/attestation/attestWithSchema/" + schemaId + "#template=\(recipient)::0:false:\(arguments)"
```
see [example new attestation url](https://base.easscan.org/attestation/attestWithSchema/0xb7cc934d4a5b37542520bfc80184538e568529d5fba5b13abe89109a23620cb6#template=0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE::0:false:0x000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003b6261666b72656967366a65786e6d78356e76667378347177616f337a676766686b75666574727661326b656b3335337a676a7a366936686d7a73610000000000)

### 3️⃣ get the latest attestation
using [easscan graphql api](https://docs.attest.org/docs/developer-tools/api)

```graphql
query Attestation {
    attestations(
        take: 1,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0xb7cc934d4a5b37542520bfc80184538e568529d5fba5b13abe89109a23620cb6" }, 
            recipient: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" },
            attester: { equals: "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" } 
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
    --data '{"query":"query Attestation {\n  attestations(\n    take: 1,\n    orderBy: { timeCreated: desc},\n    where: { schemaId: { equals: \"0xb7cc934d4a5b37542520bfc80184538e568529d5fba5b13abe89109a23620cb6\" }, recipient: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" } }\n  ) {\n    attester\n    recipient\n    decodedDataJson\n    timeCreated\n  }\n}","variables":{}}'
```

### 4️⃣ get `Snapshot` json corresponding to the latest attestation
https://ipfs.decentralized-content.com/ipfs/bafkreig6jexnmx5nvfsx4qwao3zggfhkufetrva2kek353zgjz6i6hmzsa

### 5️⃣ get nfts from an api of your choice
use `Snapshot` to display nfts in folders

## other options to consider

add an optional preview image url with each nft to make folder content appear faster?

add an optional preview image url for a folder?

# projects syncing folders

[REDACTED] soon

[nft-folder-macos](https://github.com/lil-org/nft-folder)

add yours too!

# feedback
### please [create an issue](https://github.com/lil-org/how-to-sync-nft-folders/issues) or [create a pull request](https://github.com/lil-org/how-to-sync-nft-folders/pulls)
