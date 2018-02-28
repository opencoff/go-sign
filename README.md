# go-sign - Ed25519 signature calculation and verification

## What is it?
A library to generate, verify and store Ed25519 keys and signatures.
It uses the extended library (golang.org/x/crypto) for the
underlying operations.

The generated keys and signatures are proper YAML files and human
readable.

The signature file contains a hash of the public key - so that at
verification time, the right private key may be used (in situations
where there are lots of keys).

Signatures on large files are calculated efficiently by reading them
in memory mapped mode (```mmap(2)```) and hashing the file contents
using SHA-512. The Ed25519 signature is calculated on the file-hash.

## License
GPL v2.0
