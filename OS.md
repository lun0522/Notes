# Operating Systems

## The File System

### Three Kinds of Files

#### Ordianry Files

The system has no requirement for their structure.

#### Directories

Just like ordinary files, but cannot be written on by unprivileged programs, so they are under the control of system. We can use linking to make different names refer to the same file.

#### Special Files

Each supported I/O device is associated with at least one such file.

### Mount

To **mount** a storage volume, we need the name of an existing ordinary file, and the name of a special file associated with the storage volume, which contains the structure of an independent file system. **mount** actually replaces a leaf node of our system file tree by a whole new subtree.

### Protection

When a file is created, the system marks it with the user ID of its owner, and another 10 bits for protection. Nine out of them control read-write-execute permissions. When the last bit is on, the system will temporarily change the ID of the current user to the owner's ID.

One user may create a program which uses a file that is not meant to be directly accessed by other users, and turn on this set-user-ID feature; later when other users execute this program, that private file will beome accessible.

## Implementation of the File System

The system maintains a table, called the **i-list**, where each entry contains the description of a file:

- the user and group ID of the owner
- protection bits
- addresses for the file contents
- file size
- time of creation, last use, and last modification
- number of links to the file
- a code indicating the file type (ordinary/directory/special)

The file descriptor is actually an integer to lookup for this entry, called the **i-number** (the index number of an **i-node**).

