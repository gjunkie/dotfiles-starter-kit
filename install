#!/usr/bin/env bash
LOG="${HOME}/Library/Logs/dotfiles.log"
GITHUB_USER="your github username"
GITHUB_REPO="the name of this repo"
USER_GIT_AUTHOR_NAME="your github user name"
USER_GIT_AUTHOR_EMAIL="your github email"
DIR="/usr/local/opt/${GITHUB_REPO}"

_process() {
    echo "$(date) PROCESSING:  $@" >> $LOG
    printf "$(tput setaf 6) %s...$(tput sgr0)\n" "$@"
}

_success() {
  local message=$1
  printf "%s✓ Success:%s\n" "$(tput setaf 2)" "$(tput sgr0) $message"
}

download_dotfiles() {
    _process "→ Creating directory at ${DIR} and setting permissions"
    mkdir -p "${DIR}"

    _process "→ Downloading repository to /tmp directory"
    curl -#fLo /tmp/${GITHUB_REPO}.tar.gz "https://github.com/${GITHUB_USER}/${GITHUB_REPO}/tarball/main"

    _process "→ Extracting files to ${DIR}"
    tar -zxf /tmp/${GITHUB_REPO}.tar.gz --strip-components 1 -C "${DIR}"

    _process "→ Removing tarball from /tmp directory"
    rm -rf /tmp/${GITHUB_REPO}.tar.gz

    [[ $? ]] && _success "${DIR} created, repository downloaded and extracted"

    # Change to the dotfiles directory
    cd "${DIR}"
}

link_dotfiles() {
    # symlink files to the HOME directory.
    if [[ -f "${DIR}/opt/files" ]]; then
        _process "→ Symlinking dotfiles in /configs"

        # Set variable for list of files
        files="${DIR}/opt/files"

        # Store IFS separator within a temp variable
        OIFS=$IFS
        # Set the separator to a carriage return & a new line break
        # read in passed-in file and store as an array
        IFS=$'\r\n'
        links=($(cat "${files}"))

        # Loop through array of files
        for index in ${!links[*]}
        do
            for link in ${links[$index]}
            do
                _process "→ Linking ${links[$index]}"
                # set IFS back to space to split string on
                IFS=$' '
                # create an array of line items
                file=(${links[$index]})
                # Create symbolic link
                ln -fs "${DIR}/${file[0]}" "${HOME}/${file[1]}"
            done
            # set separater back to carriage return & new line break
            IFS=$'\r\n'
        done

        # Reset IFS back
        IFS=$OIFS

        source "${HOME}/.bash_profile"
        [[ $? ]] && _success "All files have been copied"
    fi
}

install_homebrew() {
  _process "→ Installing Homebrew"
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

  _process "→ Running brew doctor"
  brew doctor
  [[ $? ]] \
  && _success "Installed Homebrew"
}

install_node() {
  if ! type -P 'npm' &> /dev/null; then
    _process "→ Installing node"

    curl https://www.npmjs.org/install.sh | sh

    [[ $? ]] \
    && _success "Installed npm"
  fi
}

install_formulae() {
  if ! type -P 'brew' &> /dev/null; then
    _error "Homebrew not found"
  else
    _process "→ Installing Homebrew packages"

    # Set variable for list of homebrew formulaes
    brews="${DIR}/opt/homebrew"

    # Update and upgrade all packages
    _process "→ Updating and upgrading Homebrew packages"
    brew update
    brew upgrade

    # Tap some necessary formulae
    brew tap homebrew/cask-versions
    brew tap homebrew/cask-drivers
    brew tap vitorgalvao/tiny-scripts

    # Store IFS within a temp variable
    OIFS=$IFS

    # Set the separator to a carriage return & a new line break
    # read in passed-in file and store as an array
    IFS=$'\r\n' formulae=($(cat "${brews}"))

    # Loop through split list of formulae
    _process "→ Checking status of desired Homebrew formulae"
    for index in ${!formulae[*]}
    do
      # Test whether a Homebrew formula is already installed
      if ! brew list ${formulae[$index]} &> /dev/null; then
        brew install ${formulae[$index]}
      fi
    done

    # Reset IFS back
    IFS=$OIFS

    brew cleanup

    [[ $? ]] && _success "All Homebrew packages installed and updated"
  fi
}

setup_git_authorship() {
  GIT_AUTHOR_NAME=eval "git config user.name"
  GIT_AUTHOR_EMAIL=eval "git config user.email"

  if [[ ! -z "$GIT_AUTHOR_NAME" ]]; then
    _process "→ Setting up Git author"

    read USER_GIT_AUTHOR_NAME
    if [[ ! -z "$USER_GIT_AUTHOR_NAME" ]]; then
      GIT_AUTHOR_NAME="${USER_GIT_AUTHOR_NAME}"
      $(git config --global user.name "$GIT_AUTHOR_NAME")
    else
      _warning "No Git user name has been set.  Please update manually"
    fi

    read USER_GIT_AUTHOR_EMAIL
    if [[ ! -z "$USER_GIT_AUTHOR_EMAIL" ]]; then
      GIT_AUTHOR_EMAIL="${USER_GIT_AUTHOR_EMAIL}"
      $(git config --global user.email "$GIT_AUTHOR_EMAIL")
    else
      _warning "No Git user email has been set.  Please update manually"
    fi
  else
    _process "→ Git author already set, moving on..."
  fi
}

install() {
  download_dotfiles
  link_dotfiles
  install_homebrew
  install_node
  install_formulae
  setup_git_authorship
}

install
