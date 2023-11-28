# Nix notes #

[NixOS](https://nixos.org/) is a Linux distribution.  It adopts
[Nix](https://nixos.org/manual/nix/stable/) as its package manager.
Nix uses a special method to manage all packages to make different
versions of the same package co-exist without conflicts.  It also
supports multi-user mode, such that different users can use different
versions of the same package.  See [About
Nix](https://nixos.org/nix/about.html).

To work with Nix, applications should be packaged in the Nix format.
There is now a Nix repo -- [nixpkgs](https://github.com/NixOS/nixpkgs)
where many popular application are packaged, such as Python.

The [Guix System Distribution
(GuixSD)](https://www.gnu.org/software/guix/) comes with [GNU Guix
package
manager](https://www.gnu.org/software/guix/manual/html_node/Package-Management.html),
which is similar to Nix.

* [Getting Started](get-start.md)
* [Making a Nix Package](packaging.md)


## Scenarios ##

When Nix would help:
* You want to install different versions of a program in your system,
  such as Python, GNOME.
    + Using several Docker containers for different versions of that
      program may need extra works and disk spaces.
    + pyenv or conda can be used for Python, and jhbuild for GNOME,
      but how to mix Python and GNOME?
* You want to use 2 programs together, but they may need different
  versions of a third program.
    + For example, program **A** needs Python2, but program **B**
      needs Python3.
    + Conda may not apply.


## Pros & Cons ##

### Pros ###

* Multiple versions of the same package can not only co-exist, but
  also be used simultaneouly.
* Allowing project-specific environments that are isolated from the
  rest of the system.

### Cons ###

* Steep learning curve


## Reference ##

* [NixOS](https://nixos.org/)
    + [Guides](https://nixos.org/learn.html)
    + [NixOS Manual](https://nixos.org/manual/nixos/stable/)
    + [NixOS community](https://nixos.org/community.html)
    + [NixOS Wiki](https://nixos.wiki/)
* [Nixpkgs](https://github.com/NixOS/nixpkgs): Nix Packages Collection
    + [Nixpkgs Guide](https://nixos.org/manual/nixpkgs/stable/)
* Nix
    + [Nix Guide](https://nixos.org/manual/nix/stable/)
    + [nix.dev](https://nix.dev/): Getting Started and Cookbook
    + [nix-shorts](https://github.com/justinwoo/nix-shorts): Collection of Short Notes
    + [Nix Pills](https://nixos.org/nixos/nix-pills/): For beginners
* [NixOps](https://github.com/NixOS/nixops): A tool for deploying to
  NixOS machines in a network or the cloud
* [CodeTriage -- Nix](https://www.codetriage.com/nixos/nix)
