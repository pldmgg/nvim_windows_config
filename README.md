# nvim_windows_config
neovim config for windows

I use Neovim on Windows...because...reasons...

# Getting neovim working on Windows
- Install requirements: git, nodejs, mingw, neovim
choco install git
choco install mingw
choco install nodejs

- Create necessary directories
$DirsToCheck = @("$env:LOCALAPPDATA\nvim", "$env:LOCALAPPDATA\nvim-data", "$HOME\.vim")
foreach ($dir in $DirsToCheck) {if (!(Test-Path -Path $dir)) {$null = New-Item -Path $dir -ItemType Directory -Force}}

- Install Packer (Reference: 'https://github.com/wbthomason/packer.nvim#requirements')
git clone https://github.com/wbthomason/packer.nvim "$env:LOCALAPPDATA\nvim-data\site\pack\packer\start\packer.nvim"

- Deploy neovim baseline config (thanks ThePrimeagen)
git clone https://github.com/pldmgg/nvim_windows_config "$env:LOCALAPPDATA\nvim"

- Install neovim
choco install neovim

- The following powershell commands should succeed. If any of them dont, check your $env:Path
(Get-Command git).Source // Should return C:\Program Files\Git\cmd\git.exe
(Get-Command gcc).Source // Should return C:\ProgramData\chocolatey\bin\gcc.exe
(Get-Command npm).Source // Should return C:\Program Files\nodejs\npm.cmd
(Get-Command nvim).Source // Should return C:\tools\neovim\nvim-win64\bin\nvim.exe

- Run nvim for the first time
cd $env:LOCALAPPDATA
nvim .

- Skip past all of the initial errors, then in neovim, run:
:PackerCompile
:PackerClean
:PackerInstall
:PackerUpdate
:PackerSync

- Exit and reenter neovim
:q
nvim .

- Next run :Mason to check on the three LSPs in our baseline config: pyright, lua_ls, and powershell_es
:Mason

- If you want to work with python, install anaconda3:
choco install anaconda3

- Make sure C:\tools\Anaconda3 and C:\tools\Anaconda3\Scripts are part of your machine path
$machinePath = ([System.Environment]::GetEnvironmentVariable('PATH', [System.EnvironmentVariableTarget]::Machine).TrimEnd(';') -split ';' | Sort-Object | Get-Unique) -join ';'
$newMachinePath = (($machinePath + ';' + 'C:\tools\Anaconda3;C:\tools\Anaconda3\Scripts').TrimEnd(';') -split ';' | Sort-Object | Get-Unique) -join ';'
[System.Environment]::SetEnvironmentVariable('PATH', $newMachinePath,[System.EnvironmentVariableTarget]::Machine)

- Make sure you see the (base) python evironment indicator in your terminal after install
conda init

- You will need to exit and re-enter your shell after conda init

- Now we need to clean up some python references
    - Delete the following from C:\ProgramData\chocolatey\bin (which probably came from the mingw install)
        python.exe
        python3.9.exe
        python3.exe
        python3w.exe
        pythonw.exe
    - Then create some symlinks:
        mklink C:\ProgramData\chocolatey\bin\python.exe C:\tools\Anaconda3\python.exe
        mklink C:\ProgramData\chocolatey\bin\python3.exe C:\tools\Anaconda3\python.exe
        mklink C:\ProgramData\chocolatey\bin\pythonw.exe C:\tools\Anaconda3\pythonw.exe
        mklink C:\ProgramData\chocolatey\bin\pip.exe C:\tools\Anaconda3\Scripts\pip.exe
    - Set PYTHONPATH and PYTHONHOME to C:\tools\Anaconda3 in Windows Machine (system) Environment Variables (just do it via Control Panel)
    - Make sure Get-Command python refers to Anacondas python.exe
        (Get-Command python).Source // Should return C:\tools\Anaconda3\python.exe
    - Use pip to install neovim
        python -m pip install neovim


- After all of this, there is still a problem with the powershell Parser. If you do :LspInfo while in a .ps1 file, it will say that "0 client(s)" are connected to the buffer,
which basically means you don't get powershell intellisense within Neovim
    - Looking at :checkhealth, the error message under vim.treesitter: require("vim.treesitter.health").check()
    Error Parser "powershell" failed to load (path: C:\Users\{user}\AppData\Local\nvim-data\site\pack\packer\start\nvim-treesitter\parser\powershell.so): ...win64\share\nvim\rutime/lua/vim/treesitter/language.lua:99: ABI version mismatch for powershell.so: supported between 13 and 14, found 6