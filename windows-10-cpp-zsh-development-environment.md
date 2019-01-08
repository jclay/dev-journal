# Creating a C++ development environment on Windows 10 with zsh

Since moving from MacOS, I've been having a hard time getting used to PowerShell (although enabling Emacs Mode goes a long way).

I still find myself missing the fantastic zsh autocomplete functionality, so the following is what I'm using to get a 
zsh based experience that works pretty well.

Note that while I've used WSL extensively at the time of writing, I'm using the MSYS2 version of MSYS2 due to improved performance and more
reliable integration with some Windows tooling.

I followed [this guide](https://medium.com/@borekb/zsh-via-msys2-on-windows-3964a943b1ce) for getting up and running with a nice zsh configuration.

## Integration with MSVC

With a working zsh install, we need to do some work to get the MSVC tooling and other setup which typically happens in the Native Tools Command Prompt.

Note that we need the VCVars powershell module installed: https://github.com/slurps-mad-rips/VCVars 

Ultimately, we take the output of the VCVars module and we perform some translation to make that new Paths work on MSYS2.
Then, I write the new environment to a `.env` file, which gets loaded by zsh when I enter the directory.

I created a hacky `build_path.ps1` in my project directory with the following:

```powershell
# build_path.ps1

# the MSVC compiler configuration you want can be specified here:
# see VCVars powershell module docs for options.
$vc_env = Invoke-VCVars -t AMD64 -h x86

# We take the output and translate paths so that they work within MSYS2/Cygwin:
# Make sure to use fwd slashes
$out_path = ""
$vcpkg_dir = "C:/vcpkg" # only needed if you're usign VCPKG
$toolchain_path = "C:/vcpkg/scripts/buildsystems/vcpkg.cmake"

# You could use the following to see what $vc_env sets:
# echo $vc_env

# We take the Path VCVars (which includes paths to MSVC tooling) gives us and we split it and translate the paths for
# cygwin to use
$vc_env.Item("Path").Split(";") | % {
    if ($_.length -gt 1) {
        $out_path += "'$(cygpath -u $_)':"
    }
}
$vc_env.Remove("Path")

# Append vcpkg to path (optional)
$out_path += ":'$(cygpath -u $vcpkg_dir)'"



# Set the rest of the keys as environment variables:
$vc_env.Keys | % { 
    $val = $vc_env.Item($_) -replace '\\', '/'
    "$_=`"$val`""
} | Out-File .env

"PATH=" + $out_path | Out-File .env -append
# make vcpkg default to using x64 builds
"VCPKG_DEFAULT_TRIPLET=" + "x64-windows" | Out-File .env -append
"CMAKE_TOOLCHAIN_FILE=" + "`"$toolchain_path`"" | Out-File .env -append
```

Ensure you are in the project directory since it will write the .env file used to setup the environment when we enter the directory within `zsh`
From a Powershell prompt, run `./build_path.ps1` you should see a .env file which looks like:

Note that only the paths where we are calling an executable from zsh need to be translated. The others have to remain the same for the rest of our tooling to work.

```
__VSCMD_PREINIT_VCToolsVersion="14.16.27023"
VSCMD_ARG_HOST_ARCH="x86"
LIBPATH="C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x86/store/references;C:/Program Files (x86)/Windows Kits/10/UnionMetadata/10.0.17763.0;C:/Program Files (x86)/Windows Kits/10/References/10.0.17763.0;C:/Windows/Microsoft.NET/Framework/v4.0.30319;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x86/store/references;C:/Program Files (x86)/Windows Kits/10/UnionMetadata/10.0.17763.0;C:/Program Files (x86)/Windows Kits/10/References/10.0.17763.0;C:/Windows/Microsoft.NET/Framework64/v4.0.30319;"
INCLUDE="C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/include;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/include;C:/Program Files (x86)/Windows Kits/NETFXSDK/4.6.1/include/um;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/ucrt;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/shared;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/um;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/winrt;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/cppwinrt;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/include;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/include;C:/Program Files (x86)/Windows Kits/NETFXSDK/4.6.1/include/um;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/ucrt;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/shared;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/um;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/winrt;C:/Program Files (x86)/Windows Kits/10/include/10.0.17763.0/cppwinrt"
CommandPromptType="Cross"
FrameworkDir="C:/Windows/Microsoft.NET/Framework/"
__DOTNET_ADD_32BIT="1"
FrameworkVersion32="v4.0.30319"
__VSCMD_PREINIT_VS150COMNTOOLS="C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/Tools/"
LIB="C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x64;C:/Program Files (x86)/Windows Kits/NETFXSDK/4.6.1/lib/um/x64;C:/Program Files (x86)/Windows Kits/10/lib/10.0.17763.0/ucrt/x64;C:/Program Files (x86)/Windows Kits/10/lib/10.0.17763.0/um/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/ATLMFC/lib/x64;C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/lib/x64;C:/Program Files (x86)/Windows Kits/NETFXSDK/4.6.1/lib/um/x64;C:/Program Files (x86)/Windows Kits/10/lib/10.0.17763.0/ucrt/x64;C:/Program Files (x86)/Windows Kits/10/lib/10.0.17763.0/um/x64;"
FrameworkDir32="C:/Windows/Microsoft.NET/Framework/"
__DOTNET_PREFERRED_BITNESS="32"
PATH='/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/HostX86/x64':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/HostX86/x86':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/VC/VCPackages':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/TestWindow':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/TeamFoundation/Team Explorer':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/MSBuild/15.0/bin/Roslyn':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Team Tools/Performance Tools/x64':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Team Tools/Performance Tools':'/c/Program Files (x86)/Microsoft Visual Studio/Shared/Common/VSPerfCollectionTools/x64':'/c/Program Files (x86)/Microsoft Visual Studio/Shared/Common/VSPerfCollectionTools/':'/c/Program Files (x86)/Microsoft SDKs/Windows/v10.0A/bin/NETFX 4.6.1 Tools/':'/c/Program Files (x86)/Windows Kits/10/bin/10.0.17763.0/x86':'/c/Program Files (x86)/Windows Kits/10/bin/x86':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/MSBuild/15.0/bin':'/c/Windows/Microsoft.NET/Framework/v4.0.30319':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/Tools/':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/HostX64/x64':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/VC/VCPackages':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/TestWindow':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/TeamFoundation/Team Explorer':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/MSBuild/15.0/bin/Roslyn':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Team Tools/Performance Tools/x64':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Team Tools/Performance Tools':'/c/Program Files (x86)/Microsoft Visual Studio/Shared/Common/VSPerfCollectionTools/x64':'/c/Program Files (x86)/Microsoft Visual Studio/Shared/Common/VSPerfCollectionTools/':'/c/Program Files (x86)/Microsoft SDKs/Windows/v10.0A/bin/NETFX 4.6.1 Tools/x64/':'/c/Program Files (x86)/Windows Kits/10/bin/10.0.17763.0/x64':'/c/Program Files (x86)/Windows Kits/10/bin/x64':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/MSBuild/15.0/bin':'/c/Windows/Microsoft.NET/Framework64/v4.0.30319':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/Tools/':'/c/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.0/bin':'/c/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.0/libnvvp':'/c/Python37/Scripts/':'/c/Python37/':'/c/Windows/system32':'/c/Windows':'/c/Windows/System32/Wbem':'/c/Windows/System32/WindowsPowerShell/v1.0/':'/c/Windows/System32/OpenSSH/':'/c/Program Files/Microsoft VS Code/bin':'/c/ProgramData/chocolatey/bin':'/c/Program Files/dotnet/':'/c/Program Files/Microsoft SQL Server/130/Tools/Binn/':'/c/Program Files (x86)/IncrediBuild':'/c/Program Files/Git/cmd':'/c/Program Files (x86)/NVIDIA Corporation/PhysX/Common':'/c/Program Files/PowerShell/6/':'/c/Program Files/nodejs/':'/usr/bin':'/mingw64/bin':'/c/Program Files/CMake/bin':'/c/Users/joel/AppData/Local/Microsoft/WindowsApps':'/':'/c/Users/joel/AppData/Roaming/npm':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/bin':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/Ninja':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/bin':'/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/Ninja'::'/c/vcpkg'
VCPKG_DEFAULT_TRIPLET=x64-windows
CMAKE_TOOLCHAIN_FILE="C:/vcpkg/scripts/buildsystems/vcpkg.cmake"
```

## Setup zsh dotenv support

In your `~/.zshrc` append the following if you're using antigen (from the guide above):

```
# append somewhere before apply antigen is called
...
antigen bundle dotenv
...
```

Now, when you restart zsh and cd into your project directory, you can verify your environment variables have been set properly:
echo $PATH or any of the other variables you want to verify.

## Visual Studio Code Setup

I'm using this with the Shell Launcher extension to enable use of Powershell or Zsh within VSCode.

I have the following in my settings.json:

```json
    "shellLauncher.shells.windows": [
        {
            "shell": "C:\\Program Files\\PowerShell\\6\\pwsh.exe",
            "label": "pwsh"
        },
        {
            "shell": "C:\\tools\\msys64\\usr\\bin\\zsh.exe",
            "label": "zsh"
        }
    ]
```

### Please open a PR or issue if there is a better way to handle any of the above.
