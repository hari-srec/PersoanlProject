# BakeFlow – Development Environment Setup

> **Step:** 1.2 · **Status:** Draft for owner review · **Primary platform:** macOS on Apple Silicon (M1–M4), zsh · Notes for Windows and Linux at the end

## Table of Contents

1. [What You Will Install](#1-what-you-will-install)
2. [Step-by-Step macOS Setup](#2-step-by-step-macos-setup)
3. [IDE Setup](#3-ide-setup)
4. [Supporting Tools](#4-supporting-tools)
5. [Verify Every Installation](#5-verify-every-installation)
6. [Ports Used Locally](#6-ports-used-locally)
7. [Common Problems and Fixes](#7-common-problems-and-fixes)
8. [Windows and Linux Notes](#8-windows-and-linux-notes)
9. [Project Files Created in Phase 2](#9-project-files-created-in-phase-2)
10. [Related Documents](#10-related-documents)

---

## 1. What You Will Install

| Tool | Recommended way | Why this way |
|------|-----------------|--------------|
| Xcode Command Line Tools | `xcode-select --install` | Compilers and `git`, needed by Homebrew |
| Homebrew | Official installer | Standard macOS package manager |
| Git (latest) | Homebrew | Newer than the Apple-bundled version |
| **JDK 21 (Eclipse Temurin)** | **SDKMAN!** | Switch between JDKs per project (for example 21 now, 25 later, OQ-24); `.sdkmanrc` pins the JDK for the repository |
| **Maven** | **Maven Wrapper (`./mvnw`)** in the repository – no global install | Everyone, including CI, uses the identical Maven version ([ADR-018](../02-system-design/design-decisions.md#adr-018--maven-with-the-maven-wrapper)) |
| **Node.js 26 LTS** | **fnm** (Fast Node Manager) | Fast, reads `.nvmrc`, switches automatically per folder |
| **Docker** | **Docker Desktop** (personal Mac) or **Colima** (company-managed Mac), see OQ-26 | Runs MySQL, Mailpit, Redis and Testcontainers |
| VS Code (or IntelliJ IDEA) | Homebrew cask | Editor with Java, Spring and React support |
| DBeaver Community | Homebrew cask | Database client |
| Bruno | Homebrew cask | API client whose collections are plain files kept in Git |

**SDKMAN! vs Homebrew for the JDK:** Homebrew's `temurin@21` cask is simpler for a single JDK. SDKMAN! is recommended because it manages several JDKs side by side and pins one per project with `.sdkmanrc`, which matters when testing a future Java 25 upgrade.

---

## 2. Step-by-Step macOS Setup

Open **Terminal** (or iTerm2). Run each block, then the check shown after it.

### 2.1 Xcode Command Line Tools

```bash
xcode-select --install
```

Check: `xcode-select -p` prints a path such as `/Library/Developer/CommandLineTools`.

### 2.2 Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Apple Silicon installs into /opt/homebrew; add it to your shell:
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
brew --version
```

### 2.3 Git

```bash
brew install git gh
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global core.autocrlf input
# SSH key for GitHub
ssh-keygen -t ed25519 -C "you@example.com"
gh auth login          # choose SSH and upload the key
```

### 2.4 JDK 21 via SDKMAN!

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk list java | grep -i tem      # find the latest 21.0.x-tem identifier
sdk install java 21.0.<x>-tem     # replace <x> with the latest patch shown
sdk default java 21.0.<x>-tem
# switch JDK automatically when entering a folder with .sdkmanrc:
sed -i '' 's/sdkman_auto_env=false/sdkman_auto_env=true/' ~/.sdkman/etc/config
```

SDKMAN! sets `JAVA_HOME` for you. Do **not** install a separate Oracle JDK.

### 2.5 Maven (via the wrapper)

Nothing to install globally. In Phase 2 the repository gets `mvnw`, `mvnw.cmd` and `.mvn/wrapper/maven-wrapper.properties`, which download the pinned Maven version on first use. Always run `./mvnw …`, never `mvn …`.

To generate the wrapper the first time (Phase 2 only), a temporary Maven can be installed with `sdk install maven`.

### 2.6 Node.js 26 LTS via fnm

```bash
brew install fnm
echo 'eval "$(fnm env --use-on-cd --version-file-strategy=recursive --shell zsh)"' >> ~/.zshrc
source ~/.zshrc
fnm install 26
fnm default 26
```

With `--use-on-cd`, entering the frontend folder switches to the version in its `.nvmrc` automatically.

### 2.7 Docker

**Option A – Docker Desktop (recommended for a personal Mac):**

```bash
brew install --cask docker-desktop   # older Homebrew versions call the cask "docker"
open -a Docker
```

Then in Docker Desktop → **Settings**:

- **Resources:** at least 4 CPUs and 6–8 GB memory (MySQL plus Testcontainers).
- **General:** "Use Virtualization framework" and "Use Rosetta for x86/amd64 emulation" both on.
- **Advanced:** "Allow the default Docker socket to be used" on. Testcontainers needs this.

**Licence note:** Docker Desktop is free for personal use, education and small businesses. Commercial use in larger organizations needs a paid subscription. If this Mac is company-managed, use Option B or check your company's licence (OQ-26).

**Option B – Colima (free, open source, no GUI):**

```bash
brew install colima docker docker-compose
mkdir -p ~/.docker/cli-plugins
ln -sfn "$(brew --prefix)/opt/docker-compose/bin/docker-compose" ~/.docker/cli-plugins/docker-compose
colima start --cpu 4 --memory 8 --vm-type vz --vz-rosetta
# Testcontainers settings for Colima (add to ~/.zshrc):
echo 'export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"' >> ~/.zshrc
echo 'export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock' >> ~/.zshrc
source ~/.zshrc
```

Run `colima start` after each reboot (or `brew services start colima`).

**Option C – OrbStack:** fast and polished, and works with Testcontainers out of the box (`brew install --cask orbstack`). Free for personal use; commercial use needs a paid licence.

---

## 3. IDE Setup

### 3.1 VS Code (recommended – one editor for Java and React)

```bash
brew install --cask visual-studio-code
# enable the "code" command: VS Code → Cmd+Shift+P → "Shell Command: Install 'code' command in PATH"
code --install-extension vscjava.vscode-java-pack          # Java Extension Pack (language server, debugger, Maven, tests)
code --install-extension vmware.vscode-boot-dev-pack       # Spring Boot Extension Pack (Spring tools, initializr, dashboard)
code --install-extension dbaeumer.vscode-eslint            # ESLint
code --install-extension esbenp.prettier-vscode            # Prettier
code --install-extension bradlc.vscode-tailwindcss         # Tailwind CSS IntelliSense
code --install-extension bierner.markdown-mermaid          # Mermaid diagrams in Markdown preview
code --install-extension editorconfig.editorconfig         # EditorConfig
code --install-extension redhat.vscode-yaml                # YAML (application.yml, GitHub Actions)
code --install-extension ms-azuretools.vscode-containers   # Container Tools (Docker files, compose, containers)
```

Recommended settings (`Cmd+,` → open `settings.json`); a shared `.vscode/settings.json` and `.vscode/extensions.json` are added to the repository in Phase 2:

```json
{
  "editor.formatOnSave": true,
  "[typescript]": { "editor.defaultFormatter": "esbenp.prettier-vscode" },
  "[typescriptreact]": { "editor.defaultFormatter": "esbenp.prettier-vscode" },
  "java.configuration.updateBuildConfiguration": "automatic",
  "java.compile.nullAnalysis.mode": "automatic",
  "files.eol": "\n"
}
```

### 3.2 IntelliJ IDEA (alternative)

```bash
brew install --cask intellij-idea   # or intellij-idea-ce where still available
```

- Very strong Java and Maven support, refactoring and debugging.
- JetBrains has been merging its Community and Ultimate editions into one product with a free tier. Some Spring-specific and JavaScript tooling may need the paid tier, so check the current edition comparison.
- Enable **annotation processing** (Settings → Build → Compiler → Annotation Processors) for MapStruct.
- Use VS Code or WebStorm for the frontend if the free tier lacks TypeScript and React features you need.

---

## 4. Supporting Tools

| Tool | Install | Purpose |
|------|---------|---------|
| **DBeaver Community** (recommended) | `brew install --cask dbeaver-community` | Browse tables, run SQL, view ER diagrams. Connect to `localhost:3306` |
| MySQL Workbench (alternative) | `brew install --cask mysqlworkbench` | Oracle's official client, good for EXPLAIN plans |
| **Bruno** (recommended) | `brew install --cask bruno` | API client. Collections are plain text files stored in the repository (`api-collections/`, Phase 2), so they are versioned with the code |
| Postman (alternative) | `brew install --cask postman` | Popular API client; collections live in Postman's cloud by default |
| Swagger UI | Built into the backend at `http://localhost:8080/swagger-ui.html` (Phase 2) | Try endpoints from the browser |
| Mermaid preview | VS Code extension above; GitHub renders Mermaid natively; [mermaid.live](https://mermaid.live) | View and edit diagrams |
| **Mailpit** web inbox | Docker container (Phase 2), `http://localhost:8025` | See emails the app sends in development |
| `jq` | `brew install jq` | Pretty-print JSON in the terminal |
| `httpie` (optional) | `brew install httpie` | Friendly command-line HTTP client |

---

## 5. Verify Every Installation

| Check | Command | Expected (example) |
|-------|---------|--------------------|
| Homebrew | `brew --version` | `Homebrew 4.x` or newer |
| Git | `git --version` | a recent 2.x version |
| Java | `java -version` | `openjdk version "21.0.x" … Temurin` |
| JAVA_HOME | `echo $JAVA_HOME` | `…/.sdkman/candidates/java/current` |
| SDKMAN! | `sdk version` | prints the SDKMAN! version |
| Maven (after Phase 2) | `./mvnw -v` | `Apache Maven 3.9.x … Java version: 21.0.x` |
| fnm | `fnm --version` | prints a version |
| Node | `node -v` | `v26.x.x` |
| npm | `npm -v` | prints the version bundled with Node 26 |
| Docker | `docker version` | both Client and Server sections; Server `OS/Arch: linux/arm64` |
| Docker Compose | `docker compose version` | `Docker Compose version v2.x` |
| Docker works | `docker run --rm hello-world` | "Hello from Docker!" |
| MySQL image runs natively | `docker run --rm mysql:8.4 mysqld --version` | `mysqld  Ver 8.4.x … (aarch64)` |
| VS Code CLI | `code --version` | prints a version |
| GitHub CLI | `gh auth status` | "Logged in to github.com" |

---

## 6. Ports Used Locally

| Port | Service | Notes |
|------|---------|-------|
| **8080** | Spring Boot backend (API, Swagger UI, Actuator) | `SERVER_PORT` can override |
| **5173** | Vite dev server (React app) | Proxies `/api` to 8080 |
| **3306** | MySQL (Docker) | Mapped from the container |
| **6379** | Redis (Docker) | Phase 4 |
| **1025** | Mailpit SMTP | The app sends mail here in development |
| **8025** | Mailpit web inbox | Open in a browser |

Testcontainers uses **random free ports**, so tests never clash with these.

### Resolving a port conflict

```bash
# Who is using port 3306?
lsof -nP -iTCP:3306 -sTCP:LISTEN
```

Then choose one of:

1. **Stop the other program**, for example a Homebrew MySQL: `brew services stop mysql`; for a MySQL installed from Oracle's `.pkg`, use System Settings → MySQL → Stop.
2. **Change BakeFlow's host port.** The Phase 2 `docker-compose.yml` reads ports from `.env`, for example `MYSQL_HOST_PORT=3307`. Then point the app at `localhost:3307`.
3. For an old container: `docker ps`, then `docker stop <name>`.

---

## 7. Common Problems and Fixes

| Problem | Likely cause | Fix |
|---------|--------------|-----|
| Testcontainers: "Could not find a valid Docker environment" | Docker not running, or the default socket is not exposed | Start Docker. Docker Desktop: enable *Settings → Advanced → Allow the default Docker socket*. Colima: set `DOCKER_HOST` and `TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE` (section 2.7). Check with `docker info` |
| Testcontainers works in the terminal but not in the IDE | The IDE was started before the environment variables existed | Quit and reopen the IDE from a terminal (`code .`) so it inherits `~/.zshrc` |
| Container is very slow, or shows a "platform (linux/amd64) does not match" warning | Image has no arm64 build and runs under emulation | Use official multi-arch images (`mysql:8.4`, `redis:8`, `axllent/mailpit` all support arm64). As a last resort set `platform: linux/amd64` and enable Rosetta |
| `bind: address already in use` on 3306 | A local MySQL is already running | See section 6 |
| MySQL container restarts or the app cannot connect at startup | MySQL is still initialising (first start takes about 20–40 s) | The Phase 2 compose file has a healthcheck, and the app waits for it. Run `docker compose logs mysql` |
| `java -version` shows the wrong version | Another JDK on `PATH` or `JAVA_HOME` | `sdk current java`; `sdk use java 21.0.<x>-tem`; remove old JDK exports from `~/.zshrc` |
| VS Code Java errors after changing JDK | Language server cache | Cmd+Shift+P → "Java: Clean Java Language Server Workspace" |
| `node -v` shows an old version | fnm not initialised in this shell | `source ~/.zshrc`; `fnm use` in the frontend folder |
| Vite shows `ECONNREFUSED` / 502 for `/api` calls | The backend is not running on 8080 | Start the backend; check `curl localhost:8080/actuator/health` |
| Maven or npm SSL errors (`PKIX path building failed`, `SELF_SIGNED_CERT_IN_CHAIN`) | Corporate network intercepts HTTPS | Import the company root certificate into the JDK trust store and set `npm config set cafile …`, or use a non-corporate network |
| Build passes on Mac but fails in CI with "class not found" | macOS file names are case-**insensitive**; Linux CI is case-**sensitive** | Rename files with `git mv` in two steps (`Foo.java → Tmp.java → foo.java`); keep names exactly as conventions say |
| Docker disk full | Old images and volumes | `docker system df`, then `docker system prune` (and `docker volume prune` only if you accept losing local data) |
| Testcontainers leaves containers behind | Test JVM killed abruptly | Ryuk cleans up automatically; otherwise `docker ps -a` and remove them |

---

## 8. Windows and Linux Notes

**Windows 11**

- Install **WSL 2** with Ubuntu (`wsl --install`) and do all development **inside WSL**: SDKMAN!, fnm and Git, with the repository stored in the Linux file system (`~/projects`), not under `C:\`, because it is much faster.
- Install **Docker Desktop** with the WSL 2 backend, and enable integration for your Ubuntu distribution.
- Use VS Code with the **WSL** extension (`ms-vscode-remote.remote-wsl`).
- Use `./mvnw` inside WSL (`mvnw.cmd` exists for native Windows but is not the recommended path).

**Linux (Ubuntu/Debian)**

- Install **Docker Engine** and the **Compose plugin** from Docker's official repository; add yourself to the `docker` group (`sudo usermod -aG docker $USER`, then log out and in).
- SDKMAN! and fnm work as on macOS (use `--shell bash` for fnm if you use bash).
- Testcontainers works without extra configuration.

---

## 9. Project Files Created in Phase 2

These files make the setup reproducible. They are listed here for reference only; they are created in Phase 2, not now.

| File | Purpose |
|------|---------|
| `.sdkmanrc` | Pins the JDK (for example `java=21.0.<x>-tem`) |
| `.mvn/wrapper/…`, `mvnw`, `mvnw.cmd` | Maven Wrapper |
| `frontend/.nvmrc` | Pins Node (`26`) |
| `frontend/.npmrc` | `save-exact=true` |
| `.editorconfig` | Indentation, line endings, charset for all editors |
| `docker-compose.yml` + `.env.example` | Local MySQL and Mailpit (Redis in Phase 4); ports and passwords from `.env`, which is git-ignored |
| `.vscode/extensions.json`, `.vscode/settings.json` | Recommended extensions and shared settings |
| `api-collections/` | Bruno request collections |

---

## 10. Related Documents

- [technology-stack.md](technology-stack.md)
- [coding-conventions.md](coding-conventions.md)
- Deployment documentation: *Planned – created in step 1.7*
