This is a "jj-ified" fork of the obra/superpowers agentic skills plugin.

The original lives here: https://github.com/obra/superpowers

This local repo is a fork because I use `jj` (the Jujutsu VCS) and the original
superpowers assumes a lot of Git-specific commands and tactics.

So I forked it and am maintaining a local branch/patch that replaces references
to Git with jj.

The bookmark `jjify` always points to the head of the branch with the commits
that patch the repo (as relative to `main`).
