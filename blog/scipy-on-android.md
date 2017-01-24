---
title: Running Jupyter and the Scipy stack on Android
date: 2017-01-23
thumbnail: scipy-on-android.png
layout: post
---


# TL;DR

Install Termux from Google Play, open it and run:

    $ apt install clang python python-dev fftw libzmq libzmq-dev
    $ LDFLAGS=" -lm -lcompiler_rt" pip install numpy matplotlib pandas jupyter
    $ jupyter notebook

Copy the URL printed to the screen and paste it into Chrome/Firefox. Enjoy!

Read on for more tips and a few tweaks.


# Some background

I bought my first tablet last October, an
[NVIDIA Shield K1](https://www.amazon.com/NVIDIA-SHIELD-K1-Tablet-Black/dp/B0171BS9CG/ref=sr_1_2?s=pc&ie=UTF8&qid=1484937529&sr=1-2&keywords=nvidia+shield+k1).
I had been putting off getting one because I never could think of a good use
for them.
I have my phone for messaging and Internet, my kindle for reading, and my
Linux laptop for working.
It seemed to me that the tablet would be a nice toy but not something I could
use and justify the purchase.

The dream would be to be able to ditch my laptop and do actual work on the
tablet.
Mark O'Connor wrote about doing just that on
[Yield
Thought](http://yieldthought.com/post/12239282034/swapped-my-macbook-for-an-ipad)
but he cheats a bit by running everything on a Linode server.
And how does anyone do scientific programming these days without a Jupyter
notebook?

I finally gave in, thinking that I would use the tablet mainly for reading
papers and taking notes.
Maybe even play a few games.
Then I discovered [Termux](https://termux.com/).


# Show, don't tell

Here is a screencast of me running a Jupyter notebook
server on my tablet.
Notice that the URL is `localhost:8888/` so this is not a remote server.

![Screencast of Termux running the Jupyter notebook on my Shield K1 tablet](/images/termux-running-jupyter.gif)


# Setting up your terminal

Install Termux from Google Play and open it.
You'll be dropped into a bash terminal, like the one below.

![Blank Termux startup screen](/images/termux-blank.png)

Termux uses the `apt` package manager so you can install packages pretty much
like you would on Debian/Ubuntu.

The first thing I do on any new computer is install git so that I can fetch my
[configuration files from Github](https://github.com/leouieda/dotfiles):

    $ apt install git

Before cloning the repository, I need to generate a new SSH key (only required
if you [use the SSH protocol with git](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)):

    $ apt install openssh
    $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    $ cat .ssh/id_rsa.pub  # copy and paste your public key to Github

Then I can clone my [dotfiles](https://github.com/leouieda/dotfiles)
repository:

    $ git clone git@github.com:leouieda/dotfiles.git

Now my Termux terminal looks just like my Linux terminal on my laptops.

![Cloning git repository with my config files](/images/termux-git.png)

![My Linux terminal to compare with termux](/images/termux-linux-terminal.png)


# Installing the Scipy stack

If you're from the pre-Anaconda era, you'll probably remember the frustration
of trying to `pip install numpy scipy matplotlib`.
Sadly, there is no Anaconda for Termux so we're stuck with using the system
python and `pip` to install packages.

But don't despair!
Things work more smoothly these days (if you follow the
[magic incantations](https://github.com/termux/termux-packages/issues/136)):


    $ # Install the required packages from Termux
    $ apt install clang python python-dev fftw libzmq libzmq-dev
    $ # Add some magic works to tell pip how to compile the C extensions
    $ LDFLAGS=" -lm -lcompiler_rt" pip install numpy matplotlib pandas jupyter

**UPDATE (24-01-2017):** I removed the `scipy` package from the above `pip
install` because it actually doesn't work. I should have tested the commands
better, sorry. It seems like it [will be a while before we get
scipy](https://github.com/termux/termux-packages/issues/471) because of all the
binary requirements (BLAS, LAPACK, and gfortran).

**UPDATE (24-01-2017), the return:** Also forgot to add `libzmq` and the
corresponding headers to the `apt install` list, as required by Jupyter.

Now you have access to things like `ipython` on the command-line:

![IPython running inside Termux](/images/termux-ipython-numpy.png)

One thing that won't work are matplotlib plots because there is no backend for
Android.
You can, however, use `%matplotlib inline` or `%matplotlib notebook` inside
Jupyter notebooks to get the plots working.

To get a Jupyter notebook server running, so the same thing you would on any
other computer:

    $ jupyter notebook

The server won't automatically open a browser but
you can copy the URL from the output and paste it into Chrome or Firefox.

![Jupyter notebook server running inside Termux](/images/termux-jupyter-startup.png)


# Getting comfortable

While it is possible to do some work using this setup (I wrote part of this
post on the tablet using Vim and pushing to the [Github
repo](https://github.com/leouieda/website)), it may not be the most productive
environment.
Here are a few tips for making life a little bit easier.

* Get a bluetooth keyboard. I bought the
  [Logitech 920-003390](https://www.amazon.com/Logitech-920-003390-Tablet-Keyboard-Android/dp/B0054L8N7M/ref=sr_1_15?s=pc&ie=UTF8&qid=1476900899&sr=1-15&keywords=Android+keyboard).
  It's not great but much better than a touch screen.
* If you want to use the touch screen, you'll need the
  [Hacker's Keyboard app](https://play.google.com/store/apps/details?id=org.pocketworkstation.pckeyboard&hl=en)
  to execute your code cells with Shift+Enter and not go crazy.
* [Remap Esc to anything else](http://vim.wikia.com/wiki/Avoid_the_escape_key)
  when using Vim. Esc shows the homescreen on Android and is a very frustrating
  habit to loose.

![Sreenshot of vim running inside termux writing this post,
inception style.](/images/termux-vim.png)


# Things that are still missing

This setup works and is way beyond what I expected to be able to accomplish
with a $200 tablet.
However, going back to pip installing numpy feels a bit like I'm back in 2010.
What I've missed the most is [Anaconda](http://continuum.io/downloads#all)
and the [conda package manager](http://conda.pydata.org/docs/).
Having a prebuilt bundle certainly makes life a lot easier.
But I miss the conda environments the most.
I use them extensively for my projects and papers.

**UPDATE (24-01-2017):** See the update above about the scipy package. So
yeah, that is still missing as well. A lot of things can be done using
numpy replacements (`numpy.fft` instead of `scipy.fftpack` etc) though they are
usually slower.

Another recent arrival that has made a huge impact on my daily work is [conda-forge](https://conda-forge.github.io/).
This project greatly democratizes conda packages.
Now anyone can build their own packages for Linux, Windows, and Mac.
It would be awesome to have some for Android as well.
Assuming that you can get conda installed, the major difficulty might
be finding a continuous integration service that runs Android and setting up
the infrastructure.


**Let me know if you try this out! Is there another setup that you use?  What
else is missing?  Do you think we'll be able to fully work like this one day?**


---

*The Jupyter logo was downloaded from their Github repository
([jupyter/design](https://github.com/jupyter/design)).
The Android logo is [CC BY 2.5](http://creativecommons.org/licenses/by/2.5)
Google Inc.,
[via Wikimedia Commons](https://commons.wikimedia.org/wiki/File%3AAndroid_robot.svg).*