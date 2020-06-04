<p align="center"> 
<img src="static/ssh-key.png" alt="SSH-Key" >
</p>

How to Lock Down Your SSH Server
--------------------------------
SSH, which stands for Secure Shell, isn't very secure by default, option for basic password authentication with no other limits. If you really want to lock down your server, you'll neet to do more configuration.

## Table of Contents
* [Don't Allow Password Logins - Use SSH Keys](#dont-allow-password-logins---use-ssh-keys).
* [Generate SSH Keys](#generate-ssh-keys).
* [Disable SSH Password Login](#disable-ssh-password-login).
* [Don't Allow Root Login](#dont-allow-root-login).
* [Set Up two-factor authentication](#set-up-two-factor-authentication).
* [General Issues](#general-issues).


## Don't Allow Password Logins - Use SSH Keys

The First thing to do is get rid of password authentication completely and switch to using SSH keys. SSh keys are a form of public key encryptionl you have a public key that acts like username, and a private key that acts like password (except this password is 2048 characters long). Your private key is stored on your dis, but is encrypted with a passphrase and ssh-agent, when you go to SSH into a server, instead of asking for password, the ssh-agent connects to the server using ssh keys.

> Even if you're already using SSH keys, you'll still want to ensure that password logins are turned off, as the two aren't mutually exclusive.

## Generate SSH Keys

You can generate a new SSH key using the `ssh-keygen` utility, installed by default UNIX systems, also you may pass the file name `ubuntu_srvr`.

```sh
ssh-keygen -f ubuntu_srvr
...
[ENTER]
[ENTER]
[ENTER]
```
This will ask you for a passphrase to encrypt the local key file with. It is not used for authentication with the server, but should still be kept secret.

`ssh-keygen` will save your private key in `~/.ssh/ubuntu_srvr`, and will alose save you public key in `~/.ssh/ubuntu_srvr.pub`. The private key stays on your hard drive, but the public key must be uploaded to the server so that the server can verify your identity, and verify that you have permission to access that server.

The server keeps a list of authorized users, usually stored in `~/.ssh/authorized_keys`, you can add your key file manually to this file, or you can use the `ssh-copy-id` utility:

```sh
ssh-copy-id -i ~/.ssh/ubuntu_srvr.pub user@ip_address
```
Replace user@host with yout own username and server hostname, you'll be asked to sign in with your old password once more, after which you shouldn't be prompted for it again, then you can disable password sign-in.

## Disable SSH Password Login

Now that you can access the server with your keys, you can turn off password authentication altogether, make sure that key-based authentication is working, or you'll be locked out of server.

On the server, open up `/etc/ssh/sshd_config` in you terminal editor, and search for the line that starts with `PasswordAuthentication`, uncomment it and change "yes" to "no"

```sh
PasswordAuthentication no 
```

Then restart `sshd` with:

```sh
systemctl restart sshd
```
Now you shoud be forecd to reconnect, and if your key file is wrong, you won't be prompted for a password.

You can also force **public key-based** authentication, which will block all other authentication methods by add the following lines to `/etc/ssh/sshd_config`:
```
AuthenticationMethods publickey
PubkeyAuthentication yes
```
then restart `sshd`.

## Don't Allow Root Login

Instead, make a new user and give that user sudo privilege. this effectively is the same thing but has one major difference: potential attackers will need to know your user account name to even begin attacking your server, because it won't be as simple as root@host.

Aside from security, it's generally good Unix policy to not be logged in as `root` all the time, because `root` doesn't create logs and doesn't prompt when accessing protected resources.

Create a new user on your SSH server:

```sh
adduser myusername
```
Set a password for that user
```sh
passwd myusername
```
You won't be logging in with this password because you'll still be using SSH Keys, but it is required. Ideally make this different from your root password. Then add this user to `/etc/sudoers` to give admin permissions:

```sh
echo "myusername ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```
Switch to that user with `su myusername`, and verify that you can switch back to the root user with sudo su (which doesn't require root's password), if you can, you have sudo access.

Now you'll want to block root login, in `/etc/ssh/sshd_config`, you will set:
```
PermitRootLogin no
```
then restart `sshd` and server should block all requests to log on as `root`.

## Set Up two-factor authentication
This is certainly overkill, but if you're paranoid about someone nabbing your private SSH keys, you caon configure SSH server to use 2FA.

The easiest way to do this is to use [Google Authenticator](https://hackertarget.com/ssh-two-factor-google-authenticator/) with an Android / iOS device, though SSH supports many two factor methods, with Authenticator App, you'll be given a QR code which you can scan from the Authenticator mobile App to link your phone to the server, and you'll also be given a few backup codes for recover in the event your phone is lost, do not store these codes on your main machine, otherwise it's not really two factor.

## General Issues

* ssh-copy-id not working Permission denied (publickey).
Edit ssh config:
```
sudo nano /etc/ssh/sshd_config
```
Change this line:
```ini
PasswordAuthentication no
```
to
```ini
PasswordAuthentication yes
```
Restart ssh daemon:
```sh
sudo systemctl restart sshd
```
Do ssh-copy-id:
```sh
ssh-copy-id someuser@<static-ip>
```
> Note: do not forget change to `PasswordAuthentication no` and restart ssh again to prevent user/pass login.

See also [SSH Agent Forwarding](SSH-AGENT-FORWARDING.md)