---
title: "Mutt Email vNext"
subtitle: "vNext Email from the past"
date: 2020-10-14T18:17:50+02:00
draft: true
tags: ["office365", "GpG", "security"]
categories: [itstuff]
---

Email is an ancient technology, but like telefax, it will not disappear for a long time yet.
But since I am still not a friend of someone evaluating and sorting my emails for me, I have to find a solution.
Here is a post about the configuration of a minimalistic email client.

<!--more-->

Assume we have a user named Dexter DeShawn with one GMail and one Office 365 account.

dexter@example.com
dexter@gmail.com

## Install Mutt (optional GPG)

`apt install mutt gpg`

## Encrypt your Passwords

create a new file `~/.pwd/.mail` for example. The content of the file should be as follows

``` config
o365: password
gmail: password
```

If you do not have a GPG key yet create one with the command `gpg --full-generate-key`  
You want RSA encryption and a key with a size of 4096.

Run `gpg --output ~/.pwd/.mail.gpg --encrypt --recipient dexter ~/.pwd/.mail` and delete the unencrypted pwd file `rm ~/.pwd/.mail`

## Configuration of your new Mail client

Create a mutt config file at `~/.muttrc`  

The content of the configuration file should be:

``` config

# Process the password
#
# All user-defined variable must start with "my_"

set my_tmpsecret=`gpg -o ~/.pwd/.tmp -d ~/.pwd/.mail.gpg`
set my_dpass=`awk '/o365:/ {print $2}' ~/.pwd/.tmp`
set my_gpass=`awk '/gmail:/ {print $2}' ~/.pwd/.tmp`
set my_del=`rm -f ~/.pwd/.tmp`


# Folder hooks
folder-hook 'account.com.live.dexter' 'source ~/.mutt/account.com.live.dexter'
folder-hook 'account.com.gmail.dexter' 'source ~/.mutt/account.com.gmail.dexter'

# Default account
source ~/.mutt/account.com.live.dexter

# Macros for switching accounts
macro index <F7> '<sync-mailbox><enter-command>source ~/.mutt/account.com.live.dexter<enter><change-folder>!<enter>'
macro index <F8> '<sync-mailbox><enter-command>source ~/.mutt/account.com.gmail.dexter<enter><change-folder>!<enter>'

# Set default text editor
set editor = "$EDITOR"

set query_command= "abook --mutt-query '%s'"
macro index,pager A "<pipe-message>abook --add-email-quiet<return>" "add the sender address to abook"


#-------- Basic Config {{{
#------------------------------------------------------
set imap_check_subscribed
set mail_check = 120
set timeout = 300
set imap_keepalive = 300
set record="~/mail/sent"
set move = no
set include
set sort = 'threads'
set sort_aux = 'reverse-last-date-received'
set auto_tag = yes
ignore "Authentication-Results:"
ignore "DomainKey-Signature:"
ignore "DKIM-Signature:"
hdr_order Date From To Cc
alternative_order text/plain text/html *
auto_view text/html
bind editor <Tab> complete-query
bind editor ^T complete
bind editor <space> noop
# }}}
#-------- Color Theme {{{
#------------------------------------------------------

color hdrdefault cyan default
color attachment yellow default

color header brightyellow default "From: "
color header brightyellow default "Subject: "
color header brightyellow default "Date: "

color quoted green default
color quoted1 cyan default
color quoted2 green default
color quoted3 cyan default

color error     red             default         # error messages
color message   white           default         # message  informational messages
color indicator white           red             # indicator for the "current message"
color status    white           blue            # status lines in the folder index sed for the mini-help line
color tree      red             default         # the "tree" display of threads within the folder index
color search    white           blue            # search matches found with search within the internal pager
color markers   red             default         # The markers indicate a wrapped line hen showing messages with looong lines

color index     yellow default  '~O'
color index     yellow default  '~N'
color index     brightred       default '~F'    # Flagged Messages are important!
color index     blue default    '~D'            # Deleted Mails - use dark color as these are already "dealt with"
# }}}

```

## Office 365 Account config

Add a account configuration file for your Office 365 account`~/.mutt/account.com.live.dexter`

``` config

unmailboxes *
set imap_user = "dexter@example.com"
set imap_pass = $my_dpass
set smtp_url = "smtp://dexter@example.com@smtp.office365.com:587/"
set smtp_pass = $my_dpass
set from = "dexter@example.com"
set realname = "Dexter DeShawn"
set folder = "imaps://outlook.office365.com:993"
set spoolfile = "+INBOX"
set postponed = "+[Live]/Drafts"
set header_cache = ~/.mutt/com.live.dexter/cache/headers
set message_cachedir = ~/.mutt/com.live.dexter/cache/bodies
set certificate_file = ~/.mutt/com.live.dexter/certificates
set ssl_force_tls = yes
set status_format = "$from -%r-Mutt: %f [Msgs:%?M?%M/?%m%?n? New:%n?%?o? Old:%o?%?d? Del:%d?%?F? Flag:%F?%?t? Tag:%t?%?p? Post:%p?%?b? Inc:%b?%?l? %l?]---(%s/%S)-%>-(%P)---"

```

## Gmail Account config

Add a config file for your Gmail account: `~/.mutt/account.com.gmail.dexter`

``` config

unmailboxes *
set imap_user = "dexter@gmail.com"
set imap_pass = $my_gpass
set smtp_url = "smtp://dexter@smtp.gmail.com:587/"
set smtp_pass = $my_gpass
set from = "dexter@gmail.com"
set realname = "Dexter DeShawn"
set folder = "imaps://imap.gmail.com:993"
set spoolfile = "+INBOX"
set postponed = "+[Gmail]/Drafts"
set header_cache = ~/.mutt/com.gmail.dexter/cache/headers
set message_cachedir = ~/.mutt/com.gmail.dexter/cache/bodies
set certificate_file = ~/.mutt/com.gmail.dexter/certificates
set ssl_starttls = yes
set ssl_force_tls = yes
set status_format = "$from -%r-Mutt: %f [Msgs:%?M?%M/?%m%?n? New:%n?%?o? Old:%o?%?d? Del:%d?%?F? Flag:%F?%?t? Tag:%t?%?p? Post:%p?%?b? Inc:%b?%?l? %l?]---(%s/%S)-%>-(%P)---"

```
