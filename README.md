### SSH Key Generation

SSH key generation: `ssh-keygen -t ed25519`
You can change the key's name later but if you do that make sure to change both public and private key name (more on this later) 

don't add a passphrase to your ssh keys unless you will remember

The main thing is to make sure you have saved your ssh keys (both public and private key part) in `C:\Users\<username>\.ssh` (for windows) 
Or whatever is the equivalent local ssh key location on your platform

### Config

Next is to make a config, literally make a file called config (no file extension) in your `C:\Users\<username>\.ssh` (or wherever yours is) directory
I like to set config as 

```
Host flip1
  HostName flip1.engr.oregonstate.edu
  User <onid>
  IdentityFile /c:/Users/<username>/.ssh/<privatekey>
  
Host os2
    HostName os2.engr.oregonstate.edu
    User <onid>
    ProxyJump flip1
```

os2 or os1 etc is optional and only if you are taking that class, you normally cannot get into these servers without going through flip first. ProxyJump lets you directly join os1 or os2 from local computer.

IdentifyFile should be the ssh key you created earlier (if you changed the key's name from the default, make sure you put the right key).

You might need IdentityFile for os2 as well if you used a different key for this.

### Authorized_keys
In your remote school directory (`<onid>/.ssh`), add a file called `authorized_keys`. Copy and paste all your **public** keys here as plain text (not private keys). **Your ssh will not work properly if you skip this**

### SSH without VSCode
With the config set up, you can ssh from local terminal by just typing out `ssh flip1`, `ssh os2`, etc. Depending what you call them in config

### VSCode Part

In VScode (hit "Extensions" in left tab). Search for "remote development" and "remote ssh" extension, install. Then go to view > command pallete > remote-ssh: open configuration file. Select the C:\Users\<username>\.ssh\config file you made earlier.

Then hit view > command pallete > remote-ssh: connect to host
If you set config correct, you should see "flip1", "os2", and whatever already.
Select the server, I think it also asks for platform (say linux), and you're in

More documentation:  https://code.visualstudio.com/docs/remote/ssh

You might have to set another identityfile for the os2 part if you have a different key for it btw.

### Git

If you have ssh keys for Git (highly recommended) make sure to add the ssh keys (public and private pair) you use for Git to your remote school directory (`<onid>/.ssh`). If in doubt, copy over all ssh keys to your school directory.