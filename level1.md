# Preparation
From https://io.netgarage.org/, we have:
- Username: level1
- Password: level1
- Ssh server: io.netgarage.org
# Start
After login successfully, we will have informed that our challenges will be in folder `/levels`
```bash
level1@io:/levels$ ls
beta           level05           level07_alt.c    level10_bis.pass  level15.pass   level18_alt    level25       level30
level01        level05.c         level08          level11           level16        level18_alt.c  level25.c     level30.c
level02        level05_alt       level08.cpp      level11.c         level16.c      level19        level26       level31
level02.c      level05_alt.c     level08_alt      level12           level16.pass   level19.c      level26.l     level31.asm
level02_alt    level06           level08_alt.cpp  level12.c         level16_alt    level20        level26.y     level32
level02_alt.c  level06.c         level09          level12.pass      level16_alt.c  level20.asm    level27
level03        level06_alt       level09.c        level13           level17        level20.pass   level27.c
level03.c      level06_alt.c     level10          level13.c         level17.c      level21        level27.pass
level04        level06_alt.pass  level10.c        level14           level17_alt    level22        level28
level04.c      level07           level10.pass     level14.c         level17_alt.c  level23        level28.c
level04_alt    level07.c         level10_bis      level15           level18        level23.c      level29
level04_alt.c  level07_alt       level10_bis.c    level15.c         level18.c      level24        level29.c
```

Run level01:
```bash
level1@io:/levels$ ./level01
Enter the 3 digit passcode to enter: 123
```

We need to guess 3 number to pass in the program (there's only 999 possibilities but ...)
![actually...](https://i.imgur.com/RUdPyQP.jpeg)

Instead, we can reverse engineering this binary and find the correct answer :v

The tool I will use is radare2:
```bash
level1@io:/levels$ r2 ./level01
Warning: Cannot initialize dynamic strings
 -- Switch between print modes using the 'p' and 'P' keys in visual mode
[0x08048080]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[0x08048080]> s main
[0x08048080]> pdf
            ;-- main:
            ;-- section..text:
            ;-- main:
            ;-- _start:
/ (fcn) entry0 31
|   entry0 ();
|           0x08048080      6828910408     push str.Enter_the_3_digit_passcode_to_enter:_Congrats_you_found_it__now_read_the_password_for_level2_from__home_level2_.pass_n_bin_sh ; loc.prompt1 ; "Enter the 3 digit passcode to enter: Congrats you found it, now read the password for level2 from /home/level2/.pass./bin/sh" @ 0x8049128 ; [1] va=0x08048080 pa=0x00000080 sz=31 vsz=31 rwx=--r-x .text
|           0x08048085      e885000000     call loc.puts
|           0x0804808a      e810000000     call loc.fscanf
|           0x0804808f      3d0f010000     cmp eax, 0x10f
|       ,=< 0x08048094      0f8442000000   je loc.YouWin
\       |   0x0804809a      e864000000     call loc.exit
```

We can see that our input will be passed into `eax` by `fscanf` then compare with `0x10f = 271` :)
```bash
Enter the 3 digit passcode to enter: 271
Congrats you found it, now read the password for level2 from /home/level2/.pass
```