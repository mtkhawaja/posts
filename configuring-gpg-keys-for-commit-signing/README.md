# Configuring GPG Keys for Commit Signing

## Install GnuPG (OpenPGP)

```shell
#/usr/bin/env zsh

brew install gnupg
```

Or if you're on a linux machine without gpg:

```shell
#!/usr/bin/env zsh 

sudo apt-get install gnupg
```

## Generate a PGP Key

```shell
#!/usr/bin/env zsh

gpg --full-generate-key
```

### Prompt 1: Key Type

Select the Default Option i.e. `(9) ECC (sign and encrypt)`

```plaintext
➜  ~ gpg --full-generate-key
gpg (GnuPG) 2.4.4; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection?
```

### Prompt 2: Elliptic curve

Select the Default Option i.e. `(1) Curve 25519`

```plaintext
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection?
```

### Prompt 3: Key Expiry

Pick a suitable expiry period.

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
```

### Remaining Prompts

Fill in your user ID (name/email) and choose a strong passphrase for the key.

## Listing Keys

```shell
#!/usr/bin/env zsh

gpg --list-secret-keys --keyid-format=long
```

## GPG Agent Configuration

The GPG Agent should be up and running automatically. Check the GPG Agent Socket path using the following command:

```shell
#!/usr/bin/env zsh
# Sample output: $HOME/.gnupg/S.gpg-agent
gpgconf --list-dirs agent-socket
```

However, make sure to set `GPG_TTY=$(tty)` in your shell configuration as per the GPG documentation:

> You should always add the following lines to your .bashrc or whatever initialization file is used for all shell invocations:
> GPG_TTY=$(tty)
> export GPG_TTY

In addition, you may want to optionally override the default cache TTLs:

```shell
#!/usr/bin/env zsh
mkdir -p "$HOME/.gnupg"
chmod 700 "$HOME/.gnupg"
# "$HOME/.gnupg/gpg-agent.conf"
# Note: See GPG Agent Configuration Options: https://www.gnupg.org/documentation/manuals/gnupg/Agent-Options.html
# --default-cache-ttl default is 600 seconds (10 minutes) | Override below =>  86400  = 1 Day
# --max-cache-ttl default is 7200 seconds (2 hours)       | Override below =>  604800 = 7 days
#
{
  echo "default-cache-ttl 86400"
  echo "max-cache-ttl 604800"
} >> "$HOME/.gnupg/gpg-agent.conf"
# Restart the agent:
gpgconf --kill gpg-agent
```

## Configuring Git

```shell
#!/usr/bin/env zsh

# Configure git to use a particular GPG key
git config --global user.signingkey <key-id>

# Configure git to sign all tags + commits by default
git config --global commit.gpgsign true
git config --global tag.gpgSign true
```

## Verifying a Commit

```shell
#!/usr/bin/env zsh

git log --show-signature
```

Sample output:

```plaintext
commit 78c95a95965d7d9ef816ed192396ba875b0962dc (HEAD -> main)
gpg: Signature made Wed Dec  3 01:10:46 2025 EST
gpg:                using EDDSA key AB244C5A61307221FAC18D84535C84F6751CC5C4
gpg: Good signature from "Muneeb Khawaja <36654508+mtkhawaja@users.noreply.github.com>" [ultimate]
Author: Muneeb Khawaja <36654508+mtkhawaja@users.noreply.github.com>
```

## Extras

## GPG Keys Uploaded to GitHub:

Any gpg public keys uploaded to GitHub will be available via the following URL `https://github.com/<username>.gpg` e.g. [https://github.com/mtkhawaja.gpg](https://github.com/mtkhawaja.gpg)

## Upload GPG Pub Keys on Public Key Servers:

- [OpenPGP Servers](https://keys.openpgp.org/upload)
- [Ubuntu Key Servers](https://keyserver.ubuntu.com)

## Digitally Signing Artifacts

```shell
#!/usr/bin/env zsh

# Different encodings for the same data:
# .asc ==> ascii armor format (by convention)
# .sig ==> binary  encoding (by convention)
gpg --armor --detach-sig "<doc-to-sign>"
# Or shortened
gpg -ab "<doc-to-sign>"
```

## Using GPG Keys to Validate Artifact Authenticity

### Sample Scenario: Validating a JAR

Suppose we're looking at the [springboot-starter-web](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.0/) release artifacts.

There are a bunch of artifacts available for download. But let's just look at the JAR and related PGP signature:

```shell
#!/usr/bin/env zsh
# Download JAR
wget "https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.0/spring-boot-starter-web-4.0.0.jar"
# Download PGP Signature in ASCII format
wget "https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.0/spring-boot-starter-web-4.0.0.jar.asc"
# MD5 Checksum for JAR
wget "https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.0/spring-boot-starter-web-4.0.0.jar.md5"
# SHA1 Checksum for JAR
wget "https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.0/spring-boot-starter-web-4.0.0.jar.sha1"
```

After downloading the artifacts, we should have the following files:

| Artifact                               | Purpose                    | Description                                                                                                                 |
|----------------------------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| spring-boot-starter-web-4.0.0.jar      | Consumption                | The compiled Java library you’ll use as a dependency.                                                                       |
| spring-boot-starter-web-4.0.0.jar.md5  | Integrity Only             | MD5 checksum of the JAR File                                                                                                |
| spring-boot-starter-web-4.0.0.jar.asc  | Integrity and Authenticity | PGP/GPG detached signature for the JAR. Used to verify authenticity and integrity using the developer’s public signing key. |
| spring-boot-starter-web-4.0.0.jar.sha1 | Integrity Only             | SHA-1 checksum of the JAR File                                                                                              |

**Note**: We don't verify the JAR integrity using the checksums in this article but you can verify these with `shasum --check` / `md5sum --check`

#### Importing Spring's PGP Key

From the [spring website](https://spring.io/GPG-KEY-spring.txt)

> Spring JARs released on Maven Central are signed with the following key.
>
> Key ID: builds@springframework.org
>
> Fingerprint: 48B0 86A7 D843 CFA2 58E8 3286 928F BF39 003C 0425
>
> Key size: RSA 4096
>
> Date: 2023-01-16
>
> You can import this key using a public key server:
>
> $ gpg --keyserver keyserver.ubuntu.com --recv 48B086A7D843CFA258E83286928FBF39003C0425
>
> You can also verify locally a manually downloaded key with:
>
> $ gpg --import --import-options show-only spring.gpg

**Note**: When possible, you should avoid getting both the public key and the artifacts you’re validating from the exact same place, because a single compromised origin could silently serve you both a
malicious key and malicious artifacts that all “verify” cleanly. Instead, treat key verification as an out-of-band process:

1. Get the fingerprint (or the key itself) via one or more independent channels e.g., official documentation, the official project website, multiple key servers etc.
2. Compare that fingerprint to the key you actually imported.

We'll go ahead and import the PGP key in from the public ubuntu key servers:

```shell
#!/usr/bin/env zsh

gpg --keyserver keyserver.ubuntu.com --recv 48B086A7D843CFA258E83286928FBF39003C0425
```

#### Validating JAR PGP Signature

And now we can validate the signature as follows:

```shell
#!/usr/bin/env zsh

gpg --verify spring-boot-starter-web-4.0.0.jar.asc spring-boot-starter-web-4.0.0.jar
```

If the signature is valid, we'll see the following output:

```plaintext
gpg: Signature made Thu Nov 20 12:41:49 2025 EST
gpg:                using RSA key 48B086A7D843CFA258E83286928FBF39003C0425
gpg: Good signature from "Spring Builds (JAR Signing) <builds@springframework.org>" [unknown]
gpg:                 aka "Spring Builds (JAR Signing) <buildmaster@springframework.org>" [unknown]
gpg:                 aka "Spring Builds (JAR Signing) <1134463+spring-builds@users.noreply.github.com>" [unknown]
gpg:                 aka "Spring Builds (JAR Signing) <spring-builds@vmware.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 48B0 86A7 D843 CFA2 58E8  3286 928F BF39 003C 0425
```

#### Trusting the Key (Optional)

Notice, however, that we're getting an ominous message in the output:

```plaintext
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
```

The warning above does not mean the signature is invalid or that the file is malicious. It just means GPG can’t vouch for the identity beyond 'this key signed this file.'

Trust and Validity are two different concepts in GPG:

* **Validity** is GnuPG’s judgment about whether a key really belongs to the identity it claims (i.e., whether the key–user ID binding is authentic).
* **Trust** (ownertrust) expresses how much you trust the key’s owner to correctly verify and sign other people’s keys. Highly trusted keys can influence GPG’s decision to mark additional keys as
  valid in the web-of-trust model.

We can trust the key by starting an interactive session to edit the key:

```shell
#!/usr/bin/env zsh

# This will start an interactive session to edit the key
gpg --edit-key "48B086A7D843CFA258E83286928FBF39003C0425"
# The next command will start a trust dialog
trust
```

After this, we should see the following prompt asking us our trust level. Refer to [the gpg documentation for more details](https://www.gnupg.org/gph/en/manual/x334.html):

```plaintext
Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say                                                                      
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu
```

Let's go with option 3, which, as per the documentation, implies:

> The owner understands the implications of key signing and properly validates keys before signing them.

Now to certify the key as **valid** (who it belongs to), we need to sign the key with our own private key:

```shell
#!/usr/bin/env zsh
# The sign command will start a new dialog
# ⚠️ Only sign third-party vendor keys if you have verified the owner's identity using strong out-of-band channels. 
lsign
```

After typing sign we'll be asked: `Really sign all user IDs? (y/N)` and if we type `y` we'll see a list of the user IDs / keys we signed.

Then type in `save` to save our changes and exit the interactive session. If we run the gpg verification again, we should see the following output:

```plaintext
gpg: Signature made Thu Nov 20 12:41:49 2025 EST
gpg:                using RSA key 48B086A7D843CFA258E83286928FBF39003C0425
gpg: Good signature from "Spring Builds (JAR Signing) <builds@springframework.org>" [full]
gpg:                 aka "Spring Builds (JAR Signing) <buildmaster@springframework.org>" [full]
gpg:                 aka "Spring Builds (JAR Signing) <1134463+spring-builds@users.noreply.github.com>" [full]
gpg:                 aka "Spring Builds (JAR Signing) <spring-builds@vmware.com>" [full]
```

#### Tampering with the Artifacts

If we tamper with the jar, e.g., let's say we append random chars at the end:

```shell
#!/usr/bin/env zsh
echo "random chars" >> "spring-boot-starter-web-4.0.0.jar"
```

And then run the verification again, we should see the following output:

```plaintext
gpg: Signature made Thu Nov 20 12:41:49 2025 EST
gpg:                using RSA key 48B086A7D843CFA258E83286928FBF39003C0425
gpg: BAD signature from "Spring Builds (JAR Signing) <builds@springframework.org>" [full]
```

## Backing up and Importing PGP Keys

### Exporting GPG Pub Key

```shell
#!/usr/bin/env zsh

gpg --armor --export "<key id>"
```

### Exporting GPG Private Key

```shell
#!/usr/bin/env zsh 

gpg  --armor --export-secret-key <key-id>
```

### Exporting Revocation Certificate

```shell
#!/usr/bin/env zsh

gpg --output revoke.asc --gen-revoke "<key-id>"
```

### Importing a PGP Key Pair from Backup

```shell
#!/usr/bin/env zsh

gpg --import <path/to/private/key/file>

# Or from STDIN

## Via Curl

curl <key-url> | gpg --import

## Clipboard (OSX)

pbpaste | gpg --import 
```

## OSX Specific Configuration

pinentry-mac is a macOS-native passphrase prompt used by GPG.
GnuPG depends on a secure pinentry program for collecting passwords and PINs, and the standard terminal-based versions often fail or
behave inconsistently in macOS GUI environments.
This tool provides a secure, reliable dialog for those prompts.

```shell
#!/usr/bin/env zsh

# Install and configure Pinentry  
# macOS must install `pinentry-mac`:

brew install pinentry-mac

echo "pinentry-program /opt/homebrew/bin/pinentry-mac" >> "$HOME/.gnupg/gpg-agent.conf"

# Restart the agent:
gpgconf --kill gpg-agent
```

## References

- [brew install gnupg](https://formulae.brew.sh/formula/gnupg)
- [On the Prevalence and Usage of Commit Signing on GitHub: A Longitudinal and Cross-Domain Study
  ](https://arxiv.org/abs/2504.19215)
- [gnupg](https://www.gnupg.org/download/index.html)
- [gnupg - agent options](https://www.gnupg.org/documentation/manuals/gnupg/Agent-Options.html)
- [GitHub GPG Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)
- [gnupg - manual](https://www.gnupg.org/gph/en/manual/x135.html)
- [Jetbrains GPG Signing](https://www.jetbrains.com/help/pycharm/2025.2/set-up-GPG-commit-signing.html?keymap=macOS&Set_up_GPG_commit_signing=&utm_medium=link&utm_campaign=PY&utm_source=product&utm_content=2025.2)
- [What is the armored option for in GnuPG?](https://unix.stackexchange.com/questions/623375/what-is-the-armored-option-for-in-gnupg)
- [GPG why is my trusted key not certified with a trusted signature?](https://security.stackexchange.com/questions/147447/gpg-why-is-my-trusted-key-not-certified-with-a-trusted-signature)
- [Validating other keys on your public keyring](https://www.gnupg.org/gph/en/manual/x334.html)
- [What's the difference between trusting a key and signing it?](https://security.stackexchange.com/questions/129540/whats-the-difference-between-trusting-a-key-and-signing-it)
- [How to suppress "WARNING: This key is not certified with a trusted signature!"](https://superuser.com/questions/1435147/how-to-suppress-warning-this-key-is-not-certified-with-a-trusted-signature)