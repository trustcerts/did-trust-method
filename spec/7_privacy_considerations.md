## Privacy Considerations

### Surveillance

If a DID is known, any person can resolve it into its DID document. The content of the DID document and changes can be monitored. Only blockchain operators have access to all transactions for analysis purposes. Companies using this DID method can alternatively use a private permissioned blockchain.

### Stored Data Compromise

Compromise of stored data on the ledger is prevented by the chain of trust, consensus, and ledger mechanisms.

Caching: If the transactions were validated before caching the final DID document, a malicious actor could manipulate the DID document afterwards (e.g. service endpoints). To prevent this from happening multiple parallel queries should be performed on the respecting DID.

### Unsolicited Traffic/Intrusion

If a DID is known, it can be resolved to a DID document. This means that all service endpoints that can also be viewed could be exposed to unsolicited traffic . However, no service endpoints need to be specified in the DID document.

### Misattribution

DID duplicates are prevented from being created by validating the incoming DID transaction against the database to find double entries. In case of a race condition where two new transactions should be persisted with the same identifier, the transactions will be checked one after another when proposing a new block.

### Correlation

The risk of correlation is analogous to the points raised in [10.2 DID Correlation Risks and Pseudonymous DIDs](https://www.w3.org/TR/did-core/#did-correlation-risks-and-pseudonymous-dids) and [10.3 DID Document Correlation Risks](https://www.w3.org/TR/did-core/#did-document-correlation-risks) of the W3C. Possible correlation factors could be the specified controller, cryptographic material or service endpoints inside the DID document.

In addition, the versioning of the DID document provided in this DID method can allow correlation by factors that existed in previous versions of the DID document.

### Identification

The direct identification of a person by his DID in this DID method is not given, because the namespace identifier is a randomly generated alphanumeric sequence. Since a DID document does not contain personally identifiable information (PII), no direct identification is possible. But it should be considered that an identification via correlation is always possible.

### Secondary Use

Information that can be viewed publicly in DID documents is constantly at risk of being misappropriated. E.g. the specified controller, cryptographic material or service endpoints. In this DID method, no personal data is written to the DID document.

### Disclosure

Only things that were intended to be public in a DID document can be disclosed such as verification methods and service endpoints. Users need to be aware that all information specified in the DID document is visible (even older contect due to versioning of the DID document).

### Exclusion

Critical or sensitive information should be located behind service endpoints protected by authentication mechanisms, as the DID document can be publicly viewed by anyone.
