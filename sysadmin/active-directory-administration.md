---
description: This page is dedicated to any and all Active Directory administration
---

# Active Directory Administration

## FILE SYSTEM ADMINISTRATION

### Getting Directory Sizes

I use the following command to generate a list of user profile's on a file server. It is useful to keep track of users that are exceeding our expectations when it comes to consuming space on a global server:

`Get-ChildItem | Where-Object { $.PSIsContainer } | ForEach-Object { $.Name + ": " + "{0:N2}" -f ((Get-ChildItem $_ -Recurse | Measure-Object Length -Sum -ErrorAction SilentlyContinue).Sum / 1MB) + " MB" }`

### Tail a File

Similar to `tail -f filename`, you can use `Get-Content` to watch a file for changes:

 `Get-Content -Path "\\server\logs\prod.server.log" -Wait` 

