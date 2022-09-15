# ResolutionExplosion

Ran into a strange issue at work where resolving gems in a bundle (not installing) was taking 3-5 minutes. It was not always this way, and it's unclear what is going on, but we did find a way to fix it that you'd never expect!


```
BUNDLER: Finished resolution (98145 steps) (Took 212.483564 seconds) (2022-09-15 13:48:59 -0400) this is where we started. That's an absurd amount of steps to resolve (and obviously a lot of time).
```

this was a gem, and had a dev dependency on rack AND a non-dev dependency on rails. Seems like rails will include rack anyway. Removing `s.add_development_dependency('rack')` changed the behavior like so:

```
BUNDLER: Finished resolution (289 steps) (Took 1.206765 seconds) (2022-09-15 14:11:36 -0400)
```

## Theory

rack 3.0.0 was released Sept 6th 2022 (today is Sept 15). This may be causing dependency resolution dead end from libraries that have not-yet been updated. Still, the explosion looks exponential and not what I would expect.

This same behavior has been verified on bundler versions verified same behavior on `2.3.11` and `2.3.22`

While removing the `rack` dependency works, pinning it to `'< 3.0.0'` also works.

I think this behavior is not a linear multiplier, but rather exponential based on my (unsharable) work project taking 100k steps (vs 250 steps). This sample reproduction only hits 800 steps (vs 100 steps).

I feel like this could be a pathological behavior that you'd be unlikely to encounter except that `rack` is a common dependency and there must be a combo of projects that are permissive and pinned to `< 3.0.0`


## Usage

```
DEBUG_RESOLVER=1 bundle lock --print | grep "Finished resolution"
```

Do this WITH and WITHOUT the gemspec having rack as a dev dependency. This reproduction is not as extreme as what I saw, but still significant:

With rack:
```
➜  resolution_explosion git:(main) ✗ DEBUG_RESOLVER=1 bundle lock --print | grep "Finished resolution"
:   0: Finished resolution (3 steps) (Took 0.00547 seconds) (2022-09-15 17:18:12 -0400)
BUNDLER: Finished resolution (816 steps) (Took 2.0365 seconds) (2022-09-15 17:18:16 -0400)
```

Without rack:
```
➜  resolution_explosion git:(main) ✗ DEBUG_RESOLVER=1 bundle lock --print | grep "Finished resolution"
:   0: Finished resolution (3 steps) (Took 0.005878 seconds) (2022-09-15 17:18:30 -0400)
BUNDLER: Finished resolution (123 steps) (Took 0.764866 seconds) (2022-09-15 17:18:33 -0400)
```
