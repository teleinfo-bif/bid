# **BIF DID protocol Specification**

Version 1.0.0

## Abort 

​         The BIF DID(hereinafter referred to as BID) method specification conforms to the requirements specified in the DID specification currently published by the W3C Credentials Community Group. For more information about DIDs and DID method specifications, please see the [DID Primer](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md) and [DID Spec](https://w3c-ccg.github.io/did-spec/).

## Abstract

​            BID provides distributed identifiers and blockchain digital identity services for people, enterprises, devices and digital objects. It aims to build a decentralized, data-secure, privacy-protected and flexible identifier system that addresses trusted connections to people, enterprises, devices and digital objects，enabling the vision of the Internet of Everything and trust ingress with everything.

​            The BID is a decentralized identification protocol and it has the features of decentralization, self-management, privacy protection, security and ease of use. Each BID corresponds to an BID Description Object (DDO).

## Status of This Document

​           This document is currently version 1.0 and may be replaced by an updated version when it is released. You Can be in making (https://github.com/teleinfo-bif/document) address to obtain the latest edition of this technical report.

## 1. BID method name space

- The namestring that shall identify this DID method is: bid

- A DID that uses this method MUST begin with the following prefix: did:bid. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is the NSI specified below.


## 2. Target System(s)

This DID method is applicable to the BIF network and has been used formally since the BIF release.

## 3. BID Specific Identifier

The BID scheme is defined by the following ABNF：

```
bif-did = "did:bid:" idstring *(":" subnamespace)
idstring = 20(base58char)
subnamespace = ALPHA *(ALPHA / DIGIT / "_" / "-")
base58char = "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" / "A" / "B" / "C" / "D" / "E" / "F" / "G" / "H" / "J" / "K" / "L" / "M" / "N" / "P" / "Q" / "R" / "S" / "T" / "U" / "V" / "W" / "X" / "Y" / "Z" / "a" / "b" / "c" / "d" / "e" / "f" / "g" / "h" / "i" / "j" / "k" / "m" / "n" / "o" / "p" / "q" / "r" / "s" / "t" / "u" / "v" / "w" / "x" / "y" / "z"
```

All bids use elliptic curve cryptography to randomly generate the public key,then hash the public key and  generate base58 encoding last take the first 20 bits .Base58 encoding uses most letters and Numbers, and removes characters such as 0Oil for readability.

Examples：did:bid:DaogpsaKqKMe4nzPy4Aq

##  4. Identity description object DDO specification

The ID description object (DDO) corresponding to the BID is stored in the BIF blockchain, written by the DDO controller to the blockchain, and opened to all users for reading.

The DDO specification contains the following information:

-   Name: a nickname to improve the convenience of the bid identifier. It can substitute the bid identifier when using it in the client.

-   publicKey: The public key information used for identity authentication, which includes the public key id, the public key type, the BID identifier corresponding to the public key, and the authorization scope (if the value is 'all', the public key can perform any operation on the bid document);

-   User identity authentication list information, including the authentication type, the public key used for authentication, and the trusted claim ID.

-   Attributes: the personal information filled in by the user in JSON format (for example: {type:age,value:22})

-   isEnable: whether this BID is enabled

sample:

```
{
  "@context": "https://w3id.org/future-method/v1",
  "id": "did:bid:DaogpsaKqKMe4nzPy4Aq",
  "name":"shiweijun",
  "publicKey": [{
    "id": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-1",
    "type": "RsaVerificationKey2018",
    "controller": "did:bid:DaogpsaKqKMe4nzPy4Aq",
    "authorizations": ["all"]，
    "bid": " did:bid:DaogpsaKqKMe4nzPy4Aw"
  }, {
    "id": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-3",
    "type": "Ieee2410VerificationKey2018",
    "controller": "did:bid:DaogpsaKqKMe4nzPy4Aq",
    "bid": " did:bid:DaogpsaKqKMe4nzPy4An"
  }],
  "authentication": [{
    "type": "ED25519SigningAuthentication",
    "publicKey": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-1",
    "proofId":"c1cb770ac94000d946e583d13ea310"
            }],
  "Attributes":[{
  		"type": "age",
  		"value":21
   }],
   "isEnable":true,
   "createTime":1561945638,
   "updateTime":1561945638
}
```



## 5. BID method specification

​         BIdContract is a smart contract implementation of the BID protocol on the BIF blockchain platform. With BIdContract, users can manage public key lists, modify profiles, and set up nickname.

###  5.1 Set BID nickname

​        The user calls the SetBidName method to set and update the bid nickname. Only the public key with authorizations value 'all' can be modified.

```
bool SetBidName(string name);
```

​         name: bid nickname, if null, it is set to BID identifier by default.

###  5.2 Query

​        Query DID documentation with given BID or nickname and return the DID document in the form of JSON string.

```
byte[] findDDOByBid(string type,string value)
```

Parameters:

​         type: type of input value. 0 for bid, 1 for name

​          value: input value

###  5.3 Add a control BID

​          BID supports authorization control, which means the user can authorize other BIDs to control the current BID, such as update the DDO document, etc. The user calls the addBid method to authorize privileges to other BIDs. Only the public key with authorizations 'all' can be added.

```
 bool addBid(string type,string bid,string authorizations)
```

Parameters:

​        type: the encryption algorithm used when generating the input bid, for example RSA,ECC

​        bid: input BID value

​        authorizations: privileges authorized to this BID

###  5.4 Delete a control BID

​         The user deletes the privileges authorized to other BIDs by calling the deleteBid method. Only the public key with authorizations 'all' can be deleted. The account authorized as all cannot be deleted by itself.

```
 bool deleteBid(string bid)
```

Parameters:

​     bid: input BID value

###  5.5 Add a Authentication credentials

​          BID supports users to perform multi-source authentication. After the authentication is completed, the user can add the abstract of the trusted statement to the DDO document for subsequent verification and management. The user calls the addProof method to add trusted claim information. Only the public key with authorizations 'all' can be added.

```
  bool addProof(string type,string proofId)
```

Parameters:

​		type: the type of multi-source authentication, for example

> ​         0: basic identity authentication (public security certification),
>
> ​        1: academic qualification,
>
> ​        2: passport authentication,
>
> ​        3: WeChat authentication, etc.

​		proofId: ID number on the chain of the trusted statement.

###  5.6 Delete a Authentication credentials

​       The user can delete the trusted statement by calling the deleteProof method. Only the public key with authorizations 'all' can be added.

```
bool deleteProof(string proofId)
```

Parameters:

​        proofId: ID number on the chain of the trusted statement.

###  5.7 Add a Personal Attribute

The user can add personal attribute information by calling the addAttr method.

```
bool addAttr(string key,string value)
```

Parameters:

​        type: the key of the property, for example, age

​        value: the value of the property

###  5.8 Delete a Personal Attribute

The user can delete personal attribute information by calling the deleteAttr method.

```
 bool deleteAttr(string type)
```

Parameters:

​          type: the key of the property, for example, age

###  5.9 Enable BID

​        The user can enable a BID by calling the enable method. Only the public key with authorizations 'all' can be enabled.

```
  bool enable()
```



### 5.10 Disable BID

​        The user can disable a BID by calling the disable method. Only the public key with authorizations 'all' can be disabled.

```
  bool disable()
```

