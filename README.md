# uv-tools

## Automatically set uv to use the active Conda environment

[Miniconda](https://docs.anaconda.com/miniconda/install) is still the simplest way to install Cuda, and the easiest way to have multiple versions of Cuda on one machine.

uv doesn't use the active Conda enviroment for `uv sync` like it does for `uv pip`, but we can fix that with a rc script:

```bash
# Function to update UV_PROJECT_ENVIRONMENT when Conda environment changes
conda_auto_env() {
    if [[ -n "$CONDA_PREFIX" ]]; then
        export UV_PROJECT_ENVIRONMENT="$CONDA_PREFIX"
    else
        unset UV_PROJECT_ENVIRONMENT
    fi
}

# Run the function initially to set the variable immediately
conda_auto_env

# Hook into shell prompts
if [[ -n "$ZSH_VERSION" ]]; then
    # Ensure add-zsh-hook is available
    autoload -Uz add-zsh-hook
    # Avoid duplicate hooks
    if ! [[ -v _conda_update_hook_added ]]; then
        add-zsh-hook precmd conda_auto_env
        _conda_update_hook_added=1
    fi
elif [[ -n "$BASH_VERSION" ]]; then
    # Ensure PROMPT_COMMAND is updated only once
    if [[ -z "$PROMPT_COMMAND" ]]; then
        PROMPT_COMMAND="conda_auto_env"
    elif [[ "$PROMPT_COMMAND" != *"conda_auto_env"* ]]; then
        PROMPT_COMMAND="conda_auto_env; $PROMPT_COMMAND"
    fi
fi

```

Note that this may clash with future uv updates with uv and you might need to remove it.
