# pythonbufferoverflow

This is a very simple python which also happens to be used more than simple text output. this is used for a stackoverflow exploit
in exploit exercises' stack0 challenge. The object of the exercise is to change a varible. This tutorial will use requre knowledge of programming and a very basic idea of how assembly works.The c code is as follows 
```
int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```
This uses a buffer which writes to write to the stack which also happens to be where local varibles are stored. 
This allows the varible to be changed.The magic happens in the assembly 
```
0x080483f4 <main+0>:	push   ebp
0x080483f5 <main+1>:	mov    ebp,esp
0x080483f7 <main+3>:	and    esp,0xfffffff0
0x080483fa <main+6>:	sub    esp,0x60
0x080483fd <main+9>:	mov    DWORD PTR [esp+0x5c],0x0
0x08048405 <main+17>:	lea    eax,[esp+0x1c]
0x08048409 <main+21>:	mov    DWORD PTR [esp],eax
0x0804840c <main+24>:	call   0x804830c <gets@plt>
0x08048411 <main+29>:	mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:	test   eax,eax
0x08048417 <main+35>:	je     0x8048427 <main+51>
0x08048419 <main+37>:	mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:	call   0x804832c <puts@plt>
0x08048425 <main+49>:	jmp    0x8048433 <main+63>
0x08048427 <main+51>:	mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:	call   0x804832c <puts@plt>
0x08048433 <main+63>:	leave  
0x08048434 <main+64>:	ret   
```

So first we look at how the stack is made, not all of it is important but we will see how varibles are stored on the stack and how the stackframe is built.
let's look at how the stack is made and it's alot like lifting up a car. First in assembly EBP, and ESP point to a memory adress in the stack. Push EBP loads the base pointer into the stack. So our stack is like a deck of cards which can only be drawn from the top. 
So first the EBP is now on the stack as it is pushed onto the stack. Then the mov instruction will set the base pointer and the stack pointer (for the c program). Now the base of the stack is the also the place where the stack pointer is. 
The next statement just is an anding operation does a and operation. The next operation moves the stack pointer 0x60 away giving the amount of space allocated by the buffer (64 charaters) since the memory adresses differ by 0x60. 
Then the varible that is to be changed is the varible set to zero. We see it since there is a move 0 varibles into a varible 
into exp offset 0x5c which is likely the varible (there's only 1 ). then the next few steps calls gets and loads up the buffer. Then the buffer is able to write into the stack this is how the exploit works. Note the mov and test eax eax instructions this is how we enter the if condition. if We load enough charaters into the buffer we sould overwrite the stack and too much will cause a segmentation fault. If we just enought into the stack so that it loads a stack value that is not zero since we overwritten it should hit the "You changed the varible message." 

