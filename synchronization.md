# Secure OpenPGP Key Pair Synchronization via IMAP

## Authors

Felix Hammerl

Tankred Hase

## Abstract

This document describes a proposal for a lightweight, secure mechanism to distribute an OpenPGP key pair to multiple OpenPGP-enabled Mail User Agents using the Internet Message Access Protocol Version 4rev1.

## Introduction

When the OpenPGP standard was created, encrypted emails were mainly handled on a single workstation, where the user’s key pair was stored. In a multi-device environment, where the need to access secure emails is no longer limited to a single workstation, it is mandatory that the user's data be available on all devices at all times. Studies like [JOHHNY] have shown that placing the burden of key management on the user is demanding to the point of unintentionally compromising the keys and improvements have only been made through automated public key lookup, but not for the private key.

To make things even more difficult, while the keys are portable in the form of a file, some mobile platforms are built under the assumption that users should not be handling files at all, hence users may be left without a secure means of transporting their key file onto a mobile device. In addition to these challenges there are important security considerations concerning the protection of the OpenPGP private key.

It is not apparent that the OpenPGP mechanism will be replaced by another means of secure email tailored to a multi-device environment in the foreseeable future. At the same time users utilize more devices in parallel to access their electronic communications. Once desktop-only, a more recent trend is email being read on mobile and responded to on the desktop. Therefore the challenge of distributing keys securely and efficiently across devices is a critical prerequisite for increased adoption of securely encrypted messaging.

IMAP v4rev1 as a ubiquitous, time-tested means of transportation to multiple devices with existing authentication mechanisms in place provides the stable and most universally available underpinning for a synchronization mechanism. This proposed key synchronization protocol builds on IMAP v4rev1 to distribute the OpenPGP private key.

## Conventions Used in This Document

"Conventions" are basic principles or procedures.  Document conventions are noted in this section.

This document is written from the point of view of the implementor of the proposed secure key synchronization protocol.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [KEYWORDS].

"User" is used to refer to a human user, whereas "client" refers to the software being run by the user.

"Provider" is used to refer to the party providing authenticated access to an IMAP mailbox.

“Private key” refers to the Radix-64 encoded ASCII-armored representation of the PGP private key block as described in [OPENPGP]. It is used to rebuild both the public key and the private key, hence it MUST contain both the public and private key packets.

Characters are 7-bit US-ASCII unless otherwise specified.

## Security considerations

This protocol covers the creation and reading of encrypted private keys. The deletion is out of scope: Apart from being easily handled by the user on their own and thus without need for further specification, deletion is also subject to the provider’s data retention policy.

The protocol assumes a potentially malicious provider that will exploit weaknesses in the passphrase used in the context of encrypting private keys following this protocol. The protocol described in this document MUST protect the encrypted private key from an attack based on low-entropy passphrases, e.g. dictionary attacks.

It is RECOMMENDED that the authentication mechanism in place by the provider is at least password-based, preferably Multi Factor Authentication or stronger.

## Key encryption/decryption

[OPENPGP] introduces various String-to-Key (S2K) transforms in section 3.7.1.3 to derive a symmetric key used to encrypt the private key packets, but they suffer the disadvantage of not integrity protecting the public key packets. Also a user chosen passphrase might not have enough entropy for a sync use-case where the encrypted key material is exposed to an untrusted service provider. This protocol overcomes this weakness by wrapping all key packets, encrypted with a key derived from a second high-entropy alphanumeric passphrase.

The passphrase SHOULD be a random high-entropy uppercase alphanumeric string of 24 characters, generated from a cryptographically secure pseudo-random number generator (CSPRNG). Should you choose to go with a user-supplied passphrase, please consider the downsides of deriving the key from a user-supplied passphrase due to a lack of entropy in user passwords.

The structure of the binary output (“packet”) will be explained in the following subsection.

## Packet Structure

This section describes the structure of a packet. The packet is a standard Symmetric-Key Encrypted OpenPGP Message described in [OPENPGP] section 3.7.2.2 in which the high-entropy alphanumeric string described above is used as a passphrase.

### Packet Structure Version 1

Version 1 packets use SHA-256 as the hash function for S2K conversion and AES-256 as the symmetric cipher. The structure for version 1 is as follows:

* literal data packet (contains the version number as well as any vendor specific attributes line-by-line):
````
Version: 1
Vendor-specific: xyz
````

* private key packets (including all subkey packets)

## Storage on IMAP

It is REQUIRED that the key is stored in an easily identifiable IMAP folder. The folder name MUST consist of US-ASCII characters. The folder name SHOULD be “openpgp_keys”. The folder MUST NOT be used to store anything else beside private keys by this method.

The message MUST consist of a single MIME node of type application/x.encrypted-pgp-key. For compatibility reasons, it is REQUIRED to use 7bit content transfer encoding with us-ascii charset. All line endings MUST be normalized to \r\n.

It is REQUIRED to use the PGP Key ID in the Message-Id MIME header in the format <[KEY ID]@[HOSTNAME]>.

    Message-Id: <[KEY ID]@[HOSTNAME]>
    Content-Type: application/x.encrypted-pgp-key; charset=us-ascii
    Content-Transfer-Encoding: 7bit

## References

### Normative References

**[KEYWORDS]** Key words for use in RFCs to Indicate Requirement Levels, RFC 2119, https://tools.ietf.org/html/rfc2119

**[OPENPGP]** OpenPGP Message Format, RFC 4880, https://tools.ietf.org/html/rfc4880

**[IMAP4]** Internet Message Access Protocol, Version 4rev1, RFC 3501, https://tools.ietf.org/html/rfc3501

**[MIME]** Multipurpose Internet Mail Extensions Part One, RFC 1521, https://tools.ietf.org/html/rfc1521

**[MIMESEC]** MIME Security with OpenPGP, RFC 3156, https://tools.ietf.org/html/rfc3156

**[MSGID]** Content-ID and Message-ID Uniform Resource Locators, RFC 2111, http://tools.ietf.org/html/rfc2111

### Informative References

**[JOHNNY]** Why Johnny Can’t Encrypt: A Usability Evaluation of PGP 5.0, http://www.gaudior.net/alma/johnny.pdf
