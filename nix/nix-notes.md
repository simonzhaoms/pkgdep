# Nix notes #

[NixOS](https://nixos.org/) is a Linux distribution.  It adopts
[Nix](https://nixos.org/nix/) as its package manager.  Nix uses a
special method to manage all packages to make different versions of
the same package co-exist without conflict.  It also supports
multi-user mode, such that different users can use different versions
of the same package.  See [About Nix](https://nixos.org/nix/about.html).

To work with Nix, applications should be packaged in the Nix format.
There is now a Nix repo where many popular application are packaged,
such as Python.


## Make a Nix package ##

### Reference ###

- [Generate Nix expressions for Python packages](https://github.com/garbas/pypi2nix)
- [.deb installation in NixOS](https://reflexivereflection.com/posts/2015-02-28-deb-installation-nixos.html)
- [Installing from outside Nix repository](https://stackoverflow.com/a/33924790)
- [Nix User Repository: User contributed nix packages](https://github.com/nix-community/NUR)
- [R -- NixOS](https://nixos.wiki/wiki/R)



## Reference ##

- [NixOS community](https://nixos.org/nixos/community.html)
- [Nix Package Manager Guide](https://nixos.org/nix/manual/)
- [NixOS Wiki](https://nixos.wiki/wiki/Main_Page)
- [Welcome to Nix Community Cookbook's documentation!](https://nix-cookbook.readthedocs.io/en/latest/)
- [CodeTriage -- Nix](https://www.codetriage.com/nixos/nix)

