# POC for meltdown/spectre

<b>I can't take all or even most of the credit. gkaindl, romansavrulin, and gregvish gave me all the pieces, I just put them together.</b>

## This is an intentionally outdated repository. My latest code has not been published due to concern for the embargo on the vulnerabilities. 

## Be aware! This will only work on certain CPU types. This vulnerability is heavily dependent on your CPU architecture, if this doesn't work for you, please try and fix it on your own.

This code will allow you to read kernel memory from a non-priviledged user account. It is not arbitrary, this information must be loaded into the L1 cache as described in the [Project Zero Blog Post](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html)  

## How to compile
```gcc meltdown.c```  
You can run it with ./a.out

## Proof
The following are screenshots showing how this all works. (This was done on an Ubuntu 17.10 system)  

First, let me show you what you should normally see from /proc/kallsyms as an unpriviledged user

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-1.png />
</p>

As you can see all the memory locations are 0000000000000000, this is because unpriviledged users arent supposed to see this. As an example, what happens when we run this with sudo?

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-2.png />
</p>

Now we have memory addresses! This is to establish the fact that this information is private and should only be shown to users with a certain level of priviledge.

Now, with that out of the way, let's run the exploit and see what we get.

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-3.png />
</p>

Did we get anything good? No, we didn't. (I should note that if you run the old exploit multiple times, these values will change, which is an indication that we are grabbing memory we are not allowed to see.) Which is a shame. Why is that? Well take a look at Project Zero's blog post, particularly variant 3.

"A PoC for variant 3 that, when running with normal user privileges, can read kernel memory on the Intel Haswell Xeon CPU under some precondition. We believe that this precondition is that the targeted kernel memory is present in the L1D cache."

So how do we get that into cache? well we give it a read command that is sure to fail and we get the following.

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-4.png />
</p>

Notice anything different about this one? (This output for the memory addresses will always be the same, until you reboot or do something catastrophic. An indication that we are successfully able to read it) Check out that first line, particularly the first set of hex. If we play that backwards (because of the Endianess), we get ffffffffb68525e0. So what? Well that's actually kind of cool. Why? Let's go back that that /proc/kallsyms section I told you was so important. Remember, I got the hex value as an unpriviledged user. What does that hex value correspond to?

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-5.png />
</p>

Bingo, we just got the address of something in kernel space memory as an unprivledged user. As a reminder, here is what happens if you look for that same thing without priviledge on the CLI.

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-6.png />
</p>

As you can see, a normal unpriviledged user wouldn't be able to identify where that construct is, but with the exploit we can find it. 

What can you do with this? I don't know. As of right now, it just seems to be a game of figuring out how to force that memory into the L1 cache and pulling it from there. If I could cobble this together in 2 days, I'm sure much more experience people could pull off even more.

The following is the CPU I tested this exploit on.

<p align="center">
  <img src=https://github.com/frichetten/meltdown-spectre-poc/raw/master/images/z-7.png />
</p>
