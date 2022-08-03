## Security Considerations

### Attacks

**Eavesdropping**:
The transport protocol is secured via TLS so a malicious actor can not read the content of the request and the response.

**Replay Attack**:
Since transactions are validated in a chronological way a replayed transaction will be denied. All transactions include a timestamp that acts like a nonce. Transaction A will add a key to the DID document. Some seconds later transaction B will remove it. A malicious actor will reply with Transaction A to add the key again, but the timestamp that is included in the signature will not match anymore. The blockchain compares the timestamps and recognizes that the replayed transaction A timestamp is lower than the transaction B timestamp and will deny it. Replaying transaction A results in the same since the timestamp for a new transaction has to be greater than the latest known one. Timestamps are given in milliseconds so a detailed comparison is possible since the consensus to validate a new block needs more than one millisecond.

**Message Insertion**:
Each transaction is signed by the issuer and validated by the receiver. The public key of the issuer is already known by a previous blockchain transaction.

**Deletion**:
Redundancy is given by the blockchain in multiple ways. On a physical layer all transactions are stored on multiple servers to prevent a single point of failure. On the application layer the blockchain never deletes transactions, but protocols a deletion action of a DID document element on the blockchain instead.

**Modification**:
On the blockchain level all, transactions are linked by the linking of the blocks. Chaining one transaction will result in another merkle tree's root hash that will not match with the one in the next block. Modifying a transaction body after the signature by adding, updating or removing content will not work since the signature involves all changes of the body.

**Denial of Service Attack**:
The blockchain is a distributed network so a client is able to request multiple nodes to get the information.

**Storage or network amplification**:
Adding and removing the same key results in the same DID document, but will require storage since all changes are logged by transactions. This can be prevented by a fee for each transaction and a rate limit for each issuer.

**Man-In-The-Middle**:
The whole transport is secured by TLS and the requests are signed by the issuer so there is no risk of a man in the middle attack.

### Residual risks

External libraries are used in this DID method. Any security issues in these libraries also affect this DID method. Also, if the end device is compromised, there is also a residual risk outside the scope of this DID method.

### Integrity protection and update authentication for method operations

All operations that change the state of a DID (create, update, deactivate) require authentication. This is done by signing the transaction body, the blockchain can specify how many or what kind of signatures are required to sign a DID in a specific context. For a DID with a higher privilege a multi signature from two other members can be a requirement.  
The signature validation is done before validators are persisting the transaction into a block and when a client requested transactions or objects from the blockchain system.

### User-host-authentication

User authentication for writing on the ledger is realized by signing transactions using a private/public key pair, where the authorized public keys are stored on the ledger. There is no kind of authentication required for reading operations.

### Unique DIDs

DIDs are proven to be uniquely assigned by checking for duplicate transactions at both block generation and block validation time.

### Network topology

Clients inside ring 3 interact with gateways and observers inside ring 2, who are responsible for reading and writing to the ledger. The clients have no direct connection to the validators in ring 1, who are responsible for generating blocks. A resolver itself can also be regarded as a proxy that can be used for caching. There is no authentication required for reading from the ledger.

### Cryptographic protection mechanisms

Transactions are checked for integrity by only allowing specified transaction formats. Transactions are cryptographically signed to ensure integrity.

### Private data

You MUST keep your private keys private in order to prevent unauthorized writes to the ledger.

### Signatures on DID documents

When resolving a DID document, you SHOULD verify the node's response by validating the signatures of the complete transaction history back to the genesis block to ensure the validity of the retrieved DID document via the chain of trust. If this is not possible or feasible, you SHOULD compare the results of querying multiple – or ideally all – nodes of the network.

### Peer-to-peer computing resources

On the DLT side each node is able to scale the http servers so they can handle multiple read requests from the clients.
