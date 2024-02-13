# Writeup, Operation Orchid, picoCTF

[Link to challenge](https://play.picoctf.org/practice/challenge/285)

The challenge asks us to download a disk image, `disk.flag.img.gz`, and find the flag.

## Preparing the image

Download the image.

```
$ wget https://artifacts.picoctf.net/c/213/disk.flag.img.gz
```

Extract the files from the gzip

```
$ gunzip disk.flag.img.gz
```

## Find the flag within the image

Take a look at the partition table to get a sense of what we're working with

```
$ mmls disk.flag.img

DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000411647   0000204800   Linux Swap / Solaris x86 (0x82)
004:  000:002   0000411648   0000819199   0000407552   Linux (0x83)
```

Partition 004 is the biggest, so let's start there.  
Copy the sector offset and see what's inside.

```
$ fls -o 0000411648 disk.flag.img

d/d 460:	home
d/d 11:	lost+found
d/d 12:	boot
d/d 13:	etc
d/d 81:	proc
d/d 82:	dev
d/d 83:	tmp
d/d 84:	lib
d/d 87:	var
d/d 96:	usr
d/d 106:	bin
d/d 120:	sbin
d/d 466:	media
d/d 470:	mnt
d/d 471:	opt
d/d 472:	root
d/d 473:	run
d/d 475:	srv
d/d 476:	sys
d/d 2041:	swap
V/V 51001:	$OrphanFiles
```

Now search for the flag. I just guessed it would be in the root directory, and was correct.  
You can specify the directory you want to list by appending it's inode (472 in this case) to the end of your command.

```
$ fls -o 0000411648 disk.flag.img 472

r/r 1875:	.ash_history
r/r * 1876(realloc):	flag.txt
r/r 1782:	flag.txt.enc
```

An alternitive method would have been to search recursively assuming we can make an educated guess about the filename.

```
$ fls -r -p -o 0000411648 disk.flag.img | grep flag

r/r * 1876(realloc):	root/flag.txt
r/r 1782:	root/flag.txt.enc
```

Here `-r` will list all files and directories recursively, `-p` will give print the path to whatever we find, `-o` is probably unnecessary but limits our search to the single partition, and `| grep` is just us passing the resulting list into a grep search.  
This alternitive method confirms that our flag is in the root directory.

## Decoding the flag

We can see that the origional `flag.txt` had been reallocated. Additionally, `flag.txt.enc` has been encoded and it's output is useless to us.

```
$ icat -o 0000411648 disk.flag.img 1782

Salted__ÍﬁÅ—”e¿ïB’JËc—$QE&$¢á4jM·KGeE–1î˚^»§7˘ ≥ÀÒÿé$ƒ'%
```

Thankfully, we see the user has not disabled their `.ash_history` file. Maybe we can learn more from there.  
Note: `.ash_history`, like `.bash_history` or `.zsh_history`, is basically a log of the user's CLI history. It usually lives in the home directory.

```
$ icat -o 0000411648 disk.flag.img 1875

touch flag.txt
nano flag.txt 
apk get nano
apk --help
apk add nano
nano flag.txt 
openssl
openssl aes256 -salt -in flag.txt -out flag.txt.enc -k unbreakablepassword1234567
shred -u flag.txt
ls -al
halt
```

There it is! It looks like the challenge author used openssl to use an aes256 encryption AND we even have the key `unbreakablepassword1234567`.  
With this information, we should be able to use openssl ourselves to decrypt the flag.  
First we'll need to get the file onto our machine from the image.

```
$ icat -o 0000411648 disk.flag.img 1782 > flag.txt.enc
$ ls

disk.flag.img  flag.txt.enc
```

Now we just decrypt the file.  
Note: The decryption works here, but I get a warning. This is just a reminder that I'm still learning and there's probably a better way of doing almost everything covered in this walkthrough.

```
$ openssl aes256 -d -in flag.txt.enc -out flag.txt -k unbreakablepassword1234567

*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bad decrypt
C047DF5BF87F0000:error:1C800064:Provider routines:ossl_cipher_unpadblock:bad decrypt:providers/implementations/ciphers/ciphercommon_block.c:107:

$ ls
disk.flag.img  flag.txt  flag.txt.enc

$ cat flag.txt
picoCTF{redacted}%
```

Not sure why there was a % at the end of my flag, but it probably has something to do with my janky decryption.
