# exiftool

## Download From
http://www.sno.phy.queensu.ca/~phil/exiftool/

## Purpose
"ExifTool is a platform-independent Perl library plus a command-line application for reading, writing and editing meta information in a wide variety of files." (http://www.sno.phy.queensu.ca/~phil/exiftool/, February 3, 2018)

## Uses
View and edit metadata in files.

## Examples
- `exiftool FILENAME` - Outputs the metadata found in the FILENAME file to the screen
- `exiftool -c "%.7f deg" -GPSPosition *.jpg` - Output the GPSPosition property of all files in the current directory ending in a .jpg extension and put the GPS coordinates in a decimal format (e.g., 10.1010101 deg N, 22.222222 deg E)
- 

## Helpful Switches and Flags
- `-csv` - Outputs data in CSV format
- `-PARAMETER` - Pulls the appropriate parameter from the metadata (e.g., `-Description` or `-GPSPosition`)