# tildearrow_hostapd_patch

This repo is just a copy of the hostapd patch to get intel ax210 cards working as access points.

As described here
https://tildearrow.org/?p=post&month=7&year=2022&item=lar

Copied from
[tildearrow.org/storage/hostapd-2.10-lar.patch](https://tildearrow.org/storage/hostapd-2.10-lar.patch";)

Example usage in Nix flake is:
```
#
# l2/flake.nix
#
{
  description = "l2 Flake";

  # https://nix.dev/manual/nix/2.24/command-ref/new-cli/nix3-flake.html#flake-inputs
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";

    # https://nixos-and-flakes.thiscute.world/nixos-with-flakes/start-using-home-manager
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, home-manager, ... }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs {
        inherit system;
        config = {
          allowUnfree = true;
        };
      };
      lib = nixpkgs.lib;
      overlays.default = final: prev: {
        hostapd = prev.hostapd.overrideDerivation (old: {
          patches = (old.patches or []) ++ [
            (final.fetchpatch {
            url = "https://tildearrow.org/storage/hostapd-2.10-lar.patch";
            #url = "https://github.com/randomizedcoder/tildearrow_hostapd_patch/hostapd-2.10-lar.patch";
            sha256 = "USiHBZH5QcUJfZSxGoFwUefq3ARc4S/KliwUm8SqvoI=";
          })
        ];
      });
  };
    in {
    nixosConfigurations = {
      l2 = lib.nixosSystem rec {
        inherit system;
        modules = [
          ./configuration.nix
          home-manager.nixosModules.home-manager
          {
            home-manager.useUserPackages = true;
            home-manager.users.das = { config, pkgs, ... }: {
              imports = [ ./home.nix ];
            };
          }
        ];
      };
    };
  };
}
```
