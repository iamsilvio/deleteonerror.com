---
date: 2021-05-16T23:30:00+02:00
draft: false

title: Sign your Git commit with a PGP key
subtitle: Use your PGP keys from your Yubikey

tags: ["Integrity", "Authenticity", "GPG", "YubiKey", "PGP", "Git", "Windows", "MacOS"]
categories: [Integrity, ItSec, HowTo]

twitter_creator_id: 94538574

---

Authenticity is a psychological and philosophical concept.
According to existential philosophy, the lack of authenticity is a sign of lack of trustworthiness.  
  
We should not only be authentic in what we do, but also in what we produce.  
  
In early 2021, a software was reported to have malicious code introduced through changes made by well known developers. In this particular case, the changes were noticed in time, but that was rather lucky. If an attacker picks a better time to make such changes and catches the right reviewer, it can rapidly lead to serious vulnerabilities that are difficult to detect.

> The commits were made to the php-src repo under the account names of two well-known PHP developers, Rasmus Lerdorf and Nikita Popov. [arstechnica](https://web.archive.org/web/20210424213823/https://arstechnica.com/gadgets/2021/03/hackers-backdoor-php-source-code-after-breaching-internal-git-server/)
  
There is no way to ensure that whoever is contributing the code is who they say they are, right? Of course there is a way, everything digital can be signed, after that, it is of course the task of the build system to validate them, but that is another challenge.

So signing your public work helps to trust you and your work.

How to generate PGP keys and what a YubiKey is will be part of another blog post.

The topic of this post will be how to use a PGP key stored on a Yubikey to sign your commits on Windows and macOS.

## On MacOS 11.*

Install gpg `brew install gnupg`, you don't need to specify a version. The `gnupg` package references always the latest version of "GNU Pretty Good Privacy (PGP) package".  

I have some gnupg config files at my [dotfiles repository](https://github.com/iamsilvio/dotfiles/tree/main/tux/.gnupg) which should be a good starting point. Copy them into `~\.gnupg` and you should be fine. Since Gnupg 2* I had to add a `scdaemon.conf` to get my Yubikey to work, but that's part of my dotfiles and I will update the config if needed.

To check if your Yubikey is recognized run

``` bash
gpg --card-status
# Reader ...........: Yubico YubiKey OTP FIDO CCID 0
# Application ID ...: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# Application type .: OpenPGP
# Version ..........: 3.4
# Manufacturer .....: Yubico
# Serial number ....: XXXXXXXX
# Name of cardholder: 
# Language prefs ...: en
# Salutation .......:
# URL of public key : [not set]
# Login data .......: 
# Signature PIN ....: not forced
# Key attributes ...: nistp384 nistp384 nistp384
# Max. PIN lengths .: 127 127 127
# PIN retry counter : 3 0 3
# Signature counter : 2
# KDF setting ......: off
# Signature key ....: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
# Encryption key....: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
# Authentication key: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
```

Import your public key

``` bash
gpg --import PATH_TO_YOUR_PUBLIC_KEY

# gpg: key FFFFFFFFFFFFFFFF: public key "neo <neo@matrix.com>" imported
# gpg: DBG: Using CREATE_BREAKAWAY_FROM_JOB flag
# gpg: Total number processed: 1
# gpg:               imported: 1

```

Set the trust of this key so GPG will not ask if you are sure to trust this key

``` bash
gpg --edit-key FFFFFFFFFFFFFFFF

#pub  rsa3072/FFFFFFFFFFFFFFFF
#     created: 2021-05-16  expires: 2021-05-16  usage: SC
#     trust: unknown       validity: unknown
#sub  rsa3072/AAAAAAAAAAAAAAAA
#     created: 2021-05-16  expires: 2021-05-16  usage: E
#[ unknown] (1). neo <neo@matrix.com>

gpg> trust

# Please decide how far you trust this user to correctly verify other users' keys
# (by looking at passports, checking fingerprints from different sources, etc.)

#  1 = I don't know or won't say
#  2 = I do NOT trust
#  3 = I trust marginally
#  4 = I trust fully
#  5 = I trust ultimately
#  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
gpg> quit
```

Now we just add three lines to our shell config, the default shell on MacOs 11 is zshell, so we add the following lines to `~\.zshrc`.

``` bash
# set the gnupg path of the brew installed version first to ensure the right one is used
export PATH="/usr/local/opt/gnupg/bin:$PATH"
# Ensure that GPG uses the same TTY as your Terminal session
export GPG_TTY=$TTY
# on shells other than zsh you may have to use 
# export GPG_TTY=$(tty)

gpgconf --launch gpg-agent
```

## On Windows 10.*

On Windows I use `scoop` as my package manager of choice, so to install `pgp4win` you just have to `scoop install gpg4win`. If you don't use a package manager you can download [gpg4win](https://www.gpg4win.org/) at this link.

I have some gnupg config files at my [dotfiles repository](https://github.com/iamsilvio/dotfiles/tree/main/tux/.gnupg) which should be a good staring point. Copy them into `%userprofile%\AppData\Roaming\gnupg` and you should be fine, except on Windows I had to add the following two lines to the `scdaemon.conf`

``` text
reader-port "Yubico YubiKey"
card-timeout 2
```

To check if your Yubikey is recognized run

``` bash
gpg --card-status`
# Reader ...........: Yubico YubiKey OTP FIDO CCID 0
# Application ID ...: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# Application type .: OpenPGP
# Version ..........: 3.4
# Manufacturer .....: Yubico
# Serial number ....: XXXXXXXX
# Name of cardholder: 
# Language prefs ...: en
# Salutation .......:
# URL of public key : [not set]
# Login data .......: 
# Signature PIN ....: not forced
# Key attributes ...: nistp384 nistp384 nistp384
# Max. PIN lengths .: 127 127 127
# PIN retry counter : 3 0 3
# Signature counter : 2
# KDF setting ......: off
# Signature key ....: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
# Encryption key....: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
# Authentication key: XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX
#       created ....: 2021-01-01 01:00:00
```

Import your public key

``` bash
gpg --import PATH_TO_YOUR_PUBLIC_KEY

# gpg: key FFFFFFFFFFFFFFFF: public key "neo <neo@matrix.com>" imported
# gpg: DBG: Using CREATE_BREAKAWAY_FROM_JOB flag
# gpg: Total number processed: 1
# gpg:               imported: 1

```

Set the trust of this key so GPG will not ask if you are sure to trust this key

``` bash
gpg --edit-key FFFFFFFFFFFFFFFF

#pub  rsa3072/FFFFFFFFFFFFFFFF
#     created: 2021-05-16  expires: 2021-05-16  usage: SC
#     trust: unknown       validity: unknown
#sub  rsa3072/AAAAAAAAAAAAAAAA
#     created: 2021-05-16  expires: 2021-05-16  usage: E
#[ unknown] (1). neo <neo@matrix.com>

gpg> trust

# Please decide how far you trust this user to correctly verify other users' keys
# (by looking at passports, checking fingerprints from different sources, etc.)

#  1 = I don't know or won't say
#  2 = I do NOT trust
#  3 = I trust marginally
#  4 = I trust fully
#  5 = I trust ultimately
#  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
gpg> quit
```

## Git Config

### MacOs specific Git config

Tell git to use gpg by  
`git config --global gpg.program gpg`

### Windows specific Git config

Git brings its own gpg version, so we have to point git to the right one  
`git config --global gpg.program %userprofile%\scoop\apps\gpg4win\current\GnuPG\bin\gpg.exe`

### Universal config

`git config --global commit.gpgsign true`
`git config --global user.signingkey YOUR__KEY__ID`

and a special one to make `git log` a litle bit more awesome
`git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset %G?' --abbrev-commit"` if you use `git lg` the output will be a compressed log and the Character at the end of the line will show the state of the signature.

- `G` for a good (valid) signature,
- `B` for a bad signature,
- `U` for a good signature with unknown validity,
- `X` for a good signature that has expired,
- `Y` for a good signature made by an expired key,
- `R` for a good signature made by a revoked key,
- `E` if the signature cannot be checked (e.g. missing key)
- and `N` for no signature

### Github

Navigate  to [settings >keys](https://github.com/settings/keys) and upload your public pgp key.
A niche addition, at the bottom of the page you can set `Flag unsigned commits as unverified`
