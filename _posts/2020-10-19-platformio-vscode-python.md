---
layout: post
title:  "Make PlatformIO in VSCode use a specific python version"
date:   2020-10-19 14:17:14 +0200
author: bronsen
categories: howto
---
re: [https://github.com/platformio/platformio-core/issues/3700](https://github.com/platformio/platformio-core/issues/3700)

# My setup

I use [fish][2] as main shell; in fish I use [pyenv][3] and [virtualfish][4].

The following steps should find their equivalents in other shells too: if you
know how to create a virtualenv using a specific python version, you should be
able to replicate the result.

# How to make PlatformIO use a specific python version

0. Don't have vscode running and don't have the platformio extension installed.
1. Install latest python 3.8: 
```
    $ pyenv install 3.8.6
```
This might take a few minutes to compile; also might complain about missing
packages. ...and this is where it gets too platform specific.
2. Create a virtual environment for platformio/vscode:
```
    $ vf new -p 3.8 platformio
```
(I could not specify 3.8.6 but only 3.8; and virtualfish found 3.8.5 first.
Later I could `vf upgrade --rebuild --python 3.8` to get 3.8.6 🤷)
3. Activate said virtual environment and start vscode:
```
    $ vf activate platformio
    $ code
```
(The binary is called `code` on my system.)
4. [Install the extension][5] in vscode.

[0]: https://platformio.org/             "collaborative platform for embedded development"
[1]: https://code.visualstudio.com/      "free cross-platform IDE from Microsoft"
[2]: https://fishshell.com/              "a command line shell for the 90s, not POSIX compatible"
[3]: https://github.com/pyenv/pyenv      "Simple Python version management"
[4]: https://virtualfish.readthedocs.io/ "virtual environment manager for fish"
[5]: https://docs.platformio.org/en/latest/integration/ide/vscode.html#installation



