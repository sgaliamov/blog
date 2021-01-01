1. Install-Module posh-git -Scope CurrentUser
1. Install-Module oh-my-posh -Scope CurrentUser
1. Install Cascadia Code PL font
1. Install-Module -Name PSReadLine -Scope CurrentUser -Force -SkipPublisherCheck.
1. if you get "WARNING: The version '2.1.0' of module 'PSReadLine' is currently in use. Retry the operation after closing the applications." do it from other shell.
1. open profile notepad $PROFILE
1. put there

``` ps1
# Shows navigable menu of all options when hitting Tab
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete

# Autocompletion for arrow keys
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward

Import-Module posh-git
Import-Module oh-my-posh
Set-Theme Paradox
```

1. Edit settings:

   ``` json
   "profiles": {
    "defaults": {
      "colorScheme": "One Half Dark",
      "useAcrylic": true,
      "acrylicOpacity": 0.9,
      "fontFace": "Cascadia Code PL",
    },
    ```

As a bonus you will get nice looking terminal in VSCode.
