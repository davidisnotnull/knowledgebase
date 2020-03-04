# Run a .NET Core app with nginx in Ubuntu
I've been doing this for a while on Ubuntu 18.04, developing on Windows 10 but publishing to Ubuntu as the host for a .NET core web app.

I've written this for someone with a Windows head, who doesn't have that much experience using Linux. Hopefully it'll be step by step enough to follow, provided you aren't frightened of using terminal commands.

## First steps

### SSH into Ubuntu
I do this using the Git Bash program that comes with Git for Windows. Seeing as I already had it installed, and don't have to set up Putty or anything like that as well.

If you don't know how to ssh into a linux box using the terminal, here's a quick sneak peek

```
ssh username@serverdomain.com
```
You can use an IP address instead of the domain name of the server, for example:
```
root@mycloudserver.com

or

root@227.54.13.128
```

You'll then be asked to accept the key and to provide the password for the user you're logging in as.

### Make sure nano is installed
If you're on Ubuntu, I think nano comes preinstalled. But just in case it is not, you'll want to run `apt-get install nano` and agree to install it and it's dependencies.

Nano is a terminal text editor that we'll be using later on to edit some files.

NB: Some people like to use vim. I do not like to use vim. End of discussion.

### Stop using root to SSH into your Ubuntu box
You don't want to be fucking about making ssh connections with your root user. In fact, you don't want the root user to be able to make ssh connections at all. Trust me.

Let's start out by creating a new user that we're going to use to ssh in going forwards.

```
adduser david
usermod -aG sudo david`
```
This command creates a new user, and then adds them to the sudo group. You'll need to be in the sudo group as it elevates your rights to do admin stuff.

Next, change to your new user by entering `su david` or whatever your new username is.

#### A bit about the sudo group
Being in the sudo group means you need to prefix your commands with the word `sudo` before running anything at administrator level. As `root` user, you don't need the sudo prefix.

e.g. to update your packages in Ubuntu, as root user, you'd enter
`apt-get update` but as my new user david, I'd have to put `sudo apt-get update` in order to get that to work. Otherwise I'll get an error message back saying I don't have permissions to run an update.

You'll also have to enter your password the first time that you use sudo. This occurs each time you log in, or if it times out whilst you're still in your current session.

### Disable root SSH access
Now you've got your new user, you can disable your root access via SSH. For the love of god, right down that fucking password for your other user and keep it where you won't lose it. Not a post-it note on your monitor.

The easiest way to block root SSH access is to modify the sshd_config file. You can open the file in nano using the following commands

```
sudo nano /etc/ssh/sshd_config
```
Find the line in this file that says `PermitRootLogin` and set it's value to `no`. You might have to uncomment the line first. Save the file and then back to the command line.

We need to restart the sshd service in order for these changes to take affect. You can do this with the command `sudo service sshd restart`.

Going forwards, you won't be able to access ssh, scp, or sftp using your root account. Which is good.

## Setup .NET Core
We're going to do this by adding the appropriate repository into the package manager.
```
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
```
Once the repository has been added, there's a single dependency that must be installed. Do this with the following commands:
```
sudo add-apt-repository universe
sudo apt-get install apt-transport-https
```
Then, we install .NET Core
```
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```
Obviously, you can define the dotnet sdk to install by setting the version number at the end.

## Setup nginx
This one is simple. You can use the following commands
```
sudo apt-get install nginx
sudo service nginx start
```
