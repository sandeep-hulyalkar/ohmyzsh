#!/bin/bash
# Consolidated Kubernetes aliases, helpers, and Zsh completion setup

# --- Helper Tools Setup ---
TOOLS_DIR="${TOOLS_DIR:-/tmp/tools}"
TOOLS_BIN_DIR="$TOOLS_DIR/bin"

function get_goyq() {
    YQ="$TOOLS_BIN_DIR/yq"
    YQ_VERSION='4.30.4' # Consider updating version if needed

    if [[ -e $YQ ]]; then
        return
    fi
    echo "Downloading yq..."
    mkdir -p "$(dirname "$YQ")"
    curl -fSL -o "$YQ" "https://github.com/mikefarah/yq/releases/download/v$YQ_VERSION/yq_linux_amd64" && chmod +x "$YQ" || echo "Failed to download yq"
}

function get_gojq() {
    GOJQ="$TOOLS_BIN_DIR/gojq"
    GOJQ_VERSION='0.12.9' # Consider updating version if needed

    if [[ -e $GOJQ ]]; then
        return
    fi
    echo "Downloading gojq..."
    mkdir -p "$TOOLS_DIR" "$(dirname "$GOJQ")"
    curl -fSL -o - "https://github.com/itchyny/gojq/releases/download/v$GOJQ_VERSION/gojq_v${GOJQ_VERSION}_linux_amd64.tar.gz" | tar xzf - -C "$TOOLS_DIR" gojq_v${GOJQ_VERSION}_linux_amd64/gojq && mv "$TOOLS_DIR/gojq_v${GOJQ_VERSION}_linux_amd64/gojq" "$TOOLS_BIN_DIR" && rm -fr "$TOOLS_DIR/gojq_v${GOJQ_VERSION}_linux_amd64" || echo "Failed to download gojq"
}

function get_kubecolor() {
    KUBECOLOR="$TOOLS_BIN_DIR/kubecolor"
    KUBECOLOR_VERSION='0.0.21' # Consider updating version if needed

    if [[ -e $KUBECOLOR ]]; then
        return
    fi
    echo "Downloading kubecolor..."
    mkdir -p "$(dirname "$KUBECOLOR")"
    curl -fSL -o - "https://github.com/kubecolor/kubecolor/releases/download/v$KUBECOLOR_VERSION/kubecolor_${KUBECOLOR_VERSION}_Linux_x86_64.tar.gz" | tar xzf - -C "$TOOLS_BIN_DIR" kubecolor || echo "Failed to download kubecolor"
}

function get_tools() {
    mkdir -p "$TOOLS_BIN_DIR"

    # Add tools bin dir to PATH if not already present at the beginning
    if [[ ":$PATH:" != *":$TOOLS_BIN_DIR:"* ]]; then
        export PATH="$TOOLS_BIN_DIR:$PATH"
        echo "Added $TOOLS_BIN_DIR to PATH"
    fi

    # Check and get tools (run in subshell to avoid polluting function namespace if sourced)
    ( get_goyq )
    ( get_gojq )
    ( get_kubecolor )
}

# Optional: Uncomment to automatically download tools when sourcing
# get_tools

# --- Optional Configurations (Uncomment if desired) ---
# function set_ps1() {
#     PS1='($(kubectl config get-contexts --no-headers | awk "function n(x) {return x==\"\"?\"default\":x} \\$1 == \"*\" {print(\"\[\033[01;33m\]\"\\$2\"\[\033[00m\]/\[\033[01;36m\]\"n(\\$5)\"\[\033[00m\]\")}")) \u@\h \[\033[01;34m\]\w\[\033[00m\] \[\033[01;32m\]\$\[\033[00m\] '
# }
# set_ps1

# function config_vim() {
#     cat <<END > ~/.vimrc
# set paste
# set pastetoggle=<F4>
# set tabstop=2
# set shiftwidth=2
# set expandtab
# END
# }
# config_vim

# function config_tmux() {
#     cat <<END > ~/.tmux.conf
# unbind-key C-b
# set-option -g prefix C-a
# bind-key C-a send-prefix
# #set -g mouse on
# END
# }
# config_tmux

# --- Aliases ---

# Determine base command (kubecolor if available, otherwise kubectl)
if [[ -x "$TOOLS_BIN_DIR/kubecolor" ]]; then
    alias k='kubecolor'
else
    alias k='kubectl'
fi

# Get
alias kg='k get'
alias kgy='kgy_f() { k get -o yaml "$@" | cy; }; kgy_f'
alias kgyy='kgyy_f() { k get -o yaml "$@" | yq e -; }; kgyy_f'
alias kga='k get all'
alias kgp='k get pod'
alias kgpy='kgpy_f() { k get pod -o yaml "$@" | cy; }; kgpy_f'
alias kgpyy='kgpyy_f() { k get pod -o yaml "$@" | yq e -; }; kgpyy_f'
alias kgj='k get job'
alias kgjy='kgjy_f() { k get job -o yaml "$@" | cy; }; kgjy_f'
alias kgjyy='kgjyy_f() { k get job -o yaml "$@" | yq e -; }; kgjyy_f'
alias kgs='k get svc'
alias kgsy='kgsy_f() { k get svc -o yaml "$@" | cy; }; kgsy_f'
alias kgsyy='kgsyy_f() { k get svc -o yaml "$@" | yq e -; }; kgsyy_f'
alias kgd='k get deployment'
alias kgdy='kgdy_f() { k get deployment -o yaml "$@" | cy; }; kgdy_f'
alias kgdyy='kgdyy_f() { k get deployment -o yaml "$@" | yq e -; }; kgdyy_f'
alias kgn='k get node'
alias kgny='kgny_f() { k get node -o yaml "$@" | cy; }; kgny_f'
alias kgnyy='kgnyy_f() { k get node -o yaml "$@" | yq e -; }; kgnyy_f'
alias kgns='k get ns'
alias kgnsy='kgnsy_f() { k get ns -o yaml "$@" | cy; }; kgnsy_f'
alias kgnsyy='kgnsyy_f() { k get ns -o yaml "$@" | yq e -; }; kgnsyy_f'
alias kgi='k get ingress'
alias kgiy='kgiy_f() { k get ingress -o yaml "$@" | cy; }; kgiy_f'
alias kgiyy='kgiyy_f() { k get ingress -o yaml "$@" | yq e -; }; kgiyy_f'
alias kgsec='k get secret'
alias kgsecy='kgsecy_f() { k get secret -o yaml "$@" | cy; }; kgsecy_f'
alias kgsecyy='kgsecyy_f() { k get secret -o yaml "$@" | yq e -; }; kgsecyy_f'
alias kgpv='k get pv'
alias kgpvc='k get pvc'
alias kgsc='k get storageclass' # Renamed from kgpsc for clarity

# Describe
alias kd='k describe'
alias kdp='k describe pod'
alias kdj='k describe job'
alias kdd='k describe deployment'
alias kds='k describe svc'
alias kdn='k describe node'
alias kdns='k describe namespace'
alias kdi='k describe ingress'
alias kdsec='k describe secret'
alias kdpv='k describe pv'
alias kdpvc='k describe pvc'
alias kdsc='k describe storageclass' # Renamed from kdpsc

# Edit
alias ke='k edit'
alias kep='k edit pod'
alias kej='k edit job'
alias ked='k edit deployment'
alias kes='k edit svc'
alias kens='k edit ns'
alias kei='k edit ingress'
alias kesec='k edit secret'

# Delete
alias kdel='k delete'
alias kdelf='k delete -f'
alias kdelp='k delete pod'
alias kdelj='k delete job'
alias kdeld='k delete deployment'
alias kdels='k delete svc'
alias kdelns='k delete ns'
alias kdeli='k delete ingress'
alias kdelsec='k delete secret'
alias kdelfin='k patch -p "{\"metadata\":{\"finalizers\":null}}" --type=merge'

# Other
alias kv='k version --short'
alias kar='k api-resources --sort-by name'
alias kac='k auth can-i'
alias ktn='k top node'
alias ktp='k top pod'
alias kex='k expose'
alias kexpl='k explain'
alias kexplr='k explain --recursive=true'
alias kc='k create'
alias kcd='k create --dry-run=client -o yaml'
alias kx='k exec -it' # Added -it for convenience
alias kl='k logs'
alias klf='k logs -f'
alias ka='k apply'
alias kaf='k apply -f'
alias krun='k run'
alias krund='k run --dry-run=client -o yaml'
alias kshell='k shell' # Note: 'k shell' might not be a standard command, depends on plugins
alias kgansr='k api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found'
alias kctx='k config use-context'
alias knsf='k config set-context --current --namespace'
# Function for setting namespace, improved error handling
kns() {
    if [[ -z "$1" ]]; then
        echo "Usage: kns <namespace>"
        return 1
    fi
    # Check if namespace exists before setting it
    if k get ns "$1" --no-headers --ignore-not-found | grep -q "$1"; then
        k config set-context --current --namespace "$1"
    else
        echo "Error: Namespace \"$1\" not found."
        return 1
    fi
}

# YAML/JSON Cleaning functions (require yq/gojq)
cy() {
    if command -v yq &> /dev/null; then
        # Original cleanyaml logic
        yq e 'del(.. | select(tag == "!!map") | (.status, .metadata.creationTimestamp, .metadata.generation, .metadata.selfLink, .metadata.uid, .metadata.resourceVersion, .metadata.managedFields, ."metadata.annotations"."kubectl.kubernetes.io/last-applied-configuration"))' -
    else
        echo "Warning: yq not found. Skipping YAML cleaning." >&2
        cat
    fi
}
cyy() {
    if command -v yq &> /dev/null; then
        # Original cyy logic
        cy | yq e 'del(.. | select((. == "" and tag == "!!str") or tag == "!!null")) | del(... | select(tag == "!!map" and length == 0))' -
    else
        echo "Warning: yq not found. Skipping YAML cleaning." >&2
        cat
    fi
}
cj() {
     if command -v gojq &> /dev/null; then
        # Original cleanjson logic
        gojq 'del(.. | select(. == "" or . == null)) | walk(if type == "object" then del(.status, .metadata.creationTimestamp, .metadata.generation, .metadata.selfLink, .metadata.uid, .metadata.resourceVersion, .metadata.managedFields, ."metadata.annotations"."kubectl.kubernetes.io/last-applied-configuration") else . end) | del(.. | select(. == {}))'
    else
        echo "Warning: gojq not found. Skipping JSON cleaning." >&2
        cat
    fi
}
# Simpler aliases for cleaning
alias cleanyaml='cy'
alias cleanyamldeeper='cyy'
alias cleanjson='cj'


# --- Zsh Autocompletion Setup ---

# Check if running in Zsh
if [ -n "$ZSH_VERSION" ]; then
   # Source kubectl completion script
   if command -v kubectl &> /dev/null; then
      source <(kubectl completion zsh)
   else
      echo "Warning: kubectl command not found. Skipping completion setup." >&2
   fi

   # Define completions for aliases (ensure kubectl completion was sourced)
   if command -v _kubectl &> /dev/null; then
      compdef _kubectl k
      # Get
      compdef _kubectl kg
      compdef _kubectl kga
      compdef _kubectl kgp
      compdef _kubectl kgpy
      compdef _kubectl kgpyy
      compdef _kubectl kgj
      compdef _kubectl kgjy
      compdef _kubectl kgjyy
      compdef _kubectl kgs
      compdef _kubectl kgsy
      compdef _kubectl kgsyy
      compdef _kubectl kgd
      compdef _kubectl kgdy
      compdef _kubectl kgdyy
      compdef _kubectl kgn
      compdef _kubectl kgny
      compdef _kubectl kgnyy
      compdef _kubectl kgns
      compdef _kubectl kgnsy
      compdef _kubectl kgnsyy
      compdef _kubectl kgi
      compdef _kubectl kgiy
      compdef _kubectl kgiyy
      compdef _kubectl kgsec
      compdef _kubectl kgsecy
      compdef _kubectl kgsecyy
      compdef _kubectl kgpv
      compdef _kubectl kgpvc
      compdef _kubectl kgsc
      # Describe
      compdef _kubectl kd
      compdef _kubectl kdp
      compdef _kubectl kdj
      compdef _kubectl kdd
      compdef _kubectl kds
      compdef _kubectl kdn
      compdef _kubectl kdns
      compdef _kubectl kdi
      compdef _kubectl kdsec
      compdef _kubectl kdpv
      compdef _kubectl kdpvc
      compdef _kubectl kdsc
      # Edit
      compdef _kubectl ke
      compdef _kubectl kep
      compdef _kubectl kej
      compdef _kubectl ked
      compdef _kubectl kes
      compdef _kubectl kens
      compdef _kubectl kei
      compdef _kubectl kesec
      # Delete
      compdef _kubectl kdel
      compdef _kubectl kdelf
      compdef _kubectl kdelp
      compdef _kubectl kdelj
      compdef _kubectl kdeld
      compdef _kubectl kdels
      compdef _kubectl kdelns
      compdef _kubectl kdeli
      compdef _kubectl kdelsec
      compdef _kubectl kdelfin
      # Other
      compdef _kubectl kv
      compdef _kubectl kar
      compdef _kubectl kac
      compdef _kubectl ktn
      compdef _kubectl ktp
      compdef _kubectl kex
      compdef _kubectl kexpl
      compdef _kubectl kexplr
      compdef _kubectl kc
      compdef _kubectl kcd
      compdef _kubectl kx
      compdef _kubectl kl
      compdef _kubectl klf
      compdef _kubectl ka
      compdef _kubectl kaf
      compdef _kubectl krun
      compdef _kubectl krund
      compdef _kubectl kshell
      compdef _kubectl kgansr
      compdef _kubectl kctx
      compdef _kubectl knsf
      # compdef _kubectl kns # Completion for functions is less reliable
   else
      echo "Warning: _kubectl completion function not found. Alias completion may not work." >&2
   fi
else
   echo "Warning: Not running in Zsh. Alias completion setup skipped." >&2
fi

echo "Kubernetes aliases and completions loaded."
