# SSH Key Generation & SSH Agent Configuration: A Quick Reference For Secure Remote Access

A quick reference guide for [generating SSH keys](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html),
configuring an [ssh-agent](https://linux.die.net/man/1/ssh-agent)
via [keychain](https://github.com/funtoo/keychain/blob/master/keychain.txt) and more.

**Note**: This article is not intended to be a comprehensive guide on the topic. For detailed information and a
comprehensive guide on SSH, I highly recommend referring to the excellent article by
DigitalOcean: '[SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)'.
Their guide covers the important aspects of SSH in great detail including server configuration.

## Generating SSH Keys

See the [ssh-keygen manual](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html) for more information regarding the
flags used below:

### RSA (Rivest-Shamir-Adleman) based Key Generation

```shell
#!/usr/bin/env bash

ssh-keygen \
  -t rsa \
  -a "<Number of KDF (key derivation function) rounds used>" \
  -b "<Number of bits in the key>" \
  -C "<Some comment such as your email address>" \
  -N "<Passphrase used for encrypting your private key>" \
  -f "</path/to/private-key>"
```

For example:

```shell
#!/usr/bin/env bash

ssh-keygen \
  -t rsa \
  -a 128 \
  -b 4096 \
  -C "user@example.com" \
  -N "hopefully-a-very-strong-passphrase" \
  -f "~/.ssh/example-server-1"

```

### ECDSA (Elliptic Curve Digital Signature Algorithm) based Key generation

```shell
#!/usr/bin/env bash

ssh-keygen \
  -t ed25519 \
  -a "Number of KDF (key derivation function) rounds used" \
  -C "Some comment such as your email address>" \
  -N "<Passphrase used for encrypting your private key>" \
  -f "</path/to/private-key>"

```

For example:

```shell
#!/usr/bin/env bash

ssh-keygen \
  -t ed25519 \
  -a 128 \
  -C "user@example.com" \
  -N "hopefully-a-very-strong-passphrase" \
  -f "~/.ssh/example-server-1"

```

**Note**: The `-b bits` option is omitted for `ed25519` based key generation. As per
the [ssh-keygen documentation](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html)

> For ECDSA keys, the -b flag determines the key length by
> selecting from one of three elliptic curve sizes: 256, 384
> or 521 bits. Attempting to use bit lengths other than
> these three values for ECDSA keys will fail. ECDSA-SK,
> Ed25519 and Ed25519-SK keys have a fixed length and the -b
> flag will be ignored.

---

## Managing Multiple SSH Key-Pairs

If you protect all of your private keys with passphrases and assuming that your servers only allow key-based access,
then you would need the following information to connect to your server:

1. Username
2. Server hostname/IP
3. Private key
4. Private Key Passphrase

Keeping track of what username, hostname, private key, and passphrase to use can easily get out of hand when different
keys for different systems are used.

### SSH Configuration

Instead, the hostname, username, and related private key can be configured under:

1. `~/.ssh/config` (User Level)
2. `/etc/ssh/ssh_config` (System Level)

See [ssh_config documentation](https://man7.org/linux/man-pages/man5/ssh_config.5.html) for more information.

```text
Host <Host>
    HostName <hostname/ip>
    User <username>
    IdentityFile </path/to/private-key>
    Port <port, default: 22>
```

Typically, you'd run something similar to the following to connect to a server via SSH:

```shell
#!/usr/bin/env bash

ssh username@hostname -i ~/keys/your-private-key

# For example

ssh admin@example.com -i ~/.ssh/custom/example-key
```

We can define a host configuration under `~/.ssh/config` as follows:

```text
Host example
    HostName example.com
    User admin
    IdentityFile ~/.ssh/custom/example-key
    Port 22
```

This allows us to ssh to `example.com` as the `admin` user without having to specify either the username or hostname by
referencing the `Host` as follows:

```bash
#!/usr/bin/env bash

ssh example

```

However, we'll still be prompted for our private key passphrase.

### ssh-agent

An `ssh-agent` can be configured in addition to ssh configuration to avoid manually providing passphrases over and over
again. As per the [ssh-agent documentation](https://man7.org/linux/man-pages/man1/ssh-agent.1.html):

> ssh-agent is a program to hold private keys used for public key
> authentication. Through use of environment variables the agent can
> be located and automatically used for authentication when logging
> in to other machines using ssh


In other words, the `ssh-agent` is a utility program that runs in the background and solves the problem of repeatedly
entering private key passphrases.

However, an `ssh-agent` on its own is scoped to a single session. This means that if you start a new session, you'll
have to re-enter all of your passphrases.

Instead of just using the `ssh-agent` on its own, we can use [Funtoo Keychain](https://www.funtoo.org/Funtoo:Keychain)
to make things easier.

### Funtoo Keychain for managing ssh-agent

According to the project wiki:

> Keychain helps you to manage SSH and GPG keys in a convenient and secure manner. It acts as a frontend to ssh-agent
> and ssh-add, but allows you to easily have one long running ssh-agent process per system, rather than the norm of one
> ssh-agent per login session.

#### Installing & Configuring Funtoo keychain

- Install keychain:

```bash
#!/usr/bin/env bash

sudo apt install keychain
```

- Configure `keychain` in `~/.bashrc` to automatically start up and prompt for private key passphrases. According
  to the [keychain documentation](https://github.com/funtoo/keychain/blob/master/keychain.txt):

> Typically, private key files are specified by filename only, without path,
> although it is possible to specify an absolute or relative path to the private key file as well. If just a private key
> filename is used, which is typical usage, keychain will look for the specified private key files in ~/.ssh, ~/.ssh2,
> or
> with the -c/--confhost
> option, inspect the ~/.ssh/config file and use the IdentityFile option to determine the location of the private key.
> Private keys can be symlinks to the actual private key.

- As such, add the following command to your `~/.bashrc` to start keychain:

```bash
#!/usr/bin/env bash

eval `keychain --eval <your-private-key-1-in-~/.ssh> </full/path/to/your/private-key-2> ...`

# For Example:
# Suppose my ~/.ssh folder has the private key 'example' and that 
# a key called 'external' is present in my opt directory. 

eval `keychain --eval example /opt/external`
```

- If you start a new session or `source ~/.bashrc`, you'll be prompted to provide the passphrases for the private keys
  you specified. After you provide the passphrases the first time, you won't be prompted again until you either restart
  the system or explicitly clear the keys using `keychain --clear`

---

## Troubleshooting

### Fixing permissions for SSH

The following table can be used as a quick reference to fix any permission-related issues for SSH key files and
configuration:

| Folder / File          | Permissions                                    | Command                            |
|------------------------|------------------------------------------------|------------------------------------|
| ~/.ssh                 | Read, Write and Execute for the User only.     | `chmod 700 ~/.ssh`                 |
| ~/.ssh/authorized_keys | Read and Write for the User only.              | `chmod 600 ~/.ssh/authorized_keys` |
| ~/.ssh/config          | Read and Write for the User only.              | `chmod 600 ~/.ssh/config`          |
| Any private key        | Read and Write for the User only.              | `chmod 600 ~/.ssh/<private-key>`   |
| Any public key         | Readable by all but writable only by the User. | `chmod 644 ~/.ssh/<private-key>`   |

---

## References / Additional Reading

1. "Funtoo Keychain", funtoo, [https://www.funtoo.org/Funtoo:Keychain](https://www.funtoo.org/Funtoo:Keychain)
2. "Funtoo Keychain Latest Documentation",
   funtoo, [https://github.com/funtoo/keychain/blob/master/keychain.txt](https://github.com/funtoo/keychain/blob/master/keychain.txt)
3. "How can I run ssh-add automatically without a password prompt", UNIX Stack
   Exchange, [https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt](https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt)
4. "How many kdf rounds for an ssh key", Crypto Stack
   Exchange, [https://crypto.stackexchange.com/questions/40311/how-many-kdf-rounds-for-an-ssh-key](https://crypto.stackexchange.com/questions/40311/how-many-kdf-rounds-for-an-ssh-key)
5. "Permissions on private key in ssh folder",
   superuser.com, [https://superuser.com/questions/215504/permissions-on-private-key-in-ssh-folder](https://superuser.com/questions/215504/permissions-on-private-key-in-ssh-folder)
6. "Safe Curves", safecurves.cr.yp.to, [https://safecurves.cr.yp.to/](https://safecurves.cr.yp.to/)
7. "ssh(1) — Linux manual
   page", [https://man7.org/linux/man-pages/man1/ssh.1.html](https://man7.org/linux/man-pages/man1/ssh.1.html)
8. "ssh-add(1) — Linux manual
   page", [https://man7.org/linux/man-pages/man1/ssh-add.1.html](https://man7.org/linux/man-pages/man1/ssh-add.1.html)
9. "ssh-agent(1): authentication agent - Linux man
   page", [https://man7.org/linux/man-pages/man1/ssh-agent.1.html](https://man7.org/linux/man-pages/man1/ssh-agent.1.html)
10. "ssh_config(5) — Linux manual
    page", [https://man7.org/linux/man-pages/man5/ssh_config.5.html](https://man7.org/linux/man-pages/man5/ssh_config.5.html)
11. "ssh-keygen(1) — Linux manual page", Linux Manual
    Page, [https://man7.org/linux/man-pages/man1/ssh-keygen.1.html](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html)
12. "SSH Essentials: Working with SSH Servers, Clients, and Keys.", DigitalOcean (Ellingwood,
    Justin.), [https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys).
13. “What Is SSH-Keygen & How to Use It to Generate a New Ssh Key?”, SSH Communications
    Security, [https://www.ssh.com/academy/ssh/keygen](https://www.ssh.com/academy/ssh/keygen)
14. “Working with SSH Key Passphrases.”
    GitHub, [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases).