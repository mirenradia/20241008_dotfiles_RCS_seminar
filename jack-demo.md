# Jack's Homebuilt git repo Demo


## Introduction

I will start on my existing work machine which already has things set up.
I will run `ls -lh` in my home directory to show the contents.
Note the things that we want to track as dotfiles, but also several things we do not.


## Branch changes on one machine

One use that is not so obvious is using branches to easily switch between different
profiles on the same machine.

One example I have is for when I am teaching.
In this case I want a bright terminal profile with large text that can be seen by a class.
I also don't want all of my 'extras', but instead need to keep it simple and familiar to
what students are likely to have.

Rarther than switching off various things in my profiles I just have a different branch
that allows me to immediately switch over for a class, and then back when I return to
work.

I will demo this by running `dotfiles checkout carpentries` and you will see the
terminal profile change.


## Fresh Setup

I will run this demo in a virtual machine (VM).
For my purposes I am using UTM on mac, with a Debian image, but this is generic.

I have set up an ssh key and linked it to my github account.

Start by observing the terminal profile and looking at vim.

Now we will bring my dotfiles over by running the curl command from the documentation.

Now we will re-source the `.bashrc`.
See how my terminal prompt has changed, and if we boot vim you can see I now have
line-numbers and some other customisations enabled.


## Making changes and storing

If I want to modify a file, I can create a branch for this machine, add, and commit
the file, then push it to a remote.

To do this I will remove the fuzzy-menu in my vim config which we can see is causing
issues on this new machine.

```
dotfiles checkout -b debian-vm
dotfiles add -u
dotfiles commit -m "Remove fuzzy-menu from vim config on debian VM"
dotfiles push origin debian-vm
```

We can see online that there is now a branch with my updated profile.
