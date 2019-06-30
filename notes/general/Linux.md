# Linux Notes

1. File System and Memory
 * [Inodes and Links](#inodes-and-links)

2. Networking

## File System and Memory

### Inodes and Links

#### Directories and Inode Numbers
* _inodes_ store all information about a file except name and actual data.  The block of file data itself does not store any metadata.
* A directory is a special type of file that associates file names with _inode numbers_.
* When a file is created it is assigned a name and an _inode number_.  _Inode numbers_ are unique in a filesystem.  The name and inode number are stored as entries in the containing directory.
* When a file is referred to in a directory, it's name is used to find its inode number.  The inode number is then used to find the corresponding inode in _inode tables_ which are stored in known locations on the file system.  
* `ls -l` lists all the information stored in an inode.  `ls -i` will show inode numbers.  `stat` is the system call used to obtain a file's inode number and information from its inode.
* Just as a filesystem can run out space for additional data, it can also run out of space for additional inodes since the number of possible inodes is fixed at the creation of the filesystem.  This is most likely when a filesystem contains a large number of small files.  `df -i` supplies information about a filesystem's inodes including percentage used.  (Should be usefully combined with `-h` option as `df -hi`.)
* When parts of an inode are lost (on a damaged filesystem), they appear in the _lost and found_ directory.

#### Links
* Because the filename itself isn't part of the file's metadata, there can be multiple names for a single inode.  A _hard link_ is an alternative name for a file that shares a common inode (and inode number) with the original filename.  Inodes count the number of references to itself so all hard links to a file must be deleted before the space can be freed.
* Hard links to directories are not allowed since that could break the DAG structure of the filesystem.
* A soft (or symbolic) link only refe32means the link can be left dangling if the original file is deleted.
* Since soft links only refer to another file _name_, they can span file systems.  To exist, they only need handle of the file's name.

#### References
* http://www.linfo.org/inode.html 

### Memory and Swap Files
* Available memory can be displayed with the `free` utility.  The options `-k`, `-m`, `-g` will display output in kilobytes, megabytes, and gigabytes respectively.

## Networking

### FTP/SFTP
* SFTP stands for SSH File Transfer Protocol or Secure File Transfer Protocol.  It is almost always preferable to FTP and it can piggy-back on an SSH connection.
* To establish an SSH connection and open an SFTP session:
```
sftp username@remote_hostname_or_IP
```
* In a session, `help` or `?` will display available commands

##### Navigating
* During and SFTP session, can navigate both the remote and local file systems.  Commands like `pwd`, `ls`, and `cd` are not the normal shell commands (not as feature-rich) and refer to the remote directory.  The equivalent commands for the local directory is `lpwd`, `lls`, and `lcd`
* `df -h` displays human-readable disk capacity of the remote host.  There is no `ldf'.  Instead, use `!` to open a local shell (and run `df`) - or use `!df -h`


#### File Transfer
* To retrieve a remote file, use:
```
get remoteFile
```
Or to save it with a different local name:
```
get remoteFile localFile
```
* To recursively copy a remote directory, use:
```
get -r remoteDirectory
```
* `put` places a local file on a remote host.  `put` uses the same flags as `get`
* Use `bye` or `exit` to end session

### SCP (Secure Copy)

* Offers more limited toolset than `sftp`.  SCP can only be used for transferring files and isn't interactive.  Listing and creating directories remotely requires `sftp`.
* SCP use a faster, more efficient file transfer algorithm that doesn't wait for packet confirmation. However, this comes at the expense of being unable to interrupt a file transfer.
* To get a remote file:
  ```
  scp user@host:/path/to/remote/file [/path/to/local.file]
  ```
* To put a local file:
  ```
  scp /path/to/local.file user@host:[path/to/remote.file]
  ```
#### References
* https://superuser.com/questions/134901/whats-the-difference-between-scp-and-sftp
