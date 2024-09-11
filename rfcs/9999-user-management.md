---
feature: I'm Gonna Build My Own Manager With Backjack and Modules!
start-date: (fill me in with today's date, YYYY-MM-DD)
author: Anderson Torres
co-authors: (find a buddy later to help out with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Enhance Nixpkgs monorepo with a basic user management system.

# Terminology
[terminology]: #terminology

Henceforth,

- The typical unprivileged user of a system will be called _basic user_;
- The typical privileged user of a system will be called _superuser_;

# Motivation
[motivation]: #motivation

[Nixpkgs](https://github.com/NixOS/nixpkgs) is by a far margin the biggest
packageset in this existence. In principle Nix evaluators already leverage
Nixpkgs for basic users. However, its raw usage is not too ergonomical,
especially when compared to the structured model of NixOS.

The average user has two extreme options here: either configure a whole OS via
superuser powers or use raw Nix commands as a basic user, some of them [not
recommended](https://stop-using-nix-env.privatevoid.net/).

This RFC proposes to fill this gap-in-the-middle: a structured model of system
management for basic users.

# Detailed design
[design]: #detailed-design

Here is an informational sketch based on `nixos-rebuild` and friends:

- A set of modules to serve as basis
- Custom modules built over the set above
- Configuration provided by the modules above
- Helper program that reads the configuration provided by the user and realizes
  it

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

```shell
$> NHMT help
$> emacs hmt-configuration.nix
$> NHMT test-container
$> NHMT build
$> NHMT list-generations
$> NHMT select-generation
$> NHMT delete-generations
```

# Drawbacks
[drawbacks]: #drawbacks

- Why not to keep this tool outside Nixpkgs monorepo?

  Nowadays it can be argued that overlays and other facilities allow to keep
  things outside the Nixpkgs monorepo. However, having a home management toolset
  inside the monorepo brings many advantages:
  
  - Direct access to the most recent enhancements available
    - New programs
    - New modules
  - No longer dealing with synchronization among projects
  - Better strategies for deduplication
    - More generally, better code sharing and refactoring
    
  Further, including the tool inside the monorepo brings to it a better
  reputation, since it is kept by the same organization of NixOS and Nixpkgs.

- The monorepo will increase in size.

  The increase in size is expected to be small. A good heuristic approximation
  of the worst case would be the current size of `nixos` directory. A quick `du`
  returns this:
  
  ```
  $> du -sh nixos/ pkgs/
  024572 nixos/
  379536 pkgs/
  ```
  
  `nixos` occupies less than 7% of the sum of both `nixos + pkgs`.
  
- Code duplications.

  They can be refactored later. Indeed it is an advantage of bringing them to
  the monorepo, because the duplicated code can be refactored more cleanly.

- Evaluation times of monorepo will increase.

  A small price to pay.

# Alternatives
[alternatives]: #alternatives

- The trivial "do nothing".

- Promote an alternative toolset.

  The most viable alternative in this field is home-manager. However it is not
  so easy to assimilate a consolidated project within another consolidated
  project.
  
  A middle approach, namely use home-manager as an inspiration and grabbing its
  most suitable ideas and code to have some quickstart, is a better idea.
  
- Promote `guix home`

  Kidding :)

# Prior art
[prior-art]: #prior-art

## Guix Home

As an example of prior art, there is our Scheme-based cousin, Guix Software
Distribution. Since at least 2022 AD they bring a similar tool, conveniently
called Guix Home.

The nicest thing about this tool is that it is tightly integrated with Guix, to
the point of `home` being a mere subcommand of `guix`.

## Home Manager

Home Manager needs no presentations.

# Unresolved questions
[unresolved]: #unresolved-questions

Given that this tool will live inside Nixpkgs monorepo, it is expected that
future packages will interact with this new tool. How those interactions should
be dealt?

Usually a maintainer of a package, say `firefox`, will be interested in being a
maintainer of both NixOS and NHMT modules.

# Future work
[future]: #future-work

- Update an extend the CI
- Set expectations on portability among present and future platforms Nixpkgs
  supports
  - Especially outside NixOS
  - Especially outside Linux
- Factor and abstract shared code among NixOS and NHMT

# References
[references]: #references

- [Home Manager](https://nix-community.github.io/home-manager/)
- [Keeping One's Home
  Tidy](https://guix.gnu.org/en/blog/2022/keeping-ones-home-tidy/), by Ludovic
  Courtès
- [Guix Home
  Configuration](https://guix.gnu.org/manual/devel/en/html_node/Home-Configuration.html)
- [Wrapper Manager](https://github.com/viperML/wrapper-manager)
- [Overlay for User Packages](https://gist.github.com/LnL7/570349866bb69467d0caf5cb175faa74) by LnL7
- [BuilEnv-based Declarative User
  Environment](https://gist.github.com/lheckemann/402e61e8e53f136f239ecd8c17ab1deb)
  by lheckellman
