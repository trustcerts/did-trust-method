## Operations

### Create

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
              "key_ops": ["verify"],
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

### Read

#### Request transactions

An observer endpoint can be requested to get the transactions to assemble a DID document locally. The request includes the DID and filter options for versioning:

- Version time: Returns all transaction from the beginning until the given timestamp
- Version number: Returns all transactions which have a lower version number. The transactions are chronologically ordered so their version number is incremented with each transaction
  If no filter is passed all transactions that affect the DID document will be returned. If both filters are passed, the one corresponding to an earlier date is used. In case the DID is not found an empty array is returned.

Here is an example request for all transactions related to the DID `did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW`:

```
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW' \
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
        "add": ["client"]
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
              "key_ops": ["verify"]
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
              "key_ops": ["verify"]
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

#### Request DID document

Instead of querying multiple transactions that have to be validated and assembled, the client can request an already assembled DID document from the blockchain.

The request includes the DID and filter options for versioning:

- Version time: Returns the DID document as it existed at the given timestamp
- Version number: Returns the DID document corresponding to the given version number
  If no filter is passed, the latest version of the DID document is returned. If both filters are passed, the one corresponding to an earlier date is used.

Each transaction that changes the DID document also includes a signature about the latest compiled version of the DID document. This allows the client to get a valid DID document by just the validation one signature of this DID document. To guarantee the chain of trust the DIDs of the validators and other issuers have to be validated too, but the amount of validations is reduced in comparison to the transaction based approach.

Here are example requests for different types of DID documents:

##### DID ID Document

Sending a request to resolve `did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY`:

```sh
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW/doc' \
  -H 'accept: application/json'
```

And the corresponding DID ID Document response:

```json
{
  "document": {
    "@context": ["https://www.w3.org/ns/did/v1"],
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
          "key_ops": ["verify"]
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
    "role": ["gateway"]
  },
  "signatures": [
    {
      "signature": "8JihByeJHCKbuL8Aw7jUiRNAG7oHyiEySGfwsJtXU7oXVZMGPpPw5h7E8tieiv4ZvzBcgjbzkCP16ezRRidgoWkW7i3Zm9TgZmn4zYc8eaLV47NiX13NEDZJvsoCWrnVUCevvsG5bGeWXW2xt3NqF1rN4p4qPNJFV5txXbVvuR8Ko7xGm6qbpKq8Nq1rP9tzTNzhc7M6hNjNCtbKFMmperF5VC5na1NFoNDhPYN9DzYkbtQ5Mi9kEpVNLULhqKh6ujEYuAHxFb4w9G5BZEsYc9iJnMtq3o9TuwFpa1crqqaXSFvdR9edmtJKzAfrBBgoTNTiLxGX45sfn6AKpeUSnyKu5on5bV",
      "identifier": "did:trust:tc:dev:id:9hAiG2jU2d4MvztmLUiEjj#3ykwjbDPk6aAEpFTrU5AGhZ1GG91bAW9HHM7fpT44Rd2"
    }
  ]
}
```

##### DID Schema Document

Sending a request to resolve `did:trust:tc:dev:sch:6BquEL34bKFVcCbv5NmZdF`:

```sh
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Asch%3A6BquEL34bKFVcCbv5NmZdF/doc' \
  -H 'accept: application/json'
```

And the corresponding DID Schema Document response:

```json
{
  "document": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:trust:tc:dev:sch:6BquEL34bKFVcCbv5NmZdF",
    "controller": ["did:trust:tc:dev:id:XLzBJd69tqEgq7oqqdEsHW"],
    "value": "{\"type\":\"object\",\"properties\":{\"name\":{\"type\":\"string\"},\"random\":{\"type\":\"string\"}},\"required\":[\"name\",\"random\"],\"additionalProperties\":false}"
  },
  "signatures": {
    "type": "Single",
    "values": [
      {
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#2kL2biA5pC76HPdkUdHQwSqLWntLRt31aoHM8reR2RoS",
        "signature": "7mfWkvdgTE8KeRZdcc7SgvD1k9m1KnKLFC2X3z5F7xAbV9sYjZTNp9DFGnyyJ8ai4VRBPMpto4XyEd2EYJx4NuNoTSecii2aib5xRHj7FuXjABC3um7UyULWXcGmsrdJ6NeBBW88Lm26es6NiHq9amGVZV8LRwiFctZCyKD1Ani2p4P2rJeh4UxTDpXt9ArnUkt6WwEZeYe7B26gzaB33MWxXe6DgFMeYHw9Fe4Gc6C7eZ5NXMQpLQxjEgbPxWaPTwLjzEoZK3Ko58jJc3Ma9tpiVU86JCLCzbunuiJ5QwM58aGMJnhsdMiueMupRvsMgdeH1mMsTRvAFgjW3M4Pnioi6Q2eqR"
      }
    ]
  },
  "metaData": {
    "created": "2022-08-01T09:48:17.226Z",
    "versionId": 1
  }
}
```

##### DID Template Document

Sending a request to resolve `did:trust:tc:dev:tmp:BKqANFfATgZSwBA6UzzE2k`:

```sh
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Atmp%3ABKqANFfATgZSwBA6UzzE2k/doc' \
  -H 'accept: application/json'
```

And the corresponding DID Template Document response:

```json
{
  "document": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:trust:tc:dev:tmp:BKqANFfATgZSwBA6UzzE2k",
    "controller": ["did:trust:tc:dev:id:XLzBJd69tqEgq7oqqdEsHW"],
    "compression": {
      "type": "JSON"
    },
    "template": "<h1>Hello {{ name }}</h1>",
    "schemaId": "did:trust:tc:dev:sch:DfcyDKKSgXgEuUGQsP8reV"
  },
  "signatures": {
    "type": "Single",
    "values": [
      {
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#2kL2biA5pC76HPdkUdHQwSqLWntLRt31aoHM8reR2RoS",
        "signature": "8mw5LbnqMU1j5ce5npacayWkpfmYuxGLKkd4RmabnczPTfk6yGVhTc4ApL7WwsTdKwkDvYm4HHT9TBCvDrusfRQNwbgoLey1t9F7gqFCgSSkTrC6yRH1SmsTMXDrcuBvdAe1stEzYQaSBszsMLbFAtnaRXGRPYLS3PGR8oZFSU5HYD9yGsALgTVoxMpY1yDkSbCynv6UKW2r9b9tmMn9MgyFUSvyXm92JzCwg61gTcXipVWEN8GrMMcrw2WuyvtYtRzcWdm9Jpssx1KnacjeEjm52aC3WtPUCzWGBnHMVH3KcqnY9f4vvZh1AhyG3qujGcFfbD82o8BonB6qsqETyJfjazsZxh"
      }
    ]
  },
  "metaData": {
    "created": "2022-08-01T09:52:25.195Z",
    "versionId": 1
  }
}
```

##### DID Hash Document

Sending a request to resolve `did:trust:tc:dev:hash:DSkA5Pw7t76ZiR4oKiM1kpJExy3zikfD5S4iwsbQQ2FR`:

```sh
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Ahash%3ADSkA5Pw7t76ZiR4oKiM1kpJExy3zikfD5S4iwsbQQ2FR/doc' \
  -H 'accept: application/json'
```

And the corresponding DID Hash Document response:

```json
{
  "document": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:trust:tc:dev:hash:DSkA5Pw7t76ZiR4oKiM1kpJExy3zikfD5S4iwsbQQ2FR",
    "controller": ["did:trust:tc:dev:id:XLzBJd69tqEgq7oqqdEsHW"],
    "algorithm": "sha256"
  },
  "signatures": {
    "type": "Single",
    "values": [
      {
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#2kL2biA5pC76HPdkUdHQwSqLWntLRt31aoHM8reR2RoS",
        "signature": "5u9ENPTkSNnBNhJcZghG44Rn99rowbzkFkHL6nX2L81tyWFbHEQMyFic4Ue4L75ypopopxWRszAugqmTFv6P2S4WQPDi7d74SCYqLmeVQPUwDEKpeg9VEfaPH5Ye2BjvuT8DA59ripNMybZftagNiDK1UrXGTq7VGoe3sWiCsGUZnUuweNEKshgYDnr856BoRMAezEwhBQU8C6aRNfgxKvpoemE77CqJUgkrqC4zwAviVXjhuC2B6mrBaQ8DXmcaG7VFRgJSfQQPefJHT9NgVoU3P7CVk3ysP3TUMhVQ42dhqre7c3sviRdgaPt34BGNGQ74Kh6dRgp5FQALqX27xtEsgYNuL1"
      }
    ]
  },
  "metaData": {
    "created": "2022-08-01T09:52:28.331Z",
    "versionId": 1
  }
}
```

#### Request meta data

The metadata of a DID document can be requested by a separate endpoint to get more information:

Request:

```sh
curl -X 'GET' \
  'https://api.observer1.example.com/did/did%3Atrust%3Atc%3Adev%3Aid%3AXLzBJ69tqEgq7oqqdEsHW/metadata' \
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

### Update

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
  "version": 1,
  "body": {
    "date": "2021-05-04T06:27:14.668Z",
    "value": {
      "id": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW",
      "service": {
        "add": [
          {
            "id": "did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver",
            "type": "resolver",
            "endpoint": "https://api.observer1.dev.trustcerts.de/did/resolve/did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW"
          }
        ],
        "remove": ["did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver"]
      }
    },
    "didDocSignature": {
      "type": "single",
      "values": [
        {
          "signature": "9ofJG4kdHLGxR32YPdUjTTJnrS98nmivnVs8nh98wCbbfdqki2DvXAoiMx8tSNN12NWin9Q6PRSVUp9Gv393gs38gUeiYJvBXAXgNbEYoVVGRkxiJ3Kf15yyVwwHZtGhV5XLbiLymexqmywb4dVW4JcqZYaqjw9g9nuXVfXdzsGbjNNh57oTjmXFAcaG5xC37mdAJqnaHNVBho3hpDrP2Zh3DPptBoV1gtJQDAGgYQnzF6NofuNC75EyQHn6hyF1UN1RHxZmGJATVgWJfwwNKQVAGsgD7KpWvHiUfmSdJvHbPNKpeqeSPVSnK2oUtowQEhwBs1ev12tdxnAHDsRiN48pNQkooG",
          "identifier": "did:trust:tc:dev:id:XLzBJ69tqEgq7oqqdEsHW#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
        }
      ]
    },
    "type": "Did",
    "version": 1
  },
  "metadata": {
    "version": 1
  },
  "signature": {
    "type": "single",
    "values": [
      {
        "signature": "7CuzvzqK2nWKRTyirueWF1aUYVLXqftYVVXA8P5wt8MpRVeZ9U6xtpkC8oGk4RwuPG8yyqY2H6TJS7GSXb41yKhNL2z3QDnBFvYj94iC9sBiTXyFsT6Tj62k4adUvZogM3mttFqt5D35c5dcKoS1R6QnTEgj98U6AzJS3VpfzxcwPwwaDuFHgR5nhA22bNK3anHhmhjeMPzghwQP2d6xwMD1BbDrzbma4kWoCxFggchHBgob8kFP1op4fTh1TfZeSbbDJ7T2pLJ1947nWMAvNPT7SxAacLuhbSZqUNqCAZAqm9cfogwg7v8hNrqV2YQjDFstdVr6cAFwz84Skc31LJoRFSnGG1",
        "identifier": "did:trust:tc:dev:id:XLzBJ69BeqEgq7oqqdEsHY#41E9VrMT1VZpiJjaKMW5xL3ERr84yNjTzRqkp4GPjXHv"
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
          }
        ],
        "remove": ["did:trust:tc:dev:id:HzrprbqrPsvmi9pe6XsuR8#resolver"]
      }
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

### Deactivate

A DID document is considered deactivated when it does not include any `verificationMethod` to execute an update method on itself. A DID can be restored by an authority by creating a new `verificationMethod` inside the DID document.

```sh
curl -X 'POST' \
  'https://api.observer1.example.com/did/did' \
  -H 'accept: application/json'
```
