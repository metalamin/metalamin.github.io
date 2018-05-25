---
title: "Recover Perl from an EXE"
excerpt: "How to recover Perl packaged in EXE with PerlApp from ActiveState"
header:
  teaser: "/assets/images/Perl-from-EXE/teaser.png"
tags:
  - ES
  - OllyDbg
  - Perl
  - Reversing
gallery1:
          - url: /assets/images/Perl-from-EXE/referenced.png
            image_path: /assets/images/Perl-from-EXE/referenced.png
            alt: "Referenced strings"
            title: "Referenced strings"
          - url: /assets/images/Perl-from-EXE/strings_2.png
            image_path: /assets/images/Perl-from-EXE/strings_2.png
            alt: "Strings"
            title: "Strings"
---
Some time ago, during one of my Red Team exercises, I came across a Perl in the form of a Windows executable. This one is used to perform a series of operations against the domain and in order to authenticate it seems to generate credentials in memory from an encrypted key file. How can I get those credentials back?

One way might have been to mount a fake [Responder](https://github.com/SpiderLabs/Responder)-style domain controller. But... Why make it easy? Also, I was curious to find out what encryption method is used, in case I would find something similar later on during the pentest.

## Executable analysis
A brief analysis with strings shows us that it is a Perl script packed in an EXE executable. In particular, it appears that **ActiveState PerlApp** was used for this purpose.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/strings.png" alt="">
  <figcaption style="text-align: center">strings decode.exe | grep -i perl</figcaption>
</figure> 

So far all the Perl to EXE packers I have found have to save the script clearly before they can launch the interpreter. In some cases they are stored in a file and in others they remain in memory. In the sample we're interested in, he unpacks it in memory. Let's look at a way to retrieve it using [OllyDbg](http://www.ollydbg.de/).

## Recovering the Perl
We load the executable into OllyDbg. But before launching the execution we will put a breakpoint somewhere after the script is loaded.
To do this we first show the text strings.

{% include gallery id="gallery1" %}


We know that after unpacking the script, the executable has to mount the string for the Perl interpreter call. Let's look for strings that look like arguments for launching the Perl interpreter and stop the execution at that point. We choose some of the chains and set the breakpoint by pressing **F2**.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/strings_4.png" alt="">
  <figcaption style="text-align: center">Parameters srings</figcaption>
</figure>

We are now ready to start the program by pressing **F9**.

<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/stop.png" alt="">
  <figcaption style="text-align: center">Stopped execution/figcaption>
</figure>

Once we have stopped at our breakpoint, we open the Memory Map by pressing **"Alt+M"**.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/memory.png" alt="">
  <figcaption style="text-align: center">Memory Map</figcaption>
</figure>

With the memory window open we do a search (**Ctrl+B**) for strings that can contain the script we are looking for. For example, calls to perl libraries that begin with **"use"** (note the space at the end).
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/busqueda.png" alt="">
  <figcaption style="text-align: center">String search</figcaption>
</figure>

We already got the script in memory and we know where it is. All we have to do is save this memory segment in a file and store the script.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/backup.png" alt="">
  <figcaption style="text-align: center">Backup</figcaption>
</figure>

And with this we have recovered the entire script including the programmer's comments.
<figure class="align-center">
  <img class="align-center" style="width: auto" src="{{ site.url }}{{ site.baseurl }}/assets/images/Perl-from-EXE/recovered.png" alt="">
  <figcaption style="text-align: center">Script Perl recovered</figcaption>
</figure>

Surely we could have done it more elegantly or more quickly, but this is the one I thought of at the time and wanted to share.
