# CoilSnake Project Builder for GitHub Actions

This action builds and creates .ebp and .bps patches for the [CoilSnake](https://github.com/pk-hack/CoilSnake) projects.

Currently supports CoilSnake's [346cfc7](https://github.com/pk-hack/CoilSnake/commit/346cfc753644bc3703b6fc4eaa0a5d6bdcb9bb4a) and CCScript's [80a03df](https://github.com/pk-hack/ccscript_legacy/commit/80a03df13cfe9bd3aab5f0d7d34aad7dd2c7bae0) builds.

This Project idea inspired (stolen) from [Lorenzooone](https://hub.docker.com/r/lorenzooone/m2gba_translation)

## Usage

### Inputs
```
- name: Building the ROM
  uses: ozer-4/coilsnake-action@latest
  with:
    rom-hash: ${{ secrets.ROM_HASH }}
    patch-file-name: 
    expanding-option: 
    title-name: 
    author-name: 
    ebp-description: 
```

| Name | Description | Required | Default |
| - | - | - | - |
| `rom-hash` | sha256 hash of the EarthBound (USA) 24Mbit headerless ROM file.<br><br>If you don't know is your ROM compatible.<br>You can check it with [Supremekirb's EB ROM Inspector](https://bitdragon.dev/tools/rom_inspector.html). <br><br>Please use it with [GitHub's secrets feature](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets). | Yes | |
| `patch-file-name` | Name of the output patch files. | No | eb |
| `expanding-option` | This will expand the ROM size to [32MBit / 48MBit].<br>You can skip this step if expansion is not required. | No | No Expansion |
| `title-name` | Project title for the EBP patch file. | No | [title](https://github.com/pk-hack/CoilSnake/blob/346cfc753644bc3703b6fc4eaa0a5d6bdcb9bb4a/coilsnake/ui/cli.py#L123) |
| `author-name` | Project description for the EBP patch file. | No | [author](https://github.com/pk-hack/CoilSnake/blob/346cfc753644bc3703b6fc4eaa0a5d6bdcb9bb4a/coilsnake/ui/cli.py#L119) |
| `ebp-description` |  Project title for the EBP patch file. | No | [description](https://github.com/pk-hack/CoilSnake/blob/346cfc753644bc3703b6fc4eaa0a5d6bdcb9bb4a/coilsnake/ui/cli.py#L121) |

### Outputs

Both patch files are saved in the /opt directory of the GitHub Actions virtual machine.

## Example
```
name: Build the Patch

on:
    push:
        tags:
            - 'v*'

env:
    FILE_NAME: pweh_patch

permissions: read-all

jobs:
    build:
        name: Building the ROM
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: Building the ROM # Builds and Creates a patch of the ROM 
              uses: ozer-4/coilsnake-action@main 
              with:
                  rom-hash: ${{ secrets.ROM_HASH }}
                  patch-file-name: ${{ env.FILE_NAME }}
                  expanding-option: 32MBit
                  title-name: Poorly Written EarthBound Hack(PWEH)
                  author-name: ozer-4
                  ebp-description: |
                      WORLD'S WORST EB HACK EVER
                      
                      made by me of course...
                      
                      wait, THIS IS NOT APEH by Sunflower(go play it!) THATS A DIFFERENT HACK
                      DO NOT CONFUSE IT WITH THIS!
            - name: Upload Patch Artifacts # Uploads the files to the other job for giving writing permisions only for the publishing process.
              uses: actions/upload-artifact@v4
              with:
                  name: pweh-patch-files
                  path: |
                      out/${{ env.FILE_NAME }}.bps
                      out/${{ env.FILE_NAME }}.ebp

    publish:
        name: Publishing the Patch
        runs-on: ubuntu-latest
        needs: build
        permissions:
            contents: write
        steps:
            - name: Download Patch Artifacts # Downloads the files
              uses: actions/download-artifact@v4
              with:
                  name: pweh-patch-files
            - name: Upload Release Assets
              uses: softprops/action-gh-release@v3 # And publishes them.
              if: github.ref_type == 'tag'
              with:
                  files: |
                      ${{ env.FILE_NAME }}.bps
                      ${{ env.FILE_NAME }}.ebp
              env:
                  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

# Credits:
* [Lorenzooone](https://github.com/Lorenzooone) - For the idea.
* [mrtenda](https://github.com/mrtenda) and [PK Hack Members](https://github.com/pk-hack) - For the [CoilSnake Project](https://github.com/pk-hack/CoilSnake)
* [![Supremekirb](https://bitdragon.dev/images/gayass-derg/88x31.png)](https://bitdragon.dev)  - For the [EB ROM Inspector](https://bitdragon.dev/tools/rom_inspector.html)
