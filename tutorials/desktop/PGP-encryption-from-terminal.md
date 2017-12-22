---
id: PGP-encryption-from-terminal
summary: This tutorial will cover installing OpenPGP, creating your keys, file encryption, decryption and signing from the command line.
categories: desktop
tags: PGP, encryption, tutorial, terminal, keys
difficulty: 2
status: draft
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues
published:
author: Nikita Belenkov <nikitov603@gmail.com>

---

# Setting up PGP encryption from the terminal

## Overview
Duration: 2:00

This tutorial will guide you through PGP encryption with GnuPG, how to set it up from the terminal and how to then use it.

GnuPG is a key-based encryption, it is an implementation of the OpenPGP standard.

The principle behind public key encryption is simple. You will be given a public key and a private key. The private key, as indicated, should remain private as to keep the entire idea of encryption secure.

A person who holds your public key and wishes to send you an encrypted message would encrypt the message with your public key. They can not decrypt their own message after they encrypt it. Only you, who holds the private key can decrypt the message.

### What you'll learn

- What GnuPG is and how public key encryption works 
- How to create your own PGP key
- How to encrypt and decrypt files with your key
- How to sign documents with your key

### What you'll need

- Ubuntu Desktop 16.04 or above
- Some basic command-line knowledge
- GnuPG version 1.4.10 or newer (should be already installed on your system)

If you have everything ready let's begin!

## Creating your keys
Duration: 6:00

First, you need to enter this command in terminal to start the key generation process:

```bash
gpg --gen-key
```
You will see the following:

```python
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 
```
You should select **(1)**, as these keys can be used for decryption and encryption.

```bash
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 
```

2048 is the default keysize and it should be enough. You can select default by just pressing enter.

```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
```
Choose the default option, which is the key that doesn't expire.

```bash
Is this correct? (y/N) 
```
Hit **y** to confirm and continue. 

```bash
You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: 
```
The process will prompt you to enter some information about yourself you will need to enter the following: 

* Real name - Do not use aliases or handles, since these disguise or obfuscate your identity. Remember this process is about authenticating you as a real individual.
* Email - Use the current email, so it is easier for others to find your public key.
* Comment - Use the comment field to include aliases or other information. (Some people use different keys for different purposes and identify each key with a comment, such as "Open Source Projects.") 


```bash
You selected this USER-ID:
    "Heinrich Heine (Poet) <heinrichh@duesseldorf.de>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? 
```
Type **O** to continue.

```bash
You need a Passphrase to protect your secret key.

Enter passphrase: 
```
Enter your passphrase twice. As always, make a strong password which would be difficult to crack. 

negative
: **Warning!**
If you forget your passphrase, the key cannot be used and any data encrypted using that key will be lost.

```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 24 more bytes)
```
You will be asked to use the computer normally in order for randomization to take place. This will make the key more random.

```bash
gpg: key 516EEE6E marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   3  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 3u
pub   2048R/516EEE6E 2017-12-22
      Key fingerprint = 7323 8EF9 B6D5 5CD1 CEAC  4A31 7576 F4C0 516E EE6E
uid                  Heinrich Heine (Poet) <heinrichh@duesseldorftest.de>
sub   2048R/1905AF07 2017-12-22
```
And now we have successfully created a key pair. Congrats!

In the created key the key-id is **516EEE6E**. Note down **your** key-id, which we will need later.

## Uploading the key to Ubuntu keyserver
Duration: 1:00
### Why should you use a keyserver?

Key servers are used to distribute your public key to other key servers and so other users can easily look your name (or email up) in the database and find your public key to send encrypted messages to you. 

This eliminates the process of physically or insecurely giving your friend your public key, and allows others to be able to find you in an online database.

### Uploading your key to the keyserver


```bash
gpg --send-keys --keyserver keyserver.ubuntu.com your-key-id
```

Once you have uploaded it to one keyserver, it will propagate to the other keyservers. Eventually, most of the keyservers will have a copy of your public key. 

## Encrypting and decrypting files
Duration: 2:00

### Why should you encrypt files like this?
GnuPG also implements file encryption which is very secure, portable and can be used for example for encrypting files or messages. 

### Encrypting a file

Use this command to encrypt a file for your friend: Change the friends-key-id to your friend's Public Key

```bash
gpg -o encrypted_file.gpg --encrypt -r friends-key-id original.file
```


### Decrypting a file

Use this command if you have received a file, which has been encrypted with your Public key:

```bash
gpg --decrypt filename.gpg
```


## Signing and checking signatures
Duration: 1:00

### Why you should sign a document?

Clearsigning is very similar to adding your signature to the bottom of a letter or important paper. It signifies that it actually came from you. Bly clearsigning, it generates a SHA1 hash of the entire file's contents and add's the SHA1 sum to the bottom of the signature. If the file has been tampered with, the signature verification will fail, which can be used to spot a forgery.



### Signing a document

To sign a document/file run this in terminal:

```bash
gpg --clearsign filename.txt
```

## That's it!

Well done you have successfully created your key and then learned how to use it!


### Next steps

* You can read about more uses for PGP at [Ubuntu Documentation](https://help.ubuntu.com/community/GnuPrivacyGuardHowto)

### Further readings

* [The GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual.html)


