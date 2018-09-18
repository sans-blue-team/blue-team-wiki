# curl

## Download From
https://www.gnu.org/software/wget/

## Purpose
"curl is used in command lines or scripts to transfer data. It is also used in cars, television sets, routers, printers, audio equipment, mobile phones, tablets, settop boxes, media players and is the internet transfer backbone for thousands of software applications affecting billions of humans daily." (https://curl.haxx.se/, February 3, 2018)

## Uses
1. Downloading files from the internet

## Examples
- `curl [flags] protocol://site`
- `curl https://www.sans.org`
- `curl -A "2018 Chevy Volt" https://www.sans.org > index.html` - Using the redirector `>` outputs whatever file curl retrieves to a file on the local system. Without the redirector or a `-o filename`, the retrieved data will be shown in standard output (the screen).

## Helpful Switches and Flags
- `-A "some string"` - Change the User Agent to "some string"
- `-o outputfile` - Specify a file for the data retrieved to be saved in