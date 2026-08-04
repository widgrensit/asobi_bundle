# asobi_bundle

[asobi](https://github.com/widgrensit/asobi) and every first-party extension,
as one dependency.

There is no code in this repository. There is an `.app.src` listing
applications and a `rebar.config` listing dependencies, and that is the whole
package.

```erlang
%% your_game/rebar.config
{deps, [{asobi_bundle, {git, "https://github.com/widgrensit/asobi_bundle.git", {branch, "main"}}}]}.

{project_plugins, [{asobi, {git, "https://github.com/widgrensit/asobi.git", {branch, "main"}}}]}.

{relx, [{release, {your_game, "1.0.0"}, [your_game, asobi_bundle, sasl]}]}.
```

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

## What it deliberately is not

- **Not a place for code.** It sits below asobi's extensions in the dependency
  order and above asobi. Anything living here would be reachable by neither.
- **Not a version policy.** It pins nothing that the individual packages do
  not. Depend on the extensions directly if you want to hold one back.
- **Not required.** Two lines in `deps` and two in `relx` do the same thing.

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
