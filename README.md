**************************************                  Rex Barq
Request for Comments: DRAFT                             xAI
Category: Informational                                 February 2025

XCheck - Specification for QR Code on Bank Cheques with Extensible JSON Format

* Status of This Memo

   This document is an Informational RFC specifying a standard for encoding
   cheque-related data in a JSON and QR code along with cryptographic security
   features. Distribution of this memo is unlimited.

* Abstract

   This document defines a standard for embedding cheque data into a JSON object
   and a QR code that can be used for digial and visual representation of bank
   checks. The standard encodes essential cheque information, including the 
   issuing bank's cryptographic signature, the drawer's identification, and 
   transaction details, in a secure and verifiable format. The specification 
   uses Tag-Length-Value (TLV) encoding, Base64 representation, and includes a
   version number to support future updates. 
   On the check itself, the he issuing bank is responsible for generating and
   signing the JSON and QR code, the beneficiary is identified by name, while
   the drawer is authenticated by the issuing bank and identified without
   requiring their signature.
   Checks encoded and signed in this way are meant to be a digital drop-in
   replacement for paper checks. Compatibility is intended to enable printing
   of an X-Check on plain paper to be cleared via legacy check clearing
   infrastructure without increasing counterfeiting and fraud risk.

* Table of Contents

   1. Introduction
   2. Requirements Terminology
   3. QR Code Specifications
      3.1 Structure of the QR Code
      3.2 TLV Encoding
      3.3 Encoding Process
      3.4 Cryptographic Signature
   4. QR Code Content TLV Field Definitions
   5. Security Considerations
   6. IANA Considerations
   7. References
      7.1 Normative References
      7.2 Informative References
   Appendix A: Example QR Code Encoding

* 1. Introduction

   Bank cheques require a secure, machine-readable format to authenticate
   and verify transaction details. This specification defines a QR code
   format for cheques that includes the issuing bank's cryptographic
   signature, drawer identification, and cheque metadata. The QR code
   ensures authenticity and facilitates automated processing while
   supporting future updates through a version number.

* 2. Requirements Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119.
* 3. QR Code Specifications

** 3.1 Structure of the QR Code

   The QR code MUST be generated and printed on bank cheques, encoded in
   Base64 format with a maximum length of 700 characters. It MUST contain
   the mandatory fields specified in Section 4.1, encoded in Tag-Length-Value
   (TLV) format, and MAY include optional issuer-specific fields (Tags 12+).

** 3.2 TLV Encoding

   - Tag: 1 byte. Tags 1-11 are reserved for mandatory fields (Section 4.1);
     Tags 12-255 are available for issuer-specific optional fields.
   - Length: 1 byte (length of Value in bytes, max 255).
   - Value: UTF-8 encoded text or raw bytes (for binary fields).
   - Implementations MUST process Tags 1-11 and MAY ignore Tags 12+.

** 3.3 Encoding Process

   1. Construct the JSON "data.mandatory" block per Section 4.2.
   2. Optionally append "data.optional" fields defined by the issuer.
   3. Compute SHA256 hash of the canonicalized "data.mandatory" block only.
   4. For each QR field (Section 4.1):
      - Mandatory Tags 1-11: Construct TLV tuples.
      - Optional Tags 12+: Include issuer-specific TLVs if desired.
   5. Concatenate TLV tuples (Tags 1-11, then 12+ if present).
   6. Encode the byte array into a Base64 string.
   7. Generate the QR image and embed the Base64 string into "qrCode".

** 3.4 Cryptographic Signature

   The QR code MUST include an ECDSA signature (Tag 10) over the SHA256 hash
   of the "data.mandatory" block (Tag 9). The JSON MUST include a signature
   over the "data.mandatory" block, excluding "data.optional" and "qrCode".

* 4. Data Formats

** 4.1 QR Code Content TLV Field Definitions

   +-----+---------------------------+----------------+-----------------------+
   | Tag | Field                    | Enforcement    | Description           |
   +-----+---------------------------+----------------+-----------------------+
   |  1  | Specification Version    | Mandatory      | Version (e.g., "1.0") |
   +-----+---------------------------+----------------+-----------------------+
   |  2  | Issuing Bank Name        | Mandatory      | Legal name of bank    |
   +-----+---------------------------+----------------+-----------------------+
   |  3  | Bank Routing Number      | Mandatory      | ABA or SWIFT code     |
   +-----+---------------------------+----------------+-----------------------+
   |  4  | Drawer Name              | Mandatory      | Full name of drawer   |
   +-----+---------------------------+----------------+-----------------------+
   |  5  | Drawer Account Number    | Mandatory      | Drawer's account      |
   +-----+---------------------------+----------------+-----------------------+
   |  6  | Beneficiary Name         | Mandatory      | Full name of payee    |
   +-----+---------------------------+----------------+-----------------------+
   |  7  | Cheque Amount            | Mandatory      | Decimal (e.g., "123.45") |
   +-----+---------------------------+----------------+-----------------------+
   |  8  | Timestamp                | Mandatory      | ISO 8601 (e.g.,       |
   |     |                          |                | "2025-02-18T10:00:00Z") |
   +-----+---------------------------+----------------+-----------------------+
   |  9  | Hash of Mandatory JSON   | Mandatory      | SHA256 of "data.mandatory" |
   +-----+---------------------------+----------------+-----------------------+
   | 10  | ECDSA Signature          | Mandatory      | Signature of Tag 9    |
   +-----+---------------------------+----------------+-----------------------+
   | 11  | ECDSA Public Key         | Mandatory      | Bank's public key     |
   +-----+---------------------------+----------------+-----------------------+
   | 12+ | Issuer-Specific Fields   | Optional       | Defined by issuer     |
   +-----+---------------------------+----------------+-----------------------+

   - **Mandatory Fields (Tags 1-11)**: MUST be present and match JSON "data.mandatory".
   - **Optional Fields (Tags 12-255)**: MAY be added by issuers; ignored by unaware parties.
   - Tag 9 hashes only the "data.mandatory" block to ensure interoperability.

** 4.2 JSON Data Format

   The cheque SHALL be accompanied by a JSON file with this structure:

   {
     "data": {
       "mandatory": {
         "version": "string",              // Tag 1
         "issuingBankName": "string",      // Tag 2
         "bankRoutingNumber": "string",    // Tag 3
         "drawerName": "string",           // Tag 4
         "drawerAccountNumber": "string",  // Tag 5
         "beneficiaryName": "string",      // Tag 6
         "chequeAmount": "string",         // Tag 7
         "timestamp": "string"             // Tag 8
       },
       "optional": {                      // Issuer-specific fields
         "[key]": "value",                // e.g., "chequeNumber": "12345"
         "[key]": "value"                 // e.g., "branchCode": "XYZ"
       }
     },
     "signature": "string",              // Base64 ECDSA signature of SHA256("data.mandatory")
     "qrCode": "string"                  // Base64 TLV of Tags 1-11 + optional Tags 12+
   }

   - **"data.mandatory"**: Contains required fields (Tags 1-8 equivalents).
     - MUST be canonicalized (e.g., sorted keys, no whitespace) before hashing.
     - SHA256 hash stored in Tag 9 and signed in "signature" and Tag 10.
   - **"data.optional"**: Object for issuer-defined key-value pairs.
     - MAY be empty or absent; ignored by parties not recognizing the keys.
     - Excluded from hash and signature to maintain interoperability.
   - **"signature"**: Signs only "data.mandatory" hash.
   - **"qrCode"**: Base64 TLV data, including Tags 1-11 and optional Tags 12+.

   **Processing Rules**:
   1. Construct "data.mandatory" and canonicalize it.
   2. Optionally add "data.optional" fields (issuer-specific).
   3. Compute SHA256 hash of "data.mandatory" only.
   4. Sign the hash → "signature".
   5. Build QR TLV (Tags 1-11 + optional Tags 12+), with Tag 9 = hash.
   6. Encode TLV as Base64 → "qrCode".
   7. Validate by checking "signature" and Tag 10 against Tag 9 hash.

* 5. Security Considerations

   - Only "data.mandatory" is signed and hashed to ensure all parties can
     verify the cheque’s core data, regardless of "data.optional" content.
   - Optional fields (Tags 12+ and "data.optional") are unsigned and MAY
     be ignored without affecting cheque validity.
   - Implementations MUST validate both signatures using Tag 11.

* 6. IANA Considerations

   This document has no IANA actions.

&7. References

7.1 Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.

   [ISO8601]  International Organization for Standardization, "Data
              elements and interchange formats -- Information interchange
              -- Representation of dates and times", ISO 8601:2004.

7.2 Informative References

   None.
* Appendix A: Example JSON and QR Encoding

   **JSON Example**:
   {
     "data": {
       "mandatory": {
         "version": "1.0",
         "issuingBankName": "Bank of America",
         "bankRoutingNumber": "123456789",
         "drawerName": "John Q. Public",
         "drawerAccountNumber": "123456789012",
         "beneficiaryName": "Jane R. Doe",
         "chequeAmount": "123.45",
         "timestamp": "2025-02-18T10:00:00Z"
       },
       "optional": {
         "chequeNumber": "987654",
         "branchCode": "BOS001",
         "memo": "Rent Payment"
       }
     },
     "signature": "MEUCIQC... [Base64 ECDSA of SHA256(data.mandatory)]",
     "qrCode": "AQABA... [Base64 TLV Tags 1-11 + e.g., Tag 12=chequeNumber]"
   }
