---
layout: post
title:  "Utilizando Git modo verbose para debugar erros"
date:   2020-06-26 13:20:00 -0300
categories: git
---

# Git - Verbose Mode: Debug Fatal Errors - ShellHacks

> How to debug fatal Git errors by increasing level of verbosity in Git.

Sometimes it may be complex to debug Git errors, like “_fatal: repository not found_” or “_fatal: authentication failed_” with the default level of verbosity in Git.

To debug different network, security, performance and many other issues in Git it is very helpful to know how to increase verbosity.

In this note i am showing how to debug Git issues from the command line by increasing verbosity of Git commands in Linux, MacOS and Windows.

**Cool Tip:** Enable DEBUG mode and increase VERBOSITY in Ansible! [Read more →](moz-extension://62849a71-f7f6-49e8-ae58-226e95d7425d/ansible-enable-debug-increase-verbosity/)

Git Verbose Mode in Linux/MacOS
-------------------------------

Debug Git command:

$ GIT\_TRACE=true \\
GIT\_CURL\_VERBOSE=true \\
GIT\_SSH\_COMMAND="ssh -vvv" \\
git clone git://host.xz/path/to/repo.git

Debug Git-related issues with the maximum level of verbosity:

$ GIT\_TRACE=true \\
GIT\_CURL\_VERBOSE=true \\
GIT\_SSH\_COMMAND="ssh -vvv" \\
GIT\_TRACE\_PACK\_ACCESS=true \\
GIT\_TRACE\_PACKET=true \\
GIT\_TRACE\_PACKFILE=true \\
GIT\_TRACE\_PERFORMANCE=true \\
GIT\_TRACE\_SETUP=true \\
GIT\_TRACE\_SHALLOW=true \\
git clone git://host.xz/path/to/repo.git

Git Verbose Mode in Windows
---------------------------

Debug Git command:

C:\\> set GIT\_TRACE=true
C:\\> set GIT\_CURL\_VERBOSE=true
C:\\> set GIT\_SSH\_COMMAND=ssh -vvv
C:\\> git clone git://host.xz/path/to/repo.git

Debug Git-related issues with the maximum level of verbosity:

C:\\> set GIT\_TRACE=true
C:\\> set GIT\_CURL\_VERBOSE=true
C:\\> set GIT\_SSH\_COMMAND=ssh -vvv
C:\\> set GIT\_TRACE\_PACK\_ACCESS=true
C:\\> set GIT\_TRACE\_PACKET=true
C:\\> set GIT\_TRACE\_PACKFILE=true
C:\\> set GIT\_TRACE\_PERFORMANCE=true
C:\\> set GIT\_TRACE\_SETUP=true
C:\\> set GIT\_TRACE\_SHALLOW=true
C:\\> git clone git://host.xz/path/to/repo.git

To avoid the error below, the variable `GIT_SSH_COMMAND=ssh -vvv` in Windows CMD must be set without quotes: “_fatal: ssh variant ‘simple’ does not support setting port_”

Git Debug Options
-----------------

The environment variables that can be used to increase verbosity in Git:

| Option | Description |
| --- | --- |
| `GIT_TRACE=true` | Enable general trace messages |
| `GIT_CURL_VERBOSE=true` | Print HTTP headers (similar to `curl -v`) |
| `GIT_SSH_COMMAND="ssh -vvv"` | Print SSH debug messages (similar to `ssh -vvv)` |
| `GIT_TRACE_PACK_ACCESS=true` | Enable trace messages for all accesses to any packs |
| `GIT_TRACE_PACKET=true` | Enable trace messages for all packets coming in or out of a given program |
| `GIT_TRACE_PACKFILE=true` | Enable tracing of packfiles sent or received by a given program |
| `GIT_TRACE_PERFORMANCE=true` | Enable performance related trace messages |
| `GIT_TRACE_SETUP=true` | Enable trace messages printing the `.git`, working tree and current working directory after Git has completed its setup phase |
| `GIT_TRACE_SHALLOW=true` | Enable trace messages that can help debugging fetching/cloning of shallow repositories |


[Source](https://www.shellhacks.com/git-verbose-mode-debug-fatal-errors/)