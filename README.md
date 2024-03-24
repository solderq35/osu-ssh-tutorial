# OSU Engineering Servers SSH Tutorial

### Nomenclature
- Capitalized terms below with dollar sign in front (such as `$PRIVATE_KEY_FILEPATH`) are to be read as a stand-in value, please substitute your own credentials with the correct capitalization and do not insert a dollar sign unless it's actually in the file / folder name. 
- `~` (for MacOS / Linux) and `%HOMEPATH%` (for Windows) are shortcuts to your computer / server's home directory. Please type in these home directory terms exactly as written into your terminal.
  - `%HOMEPATH` appears to not work for SSH config files, so substitute `/c:/Users/$USERNAME` when working with config files in Windows.
- "Local" is to be read as referring to files on your personal computer.
- "Remote" or "School" refers to files stored on the ENGR school's remote servers.
- `flip` is the one of the more common remote school servers, which you will use for almost every class. `os1` and `os2` are separate servers used for the Operating Systems 1 / Operating Systems 2 classes respectfully, and CS370 (Intro to Security) and CS499 CAND (Cybersecurity and Defense) may or maynot have separate servers as well, such as if you have Yeongjin Jang as your professor.
- Pretty much any terminal on your local computer can qualify as a "local terminal" for the sake of the instructions below. Windows users are frequently recommended to use Putty, e.g. [this article](https://it.engineering.oregonstate.edu/accessing-unix-server-using-putty-ssh), but Windows Command Prompt also supports SSH natively so Putty is unneccessary.

### SSH Key Generation

SSH key generation: Use `ssh-keygen -t ed25519` in local terminal.

**This generates two keys, a private key (`id_ed25519`), and a public key (`.id_ed25519.pub`), in the SSH folder of your home directory (`~/.ssh` for MacOS / Linux users, and `%HOMEPATH%/.ssh` for Windows users).**

If you want to save your SSH key as a different filename or location (such as if you are managing several SSH key pairs), then add the argument `-f $FILENAME` (with full filepath included). 

Examples: 
- `ssh-keygen -t ed25519 -f %HOMEPATH%/.ssh/id_test`
or
- `ssh-keygen -t ed25519 -f ~/.ssh/id_test`

You will be asked to create a passphrase when generating the SSH key. You can just hit Enter without typing anything to skip setting a passphrase for the SSH key if you want, although this may make them less secure.

The main thing is to make sure you have saved your SSH keys locally before moving on.

### Helpful Prerequisites for File Transfer
These aren't strictly necessary but they could help out with copy and pasting files from local to remote and whatnot, for the some of the SSH instructions below. 

You can do the following steps before you set up any SSH keys, just login with password and then do dual auth. Note that files on the `os1` and `os2` servers are shared with `flip`.
  
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

  - local > flip1: `scp $SOURCE_FILEPATH $ONID@flip1.engr.oregonstate.edu:$DESTINATION_DIRECTORY`
  - flip1 > local: `scp $ONID@flip1.engr.oregonstate.edu:$SOURCE_FILEPATH $DESTINATION_DIRECTORY`
  - NOTE: The source file's exact filepath must be provided, but for the destination, just list the directory (folder) you want the file to end up in.

  Below examples are for transferring SSH files between local and Flip, but you might want to try with just a random text file first (substitute `~` for `%HOMEPATH%` if your local computer is Linux or MacOS).
  - Example (local > flip1): `scp %HOMEPATH%/.ssh/id_ed25519.pub huangjef@flip1.engr.oregonstate.edu:.ssh`
  - Example (flip1 > local): `scp huangjef@flip1.engr.oregonstate.edu:.ssh/id_ed25519.pub %HOMEPATH%/.ssh`

  - ![Example of what your output here should look like](https://i.ibb.co/Q6xWn7p/ssh1.png) 
    - If you haven't finished setting up SSH keys yet you might get prompted to login with password and dual auth

### Config / Proxyjump

Next is to make a config, literally make a file called `config` (no file extension) in `%HOMEPATH%\.ssh` (for Windows), or `~/.ssh` (Linux / MacOS). Fill in the blanks with your own information. Pick the appropiate IdentityFile format for your platform. If you are in OS1 and not OS2 then change the lines referencing `os2` to `os1`. If you're not in either of the OS1 or OS2 classes yet, feel free to omit that section.

- NOTE: You can choose any of the three flip servers by editing `HostName` to be `flip1`, `flip2`, or `flip3`. This is mainly useful to switch if one of them is down, or I think in CS340 you might have to run a web server, in which case it's more convenient to stay on the same flip version.

- For Windows, `%HOMEPATH` in the SSH config appears to trigger an error, so use `/c:/Users/$USERNAME` for this part.

```
Host flip1
  HostName flip1.engr.oregonstate.edu
  User ONID
  IdentityFile /c:/Users/$USERNAME/.ssh/$PRIVATEKEY
  or 
  IdentityFile ~/.ssh/$PRIVATEKEY
  
Host os2
    HostName os2.engr.oregonstate.edu
    User ONID
    ProxyJump flip1
```

`os2` or `os1` etc is optional and only if you are taking that class. Adding this allows you to access those servers directly from your Local machine. Normally, you cannot get into these servers without going through `flip` first. The `ProxyJump` option tells SSH to go through `flip` first automatically.

IdentifyFile should be the local SSH private key you created earlier (if you changed the key's name from the default, make sure you put the right key name).

You might need IdentityFile for `os2` / `os1` as well if you used a different key for this.

Also, you can set up a config file on the remote school servers too (`~/.ssh/config` on the remote server) if you want (in order to more easily SSH between `flip` and `os1` / `os2`), although that might be redundant when you have ProxyJump.

More documentation for SSH config: https://www.ssh.com/academy/ssh/config
More documentation for ProxyJump: https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump

#### chmod 600
You may also need to restrict read / write permissions on the SSH config file if you are on Linux or MacOS.

You can do this by running `chmod 600 ~/.ssh/config`

- [More Info on SSH Config Required Permissions](https://man.archlinux.org/man/core/openssh/ssh.1.en#FILES)
- [More Info on chmod 600](https://chmodcommand.com/chmod-600/) (read / write for user only)

### Authorized_keys
In your SSH directory on the school's remote server (`~/.ssh` on the remote server), add a file called `authorized_keys`. Copy and paste all your **public** keys (i.e. `id_ed25519.pub`) from your local machine **as plain text** into the `authorized_keys` file on the remote server. Leave a blank line between each public key. **Your ssh will not work properly if you skip this.** 

- [Example of what authorized_keys should look like](https://i.ibb.co/fCDVjzS/ssh2.png)(public key details scrubbed for privacy)

- You can upload local SSH keys to ENGR servers with SCP as explained above, e.g. `scp "%HOMEPATH%/.ssh/id_ed25519.pub" $ONID@flip1.engr.oregonstate.edu:.ssh` (substitute `~` for `%HOMEPATH%` if your local computer is Linux or MacOS).

### SSH without VSCode
With the config set up, you can SSH from local terminal by just typing out `ssh flip1`, `ssh os2`, etc, depending what you call them in config

### VSCode Integration

In VScode (hit `Extensions` in left tab). Search for `Remote Development` extension pack. 
- ![Screenshot of the extension](https://i.ibb.co/BPjKcMb/ssh3.png)

Install them. Then go to `View` > `Command Palette` > `Remote-SSH: Open SSH Configuration File`. 

Select the `C:\Users\$USERNAME\.ssh\config` or `~/.ssh/config` (local) file you made earlier.
  - `%HOMEPATH` shortcut appears to not work here, use full Windows path.

Then hit `View` > `Command Palette` > `Remote-SSH: Connect to Host `

If you set config correct, you should see `flip1`, `os2`, and whatever already.

Select the server, I think it also asks for platform (select Linux for the remote server type), and you're in.

More documentation:  https://code.visualstudio.com/docs/remote/ssh

WARNING / DISCLAIMER: VSCode SSH Extension has some notable issues, including hogging CPU usage and spawning runaway processes, leading to worse performance on the ENGR servers than if you done SSH directly through your terminal.
- If you are ever worried about it, you can kill runaway processes on the ENGR servers here (select the server you were working on): https://teach.engr.oregonstate.edu/teach.php?type=kill_runaway_processes

### Git
**Why use Git SSH keys: You will no longer be prompted to enter a password when pushing and pulling from the command line, nor will you have to deal with expiring Personal Access Tokens.**

Instructions for adding SSH keys to Git: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
  - Use same process as explained above for creating the SSH keys, or you can even reuse a SSH key you already made.

Make sure to add the SSH keys (public and private pair) you use for Git to your school's remote SSH directory (`~/.ssh` on the remote server), as well as your local SSH directory (`%HOMEPATH%\.ssh` or `~/.ssh` on local machine). That will ensure you can easily use git from the command line on both your local machine and on the remote server.
If in doubt, copy over all SSH keys from local to your school directory.

- Refer above to the [helpful files for prerequisite section](#helpful-prerequisites-for-file-transfer) if you need help transferring files between local and remote.

### Bonus: CS370 / CAND SCP
This is only for if you are taking CS370 Intro to Security or CS 499 CAND with Yeongjin Jang, in which case he may have a custom server that makes SCP more complicated. You will need to set up SSH keys at least for the CS370 / CAND server first.

  **Run these after you SSH into Flip (Between Flip and CS370)**

  -----------------------------------------------------------------
  - CS370 to Flip: `scp -i ~/$PRIVATE_KEY_FILEPATH $CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:$SOURCE_FILEPATH $DESTINATION_DIRECTORY`
  - Flip to CS370: `scp -i ~/$PRIVATE_KEY_FILEPATH $SOURCE_FILEPATH $CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:$DESTINATION_DIRECTORY`

  The `-i $PRIVATE_KEY_FILEPATH` part is optional, only include that if you saved your private SSH key somewhere other than `id_ed25519`. **Remember to use your SSH key on the remote school server. Copy over your local SSH keys to School server's SSH directory if in doubt.

  - Example (CS370 > flip1): `scp -i ~/.ssh/id_ctf huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py downloads`

  - Example (flip1 > CS370): `scp -i ~/.ssh/id_ctf downloads/solution.py huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py`

  -----------------------------------------------------------------

  **Run in local terminal (Between Local and CS370, via ProxyJump through Flip)**

  - CS370 to Local:`scp -i $PRIVATE_KEY_FILEPATH -o "ProxyJump $ONID@flip1.engr.oregonstate.edu" $CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:$SOURCE_FILEPATH $DESTINATION_DIRECTORY`

  - Local to CS370: `scp -i $PRIVATE_KEY_FILEPATH -o "ProxyJump $ONID@flip1.engr.oregonstate.edu" $SOURCE_FILEPATH $CS370_USERNAME@vm-ctf1.eecs.oregonstate.edu:$DESTINATION_DIRECTORY`

  The `-i $PRIVATE_KEY_FILEPATH` part is optional, only include that if you saved your private SSH key somewhere other than `id_ed25519`. And this refers to your local machine's SSH key, not the remote school server SSH key.

  - Example (CS370 > Local): `scp -i downloads/id_ctf -o "ProxyJump huangjef@flip1.engr.oregonstate.edu" huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py downloads`

 -  Example (Local > CS370): `scp -i downloads/id_ctf -o "ProxyJump huangjef@flip1.engr.oregonstate.edu" downloads/solution.py huangjeff@vm-ctf1.eecs.oregonstate.edu:crypto/caesar/solution.py`
