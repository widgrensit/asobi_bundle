# asobi_bundle

[asobi](https://github.com/widgrensit/asobi) and every first-party extension,
as one dependency.

There is (almost) no code in this repository. There is an `.app.src` listing
applications, a `rebar.config` listing dependencies, and a `Dockerfile` +
`docker-publish.yml` that build the ready-to-run image. That is the whole
package.

```erlang
%% your_game/rebar.config
{deps, [{asobi_bundle, {git, "https://github.com/widgrensit/asobi_bundle.git", {tag, "v0.1.0"}}}]}.

{project_plugins, [{asobi, {git, "https://github.com/widgrensit/asobi.git", {tag, "v0.83.8"}}}]}.

{relx, [{release, {your_game, "1.0.0"}, [your_game, asobi_bundle, sasl]}]}.
```

First-party only, by construction. The bundle re-adds every application asobi
*ships* to a release closure; it does nothing for third-party extensions. A
self-hoster who wants a third-party (Tier E) extension adds its dependency and
`relx` entry by hand - the official image has no third-party install path.

| Application | What it is |
|---|---|
| [`asobi`](https://github.com/widgrensit/asobi) | core |
| [`asobi_quests`](https://github.com/widgrensit/asobi_quests) | quests and progress counters |
| [`asobi_seasons`](https://github.com/widgrensit/asobi_seasons) | wall-clock seasons |

## Why it exists

asobi cannot depend on its own extensions. Every extension depends on asobi,
so the reverse edge is a cycle that relx sorts into a build failure and Hex
rejects outright. That is deliberate - it is what keeps core from growing a
dependency on something built on top of it - but it means there is no single
package that pulls in the whole first-party set.

This is that package, and it needed to exist the moment anything was extracted
out of core. Before `asobi_seasons`, "everything asobi ships" and "asobi" were
the same list.

## The image

This repository builds `ghcr.io/widgrensit/asobi` - the ready-to-run node: the
game backend, the Lua runtime, the operator console, and every first-party
extension in one image. That image used to be published from the asobi repo,
back when the kernel and the batteries were the same list; since the first
extraction they diverge, so the batteries image is built here where the release
closure is `asobi` + every extension. The asobi repo no longer publishes an
image - assemblers who want a bare kernel build their own release from the
`asobi` dependency.

## Not to be confused with `asobi_engine_bundle`

Two different things wear the word "bundle":

- **`asobi_bundle`** (this repo) is an OTP meta-*application*: a build-time
  dependency closure that decides which extensions are compiled into a release.
- **`asobi_engine_bundle`** (a module inside `asobi_engine`) is a *runtime*
  concern: it fetches a tenant's Lua game code into the running node at boot.

One is what code is in the image; the other is what game runs on it.

## What it deliberately is not

- **Not a place for code.** It sits below asobi's extensions in the dependency
  order and above asobi. Anything living here would be reachable by neither.
- **Not a compatibility matrix.** It pins exactly one CI-verified row - one
  asobi minor plus the extension patches known to boot and pass `rebar3 asobi
  check` together - in `rebar.config` + a committed `rebar.lock`. When asobi's
  minor moves, the whole row moves with it. To hold one package back, depend on
  the extensions directly instead.
- **Not required.** A few lines in `deps` and `relx` do the same thing by hand.

## Validating the set

The one thing a bundle can do that a list of dependencies cannot is check
itself:

```sh
rebar3 asobi check
```

Every extension declares the tables, RPC prefixes, Lua namespaces and job
queues it owns. Run from here, the check sees all of them at once and fails if
two claim the same name or if one claims a name core reserves. Each extension
runs the same check in its own CI, but only against itself and core - a
collision between two extensions is visible from here first.

## Licence

Apache-2.0.
