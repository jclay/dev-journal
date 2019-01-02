# Towards a sane Windows 10 C++/CUDA/CMake development experience

A few notes regarding my attempts to streamline the development experience on Windows 10.

## Visual Studio Code

As Visual Studio Code becomes more feature rich, I find myself predominantly using it for C++ development instead of Visual Studio 2017. The speed, simplicity, built-in terminal and much improved font rendering make it a pleasure to use in comparison.

**Essential Extensions**

- cquery (must build from source on Windows currently)
- CMake Tools
- Microsoft C/C++ Extension
- Vim Emulation
- vscode-cudacpp for CUDA language highlighting

**Configuration**

- XCode Default Theme (Light)
- SF Mono Font
  - Size 16
  - Line Height 24
  
**CMake Tools**

A fantastic extension that can automatically find installed MSVC instances and use them to configure CMake. From there,
we get a great CMake config/edit/run experience that is exceeds any other editors/IDEs I have tried. 

**Powershell Core** 

I prefer using PowerShell core over the version included with Windows 10. I have it configured as my default terminal and I use the following in my profile for a keyboard shortcut and autocomplete experience that achieves most of what I was missing from bash/zsh. 

```
Set-PSReadlineOption -EditMode  Emacs
Set-PSReadlineKeyHandler -Chord Tab -Function MenuComplete
```

**Powershell and VCVars**

Microsoft unfortunately doesn't provide first-class support for command-line C++ development with Powershell. Instead of 
opening a new development terminal with the proper development environment set each time, I use the [VCVars](https://github.com/slurps-mad-rips/VCVars) Powershell module to load the variables from the cmd based toolset.

I added the following to my profile which is loaded into the default Visual Studio terminal:

```
$vc_env = Invoke-VCVars AMD64
setvc $vc_env
```

Now, I can use `cmake --build .` and `devenv my_sln.sln` to perform common CMake related tasks from the command line and open Visual Studio if needed.

