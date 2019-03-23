---
title:  "Exploit Development: Exception Handlers and Egg Hunters!"
date:   2019-03-22
categories: posts
tags: [posts]
excerpt: "Introduction to SEH exploits and utilizing egg hunters to achieve code execution."
---
Introduction:
---
Error handling has become a vitale element within the embodiment of software development. Far are the days of just scraping by with only getting applications to function. Developers need to be implement graceful exiting of a program in case of any type of error. As old adage states, "No good deed goes
unpunished." - the same applies to exception handlers. This blog post will give a brief introduction to exception handler exploits
along with how we can leverage a technique known as an egg hunter to smuggle shellcode when there is not enough room on our stack pointer.

Exception Handlers: How Do They work?
---
Briefly I will hit on how these work (in Windows at least). Let's get acquainted some acronyms firstly. nSEH stands for next structured exception handler (we will get to why here in a second). SEH  (structured exception handler) is the current exception handler. There are two types of handlers- operating system and developer implemented handlers. As we know, an application executed on a machine that utilizes a stack based data structure will give each function/piece of the application its own stack frame to be put onto the stack for execution. The stack frame of each function/piece will eventually be popped of when execution has commenced. The same is true for an exception handler. If there is a function that has an exception handler embedded in itself- that exception handler will also get its own stack frame. Refer to the names of our acronyms above. Notice anything? It looks like you could have a list of exception handlers. If you thought this, then you are correct! The handlers form a data structure known as linked list. You can conceputalize a linked-list as a set of data that all point to each other (at a high level). Exception handlers need to be able to point to the nSEH (next handler) and the current handler (SEH). Here is what the excpetion handlers work when they encounter an error- when an exception occurs, a majority of the registers (ESI, EAX, ECX, etc. etc.) are "zeroed out" - therefore removing the data an advesary could implement. While this sounds like a tried and true way to prevent things like stack based buffer overflows- this nomenclature is not without its flaws. We will get to how we use this newfound knowledge to obtain a shell later on. Before we move on, take one note of a crucial attribute of SEH at the time an exception is raised. SEH will be located at `esp+8`. This means the location of ESP plus 8 bytes will show the location of SEH.

Egg Hunters: Woah, Wait- Easter Is Here?
---
Let me preface this paragraph by saying egg hunters are not making an inference to the Easter bunny. At a high leve, egg hunters are a small piece of shellcode (around 32 bytes in most cases) that will be embedded with what we call as a tag. This same tag will also be appended two times in front of a larger piece of secondary shellcode, like a reverse shell that is somewhere else in memory where an exploit developer can write to, uninterrupted. The egg hunter will recursively search memory for two instances of it's tag and will determine from there, that this is the piece of opcode you want to execute. After the second stage shellcode is located, a prompt execution of that opcode will take place. The word that has gained the most notoriety as a tag over the years, is `w00t`. Egg hunters are a pretty clever way to circumvent constraints individuals may come accross when trying to smuggle shellcode. Here is a well written white paper on egg hunters: [Egg Hunters](https://www.exploit-db.com/docs/english/18482-egg-hunter---a-twist-in-buffer-overflow.pdf) Let's move on :)

Down to the Nitty-Gritty
---
We will be taking crack at the [Vulnserver](https://github.com/stephenbradshaw/vulnserver). For reference and continuity, fuzzing will be out of the scope of this post. Let's begin.

