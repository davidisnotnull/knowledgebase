# Run a .NET Core app with nginx on Ubuntu
I've been doing this for a while on Ubuntu 18.04, developing on Windows 10 but publishing to Ubuntu as the host for a .NET core web app.

I've written this for someone with a Windows head, who doesn't have that much experience using Linux. Hopefully it'll be step by step enough to follow, provided you aren't frightened of using terminal commands.

## Getting your environment ready

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

### Install the ASP.NET Core runtime
Run the following commands to get the ASP.NET Core runtime installed in Ubuntu
```
sudo apt-get install aspnetcore-runtime-3.1
```

### Install the .NET Core runtime
Run the following commands to get the .NET Core runtime installed in Ubuntu
```
sudo apt-get install dotnet-runtime-3.1
```

Most guides I've seen then go on to say you can use the dotnetcore CLI to create a new app and run it up on a service and what have you. I found that advice impractical. I'm not interested in starting a dotnetcore app on a hosting server. I want to deploy one that I've written and packaged up in VS2019 on my local Windows 10 machine.

## Configure and deploy your application to the server
For a quick deployment, you can create a publish profile for your solution that outputs to the file system, and then SFTP this to a suitable location on your server. If you're familiar with running sites on Linux, then you'll know that we'd normally put web applications into the `var/www` folder. Don't worry about this. It's actually easier to put them in the `home` folder of the user that you're using to ssh into the server with.

For example, I would create a folder called `apps` in the `home` folder for my user, and SFTP my deployed application from my local file system into this folder. If you're planning on hosting multiple applications, create a folder for each one within the `apps` folder you've just created.

### Types of deployment
When you're creating your publish profile, you'll have the option to deploy the application as self-contained, or framework-dependent.
* **Self-contained** means that you don't need to install any of the dotnetcore runtimes on your server. The deployment includes everything that you'll need to run the application in complete isolation. This is useful if you want to run apps that use different dotnetcore versions, or if you're unfamiliar with configuring dotnetcore on a linux box. However, you'll notice that this greatly increases the size of the applications on the server, and you'll have to maintain each app's runtime version independently.
* **Framework-dependent** means that the app will need to have the required runtime installed on your server (as we did previously) and ensure that the correct version is installed for your app. I prefer this, as it greatly reduces the size of your deployment package, and means that you only have to maintain the runtime version on the server rather than individually for each of the apps.

### Permissions
Linux is notorious for it's granular access permissions on files and folder structures. As a rule of thumb, I would always ensure that the www-data group is the one that's in charge of running any sort of web application. It's a bit like your IIS_IUSR group in Windows.

Now that you've got your app deployed to the server, navigate back to our `apps` folder in the terminal and use the following command to change the access permissions of the file structure.

```
sudo chown -R www-data:www-data appfoldername
```
Where `appfoldername` is the name of the folder that you deployed your application to. This command gives complete ownership of the files to the www-data user.

### Configure a service to run your app
Once your app is on the server, you'll need to create and register a service to handle it. Firstly, we're going to create a service definition file:
```
sudo nano /etc/systemd/system/kestrel-myapp.service
```
You'll then be taken into the nano editor so that you can edit the contents of the file. I would use the following as a default, but you can extend it where you need to:
```
[Unit]
Description=My First DotNetCore App in Ubuntu

[Service]
WorkingDirectory=/home/david/apps/myfirstapp
ExecStart=/usr/bin/dotnet /home/david/apps/myfirstapp/MyFirstApp.dll
Restart=always
# Restart the service after 10 secs if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-MyFirstApp
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```
Few things to note in the above:
* **Description** - this is an outline of what your app is, for your reference
* **WorkingDirectory** - this must be the folder that you deployed your app to
* **ExecStart** - make sure that the second part of this points to the location of the compiled dll file of your startup web project
* **User** - as I said earlier, I like to use www-data to look after the running of all web apps, so I've added that here. You could make it a different user, but make sure that you set the ownership of the file structure accordingly.
* **Environment** - you can also add multiple environment variables. In the above example, I've included the production environment variable, and set the print telementry message variable. You can also add in things like connection strings, e.g. `Environment=ConnectionStrings__DefaultConnection={Connection String}`. Make sure you add one per line with the `Environment=` prefix.

Now that you've added in your service details, save the file, and enable the service
```
sudo systemctl enable kestrel-myapp
```
Then, we start the service and confirm that it's running correctly.
```
sudo systemctl start kestrel-myapp
sudo systemctl status kestrel-myapp
```
You'll need to press Ctrl+C to close out of the status message once you've checked it.

### Checking the logs
As we're using systemctl to run the service, all of the log files are stored in a centralised journal. If you want to check this, you can run the following command:
```
sudo journalctl -fu kestrel-myapp
```
If you want to filter the journal by time constraints, you can use commands like `--since today` or `--until 1 hour ago`
```
sudo journalctl -fu kestrel-myapp --since "2020-03-01" --until "2020-03-07"
```
## Configure nginx as the reverse proxy
The install is simple. You can use the following commands
```
sudo apt-get install nginx
sudo service nginx start
```
The config is not so much. I am going to stick to using the standard default server config for this, and then you can add multiple sites to the one file. You can break it out into a separate config file for each application you want to host, but let's not overcomplicate things for the moment.

To edit the default site config, use
```
sudo nano /etc/nginx/sites-available/default
```
This should open the file in nano. You'll want to add in something along these lines:
```
# Default server configuration
server {
  listen 80 default_server;
  return 444;
}

# My app
server {
  listen 80;
  server_name mywebsitedomain.com *.mywebsitedomain.com;
  location / {
    proxy_pass  http://localhost:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection keep-alive;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;

  }
}

# Another app I want to host
{
  ...
}
```
A few notes on the above:
* In the first server config, we set nginx to listen on port 80. If it doesn't get a request for any of the server configs then listed afterwards, nginx will return a 444 status code, which is *connection closed without response*. This is put in place as a security measure to deny malicious or malformed requests.
* In our app server config (second one in the file) we need to make sure that the `proxy_pass` is pointing to localhost and the port number that we're running our predefined service on. The port number can be controlled in the Startup.cs file of your application, so if you want to change it, I recommend republishing your site to the server.
* `server_name` must include the domains that you want to point to this application, each one separated by a space. You can see that I've added in a wildcard. Make sure that you update your DNS records to match.
* We can set a number of the headers in the app config. I've outlined all the ways I tend to use, but you can amend these if you want to. I won't go into that here, as this is a bit out of scope. However, I will get around to writing something up on this and adding it to the knowledgebase.

Save out your new default config file, and restart your nginx server
```
sudo service nginx restart
```

Drop the mike. That should now all be up and running.

I'll write up a separate article on nginx specifics, and one on how to harden your nginx setup for security.
