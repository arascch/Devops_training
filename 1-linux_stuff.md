# Practice 1
  ## Whats the MINIX file system?
  The Minix file system is the native file system of the Minix operating system. in this the last version of minix is 3. the main aim of Minix is to replicate the structure of the Unix file system while omitting complex features and was intended to be a teaching aid.
  
  [MINIX FS has six main component](https://en.wikipedia.org/wiki/MINIX_file_system):
  1. the **boot block** : stored in first block and contain bootloader 
  2. the **superblock**: store data about the file system
  3. the **inode bitmap** : is a simple map of inodes
  4. the **zone bitmap** : works exact like inode , except it tracks the zones
  5. the inode area : each file or directory is represented as an inode. which records metadata including types, IDs for users and groups and three time stamps. an inode contain a list of addresses that point to zones in the data area.
  6. the **data area** : it is where the actual file and directory data are stored and using huge space.
  ## [Whats the Umask in Linux](https://www.liquidweb.com/kb/what-is-umask-and-how-to-use-it-effectively/) ? 
  
  Umask(user file-creation mode mask) is used by unix-based system to default permissions for newly created files and directories. 

  **working mechanism** :
  usual umask being set to 022 on most systems for all the new files we create will subtract the umask value from full permissions.
  >[!NOTE]
>for files that would be 666-022 = 644 /
>for new directories would be 777-022=755

> [!TIP]
>Umask can be expressed in **octal** or **symbolic values**.

>octal values ( like 022 ) : the three digits will affect resulting permissions for user , group and nobody users.

>symbolic values ( like g=rwx ): equivalent to octal values of 022.

some example :
>[!note]
>use stat command will show the file's permissions in octal mode  
```
[root@server1]# touch aboutme.txt
[root@server1]# stat -c '%a' aboutme.txt
644
[root@server1]#umask 026 "white this line you can change default umask"
```

  
