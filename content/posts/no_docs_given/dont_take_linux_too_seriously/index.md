+++
title = "Don't take linux too seriouly, you'll never get out of vim"
date = "2026-05-05T19:56:24-04:00"
author = "Anand Siva"
cover = "linux_too_seriously_cover.png"

tags = ["linux", "terminal", "cli", "fun", "devops"]
categories = ["devops"]
publications = ["No Docs Given"]

keywords = [
  "fun linux commands",
  "weird linux commands",
  "cmatrix terminal",
  "asciiquarium terminal",
  "sl command linux",
  "hollywood terminal"
]

description = "A fun tour through weird and entertaining terminal commands that make Linux and the command line feel a little less serious."

showFullContent = false
readingTime = true
hideComments = false
+++

One of my favorite quotes I love is "Don't take life too seriously, you'll never get out alive". What a fun saying that I really try to take to heart. Listen life has many ups and downs along the way. There are super high moments such as marriage, kids etc. But life also comes with some very heart breaking moments as well. The rest of it, well the rest of it too me just feels like a grind. A lot of routine. Wake up, get dressed, type on a computer for hours a day, come home, sleep, do it again the next day. Don't get me wrong I love my job and the work I do there, but there are only so many times I can type out `ls -ltr` 100x a day before I yearn to spice up the day. 

That is why I bought a cartoon cat backpack, wear star wars hawaain shirts, and overall love to laugh while I am at work. That moment of laughter or a joke between your co-workers can really help boost my mode and turn a crappy day into a good one. Just a few words can make a big difference. 

When I was looking at a subject for my blog today, I started writing about AWS quick start guide, then I stopped because I was so burnt out about talking about cloud stuff after a long day of work. Then I decided I wanted to have some fun with this blog and get a laugh or two when I am writing it. I decided to go for strange and fun linux commands. Now a caveat, these don't come installed with linux servers usually, they will need to be installed. I am going to be using my mac and brew install it, but I am sure there are rpm packages and such for the other linux distros also.

Since this article started out with a quote let's look at fortune first!! This command literally does what you think it does, gives you a random fortune.

```
brew install fortune

󰄛   fortune
Fess:   Well, you must admit there is something innately humorous about
        a man chasing an invention of his own halfway across the galaxy.
Rod:    Oh yeah, it's a million yuks, sure.  But after all, isn't that the
        basic difference between robots and humans?
Fess:   What, the ability to form imaginary constructs?
Rod:    No, the ability to get hung up on them.
                -- Christopher Stasheff, "The Warlock in Spite of Himself"

 󰄛   fortune
You can fool all the people all of the time if the advertising is right
and the budget is big enough.
                -- Joseph E. Levine
```

You know what would be better then just seeing a fortune on the terminal? Seeing a fortune in the terminal being said by a cow!

```
brew install cowsay

 󰄛   fortune | cowsay
 ______________________________________
/ "Not Hercules could have knock'd out \
| his brains, for he had none."        |
|                                      |
\ -- Shakespeare                       /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Now liking the cow, we can always go dragon!!

```
 󰄛   fortune | cowsay -f dragon
 _________________________________________
/ "Benson, you are so free of the ravages \
| of intelligence"                        |
|                                         |
\ -- Time Bandits                         /
 -----------------------------------------
      \                    / \  //\
       \    |\___/|      /   \//  \\
            /0  0  \__  /    //  | \ \
           /     /  \/_/    //   |  \  \
           @_^_@'/   \/_   //    |   \   \
           //_^_/     \/_ //     |    \    \
        ( //) |        \///      |     \     \
      ( / /) _|_ /   )  //       |      \     _\
    ( // /) '/,_ _ _/  ( ; -.    |    _ _\.-~        .-~~~^-.
  (( / / )) ,-{        _      `-.|.-~-.           .~         `.
 (( // / ))  '/\      /                 ~-. _ .-~      .-~^-.  \
 (( /// ))      `.   {            }                   /      \  \
  (( / ))     .----~-.\        \-'                 .~         \  `. \^-.
             ///.----..>        \             _ -~             `.  ^-`  ^-_
               ///-._ _ _ _ _ _ _}^ - - - - ~                     ~-- ,.-~
                                                                  /.-~
```

Next time someone needs an output of a command, just pipe it to cowsay first before you send it to them. I am sure they would love it.

```
 __________
< rf -rf / >
 ----------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Want to look like you are hacking a main frame like you are "The One", let's hit it!!!

You can't see it here but it is raining down on my terminal!!!!

{{< image src="/posts/no_docs_given/dont_take_linux_too_seriously/matrix.gif" style="width:100%; max-width:100%;" >}}

Not into scifi, I got you.

{{< image src="/posts/no_docs_given/dont_take_linux_too_seriously/aquarium.gif" style="width:100%; max-width:100%;" >}}

Now those are just fun ascii images and such. But what about that pesky ls command I was talking about before.

What if you accidently wrote it backwards?!

```
brew install sl

sl
```

{{< image src="/posts/no_docs_given/dont_take_linux_too_seriously/sl.gif" style="width:100%; max-width:100%;" >}}

You know how sometimes you need to get a bunch of info about a server you logged into? What do you do? run uname, df -h, lscpu.

Nah lets use fastfetch

{{< image src="/posts/no_docs_given/dont_take_linux_too_seriously/fastfetch.png" style="width:100%; max-width:100%;" >}}

Trying to get diskspace? what you going to do df -h? du -skh? ehh no boorringgg

```
󰄛   brew install ncdu

ncdu

--- /Users/amoney/anandsiva.com ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
   26.6 MiB [############################] /.git
   25.6 MiB [##########################  ] /public
   18.8 MiB [###################         ] /content
    1.5 MiB [#                           ] /themes
    1.0 MiB [#                           ] /static
    4.0 KiB [                            ] /assets
    4.0 KiB [                            ] /layouts
    4.0 KiB [                            ]  hugo.toml
    4.0 KiB [                            ]  .gitignore
    4.0 KiB [                            ] /archetypes
    4.0 KiB [                            ]  .gitmodules
e   0.0   B [                            ] /i18n
e   0.0   B [                            ] /data
    0.0   B [                            ]  .hugo_build.lock
```

Your boss about to walk by and you want to look like you are working, I got you!

```
docker run -it --rm cgr.dev/chainguard/hollywood
```

{{< image src="/posts/no_docs_given/dont_take_linux_too_seriously/hollywood.gif" style="width:100%; max-width:100%;" >}}

The most important command to learn is to chill !!!

I hope you have fun trying some of these out yourselves and maybe on a gloomy day it will put a smile on your face.
