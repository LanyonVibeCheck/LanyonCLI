# LanyonCLI

LanyonCLI is the command-line client *Lanyon*: a code generation and formal verification agent for applied mathematics and computational physics. This README guides you through how to run Lanyon from a terminal.

Everything you do belongs to a *session*, which the server keeps, so you can leave and resume it later from any computer. The client either runs as an executable, or a zipped Python package which only uses the standard library.

## Contents

- [Download](#download): the executable for your system, and a Python fallback.
- [Permissions](#permissions): let your system run the downloaded executable.
- [Server location (optional)](#server-location-optional): how you can optionally specify that lanyon runs on a specific server.
- [Running Lanyon](#running-lanyon): start it, or run it with Python.
- [Using Lanyon](#using-lanyon): commands, Lanyon's assistant, downloads and scripts.
- [Sessions](#sessions): find a session's ID, resume it, cancel a command.

## Download
This repository contains four files:

| File | Runs on |
|---|---|
| `lanyon-cli-linux` | Linux, x86-64 |
| `lanyon-cli-macos` | macOS, Apple Silicon |
| `lanyon-cli-windows.exe` | Windows, x86-64 |
| `lanyon-cli.pyz` | any system with Python 3.10 or later |

Download the executable for your system or the `.pyz` file. The `.pyz` is the fallback if your system will not run the executable (see [Run with Python](#run-with-python)).


## Permissions

The executables are not signed, and downloading them drops their permission to run. Each system needs one step before the first run.

**Linux:**

```bash
chmod +x lanyon-cli-linux
```

**macOS:**

```bash
chmod +x lanyon-cli-macos
xattr -c lanyon-cli-macos
```

The last command removes the quarantine mark that macOS adds to downloaded files. If macOS still refuses to open the file, open **System Settings**, then **Privacy & Security**, and click **Open Anyway** next to `lanyon-cli-macos`.

**Windows:**

First check Smart App Control. Open **Windows Security**, then **App & browser control**, then **Smart App Control settings**. If it is **On**, it blocks the executable and cannot make an exception for one file, so use [Run with Python](#run-with-python) instead. We don't recommend turning **Smart App Control settings** off if you can help it. If, however, it is already **Off**, then unblock the file in PowerShell:

```powershell
Unblock-File .\lanyon-cli-windows.exe
```

If Windows still shows **Windows protected your PC** when the file starts, click **More info**, then **Run anyway**.


## Running Lanyon

Run the executable from the folder that holds it.

Linux:

```bash
./lanyon-cli-linux
```

macOS:

```bash
./lanyon-cli-macos
```

Windows (PowerShell):

```powershell
.\lanyon-cli-windows.exe
```

Lanyon connects, prints a welcome with the session's ID, and waits at a `>` prompt. Type `/help` to list the commands and `/quit` to leave. To list the command-line options, start it with `--help`.


### Run with Python

`lanyon-cli.pyz` runs with Python 3.10 or later, which must already be installed. It needs nothing else. Each block below prints Python's version, then starts Lanyon.

Linux:

```bash
python3 --version
python3 lanyon-cli.pyz
```

macOS:

```bash
python3 --version
python3 lanyon-cli.pyz
```

The `python3` that comes with Apple's command-line tools can be older than 3.10. If it is, install a newer Python from [python.org](https://www.python.org/downloads/) or with Homebrew.

Windows (PowerShell):

```powershell
py --version
py lanyon-cli.pyz
```

`py` comes with the installer from [python.org](https://www.python.org/downloads/). If you installed Python another way, use `python` instead of `py`.


## Server location (optional)

By default LanyonCLI submits your request to a dispatcher which allocates you to the least busy Lanyon server. However if you have requested a special server from our representatives (such as for a GPU simulation) you will be provided with a `<server-URL>` address. To launch Lanyon on this server using Python run the following command:

```bash
python3 lanyon-cli.pyz --base-url <server-URL>
```

The same command line flag will also work for any of the provided executables.


## Using Lanyon

A session runs the stages in order. Each one builds on the result of the one before:

```text
$ lanyon-cli
**********************
* Welcome to Lanyon! *
**********************

Lanyon is a code generation and formal verification agent for applied mathematics and computational physics.
Lanyon is developed and maintained by Lanyon AI, Inc.
Type /help for help or type /quit to quit.
Type anything else to ask Lanyon's assistant.
Disclaimer: Lanyon's assistant is an AI that can make mistakes.
Session: 887fc6c5-ef6c-46fe-820d-7f8bfec5e8f9
Start with /specify "Describe your equation here"

> /specify Advect a scalar with full anisotropic diffusion in one dimension.
> /verify
> /compile
> /simulate Run to time 0.2 on 100 cells.
> /download
> /quit
```

While a stage runs, the client prints the status lines Lanyon prints, such as `Verifying algorithm specification in Lean (attempt 2)...`, and, for a simulation, what it builds and runs, such as `Building the simulation for the GPU (attempt 1)...`. Each stage then prints what it produced, a `Ready to download:` line naming the file it left on the server, and the command that comes next, such as `Verify the specification with /verify`.

| Command | What it does |
|---|---|
| `/specify <prompt>` | write a *specification*, the problem in Lanyon's formal language, and print it |
| `/deepthought <prompt>` | the same, with Lanyon's deep reasoning model |
| `/auto <prompt>`, `/deepauto <prompt>` | write a specification, checking each part as it is written |
| `/verify` | check the specification with Lanyon's symbolic tools and print the Lean proof they export. Success issues a *certificate* |
| `/compile` | turn the certified specification into C code and print it |
| `/simulate [--profile <name>] [instructions]` | run a simulation with the compiled code |
| `/download` | save every file the session produced on this computer |
| `/show specification`, `/show proof`, `/show implementation` | print the latest one again |
| `/session` | print the session's ID, what it holds, and where `/download` saves |
| `/events` | list the session's history on the server |
| `/cancel <command id>` | stop a command that is running on the server |
| `/help`, `/quit` | list the commands, leave |

Everything after the command word is the prompt, as typed. A prompt ending in `.txt` is read from that file. `/compile` needs a certificate, so run `/verify` after `/auto` and `/deepauto` too.

Anything you type that does not start with `/` is a message to *Lanyon's assistant*, an AI model on the server that answers questions about your session. Its reply appears as it is written. When it suggests a command, it asks `Run /simulate? [y/N]` (or the command it suggests), and runs it only if you answer `y`.

`/download` saves into `~/lanyon-sessions/<session id>/`. Each file has the name Lanyon gives it plus the first 12 hex digits of its SHA-256 digest, so a new result never replaces an old one. Running `/download` again skips a file that still has the server's bytes, downloads again a file whose local copy was changed, and skips a `results-<sha>/` folder that exists, without checking the files in it:

```text
~/lanyon-sessions/<session id>/
├── specification-<sha>.rkt
├── proof-<sha>.lean
├── implementation-<sha>.c
└── results-<sha>/
    └── ...
```

To save somewhere else, start Lanyon with `--download-dir <dir>` or set `LANYON_DOWNLOAD_DIR`.

The stages can also be given as options. They run in the order given, and `--download` saves the files once every stage has succeeded:

```bash
lanyon-cli --specify problem.txt --verify --compile --simulate instructions.txt --download
```

The exit status is 0 when every stage succeeded, 1 when a stage did not (later stages do not run), and 2 when nothing was sent, such as for an unknown stage anywhere in `--run`. The site's [Use it](site/using.qmd) and [Reference](site/reference.qmd) chapters cover the rest.

## Sessions

The session holds everything you do in it: your prompts, the specification, proof, implementation and simulation results, the commands you ran, and your conversation with Lanyon's assistant. It lives on the server, so you can close the terminal or lose your connection and pick it up later, from this computer or another. How long the server keeps a session is up to its operator, so download what you want to keep.

Resuming a session needs only its ID and access to the server, so keep your session IDs to yourself.

### Finding your session ID

- In a running Lanyon, type `/session`.
- The welcome printed it as `Session: <id>`, and so does the first line of a run with stage options.
- If Lanyon could not confirm that a command you cancelled with Ctrl-C had stopped, it printed the commands to resume the session and to cancel the command, with both IDs in full.
- If you started Lanyon with `--save-dir <dir>`, the ID is `session_id` in `<dir>/ids.json`.
- If you downloaded from the session, its files are in a folder named after its ID in `~/lanyon-sessions/` (on Windows, `C:\Users\<you>\lanyon-sessions\`).

### Picking up where you left off

In the commands below, replace `lanyon-cli` with the command you start Lanyon with, such as `./lanyon-cli-linux`, `.\lanyon-cli-windows.exe` or `python3 lanyon-cli.pyz`, and `<session id>` with your session's ID. `LANYON_BASE_URL` must point to the server that holds the session. When lanyon-cli uses the dispatcher, it finds the session on whichever of the event's servers holds it.

To resume the session at the `>` prompt:

```bash
lanyon-cli --session "<session id>"
```

Lanyon does not reprint the session's history, but its specification, certificate, implementation and conversation are as you left them. `/session` lists them and `/show` prints them.

To download everything the session has produced, without opening the prompt:

```bash
lanyon-cli --session "<session id>" --download
```

To cancel a command that is still running, with the command ID from the Ctrl-C message, `/events` or `ids.json`:

```bash
lanyon-cli --session "<session id>" --cancel "<command id>"
```

- A command keeps running on the server when you leave it, whether its cancel could not be confirmed, you closed the terminal or you lost your connection. Resume the session after it ends to use its results.
- The client waits up to a day for a stage (`--timeout` changes this). After that it prints `... did not reach a terminal state within 86400s` and stops waiting, but the command carries on. The hosted servers allow a simulation up to a day.
- You can resume the same session in a second terminal, for example to talk to Lanyon's assistant while a stage runs in the first.
- A session runs one stage at a time. A stage started while another runs is refused with `HTTP 409: state.SessionBusy`, and the client names the running command, how long ago it started, and the line that stops it.
- A session in which no command was ever sent cannot be resumed. Start a new one.
