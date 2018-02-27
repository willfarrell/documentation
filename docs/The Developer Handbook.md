# The Developer Handbook (WIP)
v2018.02.28 By @willfarrell

## Applications
- [Xcode](https://developer.apple.com/xcode/) - Xcode Command Line Tools
- [Alfred](https://www.alfredapp.com) - Spotlight replacement
- [Docker for Mac](https://www.docker.com/docker-mac) - Conatiners
- [Dash](https://kapeli.com/dash) - Documentation
- [Paw](https://paw.cloud) - REST API testing
- [Tower](https://www.git-tower.com/mac/) - git Client
- [Transmit](https://panic.com/transmit/) - SFTP/S3/etc
- [Kaleidoscope](https://www.kaleidoscopeapp.com) - File Diff
- [Slack](https://slack.com) - Text Comms
- [Zoom](https://zoom.us) - Video Comms

### Alfred Workflows
- [caniuse](https://github.com/willfarrell/alfred-caniuse-workflow)
- [Dash](https://github.com/Kapeli/Dash-Alfred-Workflow)
- [encode/decode](https://github.com/willfarrell/alfred-encode-decode-workflow)
- [hash](https://github.com/willfarrell/alfred-hash-workflow)
- [pkgman](https://github.com/willfarrell/alfred-pkgman-workflow)

### Chrome Extensions
- [Clear Cache](https://chrome.google.com/webstore/detail/clear-cache/cppjkneekbjaeellbfkmgnhonkkjfpdn)
- [Dimensions](http://felixniklas.com/dimensions/)
- [EditThisCookie](http://www.editthiscookie.com/)
- [Full Page Screen Capture](https://mrcoles.com/full-page-screen-capture-chrome-extension/)
- [ng-inspector](http://ng-inspector.org/)
- [React Developer Tools](https://github.com/facebook/react-devtools)

## Terminal

### [Homebrew](https://docs.brew.sh)
```bash
xcode-select --install
cd /usr/local
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
```

### bashrc
TODO research and test https://github.com/Bash-it/bash-it more

#### Variables
```
# Make nano you default editor, sorry vi
export EDITOR=nano

# Used by git for signing commits with gpg
export GPG_TTY=$(tty)
```

#### Aliases
```
alias ls="ls -G"
alias ..="cd .."
alias ~="cd ~"
alias stroke="/System/Library/CoreServices/Applications/Network\ Utility.app/Contents/Resources/stroke"

alias showFiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app'
alias hideFiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app'

alias tf=terraform

# compile swift code
alias swift="xcrun -sdk macosx swift"

alias docker-redis="docker stop redis && docker rm redis && docker run --name redis -p 6379:6379 --restart always -d redis:alpine"
alias docker-mysql="docker run --name mysql -p 3306:3306 --restart always -d -e MYSQL_DATABASE=mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes mysql:5.7"
alias docker-postgres="docker run --name postgres -p 5432:5432 --restart always -d -e POSTGRES_DB=postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password postgres:alpine"
```

#### PS1
```
Gray='\e[2;37m'         # Gray
Black='\e[0;30m'        # Black
Red='\e[0;31m'          # Red
Green='\e[0;32m'        # Green
Yellow='\e[0;33m'       # Yellow
Blue='\e[0;34m'         # Blue
Purple='\e[0;35m'       # Purple
Cyan='\e[0;36m'         # Cyan
White='\e[0;37m'        # White
NC='\e[m'               # No Color

function parse_git_branch {
	# https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh
	git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/\1/'
}

function timestamp {
	date +%X
}

# ~/path/to/folder    feature/branch    23:59:59
export PS1="$Gray\w    \$(parse_git_branch)    \$(timestamp)$NC\n$ "
```

#### Install completions
```bash
# bash-completion
brew install bash-competion
# git-completion
curl -L https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash > /usr/local/etc/bash_completion.d/git-completion.bash
# docker-completion
curl -L https://raw.githubusercontent.com/docker/docker/master/contrib/completion/bash/docker > /usr/local/etc/bash_completion.d/docker-completion.bash
# docker-compose-completion
curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/bash/docker-compose > /usr/local/etc/bash_completion.d/docker-compose.bash
```

#### Inline completions
```bash
# npm-completion
source <(npm completion)

# ssh-completion
_ssh() 
{
    local cur prev opts
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts=$(grep '^Host' ~/.ssh/config ~/.ssh/config.d/* | grep -v '[?*]' | cut -d ' ' -f 2-)

    COMPREPLY=( $(compgen -W "$opts" -- ${cur}) )
    return 0
}
complete -F _ssh ssh
```

