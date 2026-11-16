# Neovim: Requirements and Fixes

Installation notes for a Hyprland setup on Arch Linux with a dotfiles collection managed by GNU Stow. The Neovim configuration is located at `~/dotfiles/nvim/.config/nvim` and is usually stowed to `~/.config/nvim`.

## Overview

| Component | Package/action | Purpose |
|---|---|---|
| Avante.nvim | Add `ColinKennedy/mega.cmdparse` and `ColinKennedy/mega.logging` as plugin dependencies | Fixes `module mega.cmdparse not found` |
| nvim-treesitter (branch `main`) | `tree-sitter-cli`, `base-devel`, `tar`, and `curl`; then update/install parsers | Fixes `tree-sitter ENOENT` and provides the CLI and C compiler |
| Mason / `ruby_lsp` | `ruby` and, for native extensions, `base-devel`; add the RubyGems binary path to `PATH` | Enables installing and running `ruby_lsp` through Mason |

## Avante.nvim: `module mega.cmdparse not found`

Add the two Mega modules to the Avante plugin spec. First, find the relevant file in the active configuration:

```sh
grep -rn avante ~/.config/nvim
```

Add the following entries under `dependencies` in the Avante spec (for example, in a `lazy.nvim` plugin spec):

```lua
{
  "yetone/avante.nvim",
  dependencies = {
    "ColinKennedy/mega.cmdparse",
    "ColinKennedy/mega.logging",
  },
}
```

If the spec already has a `dependencies` list, add only the two entries there; keep any existing dependencies. With a Stow configuration, edit the source file in `~/dotfiles/nvim/.config/nvim`. The path `~/.config/nvim` points to the stowed configuration.

## nvim-treesitter (`main`): `tree-sitter ENOENT`

Parser processing requires the Tree-sitter CLI, a C compiler, and the necessary archive/download tools:

```sh
sudo pacman -S tree-sitter-cli
sudo pacman -S --needed base-devel
sudo pacman -S --needed tar curl
```

Among other things, `base-devel` provides the tools needed for compilation. Check that the CLI is available in the shell:

```sh
tree-sitter --version
```

Start Neovim from a shell with the correct `PATH`. Then, in Neovim, install the required parser or update the parsers, and run the diagnostic:

```vim
:TSInstall scss
:TSUpdate
:checkhealth nvim-treesitter
```

Run `:TSInstall scss` if the SCSS parser is needed; `:TSUpdate` updates the configured parsers. If you do not want to install the CLI via pacman, Mason (`:MasonInstall tree-sitter-cli`) or a Cargo installation are alternatives.

## Mason `ruby_lsp`: `failed to install ruby_lsp`

Install Ruby. For native Ruby extensions, also provide the build tools:

```sh
sudo pacman -S ruby
sudo pacman -S --needed base-devel
```

Add the user-specific RubyGems path to the shell environment. Add both lines to `~/.zshrc` (Zsh) or `~/.bashrc` (Bash):

```sh
export GEM_HOME="$(ruby -e 'puts Gem.user_dir')"
export PATH="$PATH:$GEM_HOME/bin"
```

Then reload the shell configuration or start a new terminal. Update the Mason package information in Neovim and, if the installation fails again, check the installation log:

```vim
:MasonUpdate
:MasonLog
```

If `ruby_lsp` is not needed, it can be removed from `ensure_installed`. Alternatively, its automatic activation can be excluded in `mason-lspconfig`:

```lua
require("mason-lspconfig").setup({
  automatic_enable = {
    exclude = { "ruby_lsp" },
  },
})
```

## Combined installation – quick commands

Install the required Arch packages together:

```sh
sudo pacman -S --needed tree-sitter-cli base-devel ruby
sudo pacman -S --needed tar curl
```

These two commands cover the Tree-sitter CLI, build tools, Ruby, and archive and download tools. Then configure `GEM_HOME` and `PATH` for RubyGems as described above.

## Verification checklist

- [ ] `grep -rn avante ~/.config/nvim` finds the Avante plugin spec; `mega.cmdparse` and `mega.logging` are listed in its `dependencies`.
- [ ] `tree-sitter --version` prints version information.
- [ ] Neovim is launched from a shell where `tree-sitter` is in `PATH`.
- [ ] The required parser can be installed with `:TSInstall scss` or updated with `:TSUpdate`.
- [ ] `:checkhealth nvim-treesitter` reports no missing prerequisites for the configuration in use.
- [ ] `ruby --version` works; `GEM_HOME/bin` is included in `PATH`.
- [ ] `:MasonUpdate` completes; if installation fails, `:MasonLog` provides diagnostic context for `ruby_lsp`.
- [ ] If `ruby_lsp` should not be installed, it has been removed from `ensure_installed` or listed in `automatic_enable.exclude`.
