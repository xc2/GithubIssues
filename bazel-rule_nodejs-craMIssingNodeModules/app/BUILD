load("@bazel_skylib//rules:write_file.bzl", "write_file")
load("@build_bazel_rules_nodejs//:index.bzl", "copy_to_bin")
load("@npm//react-scripts:index.bzl", "react_scripts")

copy_to_bin(
    name = "copy_static_files",
    srcs = glob(
        [
            "public/*",
            "src/**/*",
            ".env.*",
            "*.env",
        ],
    ) + [
        "package.json",
        "tsconfig.json",
    ],
)

# react-scripts can only work if the working directory is the root of the application.
# So we'll need to chdir before it runs.
write_file(
    name = "write_chdir_script",
    out = "chdir.js",
    content = ["console.log(__dirname);process.chdir(__dirname);"],
)

_RUNTIME_DEPS = [
    "chdir.js",
    ":copy_static_files",
    "@npm//react",
    "@npm//react-dom",
]

react_scripts(
    name = "problem_1_webpack_watch_not_emit_rebuild",
    args = [
        "--node_options=--require=./$(rootpath chdir.js)",
        "start",
    ],
    data = _RUNTIME_DEPS,
    tags = [
        # This tag instructs ibazel to pipe into stdin a event describing actions.
        # ibazel send EOF to stdin by default and `react-scripts start` will stop when getting EOF in stdin.
        # So use this to prevent EOF.
        "ibazel_notify_changes",
    ],
)


react_scripts(
    name = "problem_2_bazel_build_not_emit_node_modules",
    args = [
        "--node_options=--require=../../chdir.js",
        "start",
    ],
    data = _RUNTIME_DEPS,
    tags = [
        # This tag instructs ibazel to pipe into stdin a event describing actions.
        # ibazel send EOF to stdin by default and `react-scripts start` will stop when getting EOF in stdin.
        # So use this to prevent EOF.
        "ibazel_notify_changes",
    ],
)
