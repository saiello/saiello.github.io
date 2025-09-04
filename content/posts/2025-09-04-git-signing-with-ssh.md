+++
title = 'Git Signing With SSH'
date = 2025-09-04T15:00:00+02:00
draft = false
tags = ['git', 'ssh', 'til', 'tldr']
+++

Hey there, developers!

This is a short memo to start signing Git commit with Ssh keys. 

**Q: Why SSH and not GPG?** 

Because is likely you are already using SSH keys to authenticate on Github, and you are more familiar with them. 


### Configuring Signing

Configure git to use SSH format and which key to sign commits (supposing you already have one)

```
export SSH_SIGN_KEY_FILE=~/.ssh/id_ed25519.pub
git config --global user.signingkey ${SSH_SIGN_KEY_FILE}
git config --global gpg.format ssh
```

Create a file `allowed_signers` with your email and instruct git to use it

```
echo "$(git config user.email) $(cat ${SSH_SIGN_KEY_FILE})" > ~/.ssh/allowed_signers 
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

### Sign Commits

Use the `-S` option to sign your commits

```
git commit -S -m "feat(foo): add important instruction"
```

or alternatively (recommended) is configuring git to always sign your commits

```
git config --global commit.gpgsign true
```


### Verify Signature locally 

To verify that a signature has been applied to your last commit use the `--show-signature` options

```
git log -1 --show-signature
```

The output should looks like:

```
Good "git" signature for john.doe@email.com with ED25519 key SHA256:vi6**********************************
Author: John Doe <john.doe@email.com>
Date:   Thu Sep 4 15:00:58 2025 +0200

    feat(foo): add important instruction
```

### Verify Signature on Github

To allow Github verifying your commits signatures you have to add you public key as `Signing Key` right below the Authentication Key



## Resouces

- [Github SSH Commit Verification](https://github.blog/changelog/2022-08-23-ssh-commit-verification-now-supported/)
- [Github About Commit Signature Verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification#about-commit-signature-verification)
- [Gitlab Sign commits with SSH keys](https://docs.gitlab.com/user/project/repository/signed_commits/ssh/)