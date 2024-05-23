# gpg-src-test

Setup key:

```sh
# create keypair
gpg --full-generate-key

# answer questions with default values
# (9) ECC (sign and encrypt) *default*
# (1) Curve 25519 *default*
#  0 = key does not expire
# Validate input, and no expiration
# Enter username:  test key
# Enter email: test@test.com

# export public key
gpg --export --armor test@test.com

# should be like
# -----BEGIN PGP PUBLIC KEY BLOCK-----
# --
# -----END PGP PUBLIC KEY BLOCK-----

# create the action secret "GPG_PUBLIC_KEY" in github
# create the action secret "GPG_EMAIL" in github using the email provided --full-generate-key

# export private key to exterrnal drive
gpg --export-secret-key --armor --output /Volumes/"My Other Drive"

# setup the action like shown in /.github/workflows/archive.yml

```
