# Wget

## Download From
https://www.gnu.org/software/wget/

## Purpose
"GNU Wget is a free software package for retrieving files using HTTP, HTTPS, FTP and FTPS the most widely-used Internet protocols. It is a non-interactive commandline tool, so it may easily be called from scripts, cron jobs, terminals without X-Windows support, etc." (https://www.gnu.org/software/wget/, February 3, 2018)

## Uses
1. Downloading files from the internet
2. Recursively downloading web sites

## Examples
- `wget [flags] protocol://site`
- `wget https://www.sans.org`
- `wget -U "2018 Chevy Volt" -m --no-check-certificate https://www.sans.org` - 

## Helpful Switches and Flags
- `-e robots=off` - Wget, by default, abides by the robots.txt files that many sites have at their root levels (e.g., https://www.gnu.org/robots.txt). This bypasses those directives and lets Wget retrieve content a "nice" spider would not.
- `-m` - shortcut for -N (timestamping) -r (recursive) -l inf (infinite levels) --no-remove-listing
- `-U "some string"` - Change the User Agent to "some string"
- `--no-check-certificate` - Some of the web sites that you will want to recover data from will have HTTPS but invalid, expired or self-signed certificates. This switch tells Wget to ignore those bad certs.