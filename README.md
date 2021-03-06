# libsodium-net [![Build Status](https://travis-ci.org/adamcaudill/libsodium-net.svg?branch=master)](https://travis-ci.org/adamcaudill/libsodium-net) [![NuGet Version](http://img.shields.io/nuget/v/libsodium-net.svg)](https://www.nuget.org/packages/libsodium-net/) [![License](http://img.shields.io/badge/license-MIT-green.svg)](https://github.com/adamcaudill/libsodium-net/blob/master/LICENSE)

libsodium-net, or better said, [libsodium](https://github.com/jedisct1/libsodium) for .NET, is a C# wrapper around libsodium. For those that don't know, libsodium is a portable implementation of [Daniel Bernstein's](http://cr.yp.to/djb.html) fantastic [NaCl](http://nacl.cr.yp.to/) library. If you aren't familiar with NaCl, I highly suggest that you look into libsodium and NaCl before using this library.

Want to support development? Consider donating via Bitcoin to `14jumFDmuVkLiAt4TgyKt17SWHtPRbkcLr` - all donations, no matter how small are appreciated.

## Why

NaCl is a great library in that its designed has made the right choices on what to implement and how - something most developers don't know how to do. So by using it (or a wrapper), many of those details are abstracted away where you don't need to worry about them. NaCl itself is less than portable C, only targeted for *nix systems; libsodium solves this by making it portable and making a few minor changes to better suite being distributed as a compiled binary.

Crypto is hard - much harder than your average developer understands. This effort was started to make these tools readily available to the .NET community in hopes they will be used to further the goals of defending personal privacy and security.

## Installation

**Windows**: For Windows, the `libsodium` library is included in the [release](https://github.com/adamcaudill/libsodium-net/releases) packages. Or just use the [NuGet version](https://www.nuget.org/packages/libsodium-net/) which has everything you need.

**OSX**: For OSX, `libsodium-net` can easily be built in Xamarin Studio, and `libsodium` can be installed easily via `brew`:

    brew install libsodium --universal

**Linux**: As with OSX, building with Xamarin Studio is simple, or there's always the option of using `xbuild`:

    xbuild libsodium-net.sln

For `libsodium`, many package managers provide older versions, so it's recommended to build the latest version from source. Thankfully, this is a fairly painless process. See the [travis-build-libsodium.sh](https://github.com/adamcaudill/libsodium-net/blob/master/travis-build-libsodium.sh) file or the `libsodium` [README](https://github.com/jedisct1/libsodium/blob/master/README.markdown) file for details.

**Other**: Support for other Mono supported platforms hasn't been determined. It may or may not work.

Note: For all platforms, it's critical that `libsodium` be compiled for the architecture that the process is running under. If they don't match, you can expect to see errors. If your process is x86/i386, you can't use a copy of `libsodium` compiled for x64.

## Methods Supported

The following methods have been implemented and have at least basic unit tests in place to ensure they are producing the expected output. *(Listed in no particular order)*

### Asymmetric

#### crypto_sign_keypair
`Sodium.PublicKeyAuth.GenerateKeyPair()` - Generates a public/private [Ed25519](http://ed25519.cr.yp.to/) key pair based on a random seed. The public key is 32 bytes, the private key is 64 bytes.

#### crypto_sign_seed_keypair
`Sodium.PublicKeyAuth.GenerateKeyPair()` - Generates a public/private [Ed25519](http://ed25519.cr.yp.to/) key pair based on the provided seed to allow deterministic key generation. The public key is 32 bytes, the private key is 64 bytes, seed must be 32 bytes.

#### crypto_sign
`Sodium.PublicKeyAuth.Sign()` - Signs a message with [Ed25519](http://ed25519.cr.yp.to/), based on the supplied 64 byte private key.

#### crypto_sign_open
`Sodium.PublicKeyAuth.Verify()` - Verifies the signature and returns the clear-text message using [Ed25519](http://ed25519.cr.yp.to/) and the supplied 32-byte public key.

#### crypto_sign_detached
`Sodium.PublicKeyAuth.SignDetached()` - Similar to `crypto_sign`, except returns a `Signature` without the message.

#### crypto_sign_verify_detached
`Sodium.PublicKeyAuth.VerifyDetached()` - Verifies the signature and returns `true` or `false`. Throws a `CryptographicException` if verification fails.

#### crypto_sign_ed25519_pk_to_curve25519
`Sodium.PublicKeyAuth.ConvertEd25519PublicKeyToCurve25519PublicKey()` - Converts a Ed25519 public key, to a Curve25519 public key.

#### crypto_sign_ed25519_sk_to_curve25519
`Sodium.PublicKeyAuth.ConvertEd25519SecretKeyToCurve25519SecretKey()` - Converts a Ed25519 private key, to a Curve25519 private key.

#### crypto_box_keypair
`Sodium.PublicKeyBox.GenerateKeyPair()` - Generates a public/private Curve25519-XSalsa20-Poly1305 key pair based on a random seed. The public key is 32 bytes, the private key is 32 bytes.

#### crypto_box
`Sodium.PublicKeyBox.Create()` - Encrypts a message using the sender's private key, and the recipient's public key. Both keys are 32 bytes. Encryption / signing is performed via Curve25519-XSalsa20-Poly1305.

#### crypto_box_open
`Sodium.PublicKeyBox.Open()` - Decrypts and verifies the sender's signature using the recipient's private key and the sender's public key. Both keys are 32 bytes. Throws a `CryptographicException` if verification fails.

#### crypto_box_detached
`Sodium.PublicKeyBox.CreateDetached()` - Similar to `crypto_box`, except returns a `DedatchedBox` object, with the cipher text and MAC separated.

#### crypto_box_open_detached
`Sodium.PublicKeyBox.OpenDetached()` - Similar to `crypto_box_open`, except dealing with a detached MAC. See `crypto_box_detached`.

### Symmetric

#### crypto_secretbox
`Sodium.SecretBox.Create()` - This method encrypts and authenticates a message. This is currently implemented via [XSalsa20](https://en.wikipedia.org/wiki/Salsa20) and [Poly1305](https://en.wikipedia.org/wiki/Poly1305).

Details:

 * Key length: 32 bytes
 * Nonce length: 24 bytes

Note: As in any crypto system, it's important to avoid reusing a nonce, so the nonce should either be randomly generated or consist of a non-repeatable message ID. To minimize risks, my recommendation is to randomly generate the nonce.

#### crypto_secretbox_open
`Sodium.SecretBox.Open()` - This method retrieves the message encrypted via `SecretBox.Create()`. To decrypt and authenticate the data, you pass the key, nonce, and cipher text; if there is an issue decrypting the message, you will receive a generic `CryptographicException` - the exact reason for the failure isn't provided.

#### crypto_secretbox_detached
`Sodium.SecretBox.CreateDetached()` - Similar to `crypto_secretbox`, except returns a `DedatchedBox` object, with the cipher text and MAC separated.

#### crypto_box_open_detached
`Sodium.SecretBox.OpenDetached()` - Similar to `crypto_secretbox_open`, except dealing with a detached MAC. See `crypto_secretbox_detached`.

#### crypto_onetimeauth
`Sodium.OneTimeAuth.Sign()` - Provides messages authentication via [Poly1305](https://en.wikipedia.org/wiki/Poly1305). The method takes a messages (as UTF-8 string or byte array), a 32 byte key (that must be used only once), and returns a 16 bytes signature.

For this to be secure, it's required that the signing key only be used once.

#### crypto_onetimeauth_verify
`Sodium.OneTimeAuth.Verify()` - This verifies a signature generated via `crypto_onetimeauth`. To do so, it requires the original message, the 16 byte signature, and the 32 byte key.

#### crypto_auth
`Sodium.SecretKeyAuth.Sign()` - Provides messages authentication via HMAC-SHA512-256. The method takes a messages (as UTF-8 string or byte array), a 32 byte key, and returns a 32 bytes signature.

#### crypto_auth_verify
`Sodium.SecretKeyAuth.Verify()` - This verifies a signature generated via `crypto_auth`. To do so, it requires the original message, the 32 byte signature, and the 32 byte key.

#### crypto_auth_hmacsha256
`Sodium.SecretKeyAuth.SignHmacSha256()` - Provides messages authentication via HMAC-256. The method takes a messages (as UTF-8 string or byte array), a 32 byte key, and returns a 32 bytes signature.

#### crypto_auth_hmacsha256_verify
`Sodium.SecretKeyAuth.VerifyHmacSha256()` - This verifies a signature generated via `crypto_auth_hmacsha256`. To do so, it requires the original message, the 32 byte signature, and the 32 byte key.

#### crypto_auth_hmacsha512
`Sodium.SecretKeyAuth.SignHmacSha512()` - Provides messages authentication via HMAC-512. The method takes a messages (as UTF-8 string or byte array), a 32 byte key, and returns a 64 bytes signature.

#### crypto_auth_hmacsha512_verify
`Sodium.SecretKeyAuth.VerifyHmacSha512()` - This verifies a signature generated via `crypto_auth_hmacsha512`. To do so, it requires the original message, the 64 byte signature, and the 32 byte key.

#### crypto_aead_chacha20poly1305_encrypt
`Sodium.SecretAead.Encrypt()` - Encrypts a message with an authentication tag and additional data.

#### crypto_aead_chacha20poly1305_decrypt
`Sodium.SecretAead.Decrypt()` - Decrypts a cipher with an authentication tag and additional data.

#### crypto_stream_xor
`Sodium.StreamEncryption.GenerateNonce()` - Returns a 24 random byte nonce.

`Sodium.StreamEncryption.Encrypt()` - Encrypts a message via XSalsa20 using a 32 byte key and a 24 byte nonce. As always, it's critical that the nonce never be reused. This provides encryption only, not authentication.

`Sodium.StreamEncryption.Decrypt()` - Decrypts messages via XSalsa20.

#### crypto_stream_chacha20_xor
`Sodium.StreamEncryption.GenerateNonceChaCha20()` - Returns a 8 random byte nonce.

`Sodium.StreamEncryption.EncryptChaCha20()` - Encrypts a message via ChaCha20 using a 32 byte key and a 8 byte nonce. This provides encryption only, not authentication.

`Sodium.StreamEncryption.DecryptChaCha20()` - Decrypts messages via ChaCha20.

The nonce is 64 bits long. In order to **prevent** nonce reuse, if a key is being reused, it is recommended to **increment** the previous nonce instead of generating a random nonce every time a new stream is required.

### Hashing

#### crypto_hash
`Sodium.CryptoHash.Hash()` - This hashes a message (UTF-8 encoded `string` or `byte[]`) using the default algorithm, currently SHA-512. A byte array is returned.

The `Sodium.CryptoHash` class also includes the following methods:

 * `Sha512()` - Compute a SHA-512 hash (for compatibility if the default algorithm is ever changed). (`crypto_hash_sha512`)
 * `Sha256()` - Compute a SHA-256 hash. (`crypto_hash_sha256 `)

#### crypto_generichash
`Sodium.GenericHash.Hash()` - This is a multi-purpose fast hash, that has variable output size and an optional key. As with the `CryptoHash` methods, the input can be a UTF-8 encoded string, or a byte array, and a byte array is returned. 

This is the most flexible hashing option; and the one I would recommend the most.

This is currently based on [BLAKE2b](https://blake2.net/), and has the following properties:

 * Variable output from 16 to 64 bytes.
 * Optional keyed mode, accepting a key from 16 to 64 bytes.

Note: Only libsodium's simplified interface is currently supported; the streaming interface is not implemented at this time. 

#### crypto_shorthash
`Sodium.ShortHash.Hash()` - Short, high-speed hashing, currently implanted via [SipHash-2-4](https://en.wikipedia.org/wiki/SipHash) and produces an 8 byte hash.

#### crypto_generichash_blake2b_salt_personal
`Sodium.GenericHash.HashSaltPersonal()` - Hash with salt, personal and optional key.

#### crypto_pwhash_scryptsalsa208sha256_str
`Sodium.PasswordHash.ScryptHashString()` - Returns the hash is a string format, which includes the generated salt.

#### crypto_pwhash_scryptsalsa208sha256_str_verify
`Sodium.PasswordHash.ScryptHashStringVerify()` - Verifies that a hash generated with `ScryptHashString` matches the supplied password.

#### crypto_pwhash_scryptsalsa208sha256
`Sodium.PasswordHash.ScryptHashBinary()` - Derives a secret key of any size from a password and a salt. There exists an overloaded version with some predefined limits for an easy usage.

### Other

#### sodium_version_string
`Sodium.SodiumCore.SodiumVersionString()` - This returns the version string on the `libsodium` library in use. Note, this is not the version of libsodium-net.

#### randombytes_buf
`Sodium.SodiumCore.GetRandomBytes()` - Gets a number of random bytes, suitable for use as a key or nonce. In classes where appropriate, there are `GenerateKey()` and/or `GenerateNonce()` functions that return a byte array of the correct size.

#### crypto_scalarmult
`Sodium.ScalarMult.Mult()` - Diffie-Hellman (function computes a secret shared by the two keys)

#### crypto_scalarmult_base
`Sodium.ScalarMult.Base()` - Diffie-Hellman (computes the public key from the given secret key)

### Utilities and Helper

#### sodium_hex2bin
`Sodium.Utilities.HexToBinary()` - This method converts a hex-encoded string (lower-case, upper-case or with some extra chars like: Hyphen, Colon and Space) to a byte array.

#### sodium_bin2hex
`Sodium.Utilities.BinaryToHex()` - This method takes a byte array, and produces a lower-case hex-encoded string.

### Non-libsodium Methods

There are a small number of additional methods that extended or simplify the usage of this library.

`Sodium.Utilities.BinaryToHex()` - This overload takes a byte array, and produces a lower-case or upper-case hex-encoded string, it also can add some extra chars (Hyphen, Colon and Space) to generate a human readable format.

Note: This overload doesn't use the sodium_bin2hex implementation.

## Requirements & Versions

This library can be built in Visual Studio 2010, Xamarin Studio (MonoDevelop 3.x supported), and targets .NET 4.0; it is compiled against libsodium v1.0.0.

On OSX & Linux, your copy of `libsodium` must be compiled for the same architecture as your copy of Mono. If you are running a 32bit process, your copy of `libsodium` must be 32bit as well.

## Notes

Any method that takes a String, has an overload that accepts a byte array; Strings are assumed to be UTF8; if this is not the case, please convert it to bytes yourself and use the overloads that accept byte arrays.

## File Signing

Starting with version 0.4.0, all files are signed via a Certum.pl Code Signing certificate. The files are signed under the name `Open Source Developer, Adam Caudill` - this can be used to ensure that the files haven't been altered.

## License

NaCl has been released to the public domain to avoid copyright issues. libsodium is subject to the [ISC license](https://en.wikipedia.org/wiki/ISC_license), and this software is subject to the MIT license (see LICENSE).
