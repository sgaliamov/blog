---
title: How to get a decent terminal for Windows
categories: windows terminal
created: 2021-01-01
date: 2021-01-02
layout: post
---

In this small post I [show](#step-by-step-instruction) how to configure `Powerlines` for `PowerShell Core` and `Windows Terminal`.

<img src="https://sgaliamov.github.io/blog/assets/wt-overview.png" alt="Windows Terminal" height="256px">

Default shells on Windows are far from perfect.
Very, very far, to put it mildly.
I think, the reason for this is that Microsoft doesn't really want their users to go into the internal details of their products.
Therefore, they don't give us the proper tools.
Or perhaps this is a weird way of promoting Linux systems.
But in recent years, the situation has been changing for the better.
What makes me happy.

I've been using [Cmder](https://cmder.net/) for a long time.
It's a very good tool, but it always felt unnatural and incomplete.
So, I periodically try to find some alternative.
This time I'm trying [Windows Terminal](https://github.com/microsoft/terminal).
And it looks very promising:

1. It's open source.
1. It's configurable.
1. It supports multiple panes.
1. It has tabs.
1. It has command pallet.
1. It supports many shells.
1. It supports transparency and custom backgrounds.

But it's not perfect yet.
There are some small limitations that bother me:

1. You cannot change size of a pane.
1. You cannot pin tabs to bottom.
1. Urls are not clickable.
1. It does not have good autocompletion. It just uses capabilities of a shell.

## Step by step instruction

1. Install [Cascadia](https://github.com/microsoft/cascadia-code) font.\
   You need exactly `Cascadia Code PL` version because it has ligatures and powerline symbols.
1. Install `Windows Terminal`. You can use `Windows Store`, but I prefer to use a console:

   ``` ps1
   choco install microsoft-windows-terminal
   ```

1. Run the following commands in `PowerShell Core`:

   ``` ps1
   Install-Module posh-git -Scope CurrentUser
   Install-Module oh-my-posh -Scope CurrentUser
   Install-Module -Name PSReadLine -Scope CurrentUser -Force -SkipPublisherCheck
   ```

1. If you get something like

   > WARNING: The version '2.1.0' of module 'PSReadLine' is currently in use. Retry the operation after closing the applications.

   do it from `cmd`:

   ``` cmd
   pwsh -Command "Install-Module -Name PSReadLine -Scope CurrentUser -Force -SkipPublisherCheck"
   ```

1. Press `Ctrl+,` in `Windows Terminal` and put the following configuration to the settings:

   ``` json
   ...
   "profiles": {
      "defaults": {
         "colorScheme": "One Half Dark",
         "useAcrylic": true,
         "acrylicOpacity": 0.9,
         "fontFace": "Cascadia Code PL",
      },
      "list": [
   ...
   ```

   If you don't want to have the same settings for all profiles, you can apply them only for `PowerShell`.

1. Open or create `PowerShell Core` profile in your favorite editor. You can find it here `C:\Users\<your user name>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`. Or you can just run `notepad $PROFILE` in `PowerShell Core`.
1. Put there:

   ``` ps1
   # Shows navigable menu of all options when hitting Tab
   Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete

   # Autocompletion for arrow keys
   Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
   Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward

   # Configure Powerlines
   Import-Module posh-git
   Import-Module oh-my-posh
   Set-Theme Paradox
   ```

As a bonus, you will get a nice looking terminal in VSCode.

<img src="https://sgaliamov.github.io/blog/assets/vscode-terminal.png" alt="VSCode Terminal" height="256px">

More details you can find on the [documentation page](https://docs.microsoft.com/en-us/windows/terminal/).
