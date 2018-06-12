[![GoDoc](https://godoc.org/github.com/opencoff/go-sign?status.svg)](https://godoc.org/github.com/opencoff/go-sign)

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

## Example of Keys, Signature

### Ed25519 Public Key
A serialized Ed25519 public key looks like so:

    pk: uxpDh+gqXojAmxA/6vxZHzA+Uk+8wogUwvEhPBlWgvo=

### Ed25519 Private Key
And, a serialized Ed25519 private key looks like so:

    esk: t3vfqHbgUiA733KKPymFjWT8DdnBEkiMfsDHolPUdQWpvVn/F1Z4J6KYV3M5rGO9xgKxh5RAmqt+6LKgOiJAMQ==
    salt: pPHKG55UJYtJ5wU0G9hBvNQJ0DvT0a7T4Fmj4aPB84s=
    algo: scrypt-sha256
    verify: JvjRjJMKhJhBmZngC3Pvq7x3KCLKt7gar1AAz7HB4qM=
    Z: 131072
    r: 16
    p: 1

The Ed25519 private key is encrypted using Scrypt password hashing
mechanism. Any user supplied passphrase to protect the private key
is first pre-hashed using SHA-512 before being used in
```scrypt()``. In pseudo code, this operation looks like below:

    passphrase = get_user_passphrase()
    hpass      = SHA512(passphrase)
    salt       = randombytes(32)
    xorkey     = Scrypt(hpass, salt, N, r, p)
    verify     = SHA256(salt, xorkey)
    esk        = ed25519_private_key ^ xorkey

Where, ```N```, ```r```, ```p``` are Scrypt parameters. In our
implementation:

    N = 131072
    r = 16
    p = 1

```verify```  is used during the decryption of the Ed25519 private
key - before actually doing the "xor" operation. The code checks to
ensure that the supplied passphrase yields the same value as
```verify```.

### Ed25519 Signature
A generated signature looks like below after serialization:

    comment: inpfile=/tmp/file.txt
    pkhash: 36z9tCwTIVNwwDlExrB0SQ==
    signature: ow2oBP+buDbEvlNakOrsxgB5Yc/7PYyPVZCkfyu7oahw8BakF4Qf32uswPaKGZ8RVz4uXboYHdZtfrEjCgP/Cg==

Here, ```pkhash`` is a SHA256 of the public key needed to verify
this signature.

## License
GPL v2.0
