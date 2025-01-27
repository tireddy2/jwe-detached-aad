---
title: "Enhanced JWE Security with Detached Additional Authenticated Data (AAD)"
abbrev: "Enhanced JWE Security with Detached AAD"
category: std

docname: draft-reddy-jose-detached-aad
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "JOSE"

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"

normative:
  RFC2119:
  RFC7516:
  RFC8785:

--- abstract

This draft introduces a mechanism for supporting detached Additional Authenticated Data (AAD) in JWE (JSON Web Encryption), enabling the AAD to be derived from context-specific information, such as session identifiers, timestamps, or request origins, rather than transmitted in-band. This mechanism enhances security by addressing attacks such as phishing attacks and contextual misuse, particularly in scenarios where context-specific AAD is used. The draft specifies how this functionality can be integrated into JWE, considering both JWE JSON Serialization and JWE Compact Serialization.

--- middle

# Introduction

In some use cases, it is beneficial to derive the Additional Authenticated Data (AAD) for JWE from out-of-band contextual information 
rather than transmitting it directly in the message. This approach provides enhanced security by ensuring cryptographic binding of context-specific information, thus mitigating risks such as manipulation or misuse of contextual data.  Additionally, this method reduces the reliance on transmitted AAD, thereby minimizing the attack surface.

This document proposes a mechanism to handle detached AAD in JWE, where the AAD is not transmitted but is instead derived by both the 
sender and receiver. The approach described here is applicable to both JWE JSON Serialization and JWE Compact Serialization. This mechanism is particularly relevant for scenarios like OpenID for Verifiable Credentials (OID4VC), where a verifier must validate contextual information without relying on in-band AAD.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Motivation and Use Cases

A typical use case for detached AAD arises in protocols such as OID4VC, where a verifier may receive a JWE response from a wallet app. 

In such scenarios:

- The verifier sends a request to the wallet (usually a mobile app).
- The wallet responds with an encrypted message that includes contextual information (e.g., the origin of the request) as part of the AAD.
- Without proper validation of AAD, attackers could exploit vulnerabilities by manipulating contextual information, potentially leading to  malicious actions.

By requiring the verifier and wallet to securely agree on the method to derive AAD, the encrypted response can be cryptographically bound to the request's context. This ensures that the response is valid only within its intended context, mitigating risks such as phishing, replay and tampering attacks. 

# Detached AAD and Its Benefits
The introduction of detached AAD provides the following benefits:

- Improved Security: Deriving AAD directly from the context eliminates the reliance on transmitting and validating in-band AAD. This reduces the potential attack surface by minimizing the risk of tampering or replay.
- Enhanced Flexibility: Applications have greater control over the AAD's use, ensuring it is only transmitted when necessary and 
derived out-of-band otherwise.
- Consistency Across serializations: By supporting detached AAD in both JWE JSON Serialization and JWE Compact Serialization, a consistent approach to context-specific AAD can be adopted across different serializations.


# JWE JSON Serialization

When the AAD is detached, the "detached_aad" parameter is set to true, indicating that the AAD MUST be computed by both the sender and receiver from contextual information.

- If the "detached_aad" parameter is set to true, the detached AAD MUST be derived out-of-band, following the process outlined in Step 5 of {{Derived}}.
- If the "aad" parameter contains data, it is used directly as the AAD for encryption.
- If both the "detached_aad" and "aad" parameters are present, the AADs are combined and used according to the process described in Step 5 of {{Derived}}.
- The derived AAD is treated as part of the encryption context, but is never transmitted within the JWE structure.

Example:


A JSON Encoded JWE:

~~~
{
  "protected": "eyJhbGciOiJIUEtFLVAyNTYtU0hBMjU2LUExMjhHQ00iLCJlbmMiOiJkaXIiLCJraWQiOiJ1cm46aWV0ZjpwYXJhbXM6b2F1dGg6andrLXRodW1icHJpbnQ6c2hhLTI1NjpTNkFYZmRVXzZZZnp2dTBLRERKYjBzRnV3bklXUGs2TE1URXJZaFBiMzJzIiwicHNrX2lkIjoib3VyLXByZS1zaGFyZWQta2V5LWlkIiwiYXV0aF9raWQiOiJ1cm46aWV0ZjpwYXJhbXM6b2F1dGg6andrLXRodW1icHJpbnQ6c2hhLTI1NjpTNkFYZmRVXzZZZnp2dTBLRERKYjBzRnV3bklXUGs2TE1URXJZaFBiMzJzIn0",
  "encrypted_key": "BD7QVodtG-FwYASgb36zuTzUCc80aiYwS6JOOE-6_heUGyAZt-cU0818e4oYqP7ebBuW3KTM9EQA0vM5fWp6hj0",
  "ciphertext": "ZxqtYoomgVQGctnv1I_EBVI1NIeJ7qJw2iVtqwUw3fXa8FK-",
  "aad": "8J-PtOKAjeKYoO-4jyBiZXdhcmUgdGhlIGFhZCE"
}
~~~

After verification:

~~~
{
  "protectedHeader": {
    "alg": "HPKE-0",
    "enc": "dir",
    "kid": "urn:ietf:params:oauth:jwk-thumbprint:sha-256:S6AXfdU_6Yfzvu0KDDJb0sFuwnIWPk6LMTErYhPb32s",
    "psk_id": "our-pre-shared-key-id",
    "auth_kid": "urn:ietf:params:oauth:jwk-thumbprint:sha-256:S6AXfdU_6Yfzvu0KDDJb0sFuwnIWPk6LMTErYhPb32s",
    "aad_detached": true
  },
  "plaintext": "This is plaintext!",
  "aad": "This is aad!"
}
~~~

In this case, the "aad_detached" parameter set to true indicates that the AAD must be derived from contextual information shared between the sender and receiver.


# JWE Compact Serialization

## Supporting Detached AAD in JWE Compact Serialization

To enable the use of detached Additional Authenticated Data (AAD) without introducing backward compatibility issues, this specification defines a mechanism that leverages external context-specific information and an optional protected header parameter to signal the presence of detached AAD. The existing JWE Compact Serialization format, as defined in Section 7.1 of {{RFC7516}}, remains unchanged.

### Mechanism Overview

1. External Metadata for Detached AAD:
   - The detached AAD is not included in the JWE Compact Serialization format.
   
2. Optional Protected Header Parameter:
   - A new optional parameter, "detached_aad", is introduced in the JWE Protected Header.
   - When set to `true`, this parameter indicates that the AAD is detached and must be obtained through an external context.

3. Backward Compatibility:
   - This mechanism does not alter the structure of the JWE Compact Serialization format, ensuring full compatibility with existing implementations.

The "aad_detached": true parameter is included in the JWE Protected Header. For example:

~~~
{
  "alg": "HPKE-0",
  "enc": "A256GCM",
  "detached_aad": true
}
~~~


# Deriving detached AAD

## Steps for Derivation {#Derive}

When using detached AAD, the sender and receiver MUST follow the same derivation process to ensure consistent results. The derived AAD is never transmitted; instead, it is independently computed by both parties. The process involves:

1. Identify Contextual Elements 
   - Applications define the contextual elements required for AAD derivation. These elements are shared or understood by both sender and receiver. Example:  

   ~~~
   {
      "session_id": "sess-1234",
      "timestamp": "2025-01-10T12:00Z"
   }
   ~~~

2. Normalize Contextual Elements
   - Each contextual element MUST be encoded as a UTF-8 string.  
   - The elements MUST be sorted in lexicographical order of their keys (as per {{RFC8785}}), ensuring consistent ordering during derivation.
   Example:  

   ~~~
   Normalize Elements (Sorted): 
   {
      "session_id": "sess-1234",
      "timestamp": "2025-01-10T12:00Z"
   }
   ~~~
   
3. Serialize the Canonicalized JSON
   - Serialize the canonicalized JSON object, ensuring it has no extra spaces or newlines.
   Example:    
   
   ~~~
   Concatenated String: {"session_id":"sess-1234","timestamp":"2025-01-10T12:00Z"}
   ~~~

4. Apply Cryptographic Hashing
   - Apply a cryptographic hash function (e.g., SHA-256) to the serialized canonicalized JSON string to derive the AAD. The resulting hash will serve as the AAD used in the encryption operation. The use of SHA-256 is RECOMMENDED; however, other cryptographic hash functions MAY be used if they provide equivalent or stronger security properties.
   Example:    

   ~~~
   Derived AAD: "SHA256({"session_id":"sess-1234","timestamp":"2025-01-10T12:00Z"})"
   ~~~

5. Use the Derived AAD in Encryption
   - The Step 15 in Section 5.1 of of {{RFC7516}} specifies how the AAD is used in the encryption operation. In the detached AAD case, the derived AAD is treated as though it were part of the encryption context, even though it is never transmitted within the JWE structure.

   - In the case of a message without the JWE AAD but with the derived AAD, the derived AAD is used alongside the Content Encryption Key (CEK), JWE IV, and the message M to create the JWE Ciphertext and JWE Authentication Tag.
      * The Additional Authenticated Data (AAD) encryption parameter is:

      ~~~
        ASCII(Encoded Protected Header || '.' ||BASE64URL(Detached AAD)).
      ~~~

   -  In the case of a message with the JWE AAD and the derived AAD, the derived AAD is used alongside the Content Encryption Key (CEK), JWE IV, JWE AAD and the message M to create the JWE Ciphertext and JWE Authentication Tag.
      * The Additional Authenticated Data (AAD) encryption parameter is:

      ~~~
         ASCII(Encoded Protected Header || '.' ||BASE64URL(JWE AAD)|| '.' ||BASE64URL(Detached AAD)).
      ~~~

   -  Both the sender and receiver MUST compute the detached AAD independently to ensure consistency.
   -  The detached AAD is never transmitted within the JWE structure but is derived from the contextual elements out-of-band.

6. Error Handling

   - If the derived AAD does not match the expected value during decryption, the JWE MUST be treated as invalid, and the decryption process MUST fail. 

# Security Considerations

Detached AAD improves security by ensuring that AAD is tightly bound to the specific context shared by the sender and recipient. However, the following considerations apply:

* Both sender and recipient MUST use the same method to derive AAD from the context to prevent mismatches. Mismatches will result in 
  decryption failures.
* Applications using detached AAD must ensure that contextual information is validated and securely exchanged to prevent manipulation by attackers.
* Implementations should account for potential synchronization issues, such as clock drift, when deriving AAD from time-sensitive context.

#  IANA Considerations {#IANA}

## JSON Web Signature and Encryption Header Parameters

The following entry is requested to be added by IANA to the "JSON Web Signature and Encryption Header Parameters" registry:

### aad_detached

- Header Parameter Name: "detached_aad"
- Header Parameter Description: Indicates that the AAD is detached and must be conveyed through an external mechanism.
- Header Parameter Usage Location(s): JWE
- Change Controller: IETF
- Specification Document(s): RFCXXXX

# Acknowledgments
{:numbered="false"}

TODO
