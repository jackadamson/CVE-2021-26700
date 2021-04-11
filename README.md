# RCE in NPM VSCode Extension

Fixed 10th February 2021 in https://github.com/microsoft/vscode-npm-scripts/commit/cdd5e507564e0cc0f60bcccf184822be3fd73e07

https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-26700

## Summary

Remote code execution vulnerability in the [eg2.vscode-npm-script](https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script)
(Tested on version 0.3.13) VSCode extension means that a malicious `.vscode/settings.json` in a project can cause
remote code execution when a file named `package.json` is viewed.

This vulnerability did not fall under any active Microsoft bug bounty program, but was reported to and handled by MSRC.

## Description

Without a malicious `.vscode/settings.json` file in a repository, upon viewing a `package.json` file, with the
`eg2.vscode-npm-script` extension installed, the command `npm ls --depth 0 --json` will be executed.

By setting the `npm.bin` key in a projects `settings.json` eg. to `./payload.sh`, viewing the `package.json` will
instead execute `./payload.sh ls --depth 0 --json` from the directory containing the `package.json`.

This vulnerability breaks the assumption that source-code can be safely read.

## Example Attack

An example of how this could be used by an attacker is:

1. Attacker publishes repository such as attached linux-poc
2. Target clones repository to read the source code in VSCode
3. Target views `package.json`
4. `payload.sh` is executed


## Remediation Options

I've seen two ways of mitigating malicious VSCode Workspace `settings.json`

### Mitigation 1 - User Only Settings

Forbid the settings to be set per workspace. This is already used by VSCode for `git.path`, `terminal.integrated.shell.linux` and a few other settings.

This could break compatibility for some users, but has precedent.  
Reference: https://vscode.readthedocs.io/en/latest/getstarted/settings/#settings-and-security

### Mitigation 2 - Prompt for Confirmation

If this setting is set in a workspace, prompt the user to confirm the setting before executing the binary.  
This will not break existing compatibility, and is the route taken by the ESLint Extension as of version 2.1.7.

## Proof of Concept

### Linux

1. Install vulnerable version (0.3.13 or earlier) of VSCode extension `eg2.vscode-npm-script`
2. Open `linux-poc` directory as a folder in VSCode
3. In VSCode click the `package.json` file to view it
4. The file `/tmp/output.txt` will be created to demonstrate execution


### Windows

1. Install vulnerable version (0.3.13 or earlier) of VSCode extension `eg2.vscode-npm-script`
2. Open `windows-poc` directory as a folder in VSCode
3. In VSCode click the `package.json` file to view it
4. `calc.exe` will be opened

Note: I was unable to use a relative path for the binary in Windows, however this may be due to my lack of familiarity with Windows
