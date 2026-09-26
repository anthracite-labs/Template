# Arena capability probe — this session, not a platform guarantee

- **Observed:** initial probe 2026-09-25, approximately 23:36–23:49 UTC; alternate-route follow-up and expanded acquisition/dependency checks 2026-09-26 (UTC). Results are observations, not platform guarantees.
- **Repository:** `anthracite-labs/Template`, `/home/user/Template`.
- **Actual work branch:** `arena/01a0daec-template`, initially at `f32732e46fd9e23d0540425b9865382e3a821bba`. The remote `arena-capability-probe` and `main` both pointed to that same SHA at probe start (`git ls-remote --heads origin`). Issue #2 asks for `arena-capability-probe` as PR head; this Arena session is bound to `arena/01a0daec-template`, so the PR head must differ. The starting commit is identical. Do not mistake this for a probe of a different repository revision.
- **Final substantive report-bearing SHA:** `c18304627bf6128d6aeb55685f7a059c37313d08` (the 22-area report, expanded §G evidence and dispatch correction). A subsequent *provenance-only* commit records this SHA in the report; the exact resulting PR-head SHA is in PR #3. A Git commit cannot include its own SHA in its committed file contents; use `git rev-parse HEAD` on the PR head to check the tip independently.

## Executive summary

This particular sandbox is a small Debian 12 **KVM guest** (2 visible CPUs, 3.8 GiB RAM, about 20 GiB free on an ext4 filesystem). It runs native C/C++, Python, Node.js, Perl, Git, and a loopback HTTP server. User-local Python/npm installations work: `pipx` and `pnpm` were **absent before the test but installed and run in `/tmp`**; an npm-downloaded ELF executable ran. File and background-process persistence was observed **between tool calls in this session**, not across Arena sessions.

Network access is selective: GitHub's website/API, npm, and PyPI worked with certificate verification; many other resolved hosts returned `curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL` before HTTP, including Maven Central, Gradle, Google Maven/SDK downloads, and GitHub release-asset hosts. These are **route-specific failures**, not proof of general toolchain unavailability or a particular network-policy cause. A checksum-verified PyPI wheel supplied a **Temurin 21 Java runtime** (no `javac`/`jdk.compiler`) and ran scratch bytecode (§F). The expanded probe (§G) also **built/installed Lua 5.4.9 from pinned Git source** and linked an actual native consumer; installed `asdf` and its Java plugin, then found that a checksum-pinned *full* JDK still redirected to the inaccessible release-asset CDN; obtained a real Py4J JAR from a verified PyPI wheel, published it in a **scratch-only Maven-layout repository** and consumed it on a Java classpath (not with a Maven/Gradle resolver); and installed F-Droid's **alternative Python `sdkmanager` CLI**, which parsed signed Android package metadata fetched via authenticated GitHub REST. The actual Android SDK ZIP and Gradle/full-JDK distribution bytes were not acquired. None of these successes proves Java source compilation, Gradle/Maven dependency resolution, an Android build, or access to arbitrary remote dependencies. No JVM, Gradle, Android SDK/emulator, browser, or container daemon was **preinstalled at initial inventory**. `/dev/kvm` is absent, limiting accelerated emulation only; builds/Robolectric remain separate, unverified questions.

This repository has **zero** GitHub Actions workflows, runs, and artifacts at probe time. Read-only repository/PR/Issues/Actions-list API calls worked, and the required Git push (both new and existing session branch) and PR creation succeeded, but `GET /repos/anthracite-labs/Template/actions/permissions` returned **403 `Resource not accessible by integration`**. It is incorrect to assume a usable canonical runner without checking. No destructive/admin/account action, workflow dispatch, merge, public tunnel, or heavyweight emulator download was performed.

**Classification vocabulary:** `confirmed` = successfully exercised; `confirmed_unavailable` = specifically tested operation or preinstalled path absent **in this session**; `installable` = missing initially but installed and exercised user-locally; `permission_restricted` = an authoritative permission/API denial; `not_tested` = no valid test of the operation; `session_specific` = a measured observation not a permanent Arena guarantee. A row can contain multiple statuses for different sub-capabilities. See the command/evidence beside each classification and the numbered details below.

## Capability matrix — all 22 Issue #2 areas

| # | Area | Classification and direct evidence |
|---|---|---|
| 1 | Host / OS / identity | `confirmed` — `uname -a; cat /etc/os-release; id; systemd-detect-virt` → Debian 12, x86_64, uid 1001, `kvm` guest; §1. |
| 2 | CPU / memory / disk / filesystem | `session_specific` — `nproc; free -h; df -h /tmp; df -i /tmp; findmnt -T /tmp` → 2 CPUs, 3.8 GiB, 20 GiB free, ext4; `confirmed` ≥4 MiB writable by `dd`/read/delete; §2. |
| 3 | Session lifecycle | `confirmed` within session — marker written then read in another call; `start_process` server answered in a later call. `not_tested` across sessions; §3. |
| 4 | Privilege / installability | `confirmed` — `id -u`=1001, `sudo -n true`=0; `installable` user-local packages, Java **runtime**/F-Droid alternative CLI via verified PyPI and Lua from pinned Git source. Unprivileged, signed apt metadata failed on default HTTP (`Connection failed`) and HTTPS (`TLS handshake`); no OS install; §4/§F/§G. |
| 5 | Network | `session_specific` — verified GitHub/npm/PyPI artifact GETs; authenticated GitHub REST **Git blob** GET delivered 1,340,527-byte signed Android metadata with matching blob hash/signature. Tested Maven/Gradle/Google/OCI/vendor routes failed at TLS; GitHub REST/CLI *release asset* redirects to a CDN failed even when authenticated. Tested forced IPv6 to GitHub failed; §5/§F/§G. |
| 6 | Local servers | `confirmed` — `python3 -m http.server 18765 --bind 127.0.0.1` + later `curl http://127.0.0.1:18765/persist.txt` → 200; `not_tested` externally reachable preview/tunnel; §6. |
| 7 | Git | `confirmed` — clone, fetch dry run, status/diff/branch, `git commit` and `git push origin arena/01a0daec-template` (remote SHA verified); `confirmed_unavailable` preinstalled Git LFS; actual submodule fetching `not_tested`; §7/§8. |
| 8 | GitHub integration | `confirmed` file/Issue/PR/Actions-list reads, new **and existing** session-branch push, PR #3 → `main`; `permission_restricted` Actions settings GET 403; other writes classified individually in §8. |
| 9 | Variables / secrets | `confirmed` — variable **presence only** for `GITHUB_TOKEN`/`GH_TOKEN`, no values; no configured `git credential.helper`; authenticated API access is selective; §9. |
| 10 | Core CLIs | `confirmed` Bash, curl, wget, tar/zip/unzip/xz, jq, grep/sed/awk/find/xargs, make, gcc, OpenSSL, SSH, gh, Git; `confirmed_unavailable` on PATH for yq, cmake/ninja, clang, pkg-config, sqlite3 CLI, rsync, git-lfs; §10. |
| 11 | Runtimes / package managers | `confirmed` preinstalled Python/pip, Node/npm/npx/yarn/corepack, Perl; `installable` `pipx`, `pnpm`, **Java runtime**, pinned source-built **Lua**, `asdf` manager/plugin and F-Droid Python CLI into scratch; `confirmed_unavailable` *at initial PATH inventory* JVM/Go/Rust/Ruby/PHP/.NET/Swift/Lua, etc. Other runtimes' installability unverified; §11/§F/§G. |
| 12 | Java / JDK | `confirmed_unavailable` **preinstalled** `java`/`javac`; `installable` pinned Temurin 21 **runtime** via SHA-256-checked PyPI wheel. `java -version`, `Hello.class` and JAR-bound `JarBound.class` ran; image has no `javac`/`jdk.compiler`. `asdf` + plugin ran with manually seeded pinned metadata; selected full JDK SHA-256 matched Adoptium's GitHub release metadata, but its archive redirect failed. No Java source compilation/full-JDK acquisition; §12/§F/§G. |
| 13 | Gradle / Maven | `confirmed_unavailable` preinstalled CLIs; `session_specific` pinned Gradle GitHub release CLI/REST → CDN EOF, official/custom repository/proxy hosts' TLS failures; checked npm candidates were wrappers. `confirmed` manual local-only Maven-layout JAR publication/`file:` round-trip and **direct Java classpath** consumption; **not** Maven/Gradle resolver use, real Gradle cache population or remote dependency resolution (`not_tested`); §13/§F/§G. |
| 14 | Android | `confirmed_unavailable` **preinstalled** SDK/adb/sdkmanager/emulator and `/dev/kvm` acceleration; `installable` **F-Droid Python alternative `sdkmanager` CLI**, which validated/parsed **F-Droid-signed** metadata about Google SDK packages manually acquired through GitHub REST. Neither the **Google** `sdkmanager` nor any actual SDK ZIP/build-tools/platform/platform-tools/adb was installed: official/mirror ZIP GETs failed curl 35. Compilation, dependency resolution, unit/Robolectric, adb, emulator `not_tested`; §14/§F/§G. |
| 15 | Native toolchain | `confirmed` — `gcc`/`g++` compiled/ran hello; make/ld/as and libc headers available. **Pinned Lua 5.4.9 Git source built** to an installed scratch CLI/static library/headers; native C consumer linked `liblua.a` and returned 42. `confirmed_unavailable` preinstalled clang/cmake/ninja/pkg-config; §15/§G. |
| 16 | Containers / virtualization | `confirmed` KVM guest and `unshare -Ur true` succeeded; `confirmed_unavailable` Docker/Podman/skopeo/oras/crane/ctr CLIs and checked daemon sockets; tested Docker Hub/GHCR registry API routes failed at TLS. OCI layer pull and actual nested container execution `not_tested`; §16/§G. |
| 17 | Browser / web testing | `confirmed_unavailable` preinstalled browser and headless display on checked paths; `installable` `playwright-core` **library** only; smoke test/screenshots `not_tested` (no browser binary); §17. |
| 18 | Databases / services | `confirmed` Python SQLite in-memory query; `confirmed_unavailable` preinstalled SQLite CLI and checked PostgreSQL/MySQL/Redis clients/servers; service startup `not_tested`; §18. |
| 19 | Process / runtime | `confirmed` child/signal, background server, loopback, inotify event; `session_specific` open files 1024 / user processes 15734 / watches 65536; long-run ceiling `not_tested`; §19. |
| 20 | Time / locale | `session_specific` — UTC date 2026-09-25, `/etc/localtime` UTC, POSIX C-type locale; clock accuracy against independent reference `not_tested`; §20. |
| 21 | Archives / artifacts | `confirmed` tar+zip round trips, files committed and pushed (report SHA verified remotely); Actions artifact listing `confirmed` (zero); artifact download/max large size `not_tested`; §21. |
| 22 | Limits / unknowns | `not_tested` cross-session lifespan, emulator/software fallback, full-JDK Java source compilation, **actual Maven/Gradle resolver**, Android SDK build/dependency execution, OCI layers, CI execution, merge/admin/workflow writes; `session_specific` measured versions/resources and **route-specific** network results; §22/§F/§G. |

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

`confirmed_unavailable` *at the initial checked PATH inventory*: zsh, yq, cmake, ninja, clang/clang++, pkg-config, sqlite3 CLI, rsync, git-lfs, uv, bun, Java/Javac, Gradle, Maven, kotlinc, Go, Rust/cargo/rustup, Ruby/gem/bundler, PHP/Composer, dotnet, Swift, Lua. `pipx`/`pnpm` moved to `installable` after the scratch tests; **Lua also moved to `installable` after the pinned Git build in §G**. Other absent tools' installation via untested sources remains `not_tested`. HEAD requests to official Lua, Rust static, Go, dotnet, PHP, RubyGems and Debian hosts also returned curl 35 (§5), so this session's tested direct distribution paths did not prove installability. **Missing on PATH is not proven unavailable as a runtime after safe installation.**

### 12. Java / JVM

```bash
command -v java; command -v javac; java -version; javac -version
# No JAVA_HOME value printed; only presence check in §9.
ls /usr/lib/jvm  # no such directory here
gh api repos/adoptium/temurin17-binaries/releases/latest --jq '{tag_name,linux:[.assets[]|select(.name|test("^OpenJDK17U-jdk_x64_linux_hotspot_.*\\.tar\\.gz$"))|{name,size}][0:1]}'
curl -sS -L -r 0-1023 --max-redirs 2 --max-filesize 1048576 --connect-timeout 5 --max-time 20 -o /dev/null -w 'HTTP=%{http_code} received=%{size_download}B\n' 'https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.20.1%2B1/OpenJDK17U-jdk_x64_linux_hotspot_17.0.20.1_1.tar.gz'
```

`confirmed_unavailable` preinstalled JVM/JDK in checked locations: both commands missing, `JAVA_HOME` unset, no `/usr/lib/jvm`. `session_specific` download route: public release metadata identified **pinned** Temurin `jdk-17.0.20.1+1`, archive size 193,252,603 bytes; the bounded GET got a GitHub 302 then curl **35** on the release-asset CDN, 0 archive bytes. `https://api.adoptium.net/v3/binary/latest/17/ga/linux/x64/jdk/hotspot/normal/eclipse` HEAD also failed 35. No **full JDK** archive was downloaded/unpacked through that route; this initial pass could not compile or execute Java. The follow-up in §F installed and executed a smaller Temurin **runtime** from PyPI, but did not obtain `javac`/`jdk.compiler`. Source compilation and a full JDK remain unverified; one failed archive route is not proof of their impossibility.

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

`confirmed_unavailable` **preinstalled** Gradle/Maven. `session_specific`: both Gradle distribution hosts' HEADs returned curl 35; bounded pinned GitHub asset GET redirected then failed 35 at release-asset host; Maven Central jar GET, Google Maven metadata GET, Plugin Portal metadata GET all returned curl **35**, zero bytes. Subsequent authenticated GitHub CLI/API and alternate mirror checks are in §F. At that initial point one small dependency was **not** resolved. §G later acquired a JAR from verified PyPI, published it manually into a local-only Maven-layout directory and used **direct classpath linking**, but **no Maven/Gradle resolver**. A tiny scratch Gradle build, Gradle execution, and real cache writes by Gradle remain `not_tested`. `confirmed` only that the conventional `$HOME/.gradle/caches/...` location accepts a marker write/read and empty directories were removed. Cache/session persistence or distribution download through untested mirrors is **unknown**, not impossible. A successful front-page HEAD elsewhere would not establish artifact downloads.

### 14. Android capability distinctions

```bash
command -v sdkmanager; command -v adb; command -v emulator
for p in /opt/android-sdk /opt/android /usr/local/lib/android /usr/lib/android-sdk /home/user/Android/Sdk /dev/kvm; do test -e "$p" && ls -ld "$p" || echo "$p MISSING"; done
# ANDROID_HOME and ANDROID_SDK_ROOT presence was checked, not printed (§9).
curl -sS -I -L --max-redirs 4 --connect-timeout 6 --max-time 22 -o /dev/null -w 'HTTP=%{http_code}\n' 'https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip'
```

`confirmed_unavailable` preinstalled Android SDK tools/build-tools/platforms in checked locations, `sdkmanager`, `adb`, `emulator`, Android env variables, and **`/dev/kvm`**. `/dev/kvm` absence establishes no KVM-backed accelerated emulator in this guest; it does not establish that software emulation could never work. `session_specific`: tested official command-line-tools archive route failed curl 35; §F documents alternate signed apt, Google and package-registry paths. No large SDK/system-image install. The following remain separately `not_tested`: **Android dependency resolution**, **Android compilation**, **JVM unit/Robolectric tests**, **adb connection**, **emulator/managed-device run**. Failure to install tools or run an emulator is **not** evidence of inability to resolve Android dependencies or compile on a different network/runner.

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

## §F. Follow-up: alternate acquisition paths (2026-09-26, ~00:02–00:13 UTC)

PR #3's [owner comment](https://github.com/anthracite-labs/Template/pull/3#issuecomment-5841284649) correctly requested more than direct-host `curl` failures before assessing JVM/Gradle/Android. This follow-up was performed in the same checkout on `arena/01a0daec-template`. All temporary files went under `/tmp/arena-audit-followup-01a0daec`; no system package install, TLS-verification bypass (`-k`, `--no_https`, etc.), unverified archive execution, workflow mutation, or emulator/system-image download. A failed host is **only** a failed route. The initial §1–§22 inventory remains a timestamped *preinstalled-at-start* snapshot; the statuses below supersede any initial uncertainty about **Java runtime** acquisition.

### F1. Safe acquisition-route outcomes

| Capability / route | Classification | Exact probe and result |
|---|---|---|
| GitHub releases via CLI | `session_specific` failed *download route* | `gh release download 'jdk-17.0.20.1+1' --repo adoptium/temurin17-binaries --pattern 'OpenJDK17U-jdk_x64_linux_hotspot_17.0.20.1_1.tar.gz' --dir "$d"` → exit 1, `Get "[signed-CDN-URL-redacted] EOF`, 0 bytes. `gh release download v8.14.3 --repo gradle/gradle-distributions --pattern gradle-8.14.3-bin.zip --dir "$d"` → the same CDN EOF, 0 bytes. CDN presigned URL/query was **not stored in the report**. |
| GitHub REST assets | `session_specific` failed *download route* | `gh api` release metadata provided **official pinned** JDK asset `523839222` (193,252,603 bytes, SHA-256 `3808d1d15e3ec6bd5b84057fb5d84c33d8a1536a258146bcea2e603fc726e08e`) and Gradle asset `269969095` (137,393,837 bytes, SHA-256 `bd71102213493060956ec229d946beee57158dbd89d0e62b91bca0fa2c5f3531`). `curl --range 0-1023 --max-filesize 4096 -H 'Accept: application/octet-stream' https://api.github.com/repos/gradle/gradle-distributions/releases/assets/269969095` → HTTP **302**, 0 bytes, with and without configured auth; authenticated Temurin asset `.../releases/assets/523839222` also returned **302**, 0 bytes. Gradle `application/vnd.github.v3.raw` returned JSON metadata (200, 1,696 bytes), **not** the archive. `gh api -H 'Accept: application/octet-stream' .../assets/269969095` followed the redirect and failed with the same CDN EOF. Public REST access is not an in-API binary stream for these assets. |
| Debian apt (existing signed source and HTTPS) | `session_specific` metadata failures; privileged install `not_tested` | `/etc/apt/sources.list.d/debian.sources` specifies `http://deb.debian.org/debian` and `debian-security`, `Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg` (readable). Unprivileged scratch `apt-get update` against **that existing Debian HTTP transport** with `signed-by` yielded `Connection failed [IP: … 80]`, **zero** index files. Prior scratch HTTPS attempt (§4) yielded `Could not handshake: The TLS connection was non-properly terminated`, zero files. Both apt attempts exited **0 with warnings**, so exit code is not evidence metadata refreshed; attempted `/var/cache/apt/archives/partial` cleanup was denied, no system package changed. No `--allow-unauthenticated` or privileged install. |
| Alternative official/reputable HTTPS distribution endpoints | `session_specific` *route failures only* | `curl -sS -I --connect-timeout 4 --max-time 9 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' URL`: Azul (`cdn.azul.com`), BellSoft, Corretto (`corretto.aws`), Oracle/`download.java.net`, conda-forge and Anaconda, Debian/Ubuntu mirrors, Tsinghua/Tencent Gradle mirrors, Google Maven (`maven.google.com`), Google CDN redirector (`redirector.gvt1.com`), Aliyun/Huawei Maven mirrors, and `jitpack.io` all returned curl **35** before HTTP at the tested URLs. Not a universal statement about mirrors or future sessions. |
| PyPI verified alternate Temurin **runtime** | `installable` | `jdk4py==21.0.8.2` from ActiveViam's public PyPI wheel: actual GET/install/hash and running Java are proven in F2. This wheel is a **trimmed runtime**, *not* a full JDK/compiler. |
| npm/PyPI Gradle/Android candidates | `not_tested` **Gradle/Google SDK binary** execution | `npm view gradle-dist@1.0.1 dist.unpackedSize` → **5,212** bytes; `npm view gradle@1.2.4 dist.unpackedSize` → **3,625** bytes: wrappers, **not** the 137 MB Gradle ZIP. `npm view android-platform-tools@3.0.2 dist.unpackedSize` → **24,399** bytes (Node wrapper, not Google platform-tools binaries). PyPI `sdkmanager==0.7.1` is a **274,541-byte F-Droid Python source package**, not Google's CLI ZIP; its **alternative CLI** was installed and used in §G, but no Google SDK binary acquired. Checked PyPI `android-sdk`, `android-platform-tools`, `android-build-tools` names: 404; no inference about every package/source. |

Representative **exact** follow-up route commands (the remaining hosts used the same HEAD form; errors and CDN URLs were redacted before display):

```bash
d=/tmp/arena-audit-followup-01a0daec
# Metadata supplies the expected SHA-256; this is NOT an asset download.
gh api repos/gradle/gradle-distributions/releases/tags/v8.14.3 --jq '{tag_name,gradle_bin:[.assets[]|select(.name=="gradle-8.14.3-bin.zip")|{name,size,digest,id}]}'
# Public API: 302, zero bytes; authenticated API with the same bounded Range was also 302.
curl -sS --range 0-1023 --max-filesize 4096 --connect-timeout 5 --max-time 15 -H 'Accept: application/octet-stream' -o "$d/gradle-api-unauth" -w 'HTTP=%{http_code} type=%{content_type} bytes=%{size_download}\n' 'https://api.github.com/repos/gradle/gradle-distributions/releases/assets/269969095'
curl -sS --range 0-1023 --max-filesize 4096 --connect-timeout 5 --max-time 15 -H "Authorization: Bearer $GH_TOKEN" -H 'Accept: application/octet-stream' -o "$d/gradle-api-authed" -w 'HTTP=%{http_code} type=%{content_type} bytes=%{size_download}\n' 'https://api.github.com/repos/gradle/gradle-distributions/releases/assets/269969095'
# gh follows the redirect: an EOF at the release-asset CDN, no distribution ZIP.
out=$(timeout 35 gh release download v8.14.3 --repo gradle/gradle-distributions --pattern gradle-8.14.3-bin.zip --dir "$d" 2>&1); status=$?
printf '%s\n' "$out" | sed -E 's#https://(objects|release-assets)\.githubusercontent\.com[^[:space:]]*#[signed-CDN-URL-redacted]#g' | head -8
printf 'gh_release_exit=%s\n' "$status"
# Configured Debian HTTP source with its signing key: metadata only, all intended writes in scratch.
printf 'deb [signed-by=/usr/share/keyrings/debian-archive-keyring.gpg] http://deb.debian.org/debian bookworm main\n' > "$d/debian-sources.list"
mkdir -p "$d/apt/lists/partial" "$d/apt/cache/archives/partial"
timeout 30 apt-get update -o Dir::Etc::sourcelist="$d/debian-sources.list" -o Dir::Etc::sourceparts=- -o Dir::State::Lists="$d/apt/lists" -o Dir::Cache="$d/apt/cache" -o Dir::Cache::archives="$d/apt/cache/archives" -o APT::Get::List-Cleanup=false -o Debug::NoLocking=1 -o Acquire::Retries=0 -o Acquire::http::Timeout=8
# Examples of independently probed alternate routes (all curl 35, HTTP 000):
curl -sS -I --connect-timeout 4 --max-time 9 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://cdn.azul.com/zulu/bin/
curl -sS -I --connect-timeout 4 --max-time 9 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://conda.anaconda.org/conda-forge/linux-64/repodata.json.zst
curl -sS -I --connect-timeout 5 --max-time 12 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://mirrors.tuna.tsinghua.edu.cn/gradle/gradle-8.14.3-bin.zip
curl -sS -I --connect-timeout 4 --max-time 9 -o /dev/null -w 'HTTP=%{http_code} TLS=%{ssl_verify_result}\n' https://redirector.gvt1.com/edgedl/android/repository/
npm view gradle-dist@1.0.1 dist.unpackedSize --json --registry=https://registry.npmjs.org
npm view android-platform-tools@3.0.2 dist.unpackedSize --json --registry=https://registry.npmjs.org
```

Only the **official Debian source already configured in this image** was probed over HTTP, with its Debian signing key still required; HTTPS was not downgraded for an otherwise HTTPS-only source. No unsigned apt flags, TLS opt-outs, package installation, or public service were used. These checks are bounded by practical, known routes, not a claim to enumerate every mirror on the Internet. GitHub `gh`/REST content reads and git work normally; **release asset redirects** are the distinct failing route.

### F2. Installed Java runtime and bounded execution proof

The PyPI metadata identified `jdk4py==21.0.8.2`, authored by ActiveViam (public `activeviam/jdk4py`, GPL-2.0); the x86_64 Linux wheel is **33,794,570 bytes** with published SHA-256 `85addfcb57c7051dad6145b9f816fc519337e9a0c705ef01edc9dc7818ee0356`. This is a **third-party repackaging** of Temurin; the wheel checksum protects the retrieved PyPI artifact, not an independently verified Adoptium upstream signature. The project's `build_java_runtime.py` uses `jlink` with selected modules, explaining the deliberately trimmed image. Exact executed acquisition commands:

```bash
d=/tmp/arena-audit-followup-01a0daec
mkdir -p "$d/wheels"
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 download --no-deps --only-binary=:all: --index-url https://pypi.org/simple --dest "$d/wheels" 'jdk4py==21.0.8.2'
w="$d/wheels/jdk4py-21.0.8.2-py3-none-manylinux_2_17_x86_64.whl"
printf '85addfcb57c7051dad6145b9f816fc519337e9a0c705ef01edc9dc7818ee0356  %s\n' "$w" | sha256sum -c -
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install --no-deps --no-index --find-links "$d/wheels" --target "$d/jdk4py" 'jdk4py==21.0.8.2'
PYTHONPATH="$d/jdk4py" python3 -c 'from jdk4py import JAVA_HOME; print(JAVA_HOME)'
java_home="$d/jdk4py/jdk4py/java-runtime"
"$java_home/bin/java" -version
"$java_home/bin/java" --list-modules | grep -E '^(java.compiler|jdk.compiler|jdk.jshell)@' || true
printf 'public class Hello { public static void main(String[] args) { System.out.println("Hello from Java"); } }\n' > "$d/Hello.java"
"$java_home/bin/java" "$d/Hello.java"  # exit 1; compiler module absent
```

Output: `sha256sum ...: OK`; installed under `/tmp/.../jdk4py` (~101 MB uncompressed). `java -version` → `openjdk version "21.0.8" 2025-07-15 LTS`, `OpenJDK Runtime Environment Temurin-21.0.8+9`, 64-bit server VM. `java.compiler@21.0.8` exists, but `jdk.compiler` **does not**; `bin/javac`, `jar`, `jlink`, `jshell` also absent. Running a scratch `Hello.java` with `"$java_home/bin/java" "$d/Hello.java"` exited 1 with `java.lang.InternalError: Module jdk.compiler not in boot Layer`. Thus calling this a usable **full JDK** or successful source compilation would be false.

For a bounded **runtime** proof beyond `-version`, a Python standard-library script generated a minimal Java 8 class file in scratch (not compiled by `javac`); this command executed it:

```bash
python3 - "$d" <<'PY'
import pathlib, struct, sys
u2=lambda n: struct.pack('>H',n)
u4=lambda n: struct.pack('>I',n)
pool=[]
def add(tag, data): pool.append(bytes([tag])+data); return len(pool)
def utf(value): data=value.encode(); return add(1,u2(len(data))+data)
def klass(i): return add(7,u2(i))
def ref(tag,a,b): return add(tag,u2(a)+u2(b))
assert utf('Hello')==1; assert klass(1)==2
assert utf('java/lang/Object')==3; assert klass(3)==4
assert utf('<init>')==5; assert utf('()V')==6
assert ref(12,5,6)==7; assert ref(10,4,7)==8
assert utf('Code')==9; assert utf('main')==10
assert utf('([Ljava/lang/String;)V')==11
assert utf('java/lang/System')==12; assert klass(12)==13
assert utf('out')==14; assert utf('Ljava/io/PrintStream;')==15
assert ref(12,14,15)==16; assert ref(9,13,16)==17
assert utf('Hello from PyPI Java runtime')==18; assert add(8,u2(18))==19
assert utf('java/io/PrintStream')==20; assert klass(20)==21
assert utf('println')==22; assert utf('(Ljava/lang/String;)V')==23
assert ref(12,22,23)==24; assert ref(10,21,24)==25
def method(flags,name,desc,stack,locals_,code):
    body=u2(stack)+u2(locals_)+u4(len(code))+code+u2(0)+u2(0)
    return u2(flags)+u2(name)+u2(desc)+u2(1)+u2(9)+u4(len(body))+body
init=method(1,5,6,1,1,bytes.fromhex('2ab70008b1'))
main=method(9,10,11,2,1,bytes.fromhex('b200111213b60019b1'))
raw=bytes.fromhex('cafebabe')+u2(0)+u2(52)+u2(len(pool)+1)+b''.join(pool)+u2(0x21)+u2(2)+u2(4)+u2(0)+u2(0)+u2(2)+init+main+u2(0)
p=pathlib.Path(sys.argv[1])/'Hello.class'; p.write_bytes(raw)
print('class_size',len(raw),'magic',raw[:4].hex(),'version',int.from_bytes(raw[6:8],'big'))
PY
"$java_home/bin/java" -cp "$d" Hello
```

Output: `class_size 352 magic cafebabe version 52` then **`Hello from PyPI Java runtime`**. This proves execution of user-local JVM bytecode; Python authored the class file. **Java source compilation was not demonstrated**. A complete full JDK (or compiler supplied by a separately verified route) and a Gradle distribution are required before a real Java compile / Gradle dependency-resolving scratch build can be claimed. No unverified third-party compiler JAR was run merely to make the check green.

### F3. Remaining Gradle / Android verification boundary

The pinned Gradle v8.14.3 distribution's **release metadata and digest** were readable (§F1), but neither GitHub CLI/API nor direct/mirror routes delivered its bytes; npm candidate packages contain wrappers, not Gradle. A Java **runtime** alone does not prove Gradle, `javac`, Maven Central access, a writable *real* Gradle cache, or a resolved dependency. A scratch Gradle build was **not run**; direct GET of the small Commons Lang jar, Google Maven metadata and Plugin Portal metadata still failed at their routes (§13). An alternate reputable artifact/mirror would need an independently checked digest and an actual Gradle run before upgrading the classification.

For Android, a bounded `curl -sS -f -L --max-redirs 2 --range 0-1023 --max-filesize 1048576 --connect-timeout 5 --max-time 15 -o /dev/null 'https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip'` GET returned **curl 35 / HTTP 000 / 0 bytes**; bounded Google Maven metadata GET also returned curl 35 / 0 bytes. The tested Google CDN redirector failed before HTTP; official signed Debian metadata was not fetched; checked npm/PyPI candidates are not SDK binaries. The working Java runtime does **not** make `sdkmanager`, `adb`, build-tools/platforms, Android compilation, Robolectric or emulator tests proven. `/dev/kvm` remains absent, independently limiting **accelerated** emulation only. No SDK assets or license acceptance were fabricated. All unperformed build/dependency/device operations remain `not_tested`, not `confirmed_unavailable` generally.

## §G. Expanded safe acquisition and dependency probes (2026-09-26, ~00:14–00:43 UTC)

All work remained on `arena/01a0daec-template`; the new scratch root was `/tmp/arena-audit-next-01a0daec` (not tracked). **Separate toolchain acquisition from project dependency acquisition**: a runtime, version manager, local JAR, or SDK *metadata* is not a compiler, build tool, SDK binary, Maven/Gradle resolver, or Android build. These are actual in-session results, not promises about a later session. No TLS/signature/checksum opt-out, repository setting change, container run, SDK license acceptance, or system package installation was performed. The checks below extend—not retroactively alter—the initial PATH inventory (§1–§22) and earlier Java-runtime follow-up (§F).

### G1. Package-manager, mirror, custom repository and OCI routes

Commands below used TLS verification, a short timeout, a bounded `Range` GET, and an 8 KiB maximum response; a partial response **would not** have qualified an archive for extraction. The exact URLs tested here are shown because front-page reachability alone would not prove artifact retrieval:

```bash
d=/tmp/arena-audit-next-01a0daec
small='org/apache/commons/commons-lang3/3.14.0/commons-lang3-3.14.0.pom'
cli='commandlinetools-linux-16111833_latest.zip'
while IFS='|' read -r name url; do
  [ -n "$name" ] || continue
  result=$(curl -sS -f -L --range 0-511 --max-redirs 2 --max-filesize 8192 \
    --connect-timeout 4 --max-time 11 -o "$d/route-sample.tmp" \
    -w 'HTTP=%{http_code} bytes=%{size_download} verify=%{ssl_verify_result}' "$url" 2>&1)
  code=$?
  printf '%s exit=%s %s\n' "$name" "$code" "$result"
  rm -f "$d/route-sample.tmp"
done <<EOF
maven-central-2|https://repo1.maven.org/maven2/$small
aliyun-custom-maven|https://maven.aliyun.com/repository/central/$small
huawei-custom-maven|https://repo.huaweicloud.com/repository/maven/$small
jitpack|https://jitpack.io/
google-maven|https://dl.google.com/dl/android/maven2/com/android/tools/build/gradle/maven-metadata.xml
tsinghua-android-mirror|https://mirrors.tuna.tsinghua.edu.cn/AndroidSDK/android/repository/$cli
tencent-android-mirror|https://mirrors.cloud.tencent.com/AndroidSDK/android/repository/$cli
sdkman-installer|https://get.sdkman.io/
foojay-discovery|https://api.foojay.io/disco/v3.0/distributions
oci-dockerhub|https://registry-1.docker.io/v2/
oci-ghcr|https://ghcr.io/v2/
nix-cache|https://cache.nixos.org/
EOF
```

**All 12** returned curl **exit 35**, HTTP **000**, **0 bytes** before HTTP. Earlier §F also checked official Gradle/JDK hosts, the signed Debian apt source (zero index files, even though apt exited 0), Conda-forge/Anaconda, Azul/BellSoft/Corretto/Oracle, Google redirector, and other Maven/Gradle mirrors: failures of **those routes**, not an exhaustive Internet survey. `npm` search metadata (`registry.npmjs.org/-/v1/search`, terms `temurin jdk binary`, `gradle binary distribution`, `android commandlinetools`) returned JRE installers/wrappers/Android library candidates, not an independently authenticated full JDK, Gradle distribution or Google command-line-tools archive in the sampled results. A third-party wrapper fetching blocked upstream hosts is not a working alternate binary route. No unsigned apt, TLS downgrade, installation of unverified mirror bytes, or privileged/container installation was attempted.

`command -v skopeo oras crane ctr nerdctl conda mamba nix asdf sdk gradlew mvnw` found **none preinstalled**; Docker/Podman and checked daemon sockets were also absent (§16). OCI registry API routes above failed *before* registry authentication or manifest transfer, so **no OCI layer pull or containerized toolchain execution** was tested. A custom Maven/Gradle `repositories { ... }` or proxy configuration cannot itself prove resolution without a working resolver; the tested remote Maven Central, Google Maven, JitPack, Aliyun and Huawei artifact routes delivered zero bytes. Before manually creating scratch caches, checks found no preseeded `$HOME/.gradle/caches`, `$HOME/.gradle/jdks`, `$HOME/.m2/repository`, `$HOME/.sdkman/candidates/java`, `$HOME/.asdf/installs/java`, `$HOME/Android/Sdk`, or `/usr/lib/jvm`. This checks named locations only, not every file on the filesystem. §G3/G5 explicitly identify **newly seeded** metadata rather than calling it preexisting.

### G2. Official Git-source toolchain built and used locally: Lua

The Lua team's `https://github.com/lua/lua` tag `v5.4.9` resolved to **`312b9efaa1061c2c4cad08554dbc1351c3270eef`**. Git source (not an unverified mirror binary) was built with preinstalled GCC/make; optional readline support was disabled because its headers were absent. GCC/make were already installed; the resulting **Lua** interpreter and static *library* were installed only under `/tmp`. These are the build/verification commands (the source tree/log in scratch also show GCC's `-std=c99 -DLUA_USE_LINUX` compilation):

```bash
d=/tmp/arena-audit-next-01a0daec
git clone --depth 1 --branch v5.4.9 https://github.com/lua/lua.git "$d/lua-src"
git -C "$d/lua-src" rev-parse HEAD
make -C "$d/lua-src" MYCFLAGS='-std=c99 -DLUA_USE_LINUX' MYLIBS='-ldl' lua liblua.a
install -D -m755 "$d/lua-src/lua" "$d/lua-local/bin/lua"
install -D -m644 "$d/lua-src/liblua.a" "$d/lua-local/lib/liblua.a"
install -m644 "$d/lua-src/"{lua.h,lauxlib.h,lualib.h,luaconf.h} "$d/lua-local/include/"
"$d/lua-local/bin/lua" -v
"$d/lua-local/bin/lua" -e 'print("proof=" .. 6*7)'
cat > "$d/lua-embed.c" <<'C'
#include <stdio.h>
#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"
int main(void) {
    lua_State *L = luaL_newstate();
    if (!L) return 1;
    luaL_openlibs(L);
    if (luaL_dostring(L, "return 6 * 7") != LUA_OK) return 2;
    printf("embedded-lua=%lld\n", (long long)lua_tointeger(L, -1));
    lua_close(L);
    return 0;
}
C
gcc -I"$d/lua-local/include" "$d/lua-embed.c" "$d/lua-local/lib/liblua.a" -ldl -lm -o "$d/lua-embed"
"$d/lua-embed"
```

Output: `Lua 5.4.9`; **`proof=42`**; **`embedded-lua=42`**. The installed CLI, four C headers and `liblua.a` were exercised by a separately linked consumer. This **confirms a pinned Git-source build and locally usable native toolchain/library**, not an apt/Nix/container install or remote Java/Android dependency resolution. Git commit pinning is not an upstream source signature.

### G3. Java version manager, checksum-pinned metadata and full-JDK provisioning

`asdf` was cloned at tag **v0.15.0** (`31e8c93004abd76253d186b8896785895069749b`) and its Java plugin `halcyon/asdf-java` at commit **`cae4e16eadcfdca1baf61ef2bc00e16c06a04960`**, all into scratch. Sourcing the manager and adding the local plugin worked. The first `asdf list all java` (before seeding) exited 1, `No compatible versions available (java )`; the captured plugin stderr identified **curl 35 at `raw.githubusercontent.com`** and a missing cache file, *not* a claim that no Java distributions exist. The pinned plugin Git tree includes `data/jdk-linux-x86_64-ga.tsv` (7,843 rows), whose SHA-256 is **`9a71eb18ab20f127366a052417cf64f7aaec9e02cb9a743bbd31d29c8cefce7f`**. Only that reviewed Git-tracked **version/checksum metadata** was then copied to the plugin's isolated scratch cache; no full JDK was preseeded:

```bash
d=/tmp/arena-audit-next-01a0daec
# Source and plugin clones at the pins above:
git clone --depth 1 --branch v0.15.0 https://github.com/asdf-vm/asdf.git "$d/asdf-src"
git clone --depth 1 https://github.com/halcyon/asdf-java.git "$d/asdf-java-src"
export ASDF_DIR="$d/asdf-src" ASDF_DATA_DIR="$d/asdf-data" \
  ASDF_CONFIG_FILE="$d/asdf-config" TMPDIR="$d/tmp"
. "$ASDF_DIR/asdf.sh"
asdf --version
asdf plugin add java "$d/asdf-java-src"
timeout 12 asdf list all java   # before seeding: curl 35 at raw.githubusercontent.com
cache="$d/tmp/asdf-java-user.cache/releases-linux-x86_64.tsv"
cp "$d/asdf-java-src/data/jdk-linux-x86_64-ga.tsv" "$cache"
sha256sum "$d/asdf-java-src/data/jdk-linux-x86_64-ga.tsv" "$cache"
timeout 12 asdf list all java | tr ' ' '\n' | grep -F 'temurin-21.0.8+9.0.LTS'
gh api 'repos/adoptium/temurin21-binaries/releases/tags/jdk-21.0.8%2B9' \
  --jq '[.assets[]|select(.name=="OpenJDK21U-jdk_x64_linux_hotspot_21.0.8_9.tar.gz")|{name,size,digest,id}]'
timeout 20 asdf install java 'temurin-21.0.8+9.0.LTS'
```

`asdf list all java` **then exited 0 and listed** `temurin-21.0.8+9.0.LTS`. Its plugin metadata SHA-256 for `OpenJDK21U-jdk_x64_linux_hotspot_21.0.8_9.tar.gz` is **`f2dc5418092c43003db8f9005c4a286e1c0104fea96ccdd49e8ebd037cac9219`**, matching **official Adoptium GitHub release asset** `274141668` (207,098,019 bytes) returned by `gh api`. `asdf install` exited **1**, curl **35** at `release-assets.githubusercontent.com`; **no JDK install directory** existed afterward. The manager/plugin and *offline metadata* are usable; this route did **not** install `javac`, `jdk.compiler`, or any full JDK. SDKMAN and Foojay API/CLI routes in G1 also failed before delivery. Gradle/Maven toolchain auto-provisioning was **not run** because neither build-tool distribution was acquired; a verified artifact digest alone cannot stand in for its bytes or for a Java compile.

### G4. Project dependency, not toolchain: local-only Py4J JAR

The official PyPI `py4j==0.10.9.9` wheel was actually downloaded and matched PyPI's SHA-256 **`c7c26e4158defb37b0bb124933163641a2ff6e3a3913f7811b0ddbe07ed61533`**. Its real Java JAR (`py4j-0.10.9.9.data/data/share/py4j/py4j0.10.9.9.jar`, 123,954 bytes) was extracted, hashed **`a290b759ba7647fc5a3411afae20698bd069aef77915f8ed53a0efa0045ccb2e`**, and *manually* published with a scratch POM at **`probe.local:py4j-wheel:0.10.9.9`**. That coordinate is **local-only**, not canonical Py4J Maven metadata; `file:` round-tripping matched the original JAR byte for byte. Representative exact commands:

```bash
d=/tmp/arena-audit-next-01a0daec
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 download --no-deps --only-binary=:all: \
  --index-url https://pypi.org/simple --dest "$d/pypi" 'py4j==0.10.9.9'
w="$d/pypi/py4j-0.10.9.9-py2.py3-none-any.whl"
printf 'c7c26e4158defb37b0bb124933163641a2ff6e3a3913f7811b0ddbe07ed61533  %s\n' "$w" | sha256sum -c -
repo="$d/file-maven/probe/local/py4j-wheel/0.10.9.9"
mkdir -p "$repo" "$d/repro-java"
jar="$repo/py4j-wheel-0.10.9.9.jar"
unzip -p "$w" py4j-0.10.9.9.data/data/share/py4j/py4j0.10.9.9.jar > "$jar"
printf 'a290b759ba7647fc5a3411afae20698bd069aef77915f8ed53a0efa0045ccb2e  %s\n' "$jar" | sha256sum -c -
cat > "$repo/py4j-wheel-0.10.9.9.pom" <<'XML'
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>probe.local</groupId><artifactId>py4j-wheel</artifactId><version>0.10.9.9</version>
  <description>Local capability probe only; not an upstream Maven coordinate.</description>
</project>
XML
curl -fsS "file://$jar" -o "$d/local-jar-roundtrip.jar"
cmp "$jar" "$d/local-jar-roundtrip.jar"
```

To avoid claiming nonexistent Java source compilation, Python's standard library generated a minimal **Java 8 bytecode class** whose superclass is `py4j/Py4JException` (available only in that JAR). This is a *classpath dependency presence* test, not a Java compiler or Maven resolution test; the class has a static main and needs no constructor:

```bash
python3 - "$d/repro-java/JarBound.class" <<'PY'
from pathlib import Path
import struct,sys
u2=lambda n:struct.pack('>H',n)
u4=lambda n:struct.pack('>I',n)
pool=[]
def add(t,b):pool.append(bytes([t])+b);return len(pool)
def utf(s):b=s.encode();return add(1,u2(len(b))+b)
def cls(i):return add(7,u2(i))
def ref(t,a,b):return add(t,u2(a)+u2(b))
assert utf('JarBound')==1;assert cls(1)==2
assert utf('py4j/Py4JException')==3;assert cls(3)==4
assert utf('Code')==5;assert utf('main')==6
assert utf('([Ljava/lang/String;)V')==7
assert utf('java/lang/System')==8;assert cls(8)==9
assert utf('out')==10;assert utf('Ljava/io/PrintStream;')==11
assert ref(12,10,11)==12;assert ref(9,9,12)==13
assert utf('local-jar-resolved')==14;assert add(8,u2(14))==15
assert utf('java/io/PrintStream')==16;assert cls(16)==17
assert utf('println')==18;assert utf('(Ljava/lang/String;)V')==19
assert ref(12,18,19)==20;assert ref(10,17,20)==21
code=bytes.fromhex('b2000d120fb60015b1')
body=u2(2)+u2(1)+u4(len(code))+code+u2(0)+u2(0)
method=u2(9)+u2(6)+u2(7)+u2(1)+u2(5)+u4(len(body))+body
raw=bytes.fromhex('cafebabe')+u2(0)+u2(52)+u2(len(pool)+1)+b''.join(pool)+u2(0x21)+u2(2)+u2(4)+u2(0)+u2(0)+u2(1)+method+u2(0)
Path(sys.argv[1]).write_bytes(raw)
print('generated_class',len(raw),'bytes')
PY
java="$d/jdk4py/jdk4py/java-runtime/bin/java"  # verified wheel/runtime from §F
"$java" -cp "$d/repro-java:$jar" JarBound
"$java" -cp "$d/repro-java" JarBound  # EXPECT exit 1 without the JAR
```

Output: `generated_class 291 bytes`; **`local-jar-resolved`** with the JAR (exit 0); without it **`NoClassDefFoundError: py4j/Py4JException`** (exit 1). A working Python/PyPI acquisition channel can supply one verified Java *project artifact* and a Java runtime can use it, even while the tested Maven/Google artifact hosts fail. **No Gradle or Maven distribution, resolver, transitive dependencies, canonical Maven coordinates, or actual cache population by a build tool were demonstrated.** The project-dependency result must not be promoted to Gradle resolution or Android build support.

### G5. Android CLI *client* versus SDK package acquisition

The F-Droid-maintained PyPI **`sdkmanager==0.7.1` alternative Python implementation** is not Google's `cmdline-tools/.../bin/sdkmanager`. Its source archive was downloaded over PyPI HTTPS, its build metadata and relevant CLI/network paths inspected, and the bytes checked against the PyPI SHA-256 **`49063acd70f37826ae2e88a78fe774d394b0a56cbc538649f40369834e356c67`**, and pip-installed **offline** into scratch (`--no-index --no-deps --no-build-isolation` with the preinstalled `setuptools`). Its declared direct and transitive dependencies—`requests==2.32.5`, `urllib3==2.5.0`, `idna==3.10`, `certifi==2025.8.3`, `charset-normalizer==3.4.3` and `argcomplete==3.6.2`—were pinned; **each** wheel was independently compared to its SHA-256 in the corresponding PyPI release JSON before offline scratch installation. Observed hashes (in that order): `2462f94637a34fd532264295e186976db0f5d453d1cdd31473c85a6a161affb6`, `e6b01673c0fa6a13e374b50871808eb3bf7046c4b125b216f6bf1cc604cff0dc`, `946d195a0d259cbba61165e88e65941f16e9b36ea6ddb97f00452bae8b1287d3`, `f6c12493cfb1b06ba2ff328595af9350c65d6644968e5d3a2ffd78699af217a5`, `0e78314bdc32fa80696f72fa16dc61168fda4d6a0c014e0380f9d02f0e5d8a07`, `65b3133a29ad53fb42c48cf5114752c7ab66c1c38544fdf6460f450c09b42591`. The client calls `argcomplete` optional at runtime, but its package metadata declares it; it was installed and imported. `--help` exited 0; `--version` **printed `25.2.0`**, a hardcoded compatibility response in the source, **not** the PyPI package version or evidence of an installed Google SDK.

The client's normal network `--list` attempt exited 1 when fetching the detached SDK checksum signature from `raw.githubusercontent.com` (Python SSL EOF after retries). A **different, authenticated API/CLI route worked for the *metadata***: `gh api` retrieved F-Droid's `signed/checksums.json` and `.asc` as Git blobs (1,340,527 and 488 bytes). Their `git hash-object` values matched the GitHub REST blob IDs **`66da0241d0f68f270b7226223b95d2975fb6f4ee`** and **`9155f786525f8015faa05cc58a20d266c36984a1`**; SHA-256 of the JSON was **`661e5f9d64f848efeb2fe7d4f82cdbc2817c5ceb1b726e429abb0954e5626186`**. The installed client's **bundled F-Droid keyring and `gpgv` validated the detached signature** before parsing the *manually seeded* scratch cache; no signature verification was suppressed:

```bash
d=/tmp/arena-audit-next-01a0daec
# Direct PyPI archive acquisition and independent hash (the URL is from its release JSON):
url=$(curl -fsS --max-time 12 https://pypi.org/pypi/sdkmanager/0.7.1/json |
  jq -r '.urls[]|select(.filename=="sdkmanager-0.7.1.tar.gz")|.url')
curl -fsS -L --max-filesize 1000000 --connect-timeout 5 --max-time 20 \
  -o "$d/sdkmanager-0.7.1.tar.gz" "$url"
printf '49063acd70f37826ae2e88a78fe774d394b0a56cbc538649f40369834e356c67  %s\n' \
  "$d/sdkmanager-0.7.1.tar.gz" | sha256sum -c -
pins=('requests==2.32.5' 'urllib3==2.5.0' 'idna==3.10' \
  'certifi==2025.8.3' 'charset-normalizer==3.4.3')
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 download --no-deps --only-binary=:all: \
  --index-url https://pypi.org/simple --dest "$d/py-wheels" "${pins[@]}"
python3 - "$d/py-wheels" <<'PY'
from pathlib import Path
import hashlib,json,sys,urllib.request
pins={'requests':'2.32.5','urllib3':'2.5.0','idna':'3.10',
      'certifi':'2025.8.3','charset-normalizer':'3.4.3'}
files=list(Path(sys.argv[1]).glob('*.whl'))
assert len(files)==len(pins)
for name,version in pins.items():
    with urllib.request.urlopen(f'https://pypi.org/pypi/{name}/{version}/json',timeout=12) as r:
        meta=json.load(r)
    hashes={x['filename']:x['digests']['sha256'] for x in meta['urls']}
    matches=[p for p in files if p.name.startswith(name.replace('-','_')+'-'+version+'-')
             or p.name.startswith(name+'-'+version+'-')]
    assert len(matches)==1 and hashlib.sha256(matches[0].read_bytes()).hexdigest()==hashes[matches[0].name]
    print(matches[0].name,'hash OK')
PY
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install --no-index --no-deps \
  --find-links "$d/py-wheels" --target "$d/fdroid-deps" "${pins[@]}"
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install --no-index --no-deps \
  --no-build-isolation --target "$d/fdroid-cli" "$d/sdkmanager-0.7.1.tar.gz"
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 download --no-deps --only-binary=:all: \
  --index-url https://pypi.org/simple --dest "$d/py-wheels" 'argcomplete==3.6.2'
printf '65b3133a29ad53fb42c48cf5114752c7ab66c1c38544fdf6460f450c09b42591  %s\n' \
  "$d/py-wheels/argcomplete-3.6.2-py3-none-any.whl" | sha256sum -c -
PIP_DISABLE_PIP_VERSION_CHECK=1 pip3 install --no-index --no-deps \
  --find-links "$d/py-wheels" --target "$d/fdroid-deps" 'argcomplete==3.6.2'
HOME="$d/fdroid-home" ANDROID_HOME="$d/android-scratch" \
  PYTHONPATH="$d/fdroid-deps:$d/fdroid-cli" "$d/fdroid-cli/bin/sdkmanager" --help
cache="$d/fdroid-home/.cache/sdkmanager"
mkdir -p "$cache"
gh api repos/f-droid/android-sdk-transparency-log/git/blobs/66da0241d0f68f270b7226223b95d2975fb6f4ee \
  --jq .content | base64 -d > "$cache/checksums.json"
gh api repos/f-droid/android-sdk-transparency-log/git/blobs/9155f786525f8015faa05cc58a20d266c36984a1 \
  --jq .content | base64 -d > "$cache/checksums.json.asc"
git hash-object "$cache/checksums.json"; git hash-object "$cache/checksums.json.asc"
HOME="$d/fdroid-home" PYTHONPATH="$d/fdroid-deps:$d/fdroid-cli" python3 - <<'PY'
import sdkmanager
from pathlib import Path
cache=Path.home()/'.cache/sdkmanager'
sdkmanager.verify(cache/'checksums.json')   # gpgv with bundled key; raises on failure
sdkmanager.build_package_list(use_net=False) # rechecks signature; no remote refresh
print('signed_packages',len(sdkmanager.PACKAGES))
print('latest_cli_url',sdkmanager.PACKAGES[('cmdline-tools','latest')])
PY
HOME="$d/fdroid-home" ANDROID_HOME="$d/android-scratch" \
  PYTHONPATH="$d/fdroid-deps:$d/fdroid-cli" "$d/fdroid-cli/bin/sdkmanager" \
  --sdk_root "$d/android-scratch" --install 'cmdline-tools;__probe_nonexistent__'
# expected exit 1; printed: Did you mean 'cmdline-tools;latest'?
```

**Metadata result:** valid GPG signature; **1,073** parsed package keys; official `cmdline-tools;latest` was `https://dl.google.com/android/repository/commandlinetools-linux-16111833_latest.zip` with signed SHA-256 **`0877a1d048fe4a24efe2eff536ca4223f7adeb58648bb81909d33c446918cfa8`**. A bounded, TLS-verified range GET of that *exact* ZIP and the two named SDK mirrors in G1 returned curl **35**, HTTP **000**, **0 bytes**; no ZIP to checksum or install. The installed CLI could suggest that package **offline** from verified metadata; the intentionally nonexistent package performed **no download or license acceptance** and exited 1 as expected. The inspected client verifies the **metadata signature**, but its install path does not itself compare a downloaded ZIP's SHA-256; any future archive acquisition must independently match the signed checksum **before** extraction. No such archive, Google's CLI, build-tools, platforms, platform-tools/`adb`, SDK manager installation of a real package, or Android build was demonstrated. Metadata availability != SDK/toolchain acquisition; neither establishes Maven/Android project dependency resolution.

### G6. Boundaries and current classification after the expanded probe

| Acquisition type | Actually worked | Not proved |
|---|---|---|
| Native toolchain + native dependency | Pinned Lua Git source built/installed; Lua command and GCC consumer linked against local `liblua.a` ran. | A remote package-manager index or unrelated native toolchain. |
| JVM **runtime** + local Java project artifact | SHA-256-verified PyPI `jdk4py` runtime executed two scratch classes; verified Py4J wheel JAR, scratch Maven-layout copy and direct Java classpath consumer worked. | Full JDK/`javac`; Gradle/Maven CLI or resolver; remote Maven/Google artifacts or Gradle dependency caching. |
| SDK/version manager | Pinned `asdf`/Java plugin ran; pinned version metadata manually seeded; official digest matched. F-Droid Python `sdkmanager` CLI ran; authenticated API-fetched metadata verified with GPG. | A **JDK archive**, Google's actual SDK ZIP/`sdkmanager`/`adb`, Android compilation/device/emulator, or automatic Java toolchain provisioning by Gradle. |
| Repositories/mirrors/proxies and OCI | PyPI/npm/GitHub Git/API content routes had positive artifact proofs; `file:` local JAR round-trip worked. | Tested Maven mirrors, vendor/Google/Gradle CDN, SDKMAN/Foojay/Conda/Nix/OCI registry routes yielded no bytes; no OCI pull, Maven/Gradle resolver/proxy success, or preexisting JDK/SDK cache. |

**Recommendation:** Re-probe artifact GET and integrity in a future session; where a trusted full JDK, Gradle distribution and real project artifact are actually obtainable, install in scratch and run a tiny *Java source compile + Gradle dependency-resolving build* before claiming that capability. Where signed Android archive bytes are obtainable, verify their published hash before extracting and separately exercise build-tools/platform, dependencies, then tests/device work. Do not treat individual TLS failures, a Java runtime, an offline cache, metadata, or classpath-only linking as a universal conclusion.

## Confirmed / limited / installable / unverified at a glance

- **`confirmed`:** Git clone/fetch/status/diff/commit/push and PR #3 creation; native GCC/G++ execution **and** pinned Lua source build, scratch installation and linked C consumer; Python/Node/Perl; SHA-256-verified PyPI wheel/JAR acquisition and **Java bytecode classpath** proof with a local-only JAR; authenticated GitHub REST Git-blob download and **GPG verification** of signed SDK metadata; loopback HTTP, in-session file/process persistence, archives, Python SQLite, signals/inotify and selected GitHub API reads.
- **`installable`:** previously missing `pipx`, `pnpm`, Playwright-core **library**, native esbuild, Temurin 21 **runtime** (not `javac`), Lua 5.4.9 native CLI/library via pinned Git source, pinned `asdf` Java version **manager/plugin** (not a JDK), and F-Droid's **alternative Python SDK-manager client** (not Google's SDK CLI or tools), all used in disposable storage. A manually seeded *metadata* cache is not an installed toolchain binary.
- **`confirmed_unavailable` in the narrowly tested form:** at initial inventory, preinstalled JVM/Gradle/Android SDK/browser/container CLIs and other missing PATH tools; `/dev/kvm` and KVM-backed Android acceleration; `javac`/`jdk.compiler` in **this** trimmed Java image; no JDK installed by **this** `asdf install` attempt; checked preexisting cache paths empty/absent; forced IPv6 curl to GitHub failed. None proves every safe acquisition route is impossible.
- **`permission_restricted`:** GitHub Actions settings read (`403 Resource not accessible by integration`), authenticated-user API read (`403`). Other GitHub writes require separate evidence, not extrapolation from these endpoints.
- **`session_specific`:** package versions, CPU/RAM/disk/inodes, DNS/reachability, **specific host/transport/redirect** failures, time, limits and integration scope. HEAD success, an authenticated API metadata GET, and a local file/classpath copy do **not** equal a remote distribution or project dependency GET; `curl` exit 35 is not a certificate-verification error and does **not** prove general toolchain unavailability.
- **`not_tested`:** **Java source compilation using a full JDK**, Gradle/Maven scratch build/resolver/real remote dependency/caches, Google's Android SDK binary/install/build/unit/adb/emulator work, OCI layer pull/container execution, account/repository admin mutations, a privileged OS package install, very large downloads, public tunnel, nonexistent Actions logs/artifacts, and different Arena sessions. If a verified route becomes available, rerun the missing bounded proofs instead of assuming failure.

## Dispatch discrepancies and recommendation

The current-session evidence supports the narrow corrections in `.agents/ARENA-DISPATCH.md` in this PR:

1. `AGENTS.md` **can** be read explicitly (`cat AGENTS.md` worked); whether Arena loads it automatically is unverified. Replace the absolute “Arena does not read” claim with “do not assume automatic reading”; keep task-critical Issue context explicit.
2. Add a **live, safety-bounded capability check**: preinstalled vs safely installable, HEAD/front page vs actual artifact GET, and check existing workflows/permission before choosing CI. This follow-up demonstrates why a direct `curl` TLS failure at the official JDK host must stay route-specific: a verified PyPI wheel supplied a working Java runtime, though not a compiler. The dispatch rule now explicitly asks for reputable alternate CLI/API/signed-package/verified-mirror/user-local routes without disabling TLS or verification.
3. Qualify “prefer GitHub Actions when the sandbox lacks tools”: **this repository has zero workflows**, and Actions settings GET is 403. Use an existing accessible trusted runner when local verification cannot work; otherwise report the precise gap instead of assuming CI will solve it.
4. Qualify final acceptance to refer to the **actual** verification surface, not an assumed Actions pipeline. Required CI remains required where present.
5. The expanded §G evidence warrants **one further general routing clarification**: **toolchain acquisition != project dependency acquisition**. A successful Lua source build, Java runtime, offline version metadata, or direct classpath-loaded Py4J JAR each proves a different operation; none implies a full JDK/compiler, Gradle/Maven resolver, real remote Maven artifact, Android SDK binary, or Android build. `.agents/ARENA-DISPATCH.md` now asks for the actual verified artifact and bounded proof before promoting any of those claims, and names signatures/checksums as verification that must not be weakened. This is a reusable distinction, not a permanent claim about this session's hostnames or versions.

For future repositories: put task-critical context and verification in the Issue; test current-session toolchain/network and actual artifact downloads **across reasonable verified routes**; separately validate the compiler/build tool, project artifact acquisition, dependency resolver and target SDK/build operations as relevant; check whether trusted CI exists and is accessible; and record unknowns without weakening TLS, signatures, checksums, package verification, or a security-audit sandbox contract. Do **not** turn one session's package versions or selective network access into permanent template guarantees.
