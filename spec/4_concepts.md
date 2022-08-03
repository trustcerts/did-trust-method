## Concepts

### Transaction Based

DID documents are not written on the ledger but composed of multiple, chronological transactions stored on the blockchain. A versioning mechanism is integrated by default and allows the requester to get a DID document at a given timestamp. This is important since DID documents are not immutable so entries like public keys can be updated or removed.

### Chain of Trust

The updates of a DID in this method are ALWAYS in a hierarchical order. This means each transaction that changed a DID is logged on the blockchain where the signature of the controller is persisted for verification. Beside the transaction’s signature the verifier can also validate the signatures of the block in which the transaction got persisted. This allows the identification of the responsible nodes that accepted the transaction in the consensus protocol.

The preferred way to guarantee the chain of trust is by the verification of the digital signatures. In this case all requests can be sent to only one node of the network since the chain of trust is going back to the genesis block, the root of trust. If the genesis block is not locally present to the client, the client can request it from multiple nodes and validate if the results are equal. This reduces the risk of a single point of failure by receiving a compromised block and therefore a compromised chain of trust.

### Single request vs multi request

Since a DID can be updated multiple times, verifying each transaction can be time consuming. Instead, the client can request a parsed document from the server. When a DID should be changed, the client puts the signature of the changes and also the signature of the current DID document into the transaction. This allows the verifier to choose between two options.

Requesting and validating only one request is faster, but on the other hand requesting all transactions and assembling has two advantages:

- An earlier version of the DID document can be assembled since the client already has the required transaction.
- A more detailed view on the changes of a DID document is possible if this is required by defined policies.
- Since both signatures will be placed in the transaction, the client is free to choose the method it prefers.

### DID Resources

The DID Trust Method allows for different types of DID documents to be stored on the ledger, such as schemas or templates as specified in [Link].

By allowing schemas, templates and other resources to be represented as DID documents you can take advantage of the features provided by the DID Trust Method like addressing specific resources through their DID and the capability to store and resolve different versions of a DID document. This enables to retrieve previous versions of on-ledger resources by addressing them via their version time (versionTime) or their version id (versionId). Storing resources on-ledger makes it possible to take advantage of other benefits of using DLTs as a storage medium such as improved availabilty and redundancy.
