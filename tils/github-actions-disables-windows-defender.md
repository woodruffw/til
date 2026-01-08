---
title: GitHub Actions disabled Windows Defender
date: 2026-01-07
tags: [github-actions, windows, security]
---

This makes perfect sense when you think about it[^why], but I had never gone looking
for it before. But today I did:

[actions/runner-images@a3ef6b2](https://github.com/actions/runner-images/blob/a3ef6b2b8f38411b1f9938d5da919c2166fbab8b/images/windows/scripts/build/Configure-WindowsDefender.ps1)

Excerpted:

```powershell
$avPreference = @(
    # a lot of settings here
)

$avPreference | Foreach-Object {
    $avParams = $_
    Set-MpPreference @avParams
}

# https://github.com/actions/runner-images/issues/4277
# https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-antivirus-compatibility?view=o365-worldwide
$atpRegPath = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection'
if (Test-Path $atpRegPath) {
    Write-Host "Set Microsoft Defender Antivirus to passive mode"
    Set-ItemProperty -Path $atpRegPath -Name 'ForceDefenderPassiveMode' -Value '1' -Type 'DWORD'
}
```

Relevant cmdlets:

- [Set-MpPreference](https://learn.microsoft.com/en-us/powershell/module/defender/set-mppreference) (for setting Defender preferences)
- [Set-ItemProperty](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-itemproperty) (for modifying the registry key that controls Defender's passive mode)

[^why]: Performance, plus CI environments run untrusted, network-sourced code by design.
