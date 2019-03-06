# Using HLF for Secure File Transfer

This code builds on the marbles02_private tutorial in the Fabric samples (https://hyperledger-fabric.readthedocs.io/en/release-1.4/private_data_tutorial.html) to demonstrate using chaincode to securely share data stored off-chain in an IPFS file system.

## Background
In this example, we assume that the user is wanting to publish a document to an IPFS filesystem, to which other users besides those authorised to access to document may have access. I assume that the file has been encrypted by the originator and saved to the IPFS cluster, with its address and encryption key being known to the originator.

Once the document is encrypted and saved off-chain, the location and encryption key for the file are saved to the blockchain using a HLF Private Data Collection, which is configured such that only two two organizations allowed to access the data can read the entry, and hence locate and decrypt the document.

## Implementation

The data stored to the chain will include the originator of the data (e.g. Org1), the authorized recipient of the data (e.g. Org2), and the authorization under which the file is being sent. The private data for each record is the location of the file itself, and the encryption key needed to decrypt it (assuming symmetrical encryption for now...)

The code can be demonstrated by instantiating on the byfn sample network included with the HLF samples, following the pattern described in the marbles02_private tutorial, with the following substitutions:

### Installation
```
peer chaincode install -n fileTransfer -v 1.0 -p github.com/chaincode/hlf-private-data/go/
```

### Instantiation
```
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n fileTransfer -v 1.0 -c '{"Args":["init"]}' -P "OR('Org1MSP.member', 'Org2MSP.member')" --collections-config $GOPATH/src/github.com/chaincode/hlf-private-data/collections_config.json
```

### Invokation
```
export TRANSFER=$(echo -n "{\"name\":\"transfer1\",\"description\":\"first transfer\",\"originator\":\"alice\",\"recipient\":\"bob\",\"authorization\":\"auth1\",\"address\":\"file-is-here\",\"encryptionKey\":\"secret\"}" | base64 | tr -d \\n)
```
```
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fileTransfer -c '{"Args":["initFileTransfer"]}' --transient "{\"fileTransfer\":\"$TRANSFER\"}"
```
## TODO
1) Recipient accessing record leaves trace of having received data.
2) Use public & private keys of participants to encrypt/decrypt files instead of explicitly including key in on-chain records.