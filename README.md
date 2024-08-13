# why sync nft folders?

organize [your own nfts](https://warpcast.com/hot/0x9d479b85) or assemble boards [across different owners and authors](https://x.com/artignatyev/status/1803864822129529197)

see the same custom folders on zora, surreal, rainbow, opensea, etc.

# sync folders via [eas](https://docs.attest.org)

1 folder = 1 attestation

read and write folders in 5 steps

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
let folderType: UInt32 = 4242424242 // use 4242424242 when you organize your own nfts; use 69696969 when assembling custom boards
let folderId: Data = Data() // TODO: bytes32 random id
let folderName: String = "zorbs or smth"
let cid = "bafkreifc5lvk2mhi2vtt3lexm433xn7uwddadlwsvn5n37kwv6sp2koapa"

let schemaId = "0xfeb3224bb6737f8f8034186c06f79d0740f40e806965e4b442350a78cef7ec86"
let arguments = String.paddedHexString(folderType, folderId, folderName, cid)

let recipient = "0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE" // owner address; can be empty when assembling custom boards

let url = "https://base.easscan.org/attestation/attestWithSchema/" + schemaId + "#template=\(recipient)::0:false:\(arguments)"
```
see [example new attestation url](https://base.easscan.org/attestation/attestWithSchema/0x8c138d949f94e74f6503a8633bb25982946709fddc196764e26c9325b8c04f73#template=0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE::0:false:0x000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000fcde41b2000000000000000000000000000000000000000000000000000000000000003b6261666b7265696663356c766b326d686932767474336c65786d343333786e377577646461646c7773766e356e33376b7776367370326b6f6170610000000000)

### 3️⃣ get the latest attestations
using [easscan graphql api](https://docs.attest.org/docs/developer-tools/api)

#### for your own nfts organized
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0xfeb3224bb6737f8f8034186c06f79d0740f40e806965e4b442350a78cef7ec86" }, 
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
     --data '{"query":"query Attestation { attestations(take: 20, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0xfeb3224bb6737f8f8034186c06f79d0740f40e806965e4b442350a78cef7ec86\" }, recipient: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, data: { contains: \"fcde41b2\"} }) { attester recipient decodedDataJson timeCreated } }","variables":{}}'
```
#### for custom boards assembled by you
```graphql
query Attestation {
    attestations(
        take: 20,
        skip: 0,
        orderBy: { timeCreated: desc },
        where: { 
            schemaId: { equals: "0x8c138d949f94e74f6503a8633bb25982946709fddc196764e26c9325b8c04f73" }, 
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
     --data '{"query":"query Attestation { attestations(take: 20, orderBy: { timeCreated: desc }, where: { schemaId: { equals: \"0xfeb3224bb6737f8f8034186c06f79d0740f40e806965e4b442350a78cef7ec86\" }, recipient: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, attester: { equals: \"0xE26067c76fdbe877F48b0a8400cf5Db8B47aF0fE\" }, data: { contains: \"4277dc9\"} }) { attester recipient decodedDataJson timeCreated } }","variables":{}}'
```

### 4️⃣ get `FolderSnapshot` json corresponding to the attestation
https://ipfs.decentralized-content.com/ipfs/bafkreifc5lvk2mhi2vtt3lexm433xn7uwddadlwsvn5n37kwv6sp2koapa

### 5️⃣ get nfts from an api of your choice
use the latest `FolderSnapshot`s to display nfts in folders

## 📝 editing folders
create a new attestation using the same `folderId`

you can use a different `cid` or a different `folderName`

## 🗑️ removing folders
[revoke all attestations](https://docs.attest.org/docs/core--concepts/revocation) for that `folderId`

or

create a new attestation using the same `folderId` with an empty `cid`

# projects syncing folders

[nft folder macos](https://github.com/lil-org/nft-folder) to organize your own nfts via folder type `4242424242`

[cherry](https://github.com/jordanpunzalann/cherry) to assemble custom boards via folder type `69696969`

add yours too!

# feedback
### [create an issue](https://github.com/lil-org/how-to-sync-nft-folders/issues) or [create a pull request](https://github.com/lil-org/how-to-sync-nft-folders/pulls)
