# The did:trust Method v0.1

# Abstract

This specification defines a DID method based on the Trustchain and its full potential. 

# Status of this document

This document is a draft of a potential specification. It has no official standing of any kind and does not represent the support or consensus of any standards organization.

# Introduction

This section is non-normative.

The Trustchain is a blockchain system that was designed from the ground to deal with use cases in the self-sovereign identity context. It combines the advantages of a classical hierarchical public key infrastructure and the Web of Trust's decentralized approach.

Since the goal of the Trustchain blockchain is to offer multiple networks, the DID method is designed to connect every hosted Trustchain network with each other, if wished. This allows cross network references so DIDs from another network can smoothly interact with each other.

# Concepts

## Transaction Based

DID documents are not written on the ledger but composed of multiple, chronological transactions stored on the blockchain. A versioning mechanism is integrated by default and allows the requester to get a DID document at a given timestamp. This is important since DID documents are not immutable so entries like public keys can be updated or removed.

## Chain of Trust

The updates of a DID in this method are ALWAYS in a hierarchical order. This means each transaction that changed a DID is logged on the blockchain where the signature of the controller is persisted for verification. Beside the transaction’s signature the verifier can also validate the signatures of the block in which the transaction got persisted. This allows the identification of the responsible nodes that accepted the transaction in the consensus protocol.

The preferred way to guarantee the chain of trust is by the verification of the digital signatures. In this case all requests can be sent to only one node of the network since the chain of trust is going back to the genesis block, the root of trust. If the genesis block is not locally present to the client, the client can request it from multiple nodes and validate if the results are equal. This reduces the risk of a single point of failure by receiving a compromised block and therefore a compromised chain of trust.

## Single request vs multi request

Since a DID can be updated multiple times, verifying each transaction can be time consuming. Instead, the client can request a parsed document from the server. When a DID should be changed, the client puts the signature of the changes and also the signature of the current DID document into the transaction. This allows the verifier to choose between two options.

Requesting and validating only one request is faster, but on the other hand requesting all transactions and assembling has two advantages:
- An earlier version of the DID document can be assembled since the client already has the required transaction.
- A more detailed view on the changes of a DID document is possible if this is required by defined policies.
- Since both signatures will be placed in the transaction, the client is free to choose the method it prefers.

# The did:trust Format

## Method Name

The method name that identifies this DID method SHALL be: `trust`

A DID that uses this method MUST begin with the following prefix: `did:trust`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is the method identifier described below.

## DID Trust Method Identifiers

The Trust DID Method DID identifier has three components that combine to make an identifier that conforms to the DID specification. The components are:

- **DID Trust Method**: the hardcoded string did:trust: indicating that the identifier uses this DID Method specification.
- **DID Trust Namespace**: a string that identifies the unique trust ledger namespace of the DID, with a single and a subsidiary name. For example `tc:prod`.
- **Namespace Identifier**: two-element identifier unique to the DID Trust namespace. The first parameter describes the type of the DID as an internal namespace. The second, required element is an identifier that is unique within the namespace.

### Delegation

The DID has to be read from left to right where it will be separated into the three parts: DID method, DID namespace and identifier namespace. All parts are required to have a valid DID. There are no default values that will be added since the separator between the elements and sub-elements is equal.

The first part will be handled by the agent. It has to look up the DID resolver and pass the DID to it.

The next part is handled by the resolver to detect which blockchain network has to be used. The resolver has to send the DID to the known blockchain nodes to get the required transactions to resolve the DID document.

The blockchain nodes will look up based on the namespace which module is responsible for handling this DID type and return the requested transactions that are related to the DID.

### DID Trust Namespace

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
    "observers": ["http://obs1.example.com"],
    "portals": ["http://por1.example.com", "http://por2.example.com"],
  },
  "foo:prod": {
    "observers": ["http://p.foobar.com"],
    "portals": ["http://o.foobar.com"],
  }
}
```

Since there is no requirement to host central network name resolver, a known connection can be used to ask a network
- if it knows the connection endpoints to a specific trust namespace
- if it knows more endpoints for the nodes for writing and reading to increase redundancy

Implementing a global register could be one way to reduce complexity and would make network discovery automatically. This would also prevent multiple tust namespaces with the same identifier. But on the other hand a single point of failure is created. So when creating a new network the identifier has to be shared with other existing networks that have to accept the connection between the choosen trust namespace and the network.

### Namespace Identifier

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

### Default Namespaces

The trust DID method does not support default namespaces right now. This is because of the separator `:` is used for internal separation (`tc:prod`) and joining the components (`did:trust:tc:prod:id:XLzBJ69keqEgq7oqqdEsHW`). When skipping an element like the network component, like in this case `did:trust:tc:id:XLzBJ69keqEgq7oqqdEsHW`, the resolver does not know if the `id` should be the network and the node will use the default identifier namespace or not. Not having the `tc:id` locally registered does not mean the network does not exist. There could be another network out there that is defined with this network namespace.

Using one more separator like `-` would solve this problem, but we suggest to use ALWAYS full names since
- The DID documents always include the full names because of the signature process
- The definition of choosing the default value has to be globally defined for all networks

# Operations

## Create

The creation of DIDs can only be done by an issuer that already has a DID in the same trust namespace. Depending on the DID usage in the context the level of authority is defined:
- Only validators of ring one are able to create DIDs for members of ring two
- Only observers of ring two can create DIDs for members of ring three
- Members of ring three are allowed to create DIDs on ring three if they are authorized

A transaction that was signed by an unauthorized member will be rejected on the way to the consensus (either by bypassing an observer or a validator).

It is possible to define rules when creating a DID. The blockchain is able to check:
- If multiple signatures are required
- If the request implements the required values by checking against a schema. Validation is done with a whitelist protocol (only defined values in the schema are allowed)
- If the ID of the DID already exists

```sh
curl -X 'POST' \
  'https://api.observer1.example.com/did/did' \
  -H 'accept: application/json'
```

The create operation is wrapped up in a transaction that will be persisted in the blockchain and can look like this:
```json
{
        "version": 1,
        "body": {
            "date": "2021-04-28T07:00:51.008Z",
            "value": {
                "id": "did:trust:tc:dev:id:WovvD9TbdR5VvQ1aqRn2GK",
                "verificationMethod": {
                    "add": [
                        {
                            "controller": "did:trust:tc:dev:id:WovvD9TbdR5VvQ1aqRn2GK",
                            "id": "did:trust:tc:dev:id:WovvD9TbdR5VvQ1aqRn2GK#EWNKD6We6azLG5v5GEcrE3UzSkuV7FWX487JKsVmkhmg",
                            "type": "RsaVerificationKey2018",
                            "publicKeyJwk": {
                                "key_ops": [
                                    "verify"
                                ],
                                "ext": true,
                                "kty": "RSA",
                                "n": "oYi0e03uAB6BDgz_HNKztv33LMhM82qAxjH4Dqu23nboC4uzfjxwqaH4xe_Wllysl6XpP8yXzJH3D4o7QO_Cy16NuFIq23-hRg51PAI7ZdmDvDI_c41s98kt2tpibO-sd8IwfQG1Kujs6NSUrhuXrNnOzT_Cwit_srOjFL27aPyH5BYBAqp3sAfMd61eNitZhtBL0SqZVjif4Sz8Cl4zFsg-oN247uilLr2UKnIf8LcYoDnmCyM2tHvESzRbNilopAhFww45jzleQJD3nFAcETQnE5NEFbg8862cuJb5HIOv1OwQExjwJMikFYuAM9XSybue-Px5Ht6vkCr7K9TYIQ",
                                "e": "AQAB",
                                "alg": "RS256"
                            }
                        }
                    ]
                },
                "authentication": {
                    "add": [
                        "did:trust:tc:dev:id:WovvD9TbdR5VvQ1aqRn2GK#EWNKD6We6azLG5v5GEcrE3UzSkuV7FWX487JKsVmkhmg"
                    ]
                }
            },
            "didDocSignature": {
                "type": "single",
                "values": [
                    {
                        "signature": "5r14PpyDfju8KPtdxwQnmjELW1suGJ7eMDfFqdJjkmLHwaAmgtqirG4vndUiDmjTzXZMWMdjerKD8uFdxDxvBZnDsj52eYaj9z7SxJuhSNU5qcMSNWuttdKzciz5jum2dBAc8iVsZ4Zww6z46BsXyj9FrPXD21yCdokseKpRxoyy2BmKiN5RxGNNdTriG2THXBrPSg4TNsyz3Ka5cGPbFdreauMf2kEhuVrDT6HhSkkdia3YxvtJxGC6NWUzCUSVC2RRgeungqL5D6HGmmoz7hmtBkv5e1Fq9sevqFu4bS9f9fvvSEEYJggaouPDmn9P98bg667aGaCTfUR46fTyGcQcVhTvmZ",
                        "identifier": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#GyFw9t8GGZxQbs1dz8HPPcJ5nUWHMWpXC1xJQ9KSN5Yf"
                    }
                ]
            },
            "type": "did",
            "version": 1
        },
        "metadata": {
            "version": 1
        },
        "signature": {
            "type": "single",
            "values": [
                {
                    "signature": "4tkSjpWP62uHQubMiou1rUxE1HP7a6aKF9dQsjFuWEASgn2VZv33XhhwczcKQjYoYk76g1M6FXDBq241Seip6BsaPGTMy6tiKUGScuc5KXXTuMsWD2KzmCheerSrfHgFLWwiJvcUjh6btEd5Lp4QA8VbMCdC7kgH4C6Uiu8ddPxc3CjU3NCvd2iHeEkT3RxNyNWKFxsxM7MsezPC3MdrMjNNVBg76AR2kkGBcz9iDRsLszx8gisvAwAFhwhLzt5jLS1KDuibm2wm29ity1977tdkx8NkgxtEuhp7pFeY156vuXrQ2sxVbUWJ2Qy3NCMsuBTQ6je5hdAhkmZAqTW37LjeQcvTVd",
                    "identifier": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#GyFw9t8GGZxQbs1dz8HPPcJ5nUWHMWpXC1xJQ9KSN5Yf"
                }
            ]
        }
    }
```

This is a minimal version to create a DID with a registered authentication key. Two signatures are required to support different read operations. The signature on the root level is the transaction signature. It signs the changes that are inside the transaction. The `body.didDocSignature` value is the signature of the document that is the result by parsing all transactions including this one.

## Read

### Request transactions
A portal endpoint can be requested to get the transactions to assemble a DID document locally. The request includes the DID and filter options for versioning:
- Version time: Returns all transaction from the beginning until the given timestamp
- Version number: Returns all transactions which have a lower version number. The transactions are chronologically ordered so their version number is incremented with each transaction
If no filter is passed all transactions that affect the DID document will be returned. If both filters are passed, the one corresponding to an earlier date is used. In case the DID is not found an empty array is returned.

Here is an example request for all transactions related to the DID `did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW`:

```
curl -X 'GET' \
  'https://api.portal1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW' \
  -H 'accept: application/json'
```

And the corresponding response:

```json
[
  {
    "signature": [
      {
        "signature": "7CuzvzqK2nWKRTyirueWF1aUYVLXqftYVVXA8P5wt8MpRVeZ9U6xtpkC8oGk4RwuPG8yyqY2H6TJS7GSXb41yKhNL2z3QDnBFvYj94iC9sBiTXyFsT6Tj62k4adUvZogM3mttFqt5D35c5dcKoS1R6QnTEgj98U6AzJS3VpfzxcwPwwaDuFHgR5nhA22bNK3anHhmhjeMPzghwQP2d6xwMD1BbDrzbma4kWoCxFggchHBgob8kFP1op4fTh1TfZeSbbDJ7T2pLJ1947nWMAvNPT7SxAacLuhbSZqUNqCAZAqm9cfogwg7v8hNrqV2YQjDFstdVr6cAFwz84Skc31LJoRFSnGG1",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ],
    "block": {
      "id": 3,
      "createdAt": "2021-04-27T15:33:57.611Z",
      "validators": [
        "did:trust:tc:dev:id:9hAiG2jU2d4MvztmLUiEjj#3ykwjbDPk6aAEpFTrU5AGhZ1GG91bAW9HHM7fpT44Rd2",
        "did:trust:tc:dev:id:2snYM2GNHLMgHa1XaAZRzA#8M8KkVFzwjZo4fMpwE8bBf5v77giiXx2Z9T3bVhXwzxH",
        "did:trust:tc:dev:id:AyTae9NpVbcAi2f7tqEQqF#DQ8csLNyUuh1PaZvuNDfzUVy3QDNFhawJryz2UgNiVgM",
        "did:trust:tc:dev:id:CayMsTz2K9tf5FwU3Fwegp#Wo1xaJSCB4jrk1jbqrjH8h4yADfeA4ANovWBbxDEoYx"
      ]
    },
    "index": "7gfXiL2LGix7LdCZyKNKW1CYPFpdXMYXcbhGXfvHPNhi",
    "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
    "createdAt": "2021-04-27T15:33:56.274Z",
    "values": {
      "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
      "role": {
        "add": [
          "client"
        ]
      },
      "service": {
        "add": [
          {
            "id": "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver",
            "type": "resolver",
            "endpoint": "https://api.observer1.dev.trustcerts.de/did/resolve/did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW"
          }
        ]
      },
      "authentication": {
        "add": [
          "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#8tcAAEJwekzxY25JxSwEWFQ28Mr64LJnxdTrF93Lsprt"
        ]
      },
      "verificationMethod": {
        "add": [
          {
            "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#8tcAAEJwekzxY25JxSwEWFQ28Mr64LJnxdTrF93Lsprt",
            "type": "RsaVerificationKey2018",
            "controller": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
            "publicKeyJwk": {
              "e": "AQAB",
              "n": "zftnecVYTcxW5vKBRVHU7TkY19Xjf2DsnDOnOQikHpC0oAMRII7mqU0DUoeofU9w8QkfbuT_e4gZzfpQDIhhAKm_h8_kHpDv3Tb7BLgAD3teevnV7LcU6oM7IN4BpxwkAK-EGBrZoKAxnLfuf_KuVnAYwi_0wvHPQBLciVOin3MdF-PWSJp-bJ5fqVyEyIVhCwcIDmX7QhRaRyho38QLm3lxM_2gf2Fwn-MEyxlDzrBFgvmBM3s9XImuw11dihzqxZb3wVktprVwL73BQ2weOEb-oG2MgfH3GiGwU0BFf04DeEa6LzWUGgDBAQNNN62aPA5DQk0oMHzwO1rdhIiZdQ",
              "alg": "RS256",
              "ext": true,
              "kty": "RSA",
              "key_ops": [
                "verify"
              ]
            }
          }
        ]
      }
    },
    "didDocumentSignature": [
      {
        "signature": "9ofJG4kdHLGxR32YPdUjTTJnrS98nmivnVs8nh98wCbbfdqki2DvXAoiMx8tSNN12NWin9Q6PRSVUp9Gv393gs38gUeiYJvBXAXgNbEYoVVGRkxiJ3Kf15yyVwwHZtGhV5XLbiLymexqmywb4dVW4JcqZYaqjw9g9nuXVfXdzsGbjNNh57oTjmXFAcaG5xC37mdAJqnaHNVBho3hpDrP2Zh3DPptBoV1gtJQDAGgYQnzF6NofuNC75EyQHn6hyF1UN1RHxZmGJATVgWJfwwNKQVAGsgD7KpWvHiUfmSdJvHbPNKpeqeSPVSnK2oUtowQEhwBs1ev12tdxnAHDsRiN48pNQkooG",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ]
  },
  {
    "signature": [
      {
        "signature": "8LLkmL5Xob7zM34QB2rWKiKTuvdyEsU7TRx3Ny57GVcVCNcv8TbTZK96SrJtfcN8yAaM92wAu4n47MDM7sECiHUNrzkMMoCk4QFHabemTzd7uwZX1vpU4kQrKEvn49ENdbrqMbLwp8cijuTZgGvdfScNmVnQ2VNpgarTP22eLyhMpmRFD39bNPdue6VpTnsk9tPEcq2qqvAbkBjzFGCeiwxJub3tDBmCnCdp942yxDSMYKbLKxXk3Yyq13vMEHkp7gotPpQZprMeS4MvWRxVXwzAQRaZ4z86yNq89mofXEANqnZUxAXbvQrrFKP68o2evvpXw7LrVSnJqAa6wCnSyPKCDbHJwW",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ],
    "block": {
      "id": 7,
      "createdAt": "2021-04-27T15:34:29.673Z",
      "validators": [
        "did:trust:tc:dev:id:9hAiG2jU2d4MvztmLUiEjj#3ykwjbDPk6aAEpFTrU5AGhZ1GG91bAW9HHM7fpT44Rd2",
        "did:trust:tc:dev:id:2snYM2GNHLMgHa1XaAZRzA#8M8KkVFzwjZo4fMpwE8bBf5v77giiXx2Z9T3bVhXwzxH",
        "did:trust:tc:dev:id:AyTae9NpVbcAi2f7tqEQqF#DQ8csLNyUuh1PaZvuNDfzUVy3QDNFhawJryz2UgNiVgM",
        "did:trust:tc:dev:id:CayMsTz2K9tf5FwU3Fwegp#Wo1xaJSCB4jrk1jbqrjH8h4yADfeA4ANovWBbxDEoYx"
      ]
    },
    "index": "HCBRcfRumBuKTHV6V79vYoSaWifcLqVJBCWtDKZhrE7R",
    "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
    "createdAt": "2021-04-27T15:34:28.337Z",
    "values": {
      "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
      "authentication": {
        "add": [
          "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#CgpWSVaL8jAsyqvvuN3B76mnRAyR49xXpZif4cPeJq79"
        ],
        "remove": [
          "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#8tcAAEJwekzxY25JxSwEWFQ28Mr64LJnxdTrF93Lsprt"
        ]
      },
      "verificationMethod": {
        "add": [
          {
            "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#CgpWSVaL8jAsyqvvuN3B76mnRAyR49xXpZif4cPeJq79",
            "type": "RsaVerificationKey2018",
            "controller": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
            "publicKeyJwk": {
              "e": "AQAB",
              "n": "12pOMH2d1z7tI6xbJjh9gevpetAC4TINhJnMBH0DHQNGHsCZCyN2dkwhT_jlRVhjwPzWwSTUVOil-2eDfGxp_NMomLkBAoGUQrNkVwwu1xW-4QEBU0XlVJqkByi0mssLjd5R-PjW4Q6giaqlCpiZ6a5459ojgQ9vShuWAdRo5NCp9eqP-XyvgBSpXmCUtZHhhXv5nEdb-KQMJQODcpbpkhYi9ihSM5MxmHphgUapnJ9OPyMwlrVa8MpsNBtWDlPZnlpDe7J8DptkGWzMb47kv9_iHj6pzYQXzTjPxpYzmCqrphCs_U5oikbUdfxlpO8Zke2iGZJqhMQZQ0LsbYPHcQ",
              "alg": "RS256",
              "ext": true,
              "kty": "RSA",
              "key_ops": [
                "verify"
              ]
            }
          }
        ],
        "remove": [
          "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#8tcAAEJwekzxY25JxSwEWFQ28Mr64LJnxdTrF93Lsprt"
        ]
      }
    },
    "didDocumentSignature": [
      {
        "signature": "3NXMJFpxQ7TemAfigD4XcGLBwj286hSzw6NfBSFDWh24rcVZLEBMLsNGnTw3bicmgbciVpiwN72CxnkqUcMZtELtqPJQdntf7uRPCkqRHJQzzE5URSFJA3sBu4pvxQSKK7MJoDzVLJqajmBarSWFdHPpBWHucPvjrh2QS1GAxdcmH1RAY9dDuYbm7yqZKBWaCo6sSDoqGuEQt4HAyjEMDGZDUbqTGFNCHPPPUiP1iN1x5UnhKgzRhv2esz7h34kDPcMgsJeEkaCzKC9VCxREpmdCnBbVzNZS4ifCz2Gz3v88hNFPWVaCfY5ka9ZQLUPRgwFbLbTNvGp5mbkDjVgrAqUuK1JBHf",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ]
  }
]
```

All transactions have to be parsed one after another. During this procedure the client has to request multiple DIDs from the system since the chain of trust includes multiple issuers:
- The validators that created the blocks in which the transactions are stored
- The creator of the DID, e.g. an observer if the requested DID belongs to a client
- The creator of the creator

By querying all transactions the client is able to generate each previous version of a DID document since all transaction are available. When updating the DID document to the latest version, the client only has to request the missing transaction. The required information can be received via a meta data request to get more information about the DID document.

### Request DID document

Instead of querying multiple transactions that have to be validated and assembled, the client can request an already assembled DID document from the blockchain.

The request includes the DID and filter options for versioning:
- Version time: Returns the DID document as it existed at the given timestamp
- Version number: Returns the DID document corresponding to the given version number
If no filter is passed, the latest version of the DID document is returned. If both filters are passed, the one corresponding to an earlier date is used.

Here is an example request:

```sh
curl -X 'GET' \
  'https://api.portal1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW/doc' \
  -H 'accept: application/json'
```

And the corresponding response:

```json
{
  "document": {
    "@context": [
      "https://www.w3.org/ns/did/v1"
    ],
    "id": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY",
    "controller": [],
    "verificationMethod": [
      {
        "id": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv",
        "type": "RsaVerificationKey2018",
        "controller": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY",
        "publicKeyJwk": {
          "e": "AQAB",
          "n": "3VETQUKUJZsEqXIQTUz0fJs9ubjGUwbnp8-lCMIimFj0dYnaq1lrHRO7laF2q-FwnUjQp2fgdvy_6N4I4RivqCfBOE7efr9VBFooZkAeWZd356exb-tfSEhdIUX45OB8KkNn0hpXfSXhiIzyznph3w6x_ez0j8GS9QoC9y8Uov-kT763j08IL9UhAumsvXOqKHektmPZ_ljURjAG9jCidk32GeawM0xAGsejdVIfBRKvdFUEFLkeFNY8TbD7gkZ2NJJJ-4r4JENrY3FrtECZ67AKV7yNOVmIj0w9eVvHJq9rH4MnZY3sBrRhbCcOmvy4sRbEEIhintZvHW0zA1RWvw",
          "alg": "RS256",
          "ext": true,
          "kty": "RSA",
          "key_ops": [
            "verify"
          ]
        }
      }
    ],
    "authentication": [
      "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
    ],
    "service": [
      {
        "id": "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver",
        "type": "resolver",
        "endpoint": "https://api.validator1.dev.trustcerts.de/did/resolve/did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY"
      }
    ],
    "assertionMethod": [],
    "role": [
      "gateway"
    ]
  },
  "signatures": [
    {
      "signature": "8JihByeJHCKbuL8Aw7jUiRNAG7oHyiEySGfwsJtXU7oXVZMGPpPw5h7E8tieiv4ZvzBcgjbzkCP16ezRRidgoWkW7i3Zm9TgZmn4zYc8eaLV47NiX13NEDZJvsoCWrnVUCevvsG5bGeWXW2xt3NqF1rN4p4qPNJFV5txXbVvuR8Ko7xGm6qbpKq8Nq1rP9tzTNzhc7M6hNjNCtbKFMmperF5VC5na1NFoNDhPYN9DzYkbtQ5Mi9kEpVNLULhqKh6ujEYuAHxFb4w9G5BZEsYc9iJnMtq3o9TuwFpa1crqqaXSFvdR9edmtJKzAfrBBgoTNTiLxGX45sfn6AKpeUSnyKu5on5bV",
      "identifier": "did:trust:tc:dev:id:9hAiG2jU2d4MvztmLUiEjj#3ykwjbDPk6aAEpFTrU5AGhZ1GG91bAW9HHM7fpT44Rd2"
    }
  ]
}
```

Each transaction that changes the DID document also includes a signature about the latest compiled version of the DID document. This allows the client to get a valid DID document by just the validation one signature of this DID document. To guarantee the chain of trust the DIDs of the validators and other issuers have to be validated too, but the amount of validations is reduced in comparison to the transaction based approach.

### Request meta data

The metadata of a DID document can be requested by a separate endpoint to get more information:

Request:
```sh
curl -X 'GET' \
  'https://api.portal1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW/metadata' \
  -H 'accept: application/json'
```

Response:

```json
{
  "created": "2021-04-27T15:33:56.274Z",
  "versionId": 4,
  "updated": "2021-04-27T15:36:50.062Z",
  "nextUpdate": "2021-04-28T06:58:43.681Z",
  "nextVersionId": 5
}
```

The `nextUpdate` and `nextVersionId` are only present if a filter was passed and the result does not correspond to the latest version of the DID document.

## Update

The update process is equal to the creation process. The client creates a transaction that consists of the changes to the DID document, rather than passing a new DID document to replace with.

If an entry should be updated with the same id, it has to be deleted and added in the same transaction. Since it's not possible to have keys with the same identifier, the parsing algorithm will first process the delete operations of a transaction and then the execute the add operations.

This is how an example update request can look like: 

```sh
curl -X 'POST' \
  'https://api.observer1.example.com/did/did' \
  -H 'accept: application/json'
```

```json
{
    "version":1,
    "body":{
        "date":"2021-05-04T06:27:14.668Z",
        "value":{
            "id":"did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
            "service":{
                "add":[
                    {
                        "id":"did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver",
                        "type":"resolver",
                        "endpoint":"https://api.observer1.dev.trustcerts.de/did/resolve/did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW"
                    }
                ],
                "remove":[
                    "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver"
                ]
            }
        },
        "didDocSignature":{
            "type":"single",
            "values":[
                {
                    "signature":"9ofJG4kdHLGxR32YPdUjTTJnrS98nmivnVs8nh98wCbbfdqki2DvXAoiMx8tSNN12NWin9Q6PRSVUp9Gv393gs38gUeiYJvBXAXgNbEYoVVGRkxiJ3Kf15yyVwwHZtGhV5XLbiLymexqmywb4dVW4JcqZYaqjw9g9nuXVfXdzsGbjNNh57oTjmXFAcaG5xC37mdAJqnaHNVBho3hpDrP2Zh3DPptBoV1gtJQDAGgYQnzF6NofuNC75EyQHn6hyF1UN1RHxZmGJATVgWJfwwNKQVAGsgD7KpWvHiUfmSdJvHbPNKpeqeSPVSnK2oUtowQEhwBs1ev12tdxnAHDsRiN48pNQkooG",
                    "identifier":"did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
                }
            ]
        },
        "type":"Did",
        "version":1
    },
    "metadata":{
        "version":1
    },
    "signature":{
        "type":"single",
        "values":[
            {
                "signature":"7CuzvzqK2nWKRTyirueWF1aUYVLXqftYVVXA8P5wt8MpRVeZ9U6xtpkC8oGk4RwuPG8yyqY2H6TJS7GSXb41yKhNL2z3QDnBFvYj94iC9sBiTXyFsT6Tj62k4adUvZogM3mttFqt5D35c5dcKoS1R6QnTEgj98U6AzJS3VpfzxcwPwwaDuFHgR5nhA22bNK3anHhmhjeMPzghwQP2d6xwMD1BbDrzbma4kWoCxFggchHBgob8kFP1op4fTh1TfZeSbbDJ7T2pLJ1947nWMAvNPT7SxAacLuhbSZqUNqCAZAqm9cfogwg7v8hNrqV2YQjDFstdVr6cAFwz84Skc31LJoRFSnGG1",
                "identifier":"did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
            }
        ]
    }
}
```

And the corresponding response:

```json
[
  {
    "signature": [
      {
        "signature": "7CuzvzqK2nWKRTyirueWF1aUYVLXqftYVVXA8P5wt8MpRVeZ9U6xtpkC8oGk4RwuPG8yyqY2H6TJS7GSXb41yKhNL2z3QDnBFvYj94iC9sBiTXyFsT6Tj62k4adUvZogM3mttFqt5D35c5dcKoS1R6QnTEgj98U6AzJS3VpfzxcwPwwaDuFHgR5nhA22bNK3anHhmhjeMPzghwQP2d6xwMD1BbDrzbma4kWoCxFggchHBgob8kFP1op4fTh1TfZeSbbDJ7T2pLJ1947nWMAvNPT7SxAacLuhbSZqUNqCAZAqm9cfogwg7v8hNrqV2YQjDFstdVr6cAFwz84Skc31LJoRFSnGG1",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ],
    "block": {
      "id": 3,
      "createdAt": "2021-04-27T15:33:57.611Z",
      "validators": [
        "did:trust:tc:dev:id:9hAiG2jU2d4MvztmLUiEjj#3ykwjbDPk6aAEpFTrU5AGhZ1GG91bAW9HHM7fpT44Rd2",
        "did:trust:tc:dev:id:2snYM2GNHLMgHa1XaAZRzA#8M8KkVFzwjZo4fMpwE8bBf5v77giiXx2Z9T3bVhXwzxH",
        "did:trust:tc:dev:id:AyTae9NpVbcAi2f7tqEQqF#DQ8csLNyUuh1PaZvuNDfzUVy3QDNFhawJryz2UgNiVgM",
        "did:trust:tc:dev:id:CayMsTz2K9tf5FwU3Fwegp#Wo1xaJSCB4jrk1jbqrjH8h4yADfeA4ANovWBbxDEoYx"
      ]
    },
    "index": "7gfXiL2LGix7LdCZyKNKW1CYPFpdXMYXcbhGXfvHPNhi",
    "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
    "createdAt": "2021-04-27T15:33:56.274Z",
    "values": {
      "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
      "service": {
        "add": [
          {
            "id": "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver",
            "type": "resolver",
            "endpoint": "https://api.observer1.dev.trustcerts.de/did/resolve/did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW"
          },
        ],
        "remove": [
          "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver"
        ],        
      },      
    },
    "didDocumentSignature": [
      {
        "signature": "9ofJG4kdHLGxR32YPdUjTTJnrS98nmivnVs8nh98wCbbfdqki2DvXAoiMx8tSNN12NWin9Q6PRSVUp9Gv393gs38gUeiYJvBXAXgNbEYoVVGRkxiJ3Kf15yyVwwHZtGhV5XLbiLymexqmywb4dVW4JcqZYaqjw9g9nuXVfXdzsGbjNNh57oTjmXFAcaG5xC37mdAJqnaHNVBho3hpDrP2Zh3DPptBoV1gtJQDAGgYQnzF6NofuNC75EyQHn6hyF1UN1RHxZmGJATVgWJfwwNKQVAGsgD7KpWvHiUfmSdJvHbPNKpeqeSPVSnK2oUtowQEhwBs1ev12tdxnAHDsRiN48pNQkooG",
        "identifier": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
      }
    ]
  }
]
```

Updates can be done by the subject of the DID document for e.g. a key rotation or done by the authority above the subject. For example every member of ring two can update a DID document from ring three. This is required if someone in ring three lost their private key for a key rotation to prevent a lock out. The rules that define which entity can update which parts of the DID document can be defined in a policy that will be evaluated by the blockchain nodes.

## Deactivate

A DID document is considered deactivated when it does not include any `verificationMethod` to execute an update method on itself. A DID can be restored by an authority by creating a new `verificationMethod` inside the DID document.

```sh
curl -X 'POST' \
  'https://api.observer1.example.com/did/did' \
  -H 'accept: application/json'
```

# Privacy Considerations

## Surveillance

If a DID is known, any person can resolve it into its DID document. The content of the DID document and changes can be monitored. Only blockchain operators have access to all transactions for analysis purposes. Companies using this DID method can alternatively use a private permissioned blockchain.

## Stored Data Compromise

Compromise of stored data on the ledger is prevented by the chain of trust, consensus, and ledger mechanisms.

Caching: If the transactions were validated before caching the final DID document, a malicious actor could manipulate the DID document afterwards (e.g. service endpoints). To prevent this from happening multiple parallel queries should be performed on the respecting DID.


## Unsolicited Traffic/Intrusion

If a DID is known, it can be resolved to a DID document. This means that all service endpoints that can also be viewed could be exposed to unsolicited traffic . However, no service endpoints need to be specified in the DID document.

## Misattribution

DID duplicates are prevented from being created by validating the incoming DID transaction against the database to find double entries. In case of a race condition where two new transactions should be persisted with the same identifier, the transactions will be checked one after another when proposing a new block.

## Correlation

The risk of correlation is analogous to the points raised in [10.2 DID Correlation Risks and Pseudonymous DIDs](https://www.w3.org/TR/did-core/#did-correlation-risks-and-pseudonymous-dids) and [10.3 DID Document Correlation Risks](https://www.w3.org/TR/did-core/#did-document-correlation-risks) of the W3C. Possible correlation factors could be the specified controller, cryptographic material or service endpoints inside the DID document.

In addition, the versioning of the DID document provided in this DID method can allow correlation by factors that existed in previous versions of the DID document.

## Identification

The direct identification of a person by his DID in this DID method is not given, because the namespace identifier is a randomly generated alphanumeric sequence. Since a DID document does not contain personally identifiable information (PII), no direct identification is possible. But it should be considered that an identification via correlation is always possible.

## Secondary Use

Information that can be viewed publicly in DID documents is constantly at risk of being misappropriated.  E.g. the specified controller, cryptographic material or service endpoints. In this DID method, no personal data is written to the DID document.

## Disclosure

Only things that were intended to be public in a DID document can be disclosed such as verification methods and service endpoints. Users need to be aware that all information specified in the DID document is visible (even older contect due to versioning of the DID document).

## Exclusion

Critical or sensitive information should be located behind service endpoints protected by authentication mechanisms, as the DID document can be publicly viewed by anyone.


# Security Considerations

## Attacks

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

## Residual risks

External libraries are used in this DID method. Any security issues in these libraries also affect this DID method. Also, if the end device is compromised, there is also a residual risk outside the scope of this DID method. 


## Integrity protection and update authentication for method operations

All operations that change the state of a DID (create, update, deactivate) require authentication. This is done by signing the transaction body, the blockchain can specify how many or what kind of signatures are required to sign a DID in a specific context. For a DID with a higher privilege a multi signature from two other members can be a requirement.  
The signature validation is done before validators are persisting the transaction into a block and when a client requested transactions or objects from the blockchain system.

## User-host-authentication

User authentication for writing on the ledger is realized by signing transactions using a private/public key pair, where the authorized public keys are stored on the ledger. There is no kind of authentication required for reading operations.

## Unique DIDs

DIDs are proven to be uniquely assigned by checking for duplicate transactions at both block generation and block validation time.

## Network topology

Clients inside ring 3 interact with observers and portals inside ring 2, who are responsible for reading and writing to the ledger. The clients have no direct connection to the validators in ring 1, who are responsible for generating blocks. A resolver itself can also be regarded as a proxy that can be used for caching. There is no authentication required for reading from the ledger.

## Cryptographic protection mechanisms

Transactions are checked for integrity by only allowing specified transaction formats. Transactions are cryptographically signed to ensure integrity.

## Private data

You MUST keep your private keys private in order to prevent unauthorized writes to the ledger.

## Signatures on DID documents

When resolving a DID document, you SHOULD verify the node's response by validating the signatures of the complete transaction history back to the genesis block to ensure the validity of the retrieved DID document via the chain of trust. If this is not possible or feasible, you SHOULD compare the results of querying multiple – or ideally all – nodes of the network.

## Peer-to-peer computing resources

On the DLT side each node is able to scale the http servers so they can handle multiple read requests from the clients.


# Future Works
## DID Update Policy

Abuse of DID Controller with the hierarchy... (Lock Out). Changes from another stage to a stage below are only possible with multiple signatures from other members to remove the single point of failure.
