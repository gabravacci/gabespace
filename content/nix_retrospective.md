---
title: "One year with NixOS"
date: 2024-02-21T14:15:29-08:00
draft: false
tags: ['software', 'linux', 'functinal programming']
---

Last January (2023) I decided to rebuild my Linux setup from scratch (from Arch, by the way), and this time I decided to go with [NixOS](https://nixos.org/), because I've always liked some functional programming, and provided many *unique* features. Particularly, I was due for a laptop upgrade, so it'd be nice to see how reproducible my NixOS configuration was. The full configuration can be seen [here](https://github.com/gabravacci/nixOS-config), I'll discuss my takeaways from one year of usage.

## What I liked
### Reproducible
I personally didn't find the language, or configuration too hard to get used to (well, until *flakes*). I began with a simple `configuration.nix`. After a full year of experience, I believe a simple setup like this is the best way to ensure complete reproducibility and ease of use. For more complicated setups, I found it useful to import separate `.nix` files as modules with the following structure:
```
configuration.nix *
   └─ ./modules   
        ├─ ./programs
        │    └─ starship.nix (example)
        ├─ ./desktop (desktop environments)
        └─ ./hardware-configuration.nix (don't touch this)
```
Which can be evoked (in `configuration.nix`) with:
```nixos
    import ./modules/programs/starship.nix
```
This configuration file is surprisingly powerful, and for servers probably all you could ever need. Configuring basic daemons and installing default packages is extremely easy:
```nix 
    environment = {
        systemPackages = with pkgs; [
            killall
            jq
            iwd
            ripgrep
            ... 
        ];
    };
```
Notice how you install these packages using `with pkgs`, this is really useful in combination with flakes, allowing you to choose to install from different package streams (unstable or stable, for example). Keeping all of this in a single repository is very easy to maintain, and if you're cautious to ensure it's reproducible, it's easily ported over. When I moved to a new laptop, I had to add *one singular line* to fix some hardware issues and we were good to go. For those interested, it was:
```nix
    boot.kernelPackages = pkgs.linuxPackages_latest;
```
To obtain the latest Linux kernel. Very convenient.

### Configuration 
This is probably the *best* aspect of NixOS as a whole, as long as the configuration you desire is exposed through the `nix` API, it is fantastic. For example, setting up my shell was as easy as:

```nixos
    programs.zsh = { 
        enable = true;
        enableCompletion = true;
        syntaxHighlighting.enable = true;
        shellAliases = { ... };
    };
```
And this holds for a decent amount of the things you'd want to configure. Unfortunately, when the configuration options aren't exposed is when it gets messy. A common solution, and the one I adopted is [Home Manager](https://nixos.wiki/wiki/Home_Manager), which really just exposes many more useful configuration options. Using it is easy enough, a few lines of code and:
```nixos
    programs.home-manager.enable = true;
```
Home manager introduces a lot of useful configuration options, particularly for GTK theming:
```nixos
    home.gtk = {
        enable = true;
        theme = { name = "Dracula"; package = pkgs.dracula-theme; };
        iconTheme = { ... };
        font = { ... };
    };
```
If you've ever tried to get this consistent on a normal Linux distro, you know it can be a bit annoying.

But this is also where the reproducibility and conveniences of NixOS start to fail for me. Which brings me to:
## What I disliked
### Fragmentation
Home manager is (as of the current date) a community tool, very rough around the edges. Upon usage, it *immediately* breaks the synergy of pure NixOS. Now, you need to maitain two different package lists:
```nixos
    home.packages = with pkgs; [ 
        htop
        iftop
        fzf
        ffmpeg
        ...
    ];
```
These expose home manager's configuration API, which to be fair is really nice. The documentation is *somewhat* extensive, and they provide a massive list of options to search for. My main issue with home manager is more with the things it brings to the Nix ecosystem. There already is a prevalent fragmentation issue within Nix, between standard `configuration.nix` and the modern paradigm of *flakes* (not getting into those, maybe another time).

The key issue is that NixOS in a transition state, from traditional to a flake based approach, and that means the toolchain is almost *too* varied. Let's consider two popular and extensible text editors, NeoVim and Emacs. They both have *aspects* of them that interface with home manager; NeoVim gets a decent amount of configuration options *and* a plugin manager: 
```nixos
    programs.neovim = {
        enable = true;
        plugins = with pkgs.vimPlugins; [
            auto-pairs
            nvim-treesitter
            ...
        ];
        # VimScript configuration
        extraConfig = ''
            set number
            set relativenumber 
            ...
            nmap <C-z> :tabprev<CR>
        '';
    };
```
But any NeoVim user can see the issues with this. Firstly, you don't get the versatility of Lua configuration and you are limited to plugins packaged by `vimPlugins` (not many at the time, I hear NeoVim -- Nix distros are better now). This eventually led me to doing this instead:
```nixos
    programs.neovim.enable = true;

    home.file.".config/nvim/init.lua".source = ./init.lua;
    home.file.".config/nvim/lua".source = ./lua;
```
That is, sourcing my *own* Lua configuration files *through* Nix. This is not that bad in itself, but I can already keep those Lua files in a repository and copy them into `~/.config` whenever I want. 

Emacs, by being more extensible naturally suffers even more. To adequately use Emacs you need an *overlay*, which returns a list of packages based on some input. You declare it in `configuration.nix` as:
```nixos
    # adds to the standard nix package list 
    # with the flake inputs
    nixpkgs.overlays = with inputs; [
        emacs-overlay.overlay # declared in flake.nix
    ];
```
Here I imported the overlay through the flake (one of the flakes' system benefits), but overlays can also be declared normally to extend the package list. For example, Discord needs the most current version installed, so it is not uncommon to overlay it as:
```nixos
    nixpkgs.overlays = 
    (self: super: {
        discord = super.discord.overrideAttrs (
            _: { src = builtins.fetchTarball {
                url = "https://discord.com/api/...";
                sha256 = "...";
            };}
        );
    })
```
Which is just makes it so that `with pkgs; [ discord ];` compiles the tarball from source. The Emacs overlay is more complicated, making Emacs plugins available as well through:
```nixos
    home.programs.emacs = { 
        enable = true;
        package = pkgs.emacs-pgtk;
        extraPackages = (epkgs: (with epkgs; [ 
            treemacs 
            lsp-mode 
            vterm 
            evil 
            ...
        ]));
    };
```
And after that, you still need to supply *your own* `*.el` configuration files (like the NeoVim example above). To me, this just added more abstraction layers on top of the "universal" Linux configuration experience through `~/.config`. It also adds more possible levels of failure to the configuration (suppose the Emacs overlay fails, or some package on `epkgs`, or the home-manager interface...).

### Extras
These are common sentiments online, but I do agree documentation is *sparse*, and most of what you learn is through other repos. I also found the Nix language to be a bit awkward and limited; I briefly experimented with [Guix](https://guix.gnu.org/) and found Scheme to be far more pleasant to work with.

## Conclusion
NixOS, and Nix, are quite new, so these rough edges are to be expected. The issues I ran into were a result of my own added complexity, which should highlight the strongest benefit of NixOS: For simple systems, it is extremely portable, reproducible and decently configurable. I'd also say I've only scratched the surface of what's possible with NixOS, and would later like to explore different multi-user setups or FHS combinations ([Erase your darlings](https://grahamc.com/blog/erase-your-darlings/) comes to mind).
