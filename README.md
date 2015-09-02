# LCFSA
Guia para la certificacion de la Linux Foundation

* LFCS Domains 2015.02
  Note - the domains will change somewhat in March 2015. SW RAID
  with mdadm will be removed

** The Command Line
*** Editing text files on the CLI
    Covers the use of the basic text editors nano and gedit as well
    as the advanced editors _vi_ and _emacs_
    - nano
      simple CLI-based text editor
    - gedit
      simple GUI-based text editor
    - *vi/vim*
      - invocation
        + vi -r myfile :: start vi and edit myfile in recovery mode
          after a system crash - vim stores your changes in a swap
          file. Using the original file plus the swap file, you can
          mostly recover your work.
        + vi myfile :: edit myfile with vi

      - _command mode_
        initial mode that vim starts in

        - movement
          + hjkl :: Left Down Up Right
          + H :: move to top of screen
          + L :: move to bottom of screen
          + 0 :: move to beginning of line
          + $ :: move to end of line
          + :0 or 1G or gg :: move to beginning of file
          + :n or nG :: move to line 'n'
            + :10 :: move to 10th line
            + 10G :: move to 10th line
          + :$ or G :: move to last line in file
          + C-f :: move forward one page
          + C-b :: move back one page
          + /pattern :: forward search for 'pattern'
          + ?pattern :: reverse search for 'pattern'
          + n :: move to next occurrence of search pattern
          + N :: move to previous occurrence of search pattern
          + C-g :: show position and line no; type _nG_ to go to
            line number `n`

        - editing
          + a :: append text after cursor
          + A :: append text at end of current line
          + I :: insert text at the beginning of current line
          + i :: insert text before cursor
          + o :: start new line below current line, insert text
          + O :: start new line above current line, insert text
          + Nx :: delete N chars, starting at current position
          + dw :: delete the word at the current position
          + D or d$ :: delete the rest of the current line
          + Ndd or dNd :: delete N lines
          + u :: undo the previous operation
          + C-R :: redo
          + yy :: yank(copy) the current line and put in buffer
          + Nyy or yNy :: yank N lines and put in buffer
          + p :: put after the current line
          + P :: put before the current line

      - _insert mode_
        from command mode, press 'i' to enter Insert mode
      - _line mode_
        from command mode, press ':' to enter Line mode
        + :r file2 :: read in file2 and insert at current position
        + :w :: write to the current file in buffer
        + :w myfile :: write out the file to myfile
        + :w! file2 :: overwrite file2
        + :x :: exit vi and write out modified file
        + :wq :: exit vi and write out modified file
        + :ZZ :: same as ':x', ':wq' save and exit vi
        + :q! :: quit vi without saving changes

        + external commands

          + :! wc % :: word count of current file (%)
          + :sh cmd :: open an external cmd shell; upon exit
            resume vi session
          + %!fmt :: run current file through fmt and return results
    - *emacs*
      As emacs is our editor of choice, we know most of the
      commands required for daily usage. We are only making note
      of those cmds were weren't aware of previously

      - C-o :: insert a blank line
      - C-@ :: same as C-space (set mark)
      - M-g-g-n :: goto line n

*** Manipulating text files on the CLI

    - I/O Redirection
      - read from '<'
      - write to '>'
      - append to '>>'

    - Viewing files
      - cat :: list file on stdout (no scrollback)
      - tac :: list file on stdout in reverse order (no scrollback)
      - less :: useful for viewing larger files as it provides scrollback
        pauses at each screenful of text
      - head :: print first 10 lines of a file by default
      - tail :: print last 1o lines of a file by default
      - head -n X | tail -n Y :: view a range of lines from text on stdin
        + _head -n 22 | tail -n 11_
          Display from lines 12 to 22, inclusive. Note that _tail_ must
          show the last *11* lines (22,21,20,19,18,17,16,15,14,13,12)
          starting from 22 in order for line 12 to be included.
          0-9: 10 numbers, 0-10: 11 numbers, 12-22: 11 numbers

    - Manipulating Text
      - cat :: concatenate - read, print, join, view files
        + cat file1 file2 :: concatenate multiple files and display output
        + cat file1 file2 > newfile :: concatenate multiple files and
          redirect output to newfile (overwrites)
        + cat file >> file2 :: append file to end of file2
        + cat > file :: interactive mode; redirect stdin to file
          Ctrl-d to terminate input
        + cat >> file :: interactive mode; append stdin to file
          Ctrl-d to terminate input
        + *tac* can be used in all the same ways as above as it has
          the same syntax as _cat_
      - echo :: displays (echoes) text on stdout
        is often used to redirect text to files and also display the
        value of variables
        + echo -e :: use escaped char sequences like '\n' and '\t'
        + echo string > file :: write string to file
        + echo string >> file :: append string to file
        + echo $var :: print value of env. variable
      - sed :: stream editor
        filter and substitute text from an input source and move
        to stdout or an output stream

        - invocation
          + sed -e cmd <file> :: specify editing cmd, operate on
            file and send result to stdout
          + sed -f scriptfile <file> :: specify a script containing
            sed cmds, operate on file, send result to stdout
          + sed s/pattern/replace_str/ file :: substitute 1st string
            occurrence in a line
          + sed s/pattern/replace_str/g file :: subtitute _pattern_ with
            _replace_str_ globally (similar to vi _:s/foo/bar/g_)
          + sed 1,3s/pattern/replace_str/g file :: substitute all string
            occurrences in lines 1 through 3
          + sed -i s/pattern/replace_str/g file :: edit files in-place
            saves changes for string substitution in the same file; this
            option is not recommended b/c it is irreversible; if you
            absolutely must overwrite the original, '>' to a new file,
            verify changes, and then '> originalFile'

** Filesystem and Storage
    - *Archiving and Compressing files and directories*
      + Compressing files
        _tar cvf(z) archive.tar(.gz) file1 file2 file3_
      + Compressing directories
        _tar cvf(z) archive.tar(.gz) /path/to/dir_
        When you run this command, make sure your /pwd/ is not
        the directory you are trying to archive!
    - *Assembling partitions as RAID devices*
      1. Examine the drives which will compose the array
         (Assume we have two drives, sdb and sdc)
         _mdadm --examine /dev/sd[b-c]_
         We want to make sure that there is no existing RAID
         on the two disks ('no md superblock')
      2. Create partitions with fdisk
         _fdisk /dev/sdb_ (and later for /dev/sdc)
         + n :: create new partition ('1' for primary)
         + t :: change partition type ('fd' for Linux RAID auto)
         + w :: write changes and exit
         Of course in between steps you want to use *p* to print the
         partition table to check your work
      3. Create RAID md devices
         _mdadm -C /dev/md0 -l raid0 -n 2 /dev/sd[b-c]1_
         + -C or --create :: create md device
         + -l or --level :: RAID level (0,1,4,5,6,10)
           linear, stripe, mirror, raid4, raid5, raid6, raid10,
           multipath, faulty, container ...
         + -n or --raid-devices :: no. of RAID devices
      4. Verify RAID device
         _cat /proc/mdstat_


    - *Creating LVM partitions* (LFCS & LFS201 topic)
      1. Create Physical Volume (PV)
         _pvcreate /dev/sdXY_
      2. Create Volume Group (VG)
      3. Create Logical Volume (LV)
      4. Make file system on LV
         _mkfs.ext[2,3,4,] /dev/VGname/LVname_
    - *Configuring swap partitions*
      1. _fdisk /dev/sdX_
         n, p, 1, t, 82, w
         normally swap partitions are designated as type 82, but
         it is actually possible to use any partition type
      2. _mkswap /dev/sdXY_
         set up the linux swap area
      3. _swapon /dev/sdXY_
         enables the device for paging
      4. edit /etc/fstab
         _/dev/sdXY none swap defaults 0 0_
    - *File attributes*
      + lsattr :: list file attributes
        + a :: append only
          a file with 'a' attribute set can only be opened in
          append mode for writing
        + c :: compressed
          a file with 'c' attribute set is automatically compressed
          on disk by the kernel, but reads from the file return
          uncompressed data
        + d :: no dump
          if this attribute is set, the file will not be backed
          up when _dump_ is run
        + e :: extent format
          if this attrib is set, it means the file is using
          extents for mapping blocks on a disk
        + i :: immutable
          a file with this attrib set cannot be modified, deleted,
          renamed, and no link to the file can be created. This
          attrib *even prevents su from deleting or modifying*
          a file!
        + j :: data journalling
          if this attrib is set, all its data is written to the
          ext3 journal before being written to the file itself
        + s :: secure deletion
          if this attrib is set on a file, when the file is deleted
          all its block are zeroed and written back to the disk
        + t :: no tail-merging
          if this attrib is set on a file, a partial block fragment
          at the end of the file will _not_ be merged with other files
          + tail-merging :: efficiently use slack space at the end
            of large files by packing the 'tail'/last partial block
            of multiple files into a single bock
        + u :: undeletable
          if a file has this attrib set, when the file is deleted
          its contents are saved, allowing for the user to ask
          for its undeletion
        + A :: no atime updates
          if a file has this attrib set, when the file is accessed,
          its atime record is not modified
          + atime :: time a file was last accessed
            disabling this attribute can speed up file operations
        + C :: no copy on write
          if a file has this attrib set, a fs supporting CoW
          (Btrfs, for example) will not use CoW. CoW causes many
          small random writes for large files, so CoW should be
          disabled for DB files and VM images
        + D :: synchronous directory updates
        + S :: synchronous updates
        + T :: top of directory hierarchy

      + chattr :: change file attributes on Linux file systems
        part of the _e2fsprogs_ package
        + -R :: Recursively change attribs of dir's and their
          contents
        + -V :: verbose output
        + -f :: suppress error messages
          shut the *f*uck up
        + -v :: version
        + _chattr -R +C /MULTIMEDIA/VM_
          disable CoW recursively in dir .../VM and its sub-dirs
          (+C means _no Copy-on-Write_)

    - *Filesystem checking* (LFS201 topic)
      + e2fsck :: check ext2,3,4 fs
      + fsck :: a wrapper for fs-specific fs-checkers
    - *Filesystem quotas and usage* (LFS201 topic)
      https://wiki.archlinux.org/index.php/Disk_quota
      + install disk package quota
        - quota-tools (Archlinux)
        - quota (RHEL and Ubuntu)
      + set up  file system quotas
        + user quota (edit /etc/fstab)
          _/dev/sda1 /home ext4 defaults 1 1_
          This is our fstab entry for 'home'. We will edit this to
          enable a user disk quota
          _/dev/sda1 /home ext4 defaults,usrquota 1 1_

        + group quota (edit /etc/fstab)
          _/dev/sda1 /home ext4 defaults,usrquota,grpquota 1 1_

        + create quota files in the fs
          _touch /home/aquota.user_
          _touch /home/aquota.group_

        + remount partitions with quotas
          _mount -vo remount /home_

        + create quota index
          _quotacheck -vgum /home_
          to create quota index for all partitions with quota
          mount options in /etc/mtab
          _quotacheck -vguma_

        + finally, enable quotas
          _quotaon -av_

    - Finding files on a filesystem
      + find
      + locate
      + ls
    - Formatting filesystems
      - ext2/3/4 filesystems
      - mkfs
      - XFS and btrfs filesystems
    - Mounting filesystems automatically at boot time
      you need to add an entry in /etc/fstab and make sure that the
      mountpoint exists
    - Mounting networked filesystems
      1. NFS client config
         _mount -t nfs(nfs4) -o servername:/remoteDir /local/mntpt_

      2. Add NFS mount to /etc/fstab
         _server:/remote/export /local/mntpt nfsType options 0 0_
         _servername:/music   /local/music  nfs4   rsize=8192,wsize=8192,timeo=14,_netdev 0 0_
         + nfsTypes are _nfs_ (for nfs2,3) and _nfs4_
         + rsize :: the number of bytes used when reading from the server
         + wsize :: the number of bytes used when writing to the server
         + timeo :: the amount of time, in tenths of a second,
           to wait before resending a transmission after an RPC timeout
         + _netdev :: wait until the network is up before
           trying to mount the share. systemd assumes this for NFS,
           but anyway it is good practice to use it for all types of
           networked file systems

    - Partitioning storage devices
    - Troubleshooting filesystem issues
    - *Linux Filesystem Tree Layout* (LFS201 topic)
      - Linux Filesystem Hierarchy
        + /bin
        + /sbin
        + /lib
        + /usr/bin
        + /usr/sbin
        + /usr/lib

    - *Linux Filesystems and the VFS* (LFS201 topic)
      - Virtual Filesystems
        + /proc
        + /sysfs

    - *Encrypting Disks with LUKS* (LFS201 topic)
      + create a LUKS partition
        1. _cryptsetup luksFormat /dev/sdaX_
        2. _cryptsetup open --type luks /dev/sdaX LUKSname_
** Local system administration
    - Creating backups
t      + tar cvf(z)
    - Restoring backed up data
      + tar xvf(z)
    - Creating local user groups
      + groupadd groupName
    - Managing user accounts
      + user management
        + cat /etc/group
        + usermod :: modify user account
          + -a :: add to group
          + -G :: used with -a
      + group management
        + groupmod :: modify group definition
    - Managing file permissions and ownership
      + chown :: change file or dir owner
        + -R :: recursively change ownership in dir and sub-dirs
      + chmod :: change rwx permissions
    - Managing fstab entries
    - Managing local user accounts
    - Managing the startup process and related services
      + chkconfig
    - Managing user account attributes
      + chmod
    - Managing user processes
      + top (or htop)
      + ps
    - Setting file permissions and ownership
    - *System startup and shutdown* (LFS201 topic)
      - init
        + systemV
        + upstart
        + systemd
      - bootloader
        + GRUB
        + GRUB2
    - *Kernel Services and Configuration* (LFS201 topic)
    - *Kernel Modules* (LFS201 topic)
      + /proc/modules
      + lsmod
    - *Devices and udev* (LFS201 topic)
      + udevadm :: udev management tool
    - *Processes* (LFS201 topic)
    - *Signals* (LFS201 topic)
    - *System Monitoring* (LFS201 topic)
    - *Process Monitoring* (LFS201 topic)
    - *I/O monitoring and tuning* (LFS201 topic)
      + iotop
      + sar
    - *I/O Scheduling* (LFS201 topic)
    - *Memory: Monitoring Usage and Tuning* (LFS201 topic)
      + free
      + /proc/meminfo
    - *Pluggable Authentication Modules (PAM)* (LFS201 topic)
    - *Network Addresses* (LFS201 topic)
      + ip addr
      + ifconfig
    - *Network Devices and Configuration* (LFS201 topic)
    - *Basic Troubleshooting* (LFS201 topic)
    - *System Rescue* (LFS201 topic)
      - init 1, single-user mode
      - LiveCD boot, chroot
** Local Security
    - Accessing the root account
    - Using sudo to manage access to the root account
      + _visudo_ for editing /etc/sudoers
      + usermod -a -G wheel userName (RHEL, Arch)
        _usermod -a -G sudo userName_ (Ubuntu/Debian)
    - *Linux Security Modules* (LFS201 topic)
      + Ubuntu
        + apt-get install pkgname
        + apt-get update
        + apt-get dist-upgrade
        + apt-get remove pkgname
** Shell scripting
    - Basic Shell Scripting
** Software Management
    - Installing software packages
      + RPM
      + DPKG
      + yum
      + APT
