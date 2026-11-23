# Notes for AI coding agents

**Start with [docs/workflow.md](docs/workflow.md):** what to work on first (severity x reach x confidence, reach from
the hardware people run), the hardware tiers a change must not break, and the steps from claim to release. The open work, scored and sorted,
is in [docs/worklist.md](docs/worklist.md): pick from its top.

Follow [CONTRIBUTING.md](CONTRIBUTING.md): its rules apply to agents as to everyone, above all that no game files or
anything made from them leave the PC. The [development guide](docs/development.md) has the layout and the commands; the
[design notes](docs/design.md) have the architecture and the rules the GPU replay keeps.

What CONTRIBUTING.md does not spell out, and agents trip on:

- **Tests read the source.** Many checks in `tests/` cut a function out of a source file by its text and compile it
  with stubs (17 read `gpu/vk/vk_menu.cpp` alone). Renaming or reshaping a function can break a check far from your
  change: run `python -I -B tests/run_all.py` and update the check in the same PR rather than weakening it.
- **A `doc:` comment names a rule.** `// doc: docs/design.md#...` above a function says the rule it keeps is written
  there: read it before you simplify a capture, a wait or a pass split away. `tests/test_doc_anchors.py` checks that
  every anchor resolves, and that the functions behind design.md's rules keep theirs.
- **The renderer's interface is versioned.** A change to `MkwVkHost` or `MkwVkRenderer` in `gpu/vk/vk_api.h` bumps
  `MKWVK_API_VERSION`.
- **A switch has a row.** A new `RX_`, `HOS_` or `HOOKSHOT_` name read from the environment needs its row in
  `gpu/vk/switches.def`; a removed one becomes a `GONE(...)` row there (`tests/test_switch_table.py`).
- **Generated files are generated.** The SPIR-V headers in `gpu/vk/shaders/` (`*_spv.h`) are compiled from the shader
  beside them; regenerate them with glslang rather than editing them (the shaders' tests show the command).
- **Two platforms.** Windows builds with llvm-mingw, Linux with clang/gcc. Keep host code portable, and paths UTF-8
  (no `CP_ACP`; `tests/test_utf8_paths.py`).
- **No game in CI.** Checks must run without a dump. A run in the game is for you to do locally and report.
- **拉取新版本后编译前检查简中翻译。** 每次从上游拉取新版本后，在编译前必须检查 `gpu/vk/menu_i18n.h` 的 `kzh_Hans` 简体中文翻译表是否覆盖 `gpu/vk/vk_menu.cpp` 中新增的菜单项与说明文本；如有缺失先补全翻译（保持字典序并保留 printf 占位符与前后导空格），运行 `python -I tests/test_menu_i18n.py` 与 `python -I tests/test_menu_i18n_count.py` 校验通过后再进行编译。
- **Builds and game runs go through the PC's job queue** (`scripts/job_queue.py`): `--mode timing` for numbers,
  `--mode shared` for every other game run. Never on a RAM file system
  (`/tmp` is often one): [Sharing a PC](docs/development.md#sharing-a-pc-the-job-queue).
- **Test a change against a base, not in a new full build**: `scripts/dev_packs.py`, [Testing a change against a
  base](docs/development.md#testing-a-change-against-a-base). Agents sharing a PC share one base and agree before
  moving it; it is for development and A/B runs only. Bases, packs and runtimes are built from the game: never send them.

Pull requests: **open the description with why the PR is needed**, in one or two plain sentences, and **include
screenshots when the change shows on screen** (CONTRIBUTING.md, *Commits and pull requests*).

**A pull request is tested before it is opened, and its description carries the measurements.** Passing the checks
is not enough on its own:

- **Run the change in the game** and say where (the area, the GPU, the driver, the OS) and what you saw. If you
  could not run it, say so, and open it as a draft until someone has.
- **A performance change carries numbers from the same scene before and after**, on the same build otherwise: fps or
  frame time, the log line it moves (a slow submit, a stall, a timing), memory where it is about memory. Several
  runs or an in-session A/B, since a single run each way varies more than most changes gain.
- **A fix shows the bug gone:** the log line, the screenshot or the steps that failed before and pass now.
- **A new default** (a switch on for everyone, a preset, a limit) says which hardware it was measured on, and what it
  costs where it was not.
- Add a row to [tested configurations](docs/tested-configurations.md) for each run you measured.
