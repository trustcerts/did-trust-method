## The did:trust Format

### Method Name

The method name that identifies this DID method SHALL be: `trust`

A DID that uses this method MUST begin with the following prefix: `did:trust`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is the method identifier described below.

### DID Trust Method Identifiers

The Trust DID Method DID identifier has three components that combine to make an identifier that conforms to the DID specification. The components are:

- **DID Trust Method**: the hardcoded string did:trust: indicating that the identifier uses this DID Method specification.
- **DID Trust Namespace**: a string that identifies the unique trust ledger namespace of the DID, with a single and a subsidiary name. For example `tc:prod`.
- **Namespace Identifier**: two-element identifier unique to the DID Trust namespace. The first parameter describes the type of the DID as an internal namespace. The second, required element is an identifier that is unique within the namespace.

#### Delegation

The DID has to be read from left to right where it will be separated into the three parts: DID method, DID namespace and identifier namespace. All parts are required to have a valid DID. There are no default values that will be added since the separator between the elements and sub-elements is equal.

The first part will be handled by the agent. It has to look up the DID resolver and pass the DID to it.

The next part is handled by the resolver to detect which blockchain network has to be used. The resolver has to send the DID to the known blockchain nodes to get the required transactions to resolve the DID document.

The blockchain nodes will look up based on the namespace which module is responsible for handling this DID type and return the requested transactions that are related to the DID.

#### DID Trust Namespace

The Trust namespace allows the DID method to support multiple hosted networks. The first name defines a unique, human-friendly identifier for a consortium/organization that hosts one or multiple ledgers. Each organization can host multiple networks in their namespace like `staging` or `prod` or organizations specific like `europe` or `internal`. The namespace MUST be in lowercase.

```
namespace     = namestring ":" namestring ":"
namestring    = ALPHA *(ALPHA / DIGIT / "_" / "-")
```

Since the first parameter only defines the organization, the second parameter is required to reach the correct ledger, even if there is only one ledger hosted.

A resolver has to register the endpoints for writing and reading transactions:

```json
{
  "org1:dev": {
    "gateways": ["http://obs1.example.com"],
    "observers": ["http://por1.example.com", "http://por2.example.com"]
  },
  "foo:prod": {
    "gateways": ["http://p.foobar.com"],
    "observers": ["http://o.foobar.com"]
  }
}
```

Since there is no requirement to host central network name resolver, a known connection can be used to ask a network

- if it knows the connection endpoints to a specific trust namespace
- if it knows more endpoints for the nodes for writing and reading to increase redundancy

Implementing a global register could be one way to reduce complexity and would make network discovery automatically. This would also prevent multiple tust namespaces with the same identifier. But on the other hand a single point of failure is created. So when creating a new network the identifier has to be shared with other existing networks that have to accept the connection between the choosen trust namespace and the network.

#### Namespace Identifier

The namespace identifier is required to support the modular approach of the underlying blockchain. It allows handling different DID types in individual ways to boost the performance. A DID that represents a visual representation has to be stored in a different kind of database because of the data size. On the other hand a DID that is based on relation and their queries should be stored in a relational database than a key value store.

The first namespace is defined by the module that was addressed inside the trustchain:

```
namespace     = namestring ":" identifier
namestring    = ALPHA *(ALPHA / DIGIT / "_" / "-")
identifier    = *(ALPHA / DIGIT)
```

The identifier behind this depends on the namespace since every namespace defines the regulation on its own:

- In case of DID documents it can be a base58 encoded value with the length of 22 characters
- When dealing with hashes the values length is based on the algorithm and the encoding option.

```
id:XLzBJ69keqEgq7oqqdEsHW
tmp:f796e2f28ae5811737ccb8233f34e09f8bb75d2511a135543d1ca37be0199a1d
```

This gives maximum flexibility of using the DID method in different use cases. In some cases the identifier already exists (an immutable file should be addressed by a DID) so it is possible to go with the IPFS approach by using the file’s hash. Cutting down the length of a hash to be equal with the length of the id could influence the security level. On the other hand when an object should be defined, the DID can be created before using it.

#### Default Namespaces

The trust DID method does not support default namespaces right now. This is because of the separator `:` is used for internal separation (`tc:prod`) and joining the components (`did:trust:tc:prod:id:XLzBJ69keqEgq7oqqdEsHW`). When skipping an element like the network component, like in this case `did:trust:tc:id:XLzBJ69keqEgq7oqqdEsHW`, the resolver does not know if the `id` should be the network and the node will use the default identifier namespace or not. Not having the `tc:id` locally registered does not mean the network does not exist. There could be another network out there that is defined with this network namespace.

Using one more separator like `-` would solve this problem, but we suggest to use ALWAYS full names since

- The DID documents always include the full names because of the signature process
- The definition of choosing the default value has to be globally defined for all networks

#### Specifying the DID Document Type

It is possible to use the namespace identifier to further specify the type of a DID document, such as `tc:prod:sch` for DID schema documents, `tc:prod:tmp` for DID template documents and `tc:prod:hash` for DID hash documents.

Here are some example DIDs for different types of DID documents:

- DID schema document: `did:trust:tc:dev:sch:6BquEL34bKFVcCbv5NmZdF`
- DID template document: `did:trust:tc:dev:tmp:BKqANFfATgZSwBA6UzzE2k`
- DID hash document: `did:trust:tc:dev:hash:DSkA5Pw7t76ZiR4oKiM1kpJExy3zikfD5S4iwsbQQ2FR`
