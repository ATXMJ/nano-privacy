# nano development workspace

Two independent repositories for working on the Nano protocol — the original
C++ node and a community Rust port. Each is a personal fork with an `upstream`
remote pointing at the original source so upstream changes can be merged in.

## Repositories

### `nano-node/` — C++ node

| Remote   | URL                                              | Role            |
| -------- | ------------------------------------------------ | --------------- |
| `origin` | https://github.com/ATXMJ/nano-node.git           | personal fork   |
| `upstream` | https://github.com/nanocurrency/nano-node.git  | original source |

Fork of `nanocurrency/nano-node`. Build system is CMake; system prereqs are
minimal (build-essential, g++, wget, python3, zlib1g-dev, cmake, git) with the
rest pulled in as git submodules. See `nano-node/docker/ci/Dockerfile-base`
for the authoritative prereq list and `nano-node/ci/build.sh` for the build
flags. Submodules must be initialized before building:

```bash
git submodule update --init --recursive
```

### `rsnano-node/` — Rust port

| Remote   | URL                                                  | Role            |
| -------- | ---------------------------------------------------- | --------------- |
| `origin` | https://github.com/atxmj-rsnano/rsnano-node.git      | personal fork   |
| `upstream` | https://github.com/rsnano-node/rsnano-node.git     | original source |

Fork of `rsnano-node/rsnano-node`. Lives under the `atxmj-rsnano` org because
GitHub's fork network prevents forking two repos from the same network into one
account — `rsnano-node` is itself a fork of `nano-node`, so a second fork on
`ATXMJ` would collide with the existing `ATXMJ/nano-node` fork. The two repos
are treated as entirely separate projects despite the shared lineage.

## Syncing upstream changes

Both forks should be periodically merged with their upstreams to stay current
with upstream development and to carry a clean, attributable history.

```bash
# from within either repo
git fetch upstream
git checkout develop
git merge upstream/develop      # or: git rebase upstream/develop
git push origin develop
```
