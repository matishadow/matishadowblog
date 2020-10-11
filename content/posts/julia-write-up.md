---
title: "RE for Beginners Julia - write-up"
date: 2019-01-27T18:41:58+01:00
draft: false
comments: true
---

## Write-up intro
Let's make a write-up of the easiest of the easiest reverse engineering challenges (not even entry [CTF](https://ctftime.org/ctf-wtf/) level).  

I will show you how I, as a total beginner, approached it and how I made it through.

For the reversing tool I will be using [Binary Ninja](https://binary.ninja/) ($149) since it's much more affordable than [IDA Pro](https://www.hex-rays.com/products/ida/) ($589).

## Goal
The goal as with many other crackme challenges is to find the correct password.

[Link to the exercise](https://www.begin.re/julia)

## Reversing

First I checked the downloaded executable. 
> E:\tmp\re\julia>ls
> Exer2_Juila.exe

> E:\tmp\re\julia>file Exer2_Juila.exe 
> Exer2_Juila.exe; PE32 executable for MS Windows (console) Intel 80386 32-bit

As expected it is a standard 32-bit Windows executable.

Let's run it then.

> E:\tmp\re\julia>Exer2_Juila.exe
> Please provide the password.

> E:\tmp\re\julia>Exer2_Juila.exe AAAAA
> Incorrect Password.

Now open it in Binary Ninja and look for the two strings which we've seen.

![](https://matishadow.files.wordpress.com/2019/01/strings.png)

After executing the _Linear Sweep_ we get lots of functions, and we need to get to know which could be the main() function.

![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_14-27-27.png)

To do so we search for the reference of one of the previously found strings.

![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_14-32-18.png)

It has been found in _sub\_401040_ function, so I'll rename it to main().

The first check is being made against _arg1_ and in case of it not being equal to 2 we branch to code responsible for printing _Please provide the password._ string.

![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_14-44-06.png)

Now we're pretty sure it in fact is the main function and therefore we can rename _arg1_ to _argc_ and _arg2_ to _argv_.

This is how the other branch looks like.
![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_14-51-38.png)

The first block of code calculates length of the password provided as an argument.
Then it allocates a buffer of the same length (including null byte) as the provided password.
![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_15-35-34.png)

In case of malloc returning a null pointer (failed at allocating the bytes) the program exists.
We do now care about this part.

The next block copies the provided password to the allocated buffer.
![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_15-43-54.png)

Here we come to the interesting part of the function.
![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_15-46-35.png)

We can clearly see a loop there. The loop iterates over the allocated buffer until it reaches a null byte. 
During each pass it calls _sub\_401160_ function with one byte of data. 
Now let's look what's inside of this function.

![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_16-31-33.png)

Function presented above checks if the provided byte is a lower-case or an upper-case letter. 
If it is then data located at _data\_403008_ is added to the byte.
At this address there is a constant which is equal to 4.

Coming back to our main function.
![](https://matishadow.files.wordpress.com/2019/01/binaryninja_2019-01-27_16-36-24.png)

After the loop is finished the first 11 characters of the modified buffer are compared against "VIMwXliFiwx" sting. 
That means that to decode it we need to subtract 4 from each of these characters.

Let's write one line of Python (3) to decode it.
> C:\Users\matishadow>ipython -c " for char in 'VIMwXliFiwx': print(chr(ord(char) - 4), end = '')
> REIsTheBest

The password looks pretty legit but we can check it on the [website](https://www.begin.re/julia).
![](https://matishadow.files.wordpress.com/2019/01/chrome_2019-01-27_16-45-29.png)

This is indeed correct!


## Summary

Even though the encoding of the password was close to none the whole process showed how to quickly get to the interesting part of the binary.