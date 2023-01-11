# OSU Engineering Servers SSH Tutorial

### Nomenclature
- Capitalized terms below (such as LOCAL_USERNAME) are to be read as a stand-in value, please substitute your own credentials with the correct capitalization. 
- "Local" is to be read as referring to files on your personal computer.
- "Remote" or "School" refers to files stored on the ENGR school's remote servers.
- `flip` is the one of the more common remote school servers, which you will use for almost every class. `os1` and `os2` are separate servers used for the Operating Systems 1 / Operating Systems 2 classes respectfully, and CS370 (Intro to Security) and CS499 CAND (Cybersecurity and Defense) may or maynot have separate servers as well, such as if you have Yeongjin Jang as your professor.

### Helpful Prerequisites for File Transfer
These aren't strictly necessary but they could help out with copy and pasting files from local to remote and whatnot, for the SSH instructions below. 

You can do the following steps before you set up any SSH keys, just login with password and then do dual auth. Note that files on the `os1` and `os2` servers are shared with `flip`.

<details>
  <summary>Click to Expand</summary>
  
  ### Mapping Network Drive (GUI File Transfer)
  The following instructions for mapping a network drive can help you out for easily copy and pasting files with GUI between local and remote.
  - [Windows](https://it.engineering.oregonstate.edu/accessing-engineering-file-space-using-windows-file-sharing)
  - [MacOS / Linux](https://it.engineering.oregonstate.edu/accessing-engineering-file-space-using-os-x-smb-command)
  - NOTE: If you are currently on OSU campus WiFi (i.e. physically on campus), you can map the network drive right away. If you are off-campus, you need to use a VPN.
    - OSU VPN Installation: https://oregonstate.teamdynamix.com/TDClient/1935/Portal/KB/ArticleDet?ID=76790

  ### SCP (CLI File Transfer)
  If you prefer a command line tool, this will help you transfer files between local and remote.

  General SCP documentation: https://man7.org/linux/man-pages/man1/scp.1.html

  **Run in local terminal (Between Local and Flip)**

  flip1 > local: `scp ONID@flip1.engr.oregonstate.edu:SOURCE_FILEPATH DESTINATION_FILEPATH`

  local > flip1: `scp SOURCE_FILEPATH ONID@flip1.engr.oregonstate.edu:DESTINATION_FILEPATH`

  Example (flip1 > local): `scp huangjef@flip1.engr.oregonstate.edu:wizards.txt Downloads`

  Example (local > flip1): `scp Downloads/wizards.txt huangjef@flip1.engr.oregonstate.edu:wizards.txt`

</details>

### SSH Key Generation

SSH key generation: Use `ssh-keygen -t ed25519` in local terminal.

**This generates two keys, a private key (no file extension), and a public key (same file name as the private key but with `.pub` file extension). Keep this in mind for the following instructions.**

You will be asked to create a passphrase when generating the SSH key. You can just click not type in a passphrase and tap Enter to skip setting a passphrase for the SSH key if you want, although this may make them less secure.

You can change the key's name later (say if you want serveral SSH key pairs) but if you do that make sure to change both public and private key name (more on this later).

The main thing is to make sure you have saved your ssh keys locally (both public and private key part) in `C:\Users\LOCAL_USERNAME\.ssh` (for Windows), or `~/.ssh` (Linux / MacOS).

### Config / Proxyjump

Next is to make a config, literally make a file called `config` (no file extension) in `C:\Users\LOCAL_USERNAME\.ssh` (for Windows), or `~/.ssh` (Linux / MacOS). Fill in the blanks with your own information. Pick the appropiate IdentityFile format for your platform. If you are in OS1 and not OS2 then change the lines referencing `os2` to `os1`. If you're not in either of the OS1 or OS2 classes yet, feel free to omit that section.

- NOTE: You can choose any of the three flip servers by editing `HostName` to be `flip1`, `flip2`, or `flip3`. This is mainly useful to switch if one of them is down, or I think in CS340 you might have to run a web server, in which case it's more convenient to stay on the same flip version.

```
Host flip1
  HostName flip1.engr.oregonstate.edu
  User ONID
  IdentityFile /C:/Users/LOCAL_USERNAME/.ssh/PRIVATEKEY
  or 
  IdentityFile ~/.ssh/PRIVATEKEY
  
Host os2
    HostName os2.engr.oregonstate.edu
    User ONID
    ProxyJump flip1
```

`os2` or `os1` etc is optional and only if you are taking that class, you normally cannot get into these servers without going through `flip` first. ProxyJump lets you directly join `os1` or `os2` from local computer.

IdentifyFile should be the local ssh private key you created earlier (if you changed the key's name from the default, make sure you put the right key name).

You might need IdentityFile for `os2` / `os1` as well if you used a different key for this.

Also, you can set up a config file on the remote school servers too (`~/.ssh/config` on the remote server) if you want (in order to more easily SSH between `flip` and `os1` / `os2`), although that might be redundant when you have ProxyJump.

More documentation for SSH config: https://www.ssh.com/academy/ssh/config
More documentation for ProxyJump: https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump

### Authorized_keys
In your SSH directory on the school's remote server (`~/.ssh` on the remote server), add a file called `authorized_keys`. Copy and paste all your **public** keys from your local machine as plain text into the `authorized_keys` file on the remote server. Leave a blank line between each public key. **Your ssh will not work properly if you skip this.** 
- Refer above to the [helpful files for prerequisite section](#helpful-prerequisites-for-file-transfer) if you need help transferring files between local and remote.

### SSH without VSCode
With the config set up, you can ssh from local terminal by just typing out `ssh flip1`, `ssh os2`, etc, depending what you call them in config

### VSCode Integration

In VScode (hit `Extensions` in left tab). Search for `Remote Development` and `Remote SSH` extensions. 

Install them. Then go to `View` > `Command Palette` > `Remote-SSH: Open SSH Configuration File`. 

Select the `C:\Users\LOCAL_USERNAME\.ssh\config` or `~/.ssh/config` (local) file you made earlier.

Then hit `View` > `Command Palette` > `Remote-SSH: Connect to Host `

If you set config correct, you should see `flip1`, `os2`, and whatever already.

Select the server, I think it also asks for platform (select Linux for the remote server type), and you're in.

More documentation:  https://code.visualstudio.com/docs/remote/ssh

### Git
**Why use Git SSH keys: You will no longer be prompted to enter a password when pushing and pulling from the command line, nor will you have to deal with expiring Personal Access Tokens.**

Instructions for adding SSH keys to Git: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
  - Use same process as explained above for creating the SSH keys, or you can even reuse a SSH key you already made.

Make sure to add the SSH keys (public and private pair) you use for Git to your school's remote SSH directory (`~/.ssh` on the remote server), as well as your local SSH directory (`C:\Users\LOCAL_USERNAME\.ssh` or `~/.ssh` on local machine). That will ensure you can easily use git from the command line on both your local machine and on the remote server.
. If in doubt, copy over all ssh keys from local to your school directory.

- Refer above to the [helpful files for prerequisite section](#helpful-prerequisites-for-file-transfer) if you need help transferring files between local and remote.

### Bonus: CS370 / CAND SCP
This is only for if you are taking CS370 Intro to Security or CS 499 CAND with Yeongjin Jang, in which case he may have a custom server that makes SCP more complicated. You will need to set up SSH keys at least for the CS370 / CAND server first.
<details>
  <summary>Click to Expand</summary>

  **Run these after you SSH into Flip (Between Flip and CS370)**

  -----------------------------------------------------------------
  CS370 to Flip: `scp -i ~/PRIVATE_KEY_FILEPATH CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:SOURCE_FILEPATH DESTINATION_FILEPATH`
  Flip to CS370: `scp -i ~/PRIVATE_KEY_FILEPATH SOURCE_FILEPATH CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:DESTINATION_FILEPATH`

  The `-i PRIVATE_KEY_FILEPATH` part is optional, only include that if you saved your private SSH key somewhere other than `id_ed25519`. **Remember to use your SSH key on the remote school server. Copy over your local SSH keys to School server's SSH directory if in doubt.

  Example (CS370 > flip1): `scp -i ~/.ssh/id_ctf huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py downloads`

  Example (flip1 > CS370): `scp -i ~/.ssh/id_ctf downloads/solution.py huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py`

  -----------------------------------------------------------------

  **Run in local terminal (Between Local and CS370, via ProxyJump through Flip)**

  CS370 to Local:`scp -i PRIVATE_KEY_FILEPATH -o "ProxyJump ONID@flip1.engr.oregonstate.edu" CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:SOURCE_FILEPATH DESTINATION_FILEPATH`

  Local to CS370: `scp -i PRIVATE_KEY_FILEPATH -o "ProxyJump ONID@flip1.engr.oregonstate.edu" SOURCE_FILEPATH CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:DESTINATION_FILEPATH`

  The `-i PRIVATE_KEY_FILEPATH` part is optional, only include that if you saved your private SSH key somewhere other than `id_ed25519`. And this refers to your local machine's SSH key, not the remote school server SSH key.

  Example (CS370 > Local): `scp -i downloads/id_ctf -o "ProxyJump huangjef@flip1.engr.oregonstate.edu" huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py downloads`

  Example (Local > CS370): `scp -i downloads/id_ctf -o "ProxyJump huangjef@flip1.engr.oregonstate.edu" downloads/solution.py huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py`

</details>
