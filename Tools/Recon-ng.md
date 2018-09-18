# Recon-ng

## Download From
https://bitbucket.org/LaNMaSteR53/recon-ng

From a Linux/Unix/macOS command line, you can perform a `git clone`:
`git clone https://bitbucket.org/LaNMaSteR53/recon-ng.git`

## Purpose
"Recon-ng is a full-featured Web Reconnaissance framework written in Python. Complete with independent modules, database interaction, built in convenience functions, interactive help, and command completion, Recon-ng provides a powerful environment in which open source web-based reconnaissance can be conducted quickly and thoroughly." (https://bitbucket.org/LaNMaSteR53/recon-ng, February 3, 2018)

## Uses
Reconnaissance framework that has modules to perform host, IP, domain, and user-based recon/OSINT.

## Examples
- `recon-ng --no-analytics` - It is important to use the `--no-analytics` flag to prevent Recon-ng from sending usage data to Google Analytics.
- `recon-ng --no-analytics --no-check` - *[In Class Only]* This prevents Recon-ng from checking to see if it is the latest version. In class, it is important to use this flag as the labs have been tested against a certain version of the application. In the real world, remove this and ensure that you are using the latest version.

## Helpful Switches and Flags
None. Recon-ng is most commonly run through the interactive command line interface.

## Interactive Commands
- `help` - Shows the help :)
- `search` - Searches modules for a string
- `use` - Loads a module
- `workspaces` - Manage workspaces to keep your data separate