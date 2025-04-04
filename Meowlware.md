# Meowlware | Hard (600 points)

Hi Analyst,

We've been hit by the infamous LunaCrypt Ransomware Group We need your help to restore some encrypted files!

Attached is a 7-zip archive containing the pertinent files we acquired during our investigation - `48276_SuspectedLunaCryptRansomware.7z`. This archive contains the encrypted files (`victim-files/`) and the Zip archive which was downloaded from one of the threat actor's staging servers after encryption occurred (`LunaCryptDecryptor.zip`). Contained in the 7-zip is the decryptor utility created by the ransomware group.

Good Luck

password: `infected`

-----

Looking at the files, we see four encrypted files within `victim-files`, including our flag:

![](/images/meowlware_victim_files.png)

Unzipping the file that was retrieved from by the attacker from their staging server, we see two entries:

- `LunaCryptDecryptor.exe`: the decryptor utility
- `.git`: a Git repo file??

Firstly, `LynaCryptDecryptor.exe` accepts the path to the `victim-files` directory and then the decryption key which **must** be 64-characters in length.

![](/images/meowlware_decryptor_utility.png)

Looking at this further, we see this is a .NET-compiled binary which can be decompiled in DNSpy/ILSpy to almost its complete sourcecode:

```
$ file LunaCryptDecryptor.exe
LunaCryptDecryptor/LunaCryptDecryptor.exe: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
```

Opening this in DNSpy, we see the decryptor utility uses AES to decrypt the files but we don't have any clues as to how we can retrieve the key used:

![](/images/meowlware_decryptor_dnspy.png)

The `.git` artifact is a strange one as this is indicative of a Git repository. Looking at the `log`, we can see this is indeed the case with two previous commits from the threat actor - this must have been zipped and added to the staging server by mistake!

The commit with ID `2ea07856ce06f3d3b21e5db3bb37b94ed815250f` shows there were previously encryption functions which the threat actor added for testing/development:

![](/images/meowlware_git_log.png)

We can checkout this commit and look at the previous version of `LunaCryptDecryptor.exe`:

```
git checkout 2ea07856ce06f3d3b21e5db3bb37b94ed815250f
```

Now if we open this binary in DNSpy/ILSpy, we see additional functions including `CalculateKey()`, `EncryptFileAES()` and `EncryptFiles()`. Looking at the code, we see the key is actually the SHA256 hash of the first file in the directory (1) with every 5th character changed with an additional `string2` (2) declared as `6f2dc9a8e5f1b4dc2b8a3e4f6c7d9a8b1f2c3a7e4b1f2c3a7e4b1f2c3a7e4b1a`.

![](meowlware_key_calculation.png)

From the `victim-files/` directory, the first file (alphabetically) is `Bliss_(Windows_XP).png.lunacrypt`. This means that the key is the SHA256 of the original file, altered with the `string2` variable.

Searching for this filename on Google, we come across the [Bliss Wikipedia page](https://en.wikipedia.org/wiki/Bliss_(photograph)). We can save this and compute the SHA256 hash with PowerShell:

```
Get-FileHash -Algorithm SHA256 .\Bliss_(Windows_XP).png

7AF0EFB2DD0984AF998DB27EBB3243D666A94A28DFFC6F2E498967B2571C5C4E
```

With this we can alter this with the value of `string2` (``6f2dc9a8e5f1b4dc2b8a3e4f6c7d9a8b1f2c3a7e4b1f2c3a7e4b1f2c3a7e4b1a``) to get the final key:

```
6af0cfb2ed09b4af298d327e6b3293d616a93a284ffc2f2e798917b2371c4c4e
```

Using this within our Decryptor utility with the path to `victim-files/`, we can get the flag:

![](/images/meowlware_decrypting_files.png)

![](/images/meowlware_flag.png)

```
flag{lun4cryp7_15_b4d_4t_43s_ac99a742}
```