---
title: Shell aliases
---

Do you often type the same boring commands in a [terminal](https://en.wikipedia.org/wiki/Terminal_emulator) window? Do you get tired of typing them over and over again? 

Shell aliases are there for you!

## An example

Let’s take the [`ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) command. To list the folder content in a long format, you would typically use

```shell
ls -l
```

After the 50th time, especially if like me you are not a formally trained touch typist, it gets old. What if you could shave off 60% of that effort?

Enter the `alias` command:
	
```shell
alias ll='ls -l'
```

Then you can just type

```shell
ll
```

and marvel at all the keystrokes you saved!

Of course, since the shell just replaces the alias name with its value under the hood, you can still append any valid argument after the alias, like this:

```shell
ll -A
```

## Saving aliases

These defined aliases will last as long as your shell session. If you open a new terminal window, they will be forgotten.

To have your favorite aliases ready to run every time, simply list them in your shell configuration file.

If you’re using [**bash**](https://en.wikipedia.org/wiki/Bash_(Unix_shell\)), create or edit the `~/.bashrc` file, and add your aliases, one per line:

```shell
# ~/.bashrc
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -lhF'
alias lla='ls -lhAF'
```

For [**zsh**](https://en.wikipedia.org/wiki/Z_shell), the file is called `~/.zshrc`.

Open a new terminal window. Typing one of the aliases should now run it every time.

## Listing aliases

To quickly list all available aliases, you can type

```shell
alias	
```

without any arguments.

## Git aliases

In addition to the regular shell aliases, git also maintains its own set of aliases.

Provided you already have git installed, you should have an existing `~/.gitconfig` file. If it contains an alias section like this one:

```shell
[alias]
  co = checkout
```

then you should be able to type 

```shell
git co
```

in a terminal window, and it will have the same effect as typing the full `git checkout` command.

## Wrapping up

I hope this nifty feature will help you save time and finger tips with your most used commands.

Thanks to [Emma Bostian’s tweet](https://twitter.com/EmmaBostian/status/1391725278624428034) for this article’s idea. (Have a look at the comments for more alias tricks.)

## Appendix 1: shell alias examples

Here are some useful aliases I accumulated over the years. Heavily geared towards [Ruby on Rails](https://rubyonrails.org) development, since that’s what I do all day long.

```shell
# Some ls aliases
alias ll='ls -lhF'
alias la='ls -A'
alias l='ls -CF'
alias lla='ls -lhAF'

# homebrew
alias bs='brew services'

# git
alias g='git'
alias gs='git status'
alias gst='git stash --include-untracked'
alias gsp='git stash pop'
alias gco='git checkout'
alias gm='git checkout master && git pull'
alias gpl='git pull'
alias gps='git push'
alias gr='git pull --rebase --autostash origin master'
alias gri='git rebase --interactive'
alias grc='git rebase --continue'
alias grs='git rebase --skip'
alias gra='git rebase --abort'
alias g-='git checkout -'

# bundler
alias b='bundle'
alias be='bundle exec'
alias bo='bundle open'

# yarn
alias y='yarn'

# rails
alias r='rails'
alias rgc='r g controller'
alias rgm="r g migration"
alias rgmo='r g model'
alias rc='r console'
alias s='bin/rspec'
alias ss='spring stop'
alias td='tail -fn500 log/development.log'
alias tt='tail -fn500 log/test.log'
alias rr='r routes'
alias rrg='r routes -g'
alias rrc='r routes -c'
alias rs='r restart'

# rake
alias bake='bin/rake'
alias rdm='bake db:migrate'
alias rdmr='bake db:migrate:redo'
alias rdc='bake db:create'
alias rdd='bake db:drop'
alias rdrs='bake db:restore'
alias rdr='bake db:rollback'
alias rdt='bake db:test:prepare'
alias rds='bake db:seed'
alias rdb='spring stop && r db:drop && r db:create && r db:restore && r db:migrate && r db:seed && r db:test:prepare'

# heroku
alias h='heroku'
alias hl='h local -e .env'
alias hrc='h run rails c'
alias hrcs='hrc -r staging'
alias hrcp='hrc -r production'
alias hcg='h config:get'
alias hcs='h config:set'

# Jekyll
alias js='bundle exec jekyll server'

# postgresql
alias pg_start='pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start'
alias pg_stop='pg_ctl -D /usr/local/var/postgres stop'
alias pg_start_agent='launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist'
alias pg_stop_agent='launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist'
alias pg_log='tail -f /usr/local/var/postgres/server.log'
alias pgs='pg_start'
alias pgx='pg_stop'

# Tinfoil hat aliased - public ip, and exit node ip.
alias ip='curl icanhazip.com'
alias iplookup='echo $(curl -s ipinfo.io/$(curl -s icanhazip.com))'

# Lock screen
alias lock='/System/Library/CoreServices/Menu\ Extras/User.menu/Contents/Resources/CGSession -suspend'

# Get the weather
alias weather='curl wttr.in/~Brussels'

# Convert an image to webp
alias webp='convert -quality 50 -define webp:lossless=true'
```

## Appendix 2: git alias examples

```shell
[alias]
  # Add
  ad = add
  aa = add .

  # Commit
  cm = commit -m
  ca = commit --amend -m

  # Checkout
  co = checkout
  cb = checkout -b

  # Cherry-pick
  cp = cherry-pick

  # Diff
  df = diff

  # List
  tl = tag -l
  bl = branch -a
  rl = remote -v

  # Status
  st = status -s

  # Pull
  pl   = pull
  plo  = pull --rebase origin
  plom = pull --rebase origin master
  plog = pull --rebase origin gh-pages
  plu  = pull --rebase upstream
  plum = pull --rebase upstream master
  plug = pull --rebase upstream gh-pages
  poule = pull

  # Push
  ps   = push
  pso  = push origin
  psom = push origin master
  psog = push origin gh-pages
  psu  = push upstream
  psum = push upstream master
  psug = push upstream gh-pages

  # Logs
  l  = log --pretty=oneline --decorate --abbrev-commit --max-count=15
  ll = log --graph --pretty=format:'%Cred%h%Creset %an: %s %Creset%Cgreen(%cr)%Creset' --abbrev-commit --date=relative
  lg1 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
  lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all
  lg = !"git lg1"

  # Sync
  sync = plu && pso

  # Stash-on-a-branch
  wip = commit -a -m "WIP" # FIXME: doesn't add untracked files
  pop = reset HEAD^
```

I use some of these so often that I routinely forget how to type the full commands. ¯\\\_(ツ)\_/¯
