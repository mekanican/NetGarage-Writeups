# Preparation
From **previous level**, we have:
- Username: level2
- Password: ----------------- :)
- Ssh server: io.netgarage.org
# Start
`/levels/level02.c`
```cpp
//a little fun brought to you by bla

#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void catcher(int a)
{
    setresuid(geteuid(),geteuid(),geteuid());
    printf("WIN!\n");
    system("/bin/sh");
    exit(0);
}

int main(int argc, char **argv)
{
    puts("source code is available in level02.c\n");

    if (argc != 3 || !atoi(argv[2]))
            return 1;
    signal(SIGFPE, catcher);
    return abs(atoi(argv[1])) / atoi(argv[2]);
}
```

The program will read your input in `argv[1]` and `argv[2]` then return the absolute value of the first one divided by second.
Our mission is to trigger `SIGFPE` to call the `catcher` function for us.
After searching GG, we have this answer on [Stack overflow](https://stackoverflow.com/questions/46378104/why-does-integer-division-by-1-negative-one-result-in-fpe/46378352#46378352)
So we need to pass `INT_MIN` and `-1` as argument. Why?
- Easiest way is to set divisor to 0 and trigger divide_by_zero exception but it's checked inside if statement
- `int` range in C vary in [$-2^{31}; 2^{31} - 1$]
- `INT_MIN` represents in 2's complement as `0b10000...0000`
- `INT_MIN` passed into `abs()` so it must be positive right? But `-INT_MIN > INT_MAX` so there's an undefined behavior as specified [here](https://www.cplusplus.com/reference/cstdlib/abs/). But what will it become? With 2's complement inverse, `0b10000...0000` -> `0b01111...1111` -> `0b10000...0000` = `INT_MIN` again... But this won't trigger `SIGFPE` because the operation inside `abs` can be optimized by compiler.
- If we divide the result above by `-1`, it's will act as above, but this time, the compiler cannot optimize this case and start to calculate a/-1 instead of -a as above -> Trigger `SIGFPE`

```bash
level2@io:/levels$ ./level02 -4294967296 -1
source code is available in level02.c

WIN!
sh-4.3$
```