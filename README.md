This is a filesystem structure built for the Linux operating system. This 
project was built in collaboration with Dan Shawlson.

DEVELOPMENT PROCESS:
In order to implement our file system, we broke down our process into 5 parts,
as per the guidelines given in class. The first "piece" of the filesystem 
implementation is the makefile. This file when run takes an allocated disk image 
and breaks it up into blocks of a predetermined size. It writes a superblock
used for storing some of the metadata of the filesystem as a whole, an inode 
bitmap that keeps track of which inodes are free, a data bitmap which does the 
same for data nodes, and the root inode that acts as the parent directory of all
the other inodes. We also wrote a quickfs.h file in order to store all the 
constants and sturctures to be used by both the makefile and main quickfs.c file. 
The second piece of the filesystem was creating a file system module that can be 
installed and uninstalled from the kernel using a user level command. This 
required creating the main quickfs.c file and creating a module_init method and a 
module_exit method that allow the user to add and remove the module from the 
kernel.  for the third piece, we created a file_system_type structure that would 
allow the code to be mounted and unmounted as a file system by the user. This 
also required implementing a somewhat trivial mount method as well as a 
fill_super method which fills the file system's superblock with the 
necessary field data. Part 4 of our implementation involved creating the 
directory methods. These consist of create which creates a new inode structure 
and the file associated with it, lookup which find an inode based on its name, 
link which created a new inode structure and points it to an existing inode 
structure as a hard link, unlink which removes a hard link disk inode and 
decreases the number of links to its vfs inode, and the readdir method which 
iterates through all of the disk inode structures and returns the metadata 
information about them. Finally for the fifth piece we put in place the file 
operations and address space operations. This required us to implement a get_block 
method which maps a requested block from the disk memory space to the VFS memory 
space and allocates new disk space if necessary.



MEMORY LAYOUT:
Our on-disk filesystem uses a series of inode blocks and data blocks in order to 
keep track of the files and all of their metadata. Each file can occupy multiple 
data blocks and will have one inode associated with it that contains all of its
metadata used in finding and accessing the file's content. The system also contains
two bitmaps associated with the inodes and data blocks that tell whether a given
block/inode is currently being used or if they are open for write. Unlike BFS and 
EXT2 filesystems, our inode list also doubles as the directory. This is done by 
having every inode contain both the inode number and the file name. What this means
is that instead of a a disk inode being strictly associated with a file's content, 
it will be associated with a file name and you will for instance have two quickfs inodes 
for a file and its hard link instead of one. 
	Given an implementation with block size of X bytes (set in the quickfs.h 
file constants) the filesystem will have a set number of inodes (and therefore max
files) equal to 8X due to the bitmap size. Depending on how large the image size 
(S) you allocate for the file system, you will be able to have S/X blocks, 7 of 
which will be reserved for the superblock, inode bitmap, data block bitmap (4 
blocks long), and root inode. The system sets aside 4 blocks for the data block 
bitmap so that we can support up to 32X data blocks maximum. However, the 
filesystem will successfully create so long as you have enough space left over 
for at least one data block. This means that for however large you set your block 
size, you must have at least 7+8X blocks minimum to format the image. For example, if you 
create an image with 20486 blocks of size 512B, then 7 blocks will be set aside 
for the reserved blocks, you will have 8*512=4096 inodes, one of which is the root, 
and have 20486-7-4095 = 16,384 data blocks set aside for files, or 8MB. Each inode
can contain a file with a max file size of 103 blocks, which in this case would be 
52736 bytes.
