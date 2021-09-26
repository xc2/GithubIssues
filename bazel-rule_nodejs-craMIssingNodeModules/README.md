# Bazel and `react-scripts start` related problems

## Workspace setup

See `WORKSPACE`.

```
Bazel: 4.2.1
bazel_rules_nodejs: 4.2.0
OS: macOS 12 x86_64
```

## Related rules:

- `react_scripts` in `@npm//react-scripts/index.bzl`, maybe `npm_package_bin`.

## Problem 1: Webpack didn't emit a rebuild when `index.tsx` changed.

### Related Target:

`//app:problem_1_webpack_watch_not_emit_rebuild`

Related definition(see `app/BUILD` for more):

```py
react_scripts(
    name = "problem_1_webpack_watch_not_emit_rebuild",
    args = [
        "--node_options=--require=./$(rootpath chdir.js)",
        "start",
    ],
    data = _RUNTIME_DEPS,
    tags = ["ibazel_notify_changes"],
)
```

### Minimal Reproduction Steps

1. Run `ibazel run //app:problem_1_webpack_watch_not_emit_rebuild`
2. Modify content of `app/src/index.tsx`

### What's expected

Step 1:

- ibazel running messages.
- Then console cleared with webpack building messages.

Step 2:

- ibazel rebuilding messages.
- Then console cleared with new webpack building messages.

### What's actually happened

Step 2:

Only ibazel rebuilding messages. No webpack rebuilding messages. (Webpack did not rebuild as well.)

## Problem 2: ibazel rebuild `react-scripts` related target would emit a typescript error.

### Related Target:

`//app:problem_2_bazel_build_not_emit_node_modules`

Related definition(see `app/BUILD` for more):

```py
react_scripts(
    name = "problem_2_bazel_build_not_emit_node_modules",
    args = [
        "--node_options=--require=../../chdir.js",
        "start",
    ],
    data = _RUNTIME_DEPS,
    tags = ["ibazel_notify_changes"],
)
```

### Minimal Reproduction Steps

1. Run `ibazel run //app:problem_2_bazel_build_not_emit_node_modules`
2. Modify content of `app/src/index.tsx`

### What's expected

Step 1:

- ibazel running messages.
- Then console cleared with webpack building messages.

Step 2:

- ibazel rebuilding messages.
- Then console cleared with new webpack building messages.

### What's actually happened

Step 2:

- ibazel rebuild messages.
- Then console cleared with webpack `Failed to compile` message:

```
Failed to compile.

undefined
TypeScript error in undefined(undefined,undefined):
Cannot find global type 'Array'.  TS2318
```

Seems to be lack of `typescript` module .

When I looked into workspace execroot, a difference between before step 2 and after step 2 was that `<execroot>/<workspace>/node_modules/` was missing after step 2.

### Another Reproduction Steps:

1. Run `bazel run //app:problem_2_bazel_build_not_emit_node_modules`
   a. `app/`, `bazel-out/`, `external/`, `node_modules/` can be found in workspace execroot.
2. Modify content of `app/src/index.tsx`
3. Run `bazel build //app:problem_2_bazel_build_not_emit_node_modules`
   a. Only `app/`, `bazel-out/`, `external/` can be found in workspace execroot.
