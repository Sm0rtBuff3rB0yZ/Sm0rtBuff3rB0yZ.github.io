# Reverse Me (hard) [??really??]

## Challenge

Reverse the given binary to retrieve username and password in order to obtain the flag. `VU{username:password}`

## Solution

We were given a binary `Task-ReverseME-hard`, we open it inside Ghidra and quickly find the value for username and password in main function:

![alt text](images/re-hard.png?raw=true)

We convert hex to dec and run the binary:
```
iStack_18 = 0x7e2; // -> 2018 (username)
iStack_14 = 0xc56; // -> 3158 (password)
```
```
s4tb0y@s4tb0y ~/C/c/R/re_hard> ./Task-ReverseME-hard 
Enter login:2018
Enter password:3158

Congrats! The correct CTF answer is a password.
```

Thereore the flag is:

`VU{2018:3158}`
