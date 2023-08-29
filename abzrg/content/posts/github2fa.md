---
title: "How I Enabled Two-Factor Authentication for GitHub Using Command Line Tools Only"
date: 2023-08-29T05:29:59+03:30
draft: false
tags: ['cli', 'gpg']
---

Today, as I was visiting one of my repositories on GitHub, I encountered a message that prompted me to enable two-factor authentication (2FA). It pointed me to a blog that right at the beginning says:

> Last year, we announced our commitment to require all developers who contribute code on GitHub.com to enable two-factor authentication (2FA) by the end of 2023. [src](https://github.blog/2023-03-09-raising-the-bar-for-software-security-github-2fa-begins-march-13/)

While many users choose graphical user interfaces to enable this security feature, I, for reasons I won't bore you with, opted for a different approach: using command line tools such as `gpg`, `pass`, and `zbar`.
In this guide, we will explore the process of enabling 2FA for GitHub using these command line tools.


## Setting Up a GPG Key before Initializing `pass`

Before diving into the process of enabling 2FA for GitHub using command line tools, it's essential to set up a GPG (GNU Privacy Guard) key.
GPG keys are crucial for securely encrypting and decrypting sensitive data, including passwords stored in the `pass` password manager.
Here's how to set up a GPG key:

1. **Install GPG**: If you don't have GPG installed, use your package manager to install it.

   ```bash
   # debian
   sudo apt-get install gnupg

   # macos
   brew install gnupg
   ```

2. **Generate GPG Key Pair**: Generate a GPG key pair using the `gpg` command.
Follow the prompts to set up your key's parameters, including your name and email address.

   ```bash
   gpg --full-gen-key
   # I chose RSA, 4096, 0 (key does not expire),...
   # You may want to insert a passphrase for extra protection
   ```


## Enabling 2FA for GitHub Using Command Line Tools

Now that you have your GPG key set up, you're ready to enable 2FA for GitHub.

1. **Install Required Tools**: Use your package manager to install `pass`, `pass-otp`[^1], and `zbar-tools`.

   ```bash
   # debian
   sudo apt-get install pass pass-otp zbar-tools

   # macos
   brew install pass pass-otp zbar
   ```

[^1]: This extension allows you to generate One-Time Passwords (OTP) for accounts stored in your pass password store.
  OTPs are commonly used for two-factor authentication (2FA), where you need a temporary code in addition to your regular password to log in securely

1. **Initialize `pass`**: Create a password store associated with your GitHub email.
  Note that this is the exact same email address you used to generate the GPG key pair just now.

   ```bash
   pass init your.github@email.com
   ```

1. **Enable 2FA on GitHub**: Go to your profile Settings on GitHub, and In the "Access" section of the sidebar, click  Password and authentication. Under "Setup authenticator app", download the SVG image (I named it as `qr_code.svg`).

1. **Scan QR Code and Extract OTP Secret**: Use `zbarimg` to extract the OTP secret.

   ```bash
   zbarimg -q path/to/qr_code.svg
   ```

1. **Add OTP Secret**: Associate an OTP secret with an entry in your pass password store with the following command.
  You will be prompted to enter the OTP secret.
  Only enter the portion of the OTP secret from 'otpauth://' to the end, and then press Enter

   ```bash
   pass otp add github2fa
   ```

1. **Generate OTP**: Retrieve the OTP secret using `pass-otp`. When you run this command, it will output a temporary OTP that's valid for a short period of time.

   ```bash
   pass otp github2fa
   ```

1. **Complete Setup**: Enter the extracted OTP into the GitHub prompt to finalize the 2FA setup.


## References

- [GitHub Guide](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication#configuring-two-factor-authentication-using-a-totp-mobile-app)
- [(Luke Smith) PASS: a Password Manager & Two Factor Authentication (OTP) with no Cell Phone](https://www.youtube.com/watch?v=sVkURNfxPd4&pp=ygUPbHVrZSBzbWl0aCBwYXNz)
- [(Ömür Özkir) Using 2FA/OTP with pass](https://medium.com/@oem_83498/using-2fa-otp-with-pass-454a8f210f71)
