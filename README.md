DISCLAIMER: I wrote this in order to learn. I made every effort
to write it securely, but I can't make any guarantees. Use it at 
your own risk. As always, if you do find a security problem or bug, 
comments, advice, and/or patches are welcome and appreciated. Thanks!



JFPG is a file encryption and signing utility 
roughly inspired by the GPG/PGP encryption utility. It
offers a more or less similar  syntax for encryption,
decryption, signing, and verification. It uses
Dan Bernstein's Tweetnacl crypto library. JFPG
only uses  modern, high speed crypto primitives 
and all encryption is authenticated by default. 
However, this means it is not backwards-compatible
with GPG/PGP (probably a feature, not a bug). 

JFPG should compile on most Unix-like systems without any
dependencies. It has been tested and confirmed to work on 
OpenBSD, DragonFly BSD, Ubuntu 14.04 and 16.04, and OS X Yosemite. 
It may work on other systems as well. 
To compile, simply run "make".

Command syntax

	new keypairs:           jfpg -n new-key-id
	sign:                   jfpg -s -f file -k signer-secretkey
	verify sig:   	        jfpg -v -f file -p signer-pubkey
	encrypt with keypair:   jfpg -e -f file -p recipient-pubkey -k sender-secretkey
	decrypt:                jfpg -d -f file [-p sender-pubkey -s recipient-secretkey] 
	symmetrically encrypt:	jfpg -c -f file [-r rounds]

Examples

Create signing and encryption keypairs for "joe"

	jfpg -n joe

Sign "file.pdf" with the secret key "joe-signing-secretkey.ed25519"

	jfpg -s -f file.pdf -k joe-signing-secretkey.ed25519

Verify signed "file.pdf.signed" with the public key "joe-signing-pubkey.ed25519"

	jfpg -v -f file.pdf.signed -p joe-signing-pubkey.ed25519

Encrypt "file.pdf" for recipient "bob". Note that the sender's secret key is required.

	jfpg -e -f file.pdf -p bob-pubkey.curve25519 -k joe-secretkey.curve25519	

Decrypt "file.pdf.xsalsa20", assuming you are bob and the sender was joe. Sender's
public key is required.

	jfpg -d -f file.pdf.xsalsa20 -p joe-pubkey.curve25519 -k bob-secretkey.curve25519

Encrypt "file.pdf" with a password-derived key

	jfpg -c -f file.pdf

Decrypt "file.pdf.xsalsa20" with password

	jfpg -d -f file.pdf.xsalsa20 

You will need to create a new set of keys when you first use JFPG 
for signing/verifying or asymmetric encryption/decryption. 
This will 2 keypairs, a pair of Curve25519 keys for encryption/decryption
and a pair of Ed25519 keys for signing. It takes your desired key ID
(name, email, etc) as its only option.

Symmetric encryption invoked by the -c option uses scrypt as a key derivation function. 
The optional "rounds" parameter determines both the amount of time and memory consumed by scrypt. 
It is used to derive the scrypt N parameter, where N = 2^rounds.
The minimum number for the rounds option 
is 16 and the maximum is 25. Increasing this by one doubles RAM usage.

Threat Model

JFPG is designed to secure data that is (or will be) in transit or sitting on a remote server.
The asymmetric operations are not really intended to secure local data; since secret
keys are not encrypted, an attacker with filesystem access will be  able to use them. 
Full disk encryption can mitigate this and provide some protection against an attacker
with physical access when the machine is powered off. However, an attacker who is able to steal 
your secret keys can also just steal your plaintext, and possibly sniff your password anyway,
so encrypted secret keys are not a cure-all. Symmetric encryption with the -c option can provide 
some protection of locally stored files as it uses a password to generate an encryption key.
Still, this method is vulnerable to weak passwords or an attacker with the ability to capture your 
password. Securing your machine against such an attacker is beyond the scope of JFPG. 
 
Primitives used

	Signing: Ed25519
	Asymmetric key exchange and cipher: X25519 key exchange with Curve25519 keys and XSalsa20-Poly1305 
	Symmetric cipher: XSalsa20-Poly1305
	Password-based key derivation: Scrypt

Limitations

	Secret keys are not encrypted, as mentioned above. This may be added
	in the future. 

	Decrypting messages on a big-endian machine that were encrypted on a little
	endian machine or vice-versa does not work at the moment. 

	There is no forward secrecy. A given sender/receiver pair will
	calculate the same shared key for all of their messages. This may be 
	added in the future. 

	JFPG does not manage keys for you. This is a problem that is likely
	beyond the ability of a command line utility to handle properly. 

