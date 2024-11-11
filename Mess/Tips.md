### Powershell 去除版本版权信息
执行命令中加入 -Nologo便可
![[Pasted image 20240409094800.png]]
```json
vsvode修改配置
"terminal.integrated.profiles.windows":
{
 "PowerShell": 
 {
   "path": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
   "args": ["-Nologo"],
   "icon": "terminal-powershell"
 }
},
"terminal.integrated.defaultProfile.windows": "PowerShell"
```

