# PGP basic commands

> Draft
{.is-warning}

> Using `keys.openpgp.org`, feel free to change it as you need it.
{.is-info}


## Import keys

```bash
# Same command is for import private and public keys
gpg --import private-keys.asc
gpg --import pgp-public-keys.asc
gpg --import-ownertrust pgp-ownertrust.asc
```

### Import keys from keyserver

```bash
gpg --keyserver keys.openpgp.org --receive-keys <FINGERPRINT>
```

## Edit keys

```bash
gpg --edit-key <FINGERPRIN>
```

## Exporting keys

```bash
gpg --armor --export > pgp-public-keys.asc
gpg --armor --export-secret-keys > pgp-private-keys.asc
gpg --export-ownertrust > pgp-ownertrust.asc
```

### Export public keys to keyserver

```bash
gpg --keyserver keys.openpgp.org --send-keys <FINGERPRING>
```

## Signatures

```bash
gpg --local-user <your-email> --detach-sign test.txt.sig test.txt

# readable signature
gpg --armor --local-user <your-email> --detach-sign test.txt

# Verify signature
gpg --verify text.txt.sig
```

## Encrypt and sign files

```bash
gpg --encrypt --sign --armor -r person@email.com name_of_file

# or with fingerprint
gpg --armor --local-user <fingerprint> --encrypt --sign --recipient <mail@server.tld> test.md
```

## Decrypt

```bash
gpg file_name.asc
```

## Keybase

To import your pgp keys to keybase run command as follows

```bash
gpg --armor --export-secret-keys <fingerprint> | keybase pgp import
```

> Is not recommended to push your private key somewhere to internet
{.is-danger}


If you want to host your private key on keybase add `--push-secret` right after `import` so command will looks as follows

```bash
gpg --armor --export-secret-keys <fingerprint> | keybase pgp import --push-secret
```
