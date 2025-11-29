# SSH Key Generation & SSH Agent Configuration: A Quick Reference For Secure Remote Access

A quick reference guide for [generating SSH keys](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html), configuring an [ssh-agent](https://linux.die.net/man/1/ssh-agent)
via both [keychain](https://github.com/funtoo/keychain/blob/master/keychain.txt) and Bitwarden.

**Note**: This page is meant as a practical, opinionated quick reference for everyday SSH use (key generation, agent setup, permissions). For a deeper, more exhaustive guide on SSH,
I still highly recommend the excellent article by
DigitalOcean: “[SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)”,
which covers the important aspects of SSH in great detail, including server configuration.

---

## Generating SSH Keys

See the [ssh-keygen manual](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html) for more information regarding the flags used below:

### RSA (Rivest–Shamir–Adleman) based key generation

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

### ed25519 based Key generation

```shell
#!/usr/bin/env bash

ssh-keygen \
  -t ed25519 \
  -a "<Number of KDF (key derivation function) rounds used>" \
  -C "<Some comment such as your email address>" \
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

**Note**: The `-b bits` option is omitted for `ed25519` based key generation. As per the [ssh-keygen documentation](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html):

> For ECDSA keys, the -b flag determines the key length by
> selecting from one of three elliptic curve sizes: 256, 384
> or 521 bits. Attempting to use bit lengths other than
> these three values for ECDSA keys will fail. ECDSA-SK,
> Ed25519, and Ed25519-SK keys have a fixed length and the -b
> flag will be ignored.

---

### Directly within Bitwarden

You can generate and [store SSH keys directly within Bitwarden](https://Bitwarden.com/help/ssh-agent/#create-new-ssh-key).

At the time of writing:

> At this time, Bitwarden can only generate ED25519 type SSH keys.

## Managing Multiple SSH Key-Pairs

According to the [-i identity_file section of the official ssh(1) documentation](https://man7.org/linux/man-pages/man1/ssh.1.html?utm_source=chatgpt.com), the SSH client automatically searches for a
set of default private key files whenever you don’t explicitly specify an identity. These defaults are:

- ~/.ssh/id_rsa
- ~/.ssh/id_ecdsa
- ~/.ssh/id_ecdsa_sk
- ~/.ssh/id_ed25519
- ~/.ssh/id_ed25519_sk

If one or more of these exist, SSH will try them in order. If your keys use different filenames or live outside the `~/.ssh` directory, you’ll need to point SSH to the correct file using `-i <path>`
or by defining an IdentityFile entry in `~/.ssh/config` on a per-host basis.

Now, if you secure all your private keys with passphrases, and your servers only permit key-based authentication—connecting to a server usually requires four pieces of information:

1. Username
2. Server hostname or IP
3. Private key
4. Private key passphrase

When you manage multiple systems, each with different accounts and different keys, juggling all this quickly becomes painful. To bring sanity and consistency to the process, we’ll use a combination of
SSH configuration and ssh-agent to automate key selection and reduce the cognitive load.

### SSH configuration

The hostname, username, and related private key can be configured under:

1. `~/.ssh/config` (User level)
2. `/etc/ssh/ssh_config` (System level)

See the [ssh_config documentation](https://man7.org/linux/man-pages/man5/ssh_config.5.html) for more information.

```text
Host <Host>
    HostName <hostname/ip>
    User <username>
    IdentityFile </path/to/private-key>
    Port <port, default: 22>
```

Typically, you’d run something similar to the following to connect to a server via SSH:

```shell
#!/usr/bin/env bash

ssh username@hostname -i ~/keys/your-private-key

# For example:

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

This allows us to SSH to `example.com` as the `admin` user without having to specify either the username or hostname by referencing the `Host` as follows:

```shell
#!/usr/bin/env bash

ssh example
```

However, we’ll still be prompted for our private key passphrase.

### ssh-agent

An `ssh-agent` can be configured (in addition to the SSH configuration above) to avoid manually providing passphrases over and over again. As per
the [ssh-agent documentation](https://man7.org/linux/man-pages/man1/ssh-agent.1.html):

> ssh-agent is a program to hold private keys used for public key
> authentication. Through use of environment variables the agent can
> be located and automatically used for authentication when logging
> in to other machines using ssh

In other words, the `ssh-agent` is a utility program that runs in the background and solves the problem of repeatedly entering private key passphrases.

However, an `ssh-agent` on its own is scoped to a single session. This means that if you start a new session, you’ll have to re-enter all of your passphrases.

Instead of just using the `ssh-agent` on its own, we can use [Funtoo Keychain](https://www.funtoo.org/Funtoo:Keychain) to make things easier.

### Option 1: Funtoo Keychain for managing ssh-agent

According to the [project documentation](https://github.com/danielrobbins/keychain):

> Keychain helps you to manage SSH and GPG keys in a convenient and secure manner. It acts as a frontend to ssh-agent and ssh-add, but allows you to easily have one long running ssh-agent process per
> system, rather than the norm of one ssh-agent per login session.
>
> This dramatically reduces the number of times you need to enter your passphrase. With keychain, you only need to enter a passphrase once every time your local machine is rebooted. Keychain also
> makes it easy for remote cron jobs to securely "hook in" to a long running ssh-agent process, allowing your scripts to take advantage of key-based logins.
>
> Keychain also integrates with gpg-agent, so that GPG keys can be cached at the same time as SSH keys.

#### Installing & configuring Funtoo keychain

- Install keychain:

```bash
#!/usr/bin/env bash

sudo apt install keychain
```

- Configure `keychain` in `~/.bashrc` to automatically start up and prompt for private key passphrases.

According to the [keychain documentation](https://github.com/danielrobbins/keychain/blob/master/keychain.pod):

> The key files specified on the command-line will be searched for in the ~/.ssh/ directory, and keychain will expect to find the private key file with the same name, as well as a .pub public key.
> Keychain will also see if any GPG keys are specified, and if so, prompt for any passphrases to cache these keys into gpg-agent.

And

> Typically, private SSH key files are specified by filename only, without path, although it is possible to specify an absolute or relative path to the private key file as well. Private key files can
> be symlinks to the actual key as long as your system has the readlink command available. More advanced features are available for specifying keys as well -- see the --extended and --confallhosts
> options for more information.

- As such, add the following command to your shell profile (e.g. `~/.bashrc`) to start keychain:

```bash
#!/usr/bin/env bash

eval `keychain --eval <your-private-key-1-in-~/.ssh> </full/path/to/your/private-key-2> ...`

# For example:
# Suppose my ~/.ssh folder has the private key 'example' and that
# a key called 'external' is present in my opt directory.

eval `keychain --eval example /opt/external`
```

- If you start a new session or `source ~/.bashrc`, you'll be prompted to provide the passphrases for the private keys
  you specified. After you provide the passphrases the first time, you won't be prompted again until you either restart
  the system or explicitly clear the keys using `keychain --clear`

Typing in passphrases for every SSH key, even once, quickly gets old. The good news is that you can offload this entirely to a password manager.

### Option 2: Using Password Managers as SSH Agents

Many modern password managers now offer first-class SSH key management, including 1Password, Bitwarden, Keeper, NordPass, and Dashlane. Each supports storing SSH keys securely, and several (like
Bitwarden and 1Password) provide an integrated SSH agent that can load your keys automatically when you connect to a server.

Pick your poison, which in my case, is Bitwarden. I’ve been using it for years (this is not a sponsored post), it’s open source, transparent, and actively improving. What's not to love?

From a security perspective, SSH key passphrases exist so that even if someone gains access to your private key file, they still can’t use it without the passphrase. The problem is the friction:
entering these passphrases over and over, especially across multiple keys, gets real annoying really fast.

Password managers solve this elegantly.

You unlock your vault once at login using a long, strong master password (plus MFA). After that:

- Your SSH private keys are securely stored only inside the password manager, not scattered around your OS.
- The password manager’s built-in SSH agent loads and serves the keys as needed.
- You never have to type a passphrase again unless your vault locks.
- No key files sit on disk unencrypted or unmanaged.

It dramatically reduces friction while maintaining strong security — at least as strong as your trust in your password manager, which is already responsible for protecting your most sensitive
credentials. If you trust it with banking logins, AWS root keys, personal docs, and 2FA tokens, SSH keys are not a stretch.

Take a look at [Bitwardens SSH Agent Documentation](https://bitwarden.com/help/ssh-agent/#configure-Bitwarden-ssh-agent) for getting started and setting up Bitwarden as an SSH agent.

Note: This Bitwarden-based workflow works perfectly, even without defining any IdentityFile entries in your SSH configuration. As long as the Bitwarden SSH Agent is enabled and your vault is unlocked,
SSH will automatically query the agent and use the correct key. This is the approach I personally use today.

---

## Troubleshooting

### Fixing permissions for SSH

The following table can be used as a quick reference to fix any permission-related issues for SSH key files and
configuration:

| Folder / File            | Permissions                                    | Command                             |
|--------------------------|------------------------------------------------|-------------------------------------|
| `~/.ssh`                 | Read, write and execute for the user only.     | `chmod 700 ~/.ssh`                  |
| `~/.ssh/authorized_keys` | Read and write for the user only.              | `chmod 600 ~/.ssh/authorized_keys`  |
| `~/.ssh/config`          | Read and write for the user only.              | `chmod 600 ~/.ssh/config`           |
| Any private key          | Read and write for the user only.              | `chmod 600 ~/.ssh/<private-key>`    |
| Any public key           | Readable by all but writable only by the user. | `chmod 644 ~/.ssh/<public-key>.pub` |

Note: I also wrote a [small script that fixes SSH folder and file permissions](https://github.com/mtkhawaja/dotfiles/blob/osx/bin/.local/bin/scripts/utility/fix-ssh.sh).
Please audit it before using it. To run it directly via curl:

```shell
#!/usr/bin/env zsh

curl -s https://raw.githubusercontent.com/mtkhawaja/dotfiles/refs/heads/osx/bin/.local/bin/scripts/utility/fix-ssh.sh | zsh
```

---

## References / Additional Reading

1. "Funtoo Keychain", funtoo, [https://www.funtoo.org/Funtoo:Keychain](https://www.funtoo.org/Funtoo:Keychain)
2. "Funtoo Keychain Latest Documentation",
   funtoo, [https://github.com/funtoo/keychain/blob/master/keychain.txt](https://github.com/funtoo/keychain/blob/master/keychain.txt)
3. Github Repo, danielrobbins,  [https://github.com/danielrobbins/keychain](https://github.com/danielrobbins/keychain)
4. "How can I run ssh-add automatically without a password prompt", UNIX Stack
   Exchange, [https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt](https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt)
5. "How many kdf rounds for an ssh key", Crypto Stack
   Exchange, [https://crypto.stackexchange.com/questions/40311/how-many-kdf-rounds-for-an-ssh-key](https://crypto.stackexchange.com/questions/40311/how-many-kdf-rounds-for-an-ssh-key)
6. "Permissions on private key in ssh folder",
   superuser.com, [https://superuser.com/questions/215504/permissions-on-private-key-in-ssh-folder](https://superuser.com/questions/215504/permissions-on-private-key-in-ssh-folder)
7. "Safe Curves", safecurves.cr.yp.to, [https://safecurves.cr.yp.to/](https://safecurves.cr.yp.to/)
8. "ssh(1) — Linux manual
   page", [https://man7.org/linux/man-pages/man1/ssh.1.html](https://man7.org/linux/man-pages/man1/ssh.1.html)
9. "ssh-add(1) — Linux manual
   page", [https://man7.org/linux/man-pages/man1/ssh-add.1.html](https://man7.org/linux/man-pages/man1/ssh-add.1.html)
10. "ssh-agent(1): authentication agent - Linux man
    page", [https://man7.org/linux/man-pages/man1/ssh-agent.1.html](https://man7.org/linux/man-pages/man1/ssh-agent.1.html)
11. "ssh_config(5) — Linux manual
    page", [https://man7.org/linux/man-pages/man5/ssh_config.5.html](https://man7.org/linux/man-pages/man5/ssh_config.5.html)
12. "ssh-keygen(1) — Linux manual page", Linux Manual
    Page, [https://man7.org/linux/man-pages/man1/ssh-keygen.1.html](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html)
13. "SSH Essentials: Working with SSH Servers, Clients, and Keys.", DigitalOcean (Ellingwood,
    Justin.), [https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys).
14. “What Is SSH-Keygen & How to Use It to Generate a New Ssh Key?”, SSH Communications
    Security, [https://www.ssh.com/academy/ssh/keygen](https://www.ssh.com/academy/ssh/keygen)
15. “Working with SSH Key Passphrases.”
    GitHub, [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases).
16. SSH Agent, Bitwarden, [https://Bitwarden.com/help/ssh-agent/](https://Bitwarden.com/help/ssh-agent/)\
17. BW ssh-agent and .ssh/config, Bitwarden Community Forums, [https://community.Bitwarden.com/t/bw-ssh-agent-and-ssh-config/82363](https://community.Bitwarden.com/t/bw-ssh-agent-and-ssh-config/82363)