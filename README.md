| `                RFC                  ` | `************************************` |
| --------------------------------------- | -------------------------------------- |
| `                                     ` | `             Author: Rex Barq       ` |
| `Request for Comments: DRAFT          ` | `             Organization: Hashpin  ` |
| `Category: Informational              ` | `             Date: March 2025       ` |
    
# X-Check - A Specification for Cryptographially Secured Digital Bank Checks as Drop-In Replacement for Paper Checks

# Status of This Memo

  This document is an Informational RFC specifying a standard for encoding
  check-related data in a JSON and QR code along with cryptographic security
  features. Distribution of this memo is unlimited.
 
# Copyright and License

  Copyright (c) 2025 Rex Barq

  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this specification and associated documentation files (the "Specification"),
  to deal in the Specification without restriction, including without limitation
  the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
  sell copies of the Specification, and to permit persons to whom the Specification
  is furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in all
  copies or substantial portions of the Specification.

  THE SPECIFICATION IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
  FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
  COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
  IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
  WITH THE SPECIFICATION OR THE USE OR OTHER DEALINGS IN THE SPECIFICATION.

# Abstract
 
   This document defines a standard for embedding check data into a JSON object
   and a printable QR code that can be used for digial and visual representation
   of bank checks. The standard encodes essential check information, including the 
   issuing bank's cryptographic signature, the drawer's identification, and 
   transaction details, in a secure and verifiable format. The specification 
   uses Tag-Length-Value (TLV) encoding, Base64 representation, and includes a
   version number to support future updates. 
   
   On the check itself, the he issuing bank is responsible for generating and
   signing the JSON and QR code, the beneficiary is identified by name, while
   the drawer is authenticated by the issuing bank and identified without
   requiring their signature.
   
   Checks encoded and signed in this way are meant to be a digital drop-in
   replacement for paper checks that will work on legacy image-based check
   clearance systems. The compatibility extends to printing of an X-Check on 
   plain paper to be cleared via legacy check clearing infrastructure with
   minimal changes while increasing anti-counterfeiting and fraud resistance
   characteristics.

# Table of Contents

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

# 1. Introduction

   It is commonly believed that checks are an outdated technology that is
   easily replaced by modern financial products, like bank transfers and
   other forms of direct payment.

   Checks are a flexible negotiable instrument that represent a legally
   recognised IOU with the ability to be encashed, deposited in a bank
   account, or even destroyed (if they were used as a security deposit).

   Consider that the drawer writing a cheque does not need to know the
   payee's bank details, and the payee has complete freedom in what
   to do with the check, including endorsing it and giving it to someone
   else to encash! This resembles properties of bearer instruments and
   makes **checks a powerful instrument of financial freedom and privacy**
   that is tenacious in countries that have adopted it and will remain
   resistant to replacement by objectively inferior instruments.

   The misconception that cheques are an inferior fuels a pernicious lack
   of focus on digitisation. In fact, checks are exceptionally well adapted
   to cryptographic digitisation. This is because the issuer of the cheque
   is the bank (drawee) and not the bank customer (drawer), which makes
   digitally issuing and signing a cheque easy to implement in practice.
   
   In effect, **a cryptographically secured check that can be transmitted
   via an insecure channel, like email, to the recipient (payee) could easily
   be generated from a bank's mobile app or in bulk as a business service
   with minimal changes to legacy systems.**

   The idea for a cryptographically secured paper cheque was implemented at
   Emirates NBD in 2018 [ENBD-CHEQUECHAIN] and is still in use as of this
   writing.

   This specification defines a QR code and JSON data format for checks 
   that includes the drawes's cryptographic signature, drawer identification, 
   payee, and other check data. The concise cryptographically secured and
   self-sufficient QR code is inspired by the one defined in the Saudi 
   electronic invoicing standard published by ZATCA [ZATCA-FATOORA] and used
   on VAT invoices in the Kingdom of Saudi Arabia. 
   
   A self-sufficient cryptographically secured QR code ensures the ability
   to digitally authenticate the check in legacy image-based check clearing
   systems while providing an increased level of counterfeit and fraud
   protection.

# 2. Requirements Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119.
   
# 3. QR Code Specifications

## 3.1 Structure of the QR Code

   The QR code MUST be generated and printed on bank checks, encoded in
   Base64 format with a maximum length of 700 characters. It MUST contain
   the mandatory fields specified in Section 4.1, encoded in Tag-Length-Value
   (TLV) format, and MAY include optional issuer-specific fields (Tags 12+).

## 3.2 TLV Encoding

   - Tag: 1 byte. Tags 1-11 are reserved for mandatory fields (Section 4.1);
     Tags 12-255 are available for issuer-specific optional fields.
   - Length: 1 byte (length of Value in bytes, max 255).
   - Value: UTF-8 encoded text or raw bytes (for binary fields).
   - Implementations MUST process Tags 1-11 and MAY ignore Tags 12+.

## 3.3 Encoding Process

   1. Construct the JSON "data.mandatory" block per Section 4.2.
   2. Optionally append "data.optional" fields defined by the issuer.
   3. Compute SHA256 hash of the canonicalized "data.mandatory" block only.
   4. For each QR field (Section 4.1):
      - Mandatory Tags 1-11: Construct TLV tuples.
      - Optional Tags 12+: Include issuer-specific TLVs if desired.
   5. Concatenate TLV tuples (Tags 1-11, then 12+ if present).
   6. Encode the byte array into a Base64 string.
   7. Generate the QR image and embed the Base64 string into "qrCode".

## 3.4 Cryptographic Signature

   The QR code MUST include an ECDSA signature (Tag 10) over the SHA256 hash
   of the "data.mandatory" block (Tag 9). The JSON MUST include a signature
   over the "data.mandatory" block, excluding "data.optional" and "qrCode".

# 4. Data Formats

## 4.1 QR Code Content TLV Field Definitions
```
   +-----+---------------------------+----------------+----------------------------+
   | Tag | Field                     | Enforcement    | Description                |
   +-----+---------------------------+----------------+----------------------------+
   |  1  | Specification Version     | Mandatory      | Version (e.g., "1.0")      |
   +-----+---------------------------+----------------+----------------------------+
   |  2  | Issuing Bank Name         | Mandatory      | Legal name of bank         |
   +-----+---------------------------+----------------+----------------------------+
   |  3  | Bank Routing Number       | Mandatory      | ABA or SWIFT code          |
   +-----+---------------------------+----------------+----------------------------+
   |  4  | Drawer Name               | Mandatory      | Full name of drawer        |
   +-----+---------------------------+----------------+----------------------------+
   |  5  | Drawer Account Number     | Mandatory      | Drawer's account           |
   +-----+---------------------------+----------------+----------------------------+
   |  6  | Beneficiary Name          | Mandatory      | Full name of payee         |
   +-----+---------------------------+----------------+----------------------------+
   |  7  | Check Amount              | Mandatory      | Decimal (e.g., "123.45")   |
   +-----+---------------------------+----------------+----------------------------+
   |  8  | Timestamp                 | Mandatory      | ISO 8601 (e.g.,            |
   |     |                           |                | "2025-02-18T10:00:00Z")    |
   +-----+---------------------------+----------------+----------------------------+
   |  9  | Hash of Mandatory JSON    | Mandatory      | SHA256 of "data.mandatory" |
   +-----+---------------------------+----------------+----------------------------+
   | 10  | ECDSA Signature           | Mandatory      | Signature of Tag 9         |
   +-----+---------------------------+----------------+----------------------------+
   | 11  | ECDSA Public Key          | Mandatory      | Bank's public key          |
   +-----+---------------------------+----------------+----------------------------+
   | 12+ | Issuer-Specific Fields    | Optional       | Defined by issuer          |
   +-----+---------------------------+----------------+----------------------------+
```
   - **Mandatory Fields (Tags 1-11)**: MUST be present and match JSON "data.mandatory".
   - **Optional Fields (Tags 12-255)**: MAY be added by issuers; ignored by unaware parties.
   - Tag 9 hashes only the "data.mandatory" block to ensure interoperability.

## 4.2 JSON Data Format

   The check SHALL be accompanied by a JSON file with this structure:
```
   {
     "data": {
       "mandatory": {
         "version": "string",              // Tag 1
         "issuingBankName": "string",      // Tag 2
         "bankRoutingNumber": "string",    // Tag 3
         "drawerName": "string",           // Tag 4
         "drawerAccountNumber": "string",  // Tag 5
         "beneficiaryName": "string",      // Tag 6
         "checkAmount": "string",          // Tag 7
         "timestamp": "string"             // Tag 8
       },
       "optional": {                       // Issuer-specific fields
         "[key]": "value",                 // e.g., "checkNumber": "12345"
         "[key]": "value"                  // e.g., "branchCode": "XYZ"
       }
     },
     "signature": "string",                // Base64 ECDSA signature of SHA256("data.mandatory")
     "qrCode": "string"                    // Base64 TLV of Tags 1-11 + optional Tags 12+
   }
```
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

# 5. Security Considerations

   - Only "data.mandatory" is signed and hashed to ensure all parties can
     verify the check’s core data, regardless of "data.optional" content.
   - Optional fields (Tags 12+ and "data.optional") are unsigned and MAY
     be ignored without affecting check validity.
   - Implementations MUST validate both signatures using Tag 11.

# 6. IANA Considerations

   This document has no IANA actions.

## 7.1 Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.

   [ISO8601]  International Organization for Standardization, "Data
              elements and interchange formats -- Information interchange
              -- Representation of dates and times", ISO 8601:2004.

   [RFC8259]  Bray, T., "The JavaScript Object Notation (JSON) Data
              Interchange Format", RFC 8259, December 2017.

   [ISO7816-4] International Organization for Standardization, "Identification
              cards -- Integrated circuit cards -- Part 4: Organization,
              security and commands for interchange", ISO/IEC 7816-4:2020,
              Section 5.2 (TLV structure), October 2020.

## 7.2 Informative References

   [ZATCA-FATOORA] Zakat, Tax and Customs Authority (ZATCA), "Electronic Invoice
              Security Features Implementation Standards", Version 1.0,
              November 2020, <https://zatca.gov.sa/en/E-Invoicing/Documents/
              Security_Features_Implementation_Standards.pdf>.
              
   [ENBD-CHEQUECHAIN] Emirates NBD, "Emirates NBD Leads Banking Sector in Cheque
              Security by Successfully Rolling Out 'Cheque Chain' at Scale",
              April 15, 2018, <https://www.emiratesnbd.com/en/media-centre/
              media-centre-info/?media=emirates-nbd-leads-banking-sector-in-
              cheque-security-by-successfully-rolling-out-cheque-chain-at-scale>.              

# Appendix A: Example JSON and QR Encoding
```
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
         "checkAmount": "123.45",
         "timestamp": "2025-02-18T10:00:00Z"
       },
       "optional": {
         "checkNumber": "987654",
         "branchCode": "BOS001",
         "memo": "Rent Payment"
       }
     },
     "signature": "MEUCIQC... [Base64 ECDSA of SHA256(data.mandatory)]",
     "qrCode": "AQABA... [Base64 TLV Tags 1-11 + e.g., Tag 12=checkNumber]"
   }
```
