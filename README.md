# warp-terminal-flake

A [Home Manager](https://github.com/nix-community/home-manager) module for [Warp](https://www.warp.dev). It installs `pkgs.warp-terminal` from nixpkgs and manages Warp's configuration files declaratively.

## Usage

Add the flake as an input and import the module:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
    warp-terminal.url = "github:kimhappy/warp-terminal-flake";
  };

  outputs = { nixpkgs, home-manager, warp-terminal, ... }: {
    homeConfigurations.me = home-manager.lib.homeManagerConfiguration {
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
      modules = [
        warp-terminal.homeModules.default
        ./home.nix
      ];
    };
  };
}
```

Then configure Warp in `home.nix`:

```nix
{
  programs.warp-terminal = {
    enable = true;

    settings = {
      appearance.text = {
        font_name = "JetBrains Mono";
        font_size = 14.0;
      };
      appearance.themes.theme = "dracula";
      text_editing.vim_mode_enabled = true;
    };

    keybindings."workspace:toggle_warp_drive" = "ctrl-shift-D";

    tabConfigs.dev_server = {
      name = "Dev Server";
      panes = [
        {
          id = "main";
          type = "terminal";
          directory = "~/code/my-app";
          commands = [ "npm run dev" ];
        }
      ];
    };
  };
}
```

## Options

All options live under `programs.warp-terminal`. Each option's description in [flake.nix](flake.nix) includes an example.

| Option | Written to (Linux) | Written to (macOS) |
| --- | --- | --- |
| `settings` | `$XDG_CONFIG_HOME/warp-terminal/settings.toml` | `~/.warp/settings.toml` |
| `keybindings` | `$XDG_CONFIG_HOME/warp-terminal/keybindings.yaml` | `~/.warp/keybindings.yaml` |
| `themes` | `$XDG_DATA_HOME/warp-terminal/themes/` | `~/.warp/themes/` |
| `tabConfigs` | `$XDG_DATA_HOME/warp-terminal/tab_configs/` | `~/.warp/tab_configs/` |
| `workflows` | `$XDG_DATA_HOME/warp-terminal/workflows/` | `~/.warp/workflows/` |
| `launchConfigurations` | `$XDG_DATA_HOME/warp-terminal/launch_configurations/` | `~/.warp/launch_configurations/` |
| `mcpServers` | `~/.warp/.mcp.json` | `~/.warp/.mcp.json` |
| `skills` | `~/.warp/skills/` | `~/.warp/skills/` |

The remaining options are:

- `package`: the Warp package. Set it to `null` to manage only the configuration.
- `validateSettings`: validates `settings` at build time against the JSON schema bundled with `package`. Enabled whenever `package` is set.
- `enableMcpIntegration`: also writes the servers of Home Manager's `programs.mcp.servers` to `~/.warp/.mcp.json`.

## Caveats

- Managed files are read-only symlinks into the Nix store. Changes made in the Warp Settings panel are not persisted to a managed `settings.toml` or `keybindings.yaml`.
- Only the Stable release channel is supported, matching `pkgs.warp-terminal`.

