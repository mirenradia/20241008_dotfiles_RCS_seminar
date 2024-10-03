# Chezmoi demo

## Setup

For the purposes of the demo, I will use a [docker](https://www.docker.com/)
container. Since I already had it lying around, I used the
[catthehacker/ubuntu:full-24.04](https://github.com/catthehacker/docker_images/pkgs/container/ubuntu/277469423?tag=full-24.04)
image which is based on the images used for Ubuntu 24.04 GitHub action runners.
This demo should work on any Ubuntu image which has the follow prerequisite
packages (can be installed in the container after starting it) above a minimal
installation:
* `git`
* `curl`
* `python3`
* `python3-pip`
* `sudo`
* `pipx`

There may be other packages I have missed from the list above but the above
image has all the prerequisites installed. It can be obtained with the
```bash
docker pull ghcr.io/catthehacker/ubuntu:full-24.04
```
> [!WARNING]
> This image is very large (~20 GB download and ~60 GB when uncompressed) so it
> will probably take a while to download and extract.


Now start the container:
```bash
docker run -it --name=chezmoi_demo -e SHELL=/bin/bash -w /home/runner catthehacker/ubuntu:full-24.04
```

We make sure to set the `SHELL` environment variable and set the working
directory to the home directory of the default `runner` user.

## Install Chezmoi and set up dotfiles

First set the `GITHUB_USERNAME` environment variable:
```bash
export GITHUB_USERNAME=mirenradia
```

Chezmoi and my dotfiles can be installed with a single command (which I always
forget so I look it up in the [documentation](https://www.chezmoi.io/)):
```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply $GITHUB_USERNAME
```
My dotfiles include several chezmoi configuration variables which will be
prompted for with the above command. Most of them are booleans which can be set
using either `y`/`n` (yes/no) or `t`/`f` (true/false) but there is also a
string. After these configuration variables are set, Chezmoi will run some scripts which
installs the software for my fancy prompt, fzf and the vim plugin manager I
use (Vundle).

For various reasons, I decided not to include `.bashrc` directly in my dotfiles
but rather write to a `.bashrc_common` file which is `source`d from the usual
`.bashrc` file. I manually add the following line to `.bashrc` using my
preferred editor (vim):
```
source ~/.bashrc_common
```
It would probably be better to automate this step with a script but I have not
done that yet.

I can then stop the container using `Ctrl`+`D` or `exit` and restart it:

```bash
docker start -i chezmoi_demo
```

If everything worked, I should have my environment just the way I like with my
fancy coloured prompt, fzf, vim setup, etc.

## Basic Chezmoi commands

To edit a dotfile that is managed my chezmoi, e.g. `.vimrc`, I can do
```bash
chezmoi edit ~/.vimrc
```
This will open a temporary copy of the file from my chezmoi *source* directory
and will save it there too if there are changes. You may have noticed that the
filename that is opened is also appended with `.tmpl`. This is because it is a
Chezmoi *template* file which adjusts based on my configuration variables. In
this case, I have some conditional code
```
{{- if or .myubuntu .myfedora -}}
...
{{- else -}}
...
{{ end }}
```
The above code is reasonably self-explanatory but the full templating syntax is
explained in the [chezmoi
documentation](https://www.chezmoi.io/user-guide/templating/).

For the sake of this demo, let's say I no longer want to enable my line
numbering in Vim, so let's comment it out these lines
```vim
 15 set number relativenumber
 16 augroup numbertoggle
 17   autocmd!
 18   autocmd BufEnter,FocusGained,InsertLeave * set relativenumber
 19   autocmd BufLeave,FocusLost,InsertEnter   * set norelativenumber
 20 augroup END
 ```
I have a Vim plugin (vim-commentary) which allows me to easily comment these
out with a single command. After commenting this out, I save and exit. Note
that if I open `~/.vimrc`, it won't have changed since I have only made the
changes in my source directory. I can see the `diff` with my actual home
directory by doing
```
chezmoi diff
```
Note that my `install_vundle.sh` script has also come up in the diff. This is
because I have set it up to hash the `.vimrc` file and re-run if it changes so
that if I add a new plugin, it will be automatically installed.
I can apply these changes by doing
```
chezmoi apply
```
In this case, it will update `~/.vimrc` and run the `install_vundle.sh` script
(which doesn't have much to do since I didn't modify the Vundle plugins).

I could have edited and applied the changes in a single step by doing
```
chezmoi edit --apply ~/.vimrc
```

Once, I am happy, I can `cd` to the source repository using
```
chezmoi cd
```
where I will notice the working tree is now dirty. I can commit and push the
changes in the usual way. On a different machine, to obtain the changes, I can
do
```
chezmoi update
```
This is equivalent to running `git pull` in the source directory followed by
`chezmoi apply`.

## Adding an existing file

If you already have a dotfile you want to add to your dotfiles repository,
you can do
```
chezmoi add ~/path/to/.my_new_dotfile
```
You can also pass with the `--template` argument if you know that you will want
this file to depend on your configuration variables.

## Further documentation

Chezmoi has a [quick start guide](https://www.chezmoi.io/quick-start/) to help
you get running. Further
documentation can be found in the [user
guide](https://www.chezmoi.io/user-guide/command-overview/) and
[reference](https://www.chezmoi.io/reference/).
