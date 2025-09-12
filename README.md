# Workspace Configuration

Greetings to my future self! (and any other poor soul that somehow ended up here)

The following constitutes a set of notes and instructions on how to configure a new computer to match my workflow preferences.
For now, this is specific to MacOS, but will eventually be extended to some Linux distributions.

## Basic downloads and installations

The first thing to install is [Homebrew](https://brew.sh/) with the following terminal command.
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Then, download and install
- [Visual Studio Code](https://code.visualstudio.com/download) and
- [MacTex](https://code.visualstudio.com/download).

Finally, use Homebrew to install Git, CMake and GLFW (for OpenGL).
```
brew install git
brew install cmake
brew install glfw
```
## Configuration

### LaTeX

In Visual Studio Code, install the [LaTeX Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop) extension by James Yu.
For this extension, change the following settings:

> Latex-workshop > Latex > Auto Clean: Run\
> Set to "onBuilt".

> Latex-workshop > Latex > Clean: File Types\
> Add the item "%DOCFILE%.synctex.gz".

> Latex-workshop > Latex > Clean: Method\
> Set to "glob".

> Latex-workshop > Latex> Clean > Subfolder: Enabled\
> Check the "Delete LaTeX auxiliary files recursively" box.

### C++

In Visual Studio Code, install the three following extensions:

- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) by Microsoft,
- [C/C++ Themes](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-themes) by Microsoft and
- [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) by Microsoft

### Git

In Visual Studio Code, install the [GitHub Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) extension by GitHub.
For this extension, it is necessary to sign in to a GitHub account from Visual Studio Code.
([Further instuctions](https://code.visualstudio.com/docs/sourcecontrol/github))

It is also necessary to set a username and email address in Git from the terminal.
```
git config --global user.name "yourusername"
git config --global user.email "your@email.com"
```
(Further detail for setting the [username](https://docs.github.com/en/get-started/git-basics/setting-your-username-in-git) and the [email address](https://docs.github.com/en/account-and-profile/how-tos/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address).)

## Useful commands and functionnality

### C++

Once a `CMakeLists.txt` file is included in the C++ project directory and the proper configurations have been included in the file,
the code is the compiled from the Visual Studio Code terminal interface using the following commands.

First ensure a `build` directory is included in the project directory, then create the local build files with
```
cmake -S . -B build
```
The code can then be compiled with
```
cmake --build ./build
```
Finally, the compiled code can be run with
```
./build/ProjectName
```
