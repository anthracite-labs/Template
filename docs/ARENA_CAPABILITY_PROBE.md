# Arena capability probe — this session, not a platform guarantee

- **Observed:** 2026-09-25, approximately 23:36–23:49 UTC (`date -u`); GitHub write observations added after the initial report commit.
- **Repository:** `anthracite-labs/Template`, `/home/user/Template`.
- **Actual work branch:** `arena/01a0daec-template`, initially at `f32732e46fd9e23d0540425b9865382e3a821bba`. The remote `arena-capability-probe` and `main` both pointed to that same SHA at probe start (`git ls-remote --heads origin`). Issue #2 asks for `arena-capability-probe` as PR head; this Arena session is bound to `arena/01a0daec-template`, so the PR head must differ. The starting commit is identical. Do not mistake this for a probe of a different repository revision.

## Executive summary

This particular sandbox is a small Debian 12 **KVM guest** (2 visible CPUs, 3.8 GiB RAM, about 20 GiB free on an ext4 filesystem). It runs native C/C++, Python, Node.js, Perl, Git, and a loopback HTTP server. User-local Python/npm installations work: `pipx` and `pnpm` were **absent before the test but installed and run in `/tmp`**; an npm-downloaded ELF executable ran. File and background-process persistence was observed **between tool calls in this session**, not across Arena sessions.

Network access is selective: GitHub's website/API, npm, and PyPI worked with certificate verification; many other resolved hosts returned `curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL` before HTTP, including Maven Central, Gradle, Google Maven/SDK downloads, Rust/Go registries, and GitHub release-asset hosts. The precise network-policy cause is **not established**. A reachable registry front page is not proof an artifact can be fetched; direct Maven/Google artifact GETs failed. A pinned JDK and Gradle distribution could be identified, but **not downloaded through the tested routes**. No JVM, Gradle, Android SDK/emulator, browser, or container daemon was preinstalled. This does **not** prove these toolchains or their builds are generally impossible here. `/dev/kvm` is absent, so accelerated Android emulation is unavailable in this guest; Android dependency resolution/builds/Robolectric remain separate, unverified questions.

This repository has **zero** GitHub Actions workflows, runs, and artifacts at probe time. Read-only repository/PR/Issues/Actions-list API calls worked, and the required Git push (both new and existing session branch) and PR creation succeeded, but `GET /repos/anthracite-labs/Template/actions/permissions` returned **403 `Resource not accessible by integration`**. It is incorrect to assume a usable canonical runner without checking. No destructive/admin/account action, workflow dispatch, merge, public tunnel, or heavyweight emulator download was performed.

**Classification vocabulary:** `confirmed` = successfully exercised; `confirmed_unavailable` = specifically tested operation or preinstalled path absent **in this session**; `installable` = missing initially but installed and exercised user-locally; `permission_restricted` = an authoritative permission/API denial; `not_tested` = no valid test of the operation; `session_specific` = a measured observation not a permanent Arena guarantee. A row can contain multiple statuses for different sub-capabilities. See the command/evidence beside each classification and the numbered details below.

## Capability matrix — all 22 Issue #2 areas

| # | Area | Classification and direct evidence |
|---|---|---|
| 1 | Host / OS / identity | `confirmed` — `uname -a; cat /etc/os-release; id; systemd-detect-virt` → Debian 12, x86_64, uid 1001, `kvm` guest; §1. |
| 2 | CPU / memory / disk / filesystem | `session_specific` — `nproc; free -h; df -h /tmp; df -i /tmp; findmnt -T /tmp` → 2 CPUs, 3.8 GiB, 20 GiB free, ext4; `confirmed` ≥4 MiB writable by `dd`/read/delete; §2. |
| 3 | Session lifecycle | `confirmed` within session — marker written then read in another call; `start_process` server answered in a later call. `not_tested` across sessions; §3. |
| 4 | Privilege / installability | `confirmed` — `id -u`=1001, `sudo -n true`=0; `installable` user-local packages in §11. Apt's attempted *scratch* metadata refresh failed TLS; privileged system update/install `not_tested`; §4. |
| 5 | Network | `session_specific` — verified HTTPS 200 at `github.com`, `api.github.com`, `registry.npmjs.org`, `pypi.org`; `curl` 35 at Maven/Gradle/Google and other hosts; `confirmed` small GitHub API file download; `confirmed_unavailable` only for tested `curl -6 https://github.com/`; §5. |
| 6 | Local servers | `confirmed` — `python3 -m http.server 18765 --bind 127.0.0.1` + later `curl http://127.0.0.1:18765/persist.txt` → 200; `not_tested` externally reachable preview/tunnel; §6. |
| 7 | Git | `confirmed` — clone, fetch dry run, status/diff/branch, `git commit` and `git push origin arena/01a0daec-template` (remote SHA verified); `confirmed_unavailable` preinstalled Git LFS; actual submodule fetching `not_tested`; §7/§8. |
| 8 | GitHub integration | `confirmed` file/Issue/PR/Actions-list reads, new **and existing** session-branch push, PR #3 → `main`; `permission_restricted` Actions settings GET 403; other writes classified individually in §8. |
| 9 | Variables / secrets | `confirmed` — variable **presence only** for `GITHUB_TOKEN`/`GH_TOKEN`, no values; no configured `git credential.helper`; authenticated API access is selective; §9. |
| 10 | Core CLIs | `confirmed` Bash, curl, wget, tar/zip/unzip/xz, jq, grep/sed/awk/find/xargs, make, gcc, OpenSSL, SSH, gh, Git; `confirmed_unavailable` on PATH for yq, cmake/ninja, clang, pkg-config, sqlite3 CLI, rsync, git-lfs; §10. |
| 11 | Runtimes / package managers | `confirmed` Python/pip, Node/npm/npx/yarn/corepack, Perl; `installable` `pipx` and `pnpm` into `/tmp`; `confirmed_unavailable` preinstalled JVM/Go/Rust/Ruby/PHP/.NET/Swift/Lua, etc.; **their installability not inferred**; §11. |
| 12 | Java / JDK | `confirmed_unavailable` preinstalled `java`, `javac`, `JAVA_HOME`; `session_specific` pinned Temurin archive GET redirected then TLS error 35; user-local JDK execution/Java compilation `not_tested`; §12. |
| 13 | Gradle / Maven | `confirmed_unavailable` preinstalled CLIs; `session_specific` distribution/Maven Central/Google Maven/Plugin Portal fetch failure; `confirmed` home cache *path* writable; Gradle execution/resolution/cache use `not_tested`; §13. |
| 14 | Android | `confirmed_unavailable` preinstalled SDK/adb/sdkmanager/emulator and `/dev/kvm` hardware acceleration; `session_specific` SDK URL TLS failure; compilation, unit/Robolectric tests, adb connection, emulator run `not_tested`; §14. |
| 15 | Native toolchain | `confirmed` — `gcc` and `g++` compiled/ran hello, make/ld/as and libc header available; `confirmed_unavailable` preinstalled clang/cmake/ninja/pkg-config; §15. |
| 16 | Containers / virtualization | `confirmed` KVM guest and `unshare -Ur true` succeeded; `confirmed_unavailable` Docker/Podman/etc. CLIs and local daemon sockets on checked paths; actual nested container execution `not_tested`; §16. |
| 17 | Browser / web testing | `confirmed_unavailable` preinstalled browser and headless display on checked paths; `installable` `playwright-core` **library** only; smoke test/screenshots `not_tested` (no browser binary); §17. |
| 18 | Databases / services | `confirmed` Python SQLite in-memory query; `confirmed_unavailable` preinstalled SQLite CLI and checked PostgreSQL/MySQL/Redis clients/servers; service startup `not_tested`; §18. |
| 19 | Process / runtime | `confirmed` child/signal, background server, loopback, inotify event; `session_specific` open files 1024 / user processes 15734 / watches 65536; long-run ceiling `not_tested`; §19. |
| 20 | Time / locale | `session_specific` — UTC date 2026-09-25, `/etc/localtime` UTC, POSIX C-type locale; clock accuracy against independent reference `not_tested`; §20. |
| 21 | Archives / artifacts | `confirmed` tar+zip round trips, files committed and pushed (report SHA verified remotely); Actions artifact listing `confirmed` (zero); artifact download/max large size `not_tested`; §21. |
| 22 | Limits / unknowns | `not_tested` cross-session lifespan, emulator/software fallback, JVM build, CI execution and unnecessary/forbidden GitHub mutations (merge/admin/workflow writes); `session_specific` all measured versions/resources/network results; §22. |

## Evidence and exact commands

The read-first files `.agents/ARENA-DISPATCH.md`, `.agents/CAPABILITIES.md`, `docs/PROJECT_STATE.md` and the invoked `prototype` skill were inspected before probing. This is an environment experiment, not a product UI/state-design prototype; its throwaway code lives only in scratch and the answer is this report. Commands below were run from `/home/user/Template` unless the path or note says otherwise. The disposable scratch directory was `/tmp/arena-audit-01a0daec` (`umask 077`); its contents are **not** part of the PR. The installed versions are recorded as dated evidence, not promised by the template. Errors/HTTP status matter: do not reinterpret TLS-handshake errors as missing DNS, bad credentials, or rejected certificates.

### 1. Host / OS / identity

```bash
date -u '+UTC %Y-%m-%dT%H:%M:%SZ'; date '+LOCAL %Y-%m-%dT%H:%M:%S%z %Z'
uname -a; cat /etc/os-release; uname -m; hostname; id; whoami
printf 'shell=%s home=%s pwd=%s\n' "$SHELL" "$HOME" "$PWD"
cat /proc/1/cgroup; systemd-detect-virt; tr '\0' ' ' </proc/1/cmdline
```

`confirmed`: `Linux e2b.local 6.1.158+ ... x86_64`; `/etc/os-release`: `Debian GNU/Linux 12 (bookworm)`; `uid=1001(user) gid=1001(user) groups=...27(sudo)`; `/bin/bash`, `/home/user`, `/home/user/Template`; `/proc/1/cgroup` = `0::/init.scope`, PID 1 = `/sbin/init`, `systemd-detect-virt` = `kvm`. Thus virtualization is observed, but a Docker-style container is not demonstrated. Directory listing of `/home/user` showed `Template` and normal shell dotfiles; no other project checkout was assumed. Hostname and kernel are `session_specific`.

### 2. CPU, memory, disk, filesystem and temp

```bash
nproc; lscpu | grep -E '^(Architecture|CPU\(s\)|Model name|Hypervisor vendor|Virtualization type)'
free -h; df -h /home/user/Template /home/user /tmp /; df -i /home/user/Template /home/user /tmp /
findmnt -T /home/user/Template -o TARGET,SOURCE,FSTYPE,OPTIONS
findmnt -T /tmp -o TARGET,SOURCE,FSTYPE,OPTIONS
stat -c '%n %d (%D)' /home/user/Template /home/user /tmp; du -sh .
mkdir -p /tmp/arena-audit-01a0daec
dd if=/dev/zero of=/tmp/arena-audit-01a0daec/four-mib.bin bs=1048576 count=4 status=none
wc -c /tmp/arena-audit-01a0daec/four-mib.bin
rm /tmp/arena-audit-01a0daec/four-mib.bin
```

`session_specific`: `nproc`=2, Xeon virtual CPU, 3.8 GiB memory (~3.6 GiB available at measurement), **0 swap**. `/dev/root` on ext4 `rw,relatime,discard`: 21 GiB total / ~20 GiB available / 4% used; 5,714,800 inodes, ~1% used. Repo, home and `/tmp` all had device ID `65024 (fe00)` and resolve to the same `/` mount. `du -sh .` ~884 KiB at start. `confirmed`: a 4,194,304-byte temp file was written/read and deleted. This is a **lower bound**, not a maximum writable-file-size test; the disk was not filled. Writable home path was also exercised in §13. Mount output was inspected only for safe filesystem/type/options, not sensitive mount data.

### 3 & 6. Persistence and listening sockets

```bash
printf 'session-marker-from-command-1\n' > /tmp/arena-audit-01a0daec/persist.txt
# In a separate tool invocation:
cat /tmp/arena-audit-01a0daec/persist.txt
# Started with Arena start_process (not a short-lived bash background job):
python3 -m http.server 18765 --bind 127.0.0.1 --directory /tmp/arena-audit-01a0daec
# In another tool invocation:
curl -fsS --connect-timeout 3 --max-time 5 -w 'HTTP=%{http_code}\n' http://127.0.0.1:18765/persist.txt
ss -lnt '( sport = :18765 )'
```

`confirmed`: marker read back unchanged; server was reported listening on `127.0.0.1:18765` and returned marker with `HTTP=200` in a later invocation; server was then stopped through Arena's process tool. IPv6 `curl http://[::1]:18765/` failed with curl 7 because this server bound only IPv4 loopback. `not_tested`: public preview, tunnels, maximum process lifespan, restart persistence, and storage between **different Arena sessions**. No public tunnel/service was created. No session-lifetime deadline was evident in the checked metadata.

### 4. Privilege and package installation safety

```bash
id -u; command -v sudo; sudo -n true; command -v apt-get; command -v apt
apt-cache stats; apt-get -s install openjdk-17-jdk-headless
# Only a bounded UNPRIVILEGED metadata attempt, with a PUBLIC source in scratch:
scratch=/tmp/arena-audit-01a0daec/apt
mkdir -p "$scratch/lists/partial" "$scratch/cache/archives/partial"
printf 'deb https://deb.debian.org/debian bookworm main\n' > "$scratch/sources.list"
timeout 35 apt-get update -o Dir::Etc::sourcelist="$scratch/sources.list" -o Dir::Etc::sourceparts=- -o Dir::State::Lists="$scratch/lists" -o Dir::Cache="$scratch/cache" -o Dir::Cache::archives="$scratch/cache/archives" -o APT::Get::List-Cleanup=false -o Debug::NoLocking=1 -o Acquire::Retries=0 -o Acquire::https::Timeout=8
```

`confirmed`: not root; passwordless `sudo -n true` exit 0 (the **only** sudo invocation), apt/apt-get installed; other queried OS managers `dnf`, `yum`, `apk`, `pacman`, `brew` absent. `apt-get -s install` returned exit 100 / `Unable to locate package openjdk-17-jdk-headless` with currently empty package lists. The scratch metadata attempt reported `Could not handshake: The TLS connection was non-properly terminated` to Debian and downloaded **zero** index files. It exited 0 despite `W: Failed to fetch ...` (exit code alone is misleading). Apt also attempted a cleanup at `/var/cache/apt/archives/partial` and got `Permission denied`; it did not install packages or change system package indexes. The first scratch attempt omitted the `Dir::Cache::archives` override; repeating with the override above produced the same TLS failure and cleanup warning. **No privileged apt update/install was attempted**; feasibility after different network access or properly isolated metadata refresh is `not_tested`. User-local successful installs are in §11.

### 5. DNS, HTTP, HTTPS, IPv4/IPv6, TLS and download

```bash
getent ahostsv4 github.com; getent ahostsv6 github.com
getent ahostsv4 registry.npmjs.org
# Same command form for each public host in the table:
curl -sS -I -L --max-redirs 3 --connect-timeout 5 --max-time 15 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result} redirects=%{num_redirects}\n' https://repo.maven.apache.org/maven2/
curl -4 -sS -I --connect-timeout 5 --max-time 10 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://github.com/
curl -6 -sS -I --connect-timeout 5 --max-time 10 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://github.com/
curl -sS -I -L --max-redirs 2 --connect-timeout 5 --max-time 12 -o /dev/null -w 'HTTP=%{http_code} redirects=%{num_redirects}\n' http://github.com/
```

| Tested target (`curl -I`, normal certificate verification) | Observation (`session_specific`) |
|---|---|
| `https://github.com/`, `https://api.github.com/`, `https://registry.npmjs.org/`, `https://pypi.org/` | HTTP 200, curl exit 0, TLS verify result 0. |
| `https://files.pythonhosted.org/` | HTTP 404 at root **with TLS verify 0**; host was reached. Actual Python/npm package GETs succeeded (§11). |
| `https://raw.githubusercontent.com/`, `https://objects.githubusercontent.com/`, `https://services.gradle.org/`, `https://downloads.gradle.org/`, `https://repo.maven.apache.org/maven2/`, `https://repo1.maven.org/maven2/`, `https://dl.google.com/`, `https://plugins.gradle.org/m2/` | DNS resolved, then curl exit **35**, `OpenSSL SSL_connect: SSL_ERROR_SYSCALL`, no HTTP response. Direct artifact GET failures are in §§12–14. |
| `https://crates.io/`, `https://static.crates.io/`, `https://proxy.golang.org/`, `https://registry-1.docker.io/v2/`, `https://example.com/` | Same TLS-handshake error 35, no HTTP status. Not a general statement about all hosts/registries. |
| `http://example.com/` | Curl exit 52, `Empty reply from server`; HTTP is not generically permitted. |
| `http://github.com/` | HEAD 301 to HTTPS; `curl -I -L` followed 1 redirect to HTTP 200 / TLS verify 0. |
| `curl -4 ... https://github.com/`; `curl -6 ... https://github.com/` | IPv4 200; forced IPv6 exit 7 `Couldn't connect to server`. Only this host/path was tested. |

`getent` returned IPv4 addresses for GitHub and npm; a separate `.invalid` name did not resolve. `openssl s_client -connect services.gradle.org:443 -servername services.gradle.org -brief` returned `unexpected eof while reading`, consistent with a handshake interruption, **not proof of its cause**. A GitHub archive HEAD redirected to `codeload.github.com` (not downloaded). To test an actual small file, this was executed:

```bash
curl -sS -f --connect-timeout 5 --max-time 15 -o /tmp/arena-audit-01a0daec/LICENSE-response.json -w 'HTTP=%{http_code} bytes=%{size_download} TLS=%{ssl_verify_result}\n' 'https://api.github.com/repos/anthracite-labs/Template/contents/LICENSE?ref=main'
jq -r '.content' /tmp/arena-audit-01a0daec/LICENSE-response.json | base64 -d > /tmp/arena-audit-01a0daec/public-LICENSE
cmp -s LICENSE /tmp/arena-audit-01a0daec/public-LICENSE
```

`confirmed`: HTTP 200, 2,347 response bytes, TLS 0, decoded public file equals local `LICENSE`. Direct GET of `https://raw.githubusercontent.com/anthracite-labs/Template/main/LICENSE` failed with curl 35. No HTTP proxy environment variables were set in the checked list (§9). No inference is made about external DNS policy, other times, or TLS certificate failures at blocked hosts.

### 7. Git

```bash
git --version; git status -sb; git diff --stat; git branch -av; git remote -v
git ls-remote --heads origin arena-capability-probe main
git fetch --dry-run origin main
git clone -q --depth 1 --branch main https://github.com/anthracite-labs/Template.git /tmp/arena-audit-01a0daec/clone
git -C /tmp/arena-audit-01a0daec/clone rev-parse --short HEAD
git lfs version; git submodule status
GIT_TERMINAL_PROMPT=0 git push --dry-run origin HEAD:refs/heads/arena/01a0daec-template
```

`confirmed`: Git 2.39.5; clean initial worktree; shallow clone and fetch dry run exit 0; clone HEAD `f32732e`; HTTPS `origin` points at `https://github.com/anthracite-labs/Template.git`; local `main`, `origin/main`, `arena/01a0daec-template` and remote `arena-capability-probe` initially shared the same SHA. `git submodule status` exited successfully but this checkout has no submodules to actually fetch. `git lfs version` returned `git: 'lfs' is not a git command`. Push dry-run printed `[new branch] HEAD -> arena/01a0daec-template` and exited 0 **without writing a remote ref** (`ls-remote` still showed only `arena-capability-probe` and `main`). The subsequent **actual** report commit/push created the session branch; see §8. A dry run alone is never proof of write permission.

### 8. GitHub integration / permission surface

Read-only REST commands and responses (no authentication material printed):

```bash
gh api repos/anthracite-labs/Template --jq '{full_name,default_branch,private,permissions}'
gh api repos/anthracite-labs/Template/contents/docs/PROJECT_STATE.md --jq '{path,size,type}'
gh api repos/anthracite-labs/Template/issues/2 --jq '{number,state,title,comments}'
gh api repos/anthracite-labs/Template/issues/2/labels --jq '[.[].name]'
gh api repos/anthracite-labs/Template/pulls/1/reviews --jq '[.[]|{id,state}]'
gh api repos/anthracite-labs/Template/pulls/1/comments --jq '[.[]|{id,path}]'
gh api repos/anthracite-labs/Template/commits/f32732e46fd9e23d0540425b9865382e3a821bba/check-runs --jq '{total_count}'
gh api repos/anthracite-labs/Template/actions/workflows --jq '{total_count}'
gh api repos/anthracite-labs/Template/actions/runs --jq '{total_count}'
gh api repos/anthracite-labs/Template/actions/artifacts --jq '{total_count}'
gh api repos/anthracite-labs/Template/actions/permissions
gh api -i repos/anthracite-labs/Template/actions/permissions 2>&1 | grep -Ei 'HTTP/|x-accepted-github-permissions|Resource not accessible'
gh api repos/anthracite-labs/Template/installation --jq '{permissions,repository_selection,app_slug}'
gh api user --jq '{login,type}'
```

`confirmed`: repository file/Issue/label/merged PR metadata were readable (issue 2 open, label `ready-for-agent`); existing PR #1 reviews/comments endpoints returned `[]`; main commit check-runs returned count 0; workflow/run/artifact list endpoints each returned count **0**. Empty lists prove the endpoint was reachable, **not** that populated reviews, logs or artifacts can be retrieved. `permission_restricted`: `gh api .../actions/permissions` → HTTP **403** with `{"message":"Resource not accessible by integration"}` and header `X-Accepted-Github-Permissions: administration=read`; `gh api user` returned the same 403 for the authenticated-user endpoint. A separate read-only `GET /repos/anthracite-labs/Template/installation` failed **401** `A JSON web token could not be decoded` (that endpoint expects an App JWT, not this session's installation-style token); no installation permissions could be enumerated via that route. This is not evidence of a broken GitHub connection: repository reads, push and PR creation worked. Repository JSON returned `permissions: {admin:false,maintain:false,pull:false,push:false,triage:false}`; for this app/integration context, those user-oriented fields **did not predict actual write capability**: the required Git push and PR creation both succeeded. Use the real permission response or a necessary authorized write, not these flags alone.

| Requested GitHub ability | Classification after authorized deliverable writes | Specific evidence / safety boundary |
|---|---|---|
| Read repository files / Issues / labels | `confirmed` | `gh api .../contents/docs/PROJECT_STATE.md` → 200/path/size; issue #2 and labels readable. |
| Read PR reviews / PR comments / checks | `confirmed` for GET endpoints | PR #1 reviews `[]`, comments `[]`, main check-runs count 0. No populated review or check payload exists to inspect. |
| Read Actions run metadata / artifact metadata | `confirmed` for list endpoints | `gh api .../actions/runs`, `.../artifacts` → 200 / `total_count:0`. |
| Read workflow job logs / download artifacts | `not_tested` | Zero runs/jobs/artifacts; no valid ID to fetch; permissions for these specific downloads unverified. |
| Commit locally / push ordinary files / create the allowed remote **session** branch | `confirmed` | `git commit` → `1682a39c425e38f7487861d78e16380d2973ac12`; `git push origin arena/01a0daec-template` → `[new branch]`; `git ls-remote` verified the same SHA. |
| Push an **existing** branch | `confirmed` | Second report commit `ded45ee790b83362ac44924e59e8f4bd6254f35a`: `GIT_TERMINAL_PROMPT=0 git push origin arena/01a0daec-template` → `1682a39..ded45ee`; `git ls-remote` and later PR GET confirmed the updated SHA. Only the allowed session branch was used. |
| Open PR targeting `main` | `confirmed` | `gh pr create --repo anthracite-labs/Template --base main --head arena/01a0daec-template ...` → [PR #3](https://github.com/anthracite-labs/Template/pull/3); `gh api .../pulls/3` → state `open`, `merged:false`, correct head/base. |
| Create a new *other* branch | `not_tested` | Only the permitted session branch was created; this session may not create another branch merely to probe. |
| Comment on a PR or Issue / write Issues or labels | `not_tested` | Would create visible content or change repository state merely to probe; read-only list is not proof of write. A completion report in the PR body does not test commenting. |
| Dispatch workflow / rerun / cancel workflow | `not_tested` | No workflows or runs; dispatch/rerun/cancel would have side effects. |
| Write Actions workflow file | `not_tested` | Out of scope of this documentation-only change; no workflow-file write test. |
| Merge PR | `not_tested` | Explicitly forbidden; no merge attempted. |
| Modify repository/Actions settings | `permission_restricted` for **reading Actions settings**; writes `not_tested` | Actions settings GET 403 `Resource not accessible by integration`, required permission header `administration=read`. No admin write attempted; does not establish every repository setting's permission individually. |

**Post-write evidence (2026-09-25 ~23:46–23:47 UTC):**

```bash
git add -- docs/ARENA_CAPABILITY_PROBE.md .agents/ARENA-DISPATCH.md
git diff --cached --check
git commit -m 'Document live Arena capability audit and update dispatch routing'
GIT_TERMINAL_PROMPT=0 git push origin arena/01a0daec-template
git ls-remote --heads origin arena/01a0daec-template arena-capability-probe main
gh pr create --repo anthracite-labs/Template --base main --head arena/01a0daec-template --title 'Audit live Arena sandbox capabilities (Issue #2)' --body-file /tmp/arena-audit-01a0daec/pr-body.md
gh api repos/anthracite-labs/Template/pulls/3 --jq '{number,state,merged,head:.head.ref,head_sha:.head.sha,base:.base.ref,html_url}'
```

`confirmed`: local commit `1682a39c425e38f7487861d78e16380d2973ac12` contained the new report and dispatch changes; actual push exit 0 created remote `arena/01a0daec-template`; `ls-remote` matched that SHA; `gh pr create` exit 0 returned `https://github.com/anthracite-labs/Template/pull/3`; PR GET reports `state:open`, `merged:false`, head `arena/01a0daec-template`, base `main`. Remote `arena-capability-probe` and `main` remained at starting SHA `f32732e46fd9e23d0540425b9865382e3a821bba`. `gh pr view 3 --json statusCheckRollup` returned zero checks. This is the actual safe write test on the necessary deliverables, not a gratuitous Issue/admin mutation. After PR creation, the report was updated and pushed again:

```bash
git add -- docs/ARENA_CAPABILITY_PROBE.md
git diff --cached --check
git commit -m 'Record actual GitHub write permissions in capability report'
GIT_TERMINAL_PROMPT=0 git push origin arena/01a0daec-template
git ls-remote --heads origin arena/01a0daec-template
```

`confirmed`: second commit `ded45ee790b83362ac44924e59e8f4bd6254f35a` updated the report; push to the **existing** remote session branch exited 0 with `1682a39..ded45ee`; `git ls-remote` returned the new SHA. A subsequent PR GET showed `commits:2`, `head_sha:ded45ee...`, `state:open`, `merged:false`; the immediately preceding GET briefly showed the prior head SHA, so remote-ref verification was used rather than trusting an instantaneous PR metadata read. The PR body carries the final report-bearing SHA and completion findings.

### 9. Variables / secrets boundary

```bash
for name in JAVA_HOME ANDROID_HOME ANDROID_SDK_ROOT GRADLE_USER_HOME CI ARENA_SESSION_ID GITHUB_ACTIONS GITHUB_TOKEN GH_TOKEN GIT_ASKPASS SSH_AUTH_SOCK HTTP_PROXY HTTPS_PROXY ALL_PROXY NO_PROXY; do
  if [ "${!name+x}" ]; then printf '%s: set (redacted)\n' "$name"; else printf '%s: unset\n' "$name"; fi
done
test -n "$(git config --get credential.helper 2>/dev/null)" && echo yes || echo no
```

`confirmed`: variables exist (`env | wc -l`=16); `GITHUB_TOKEN` and `GH_TOKEN` are **set; their values were never printed or recorded**. The checked CI/Arena-session, Java/Android, proxy, SSH agent and Git askpass variables were unset. No configured `git credential.helper` was found (this does not mean no other authentication integration exists); `gh api` can access selected endpoints. No credential file contents, complete environment dump, tokens or private keys were read. Presence/absence is `session_specific`.

### 10 & 11. Core CLIs, runtimes and user-local installs

```bash
# Inventory used command -v for each name (no invocation for missing tools):
for bin in bash sh zsh curl wget tar zip unzip xz jq yq grep sed awk find xargs make cmake ninja gcc g++ clang clang++ ld as pkg-config sqlite3 openssl ssh scp rsync gh git git-lfs python3 python pip3 pip pipx uv node npm npx pnpm yarn bun java javac gradle mvn kotlinc go rustc cargo rustup ruby gem bundle php composer dotnet swift perl lua sudo apt-get apt dnf yum apk pacman brew docker podman buildah containerd nerdctl chromium chromium-browser google-chrome firefox playwright Xvfb adb sdkmanager emulator psql postgres mysql mysqld redis-cli redis-server; do command -v "$bin" >/dev/null || :; done
git --version; gh --version; curl --version; gcc --version; make --version
python3 --version; pip3 --version; node --version; npm --version
yarn --version; corepack --version; perl -e 'print "$^V\n"'
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install -q --no-cache-dir --no-deps --index-url https://pypi.org/simple --target /tmp/arena-audit-01a0daec/python 'packaging==24.2'
PYTHONPATH=/tmp/arena-audit-01a0daec/python python3 -c 'import packaging; print(packaging.__version__)'
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install -q --no-cache-dir --index-url https://pypi.org/simple --target /tmp/arena-audit-01a0daec/pipx 'pipx==1.7.1'
PYTHONPATH=/tmp/arena-audit-01a0daec/pipx python3 -m pipx --version
npm --prefix /tmp/arena-audit-01a0daec/pnpm install --no-save --no-audit --no-fund --registry=https://registry.npmjs.org 'pnpm@9.15.9'
/tmp/arena-audit-01a0daec/pnpm/node_modules/.bin/pnpm --version
npm --prefix /tmp/arena-audit-01a0daec/esbuild install --no-save --no-audit --no-fund --registry=https://registry.npmjs.org 'esbuild@0.25.5'
/tmp/arena-audit-01a0daec/esbuild/node_modules/@esbuild/linux-x64/bin/esbuild --version
python3 -c 'p="/tmp/arena-audit-01a0daec/esbuild/node_modules/@esbuild/linux-x64/bin/esbuild"; print(open(p,"rb").read(4).hex())'
```

`confirmed` preinstalled: Bash/sh, curl 7.88.1, wget, archive and POSIX CLI tools, jq, make 4.3, gcc/g++ 12.2, ld/as, OpenSSL, ssh/scp, gh 2.23.0, Git 2.39.5, Python 3.11.2/pip 23.0.1, Node v22.22.3/npm 10.9.8/npx, yarn 1.22.22/corepack 0.34.6, Perl v5.36.0. `installable`, **not preinstalled**: pipx 1.7.1 and pnpm 9.15.9 worked under `/tmp`; `packaging` 24.2 imported; npm-downloaded esbuild 0.25.5 ran as a Linux ELF (`7f454c46` magic). This proves small user-local installs and execution of one downloaded native binary, not arbitrary binary trust or universal registry reachability.

`confirmed_unavailable` *on checked PATH*: zsh, yq, cmake, ninja, clang/clang++, pkg-config, sqlite3 CLI, rsync, git-lfs, uv, bun, Java/Javac, Gradle, Maven, kotlinc, Go, Rust/cargo/rustup, Ruby/gem/bundler, PHP/Composer, dotnet, Swift, Lua. `pipx`/`pnpm` moved to `installable` after the scratch tests; the others' installation via a different mirror/package source is `not_tested`. HEAD requests to official Lua, Rust static, Go, dotnet, PHP, RubyGems and Debian hosts also returned curl 35 (§5), so this session's tested direct distribution paths did not prove installability. **Missing on PATH is not proven unavailable as a runtime after safe installation.**

### 12. Java / JVM

```bash
command -v java; command -v javac; java -version; javac -version
# No JAVA_HOME value printed; only presence check in §9.
ls /usr/lib/jvm  # no such directory here
gh api repos/adoptium/temurin17-binaries/releases/latest --jq '{tag_name,linux:[.assets[]|select(.name|test("^OpenJDK17U-jdk_x64_linux_hotspot_.*\\.tar\\.gz$"))|{name,size}][0:1]}'
curl -sS -L -r 0-1023 --max-redirs 2 --max-filesize 1048576 --connect-timeout 5 --max-time 20 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.20.1%2B1/OpenJDK17U-jdk_x64_linux_hotspot_17.0.20.1_1.tar.gz'
```

`confirmed_unavailable` preinstalled JVM/JDK in checked locations: both commands missing, `JAVA_HOME` unset, no `/usr/lib/jvm`. `session_specific` download route: public release metadata identified **pinned** Temurin `jdk-17.0.20.1+1`, archive size 193,252,603 bytes; the bounded GET got a GitHub 302 then curl **35** on the release-asset CDN, 0 archive bytes. `https://api.adoptium.net/v3/binary/latest/17/ga/linux/x64/jdk/hotspot/normal/eclipse` HEAD also failed 35. No archive was downloaded/unpacked; Java hello-world compilation/execution is therefore `not_tested`, **not** `confirmed_unavailable` for every possible installation route. No large fallback JDK install was made.

### 13. Gradle / Maven / dependency resolution

```bash
command -v gradle; command -v mvn; ls "$HOME/.gradle"  # absent at start
curl -sS -I -L --max-redirs 4 --connect-timeout 6 --max-time 22 -o /dev/null -w 'HTTP=%{http_code}\n' 'https://services.gradle.org/distributions/gradle-8.14.3-bin.zip'
curl -sS -L -r 0-1023 --max-redirs 2 --max-filesize 1048576 --connect-timeout 5 --max-time 20 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://github.com/gradle/gradle-distributions/releases/download/v8.14.3/gradle-8.14.3-bin.zip'
curl -sS -f -L --max-filesize 2097152 --connect-timeout 5 --max-time 15 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://repo.maven.apache.org/maven2/org/apache/commons/commons-lang3/3.14.0/commons-lang3-3.14.0.jar'
curl -sS -f -L --max-filesize 2097152 --connect-timeout 5 --max-time 15 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://dl.google.com/dl/android/maven2/com/android/tools/build/gradle/maven-metadata.xml'
curl -sS -f -L --max-filesize 2097152 --connect-timeout 5 --max-time 15 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://plugins.gradle.org/m2/org/jetbrains/kotlin/kotlin-gradle-plugin/maven-metadata.xml'
probe="$HOME/.gradle/caches/arena-audit-01a0daec"; mkdir -p "$probe"
printf 'home-cache-marker\n' > "$probe/marker"; cat "$probe/marker"
rm "$probe/marker"; rmdir "$probe" "$HOME/.gradle/caches" "$HOME/.gradle"
```

`confirmed_unavailable` preinstalled Gradle/Maven. `session_specific`: both Gradle distribution hosts' HEADs returned curl 35; bounded pinned GitHub asset GET redirected then failed 35 at release-asset host; Maven Central jar GET, Google Maven metadata GET, Plugin Portal metadata GET all returned curl **35**, zero bytes. Consequently one small dependency was **not** resolved; a tiny scratch Gradle build, Gradle execution, and real cache writes by Gradle are `not_tested`. `confirmed` only that the conventional `$HOME/.gradle/caches/...` location accepts a marker write/read and empty directories were removed. Cache/session persistence or distribution download through untested mirrors is **unknown**, not impossible. A successful front-page HEAD elsewhere would not establish artifact downloads.

### 14. Android capability distinctions

```bash
command -v sdkmanager; command -v adb; command -v emulator
for p in /opt/android-sdk /opt/android /usr/local/lib/android /usr/lib/android-sdk /home/user/Android/Sdk /dev/kvm; do test -e "$p" && ls -ld "$p" || echo "$p MISSING"; done
# ANDROID_HOME and ANDROID_SDK_ROOT presence was checked, not printed (§9).
curl -sS -I -L --max-redirs 4 --connect-timeout 6 --max-time 22 -o /dev/null -w 'HTTP=%{http_code}\n' 'https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip'
```

`confirmed_unavailable` preinstalled Android SDK tools/build-tools/platforms in checked locations, `sdkmanager`, `adb`, `emulator`, Android env variables, and **`/dev/kvm`**. `/dev/kvm` absence establishes no KVM-backed accelerated emulator in this guest; it does not establish that software emulation could never work. `session_specific`: tested official command-line-tools archive route failed curl 35; no large SDK/system-image install. The following remain separately `not_tested`: **Android dependency resolution**, **Android compilation**, **JVM unit/Robolectric tests**, **adb connection**, **emulator/managed-device run**. Failure to install tools or run an emulator is **not** evidence of inability to resolve Android dependencies or compile on a different network/runner.

### 15 & 16. Native builds, containers and virtualization

```bash
printf '#include <stdio.h>\nint main(void){ puts("hello native probe"); return 0; }\n' | gcc -x c - -o /tmp/arena-audit-01a0daec/hello
/tmp/arena-audit-01a0daec/hello
printf '#include <iostream>\nint main(){std::cout << "hello C++ probe\\n";}\n' | g++ -x c++ - -o /tmp/arena-audit-01a0daec/hello-cpp
/tmp/arena-audit-01a0daec/hello-cpp
make --version; ld --version; test -f /usr/include/stdio.h
command -v docker; command -v podman; command -v buildah; command -v containerd; command -v nerdctl
test -e /var/run/docker.sock; test -e /run/podman/podman.sock; test -e /dev/kvm
unshare -Ur true
grep -E '^(Seccomp|NoNewPrivs|CapEff):' /proc/self/status
```

`confirmed`: GCC/G++ compiled and ran `hello native probe` / `hello C++ probe`; GNU make 4.3, GNU ld 2.40, `as`, libc header. `confirmed_unavailable` preinstalled clang, cmake, ninja, pkg-config. `unshare -Ur true` exited 0, demonstrating unprivileged **user namespaces**; process status showed `Seccomp: 0`, `CapEff: 0000000000000000`. `systemd-detect-virt`=kvm reflects the guest, but `/dev/kvm` absent. Docker/Podman/Buildah/containerd/nerdctl CLIs and tested daemon sockets absent. **No image pull, nested-container start, or container security inference** from CLI absence: execution is `not_tested`.

### 17 & 18. Browser, screenshots, databases, services

```bash
command -v chromium; command -v chromium-browser; command -v google-chrome
command -v firefox; command -v MiniBrowser; command -v Xvfb; command -v playwright
python3 -c 'import importlib.util; print({n: bool(importlib.util.find_spec(n)) for n in ("playwright","selenium","sqlite3","psycopg2","pymysql","redis")})'
npm list -g --depth=0 --json | jq -c '.dependencies|keys'
npm --prefix /tmp/arena-audit-01a0daec/playwright install --no-save --no-audit --no-fund --ignore-scripts --registry=https://registry.npmjs.org 'playwright-core@1.55.0'
node -e 'const p="/tmp/arena-audit-01a0daec/playwright/node_modules/playwright-core"; const fs=require("fs"); const pw=require(p); console.log(require(p+"/package.json").version,fs.existsSync(pw.chromium.executablePath()));'
python3 -c 'import sqlite3; c=sqlite3.connect(":memory:"); c.execute("create table t(x integer)"); c.execute("insert into t values (3)"); print(c.execute("select x from t").fetchone()[0])'
command -v sqlite3; command -v psql; command -v postgres; command -v mysql; command -v mysqld; command -v redis-cli; command -v redis-server
```

`confirmed_unavailable` on checked paths: Chromium/Chrome, Firefox, WebKit MiniBrowser, Xvfb, browser automation CLIs; no Python Playwright/Selenium; global npm list had only corepack/npm. `installable`: **Playwright-core library only**, v1.55.0, via scratch npm install; the resolved Chromium executable path did **not** exist (`false`). Local-page headless-browser test and browser screenshots are `not_tested`. Ordinary files/archives **can** be produced (§21); do not equate that to screenshots.

`confirmed`: Python standard-library SQLite `:memory:` query returned `3`; `confirmed_unavailable` preinstalled sqlite3 *CLI* and checked PostgreSQL/MySQL/Redis clients/servers and Python connectors. No heavyweight database/service was started. Docker-backed services are `not_tested` for the reasons in §16.

### 19 & 20. Processes, watchers, limits and clock

```bash
ulimit -a
cat /proc/sys/fs/inotify/max_user_watches
python3 -c 'import subprocess,signal; p=subprocess.Popen(["sleep","60"]); p.send_signal(signal.SIGTERM); print("exit",p.wait(timeout=5))'
python3 -c 'import ctypes,os,select; p="/tmp/arena-audit-01a0daec"; c=ctypes.CDLL("libc.so.6",use_errno=True); fd=c.inotify_init1(os.O_NONBLOCK|os.O_CLOEXEC); wd=c.inotify_add_watch(fd,p.encode(),0x100); name=p+"/watch-file"; open(name,"w").write("x"); print("fd",fd,"watch",wd,"events_bytes",len(os.read(fd,4096)) if select.select([fd],[],[],1)[0] else 0); os.unlink(name); os.close(fd)'
date -u '+%Y-%m-%d %H:%M:%S UTC'; date '+%Y-%m-%d %H:%M:%S %z %Z'
readlink /etc/localtime; locale | head -4
```

`confirmed`: child exited `-15` after SIGTERM; inotify watch produced 32 event bytes; localhost and a background server worked (§6). `session_specific`: open-file soft limit 1024, max user processes 15734, stack 8192 KiB, core file limit 0, inotify max watches 65536; long-running command/tool limit beyond the brief server is `not_tested`. Clock samples showed UTC `2026-09-25 23:36` through `23:43`, local offset `+0000`, timezone `/usr/share/zoneinfo/Etc/UTC`, locale `LANG=` / `LC_CTYPE="POSIX"`. Date agreed with this session's stated date; independent clock accuracy was **not** measured.

### 21 & 22. Archives, artifacts, limits and explicit unknowns

```bash
mkdir -p /tmp/arena-audit-01a0daec/{tar-out,zip-out}
tar -C /tmp/arena-audit-01a0daec -czf /tmp/arena-audit-01a0daec/sample.tar.gz persist.txt
tar -C /tmp/arena-audit-01a0daec/tar-out -xzf /tmp/arena-audit-01a0daec/sample.tar.gz
(cd /tmp/arena-audit-01a0daec && zip -q sample.zip persist.txt)
unzip -oq /tmp/arena-audit-01a0daec/sample.zip -d /tmp/arena-audit-01a0daec/zip-out
cmp /tmp/arena-audit-01a0daec/persist.txt /tmp/arena-audit-01a0daec/tar-out/persist.txt
cmp /tmp/arena-audit-01a0daec/persist.txt /tmp/arena-audit-01a0daec/zip-out/persist.txt
gh api repos/anthracite-labs/Template/actions/artifacts --jq '{total_count}'
```

`confirmed`: tar.gz (147 bytes) and zip (202 bytes) were created, extracted, and content-checked; bounded 4 MiB disposable write (§2); this report was committed and pushed, with the remote SHA verified (§8). Actions artifacts list count 0, so **download through Arena's GitHub integration is not tested**. Maximum large artifact/file size is unmeasured. Other deliberate unknowns: cross-session storage and process survival, public preview reachability, JDK/Gradle/Android builds, browser execution and screenshots, actual CI runner access, populated workflow logs/artifacts, comments/labels/workflow writes, merges and settings mutation. These unknowns must not be promoted to confirmed failures or successes.

## Confirmed / limited / installable / unverified at a glance

- **`confirmed`:** local Git clone/fetch/status/diff/commit, push of the new **and existing** remote session branch, PR #3 creation, native compilation/execution, Python/Node/Perl, small verified GitHub API download, npm/PyPI package downloads, loopback HTTP, file and background process persistence within this session, archives, SQLite via Python, signals and inotify, selected read-only GitHub APIs.
- **`installable`:** previously missing `pipx`, `pnpm`, Playwright-core **library**, and a native esbuild binary, installed in user scratch without sudo; not a claim all packages or browsers are installable.
- **`confirmed_unavailable` in the narrowly tested form:** preinstalled JVM/Gradle/Android SDK/browser/container CLI and listed other missing PATH tools; `/dev/kvm` and KVM-backed Android acceleration; curl forced IPv6 to GitHub in this test. None means every alternative installation route is impossible.
- **`permission_restricted`:** GitHub Actions settings read (`403 Resource not accessible by integration`), authenticated-user API read (`403`). Other GitHub writes require separate evidence, not extrapolation from these endpoints.
- **`session_specific`:** package versions, CPU/RAM/disk/inodes, DNS/reachability, TLS-handshake failures, time, limits and integration scope. HEAD success is not artifact GET success; `curl` exit 35 is not a certificate-verification error.
- **`not_tested`:** anything requiring account/repository administration, destructive or unnecessary visible changes, a privileged OS package install, very large downloads, emulator/system image, public tunnel, nonexistent Actions logs/artifacts, different Arena sessions, and builds dependent on missing downloads.

## Dispatch discrepancies and recommendation

The current-session evidence supports the narrow corrections in `.agents/ARENA-DISPATCH.md` in this PR:

1. `AGENTS.md` **can** be read explicitly (`cat AGENTS.md` worked); whether Arena loads it automatically is unverified. Replace the absolute “Arena does not read” claim with “do not assume automatic reading”; keep task-critical Issue context explicit.
2. Add a **live, safety-bounded capability check**: preinstalled vs safely installable, HEAD/front page vs actual artifact GET, and check existing workflows/permission before choosing CI. The exact toolchain/network is session-specific.
3. Qualify “prefer GitHub Actions when the sandbox lacks tools”: **this repository has zero workflows**, and Actions settings GET is 403. Use an existing accessible trusted runner when local verification cannot work; otherwise report the precise gap instead of assuming CI will solve it.
4. Qualify final acceptance to refer to the **actual** verification surface, not an assumed Actions pipeline. Required CI remains required where present.

For future repositories: put task-critical context and verification in the Issue, test current-session toolchain/network and actual artifact downloads, check whether trusted CI exists and can be accessed before routing to it, then record unknowns without weakening a security-audit sandbox contract. Do **not** turn one session's package versions or selective network access into permanent template guarantees.
