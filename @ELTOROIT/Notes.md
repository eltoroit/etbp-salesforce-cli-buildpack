# Buildpacks

- https://devcenter.heroku.com/articles/buildpack-api
- Adding a build pack
  - `https://github.com/eltoroit/etbp-salesforce-cli-buildpack.git`
  - `https://github.com/eltoroit/etbp-salesforce-cli-buildpack.git#Tests`

## Notes

- Use `sh` or `bash` to ensure compatibility with future Heroku stacks
- A buildpack consists of three scripts:
  - `bin/detect`: Determines whether to apply this buildpack to an app
  - `bin/compile`: Used to perform the transformation steps on the app
  - `bin/release`: Provides metadata back to the runtime

## bin/detect

- This script helps detect if the buildpack is applicable to this project
- `bin/detect BUILD_DIR`
  - $1: BUILD_DIR

```
# this pack is valid for apps with a hello.txt in the root
if [ -f $1/hello.txt ]; then
  echo "HelloFramework"
  exit 0
else
  exit 1
fi
```

- The name sent to stdout will be displayed as the application type during push
- It must return an exit code of 0 if the app present at BUILD_DIR can be serviced by this buildpack
  - If the exit code is 0, the script must print a human-readable framework name to stdout

## bin/compile

- This script performs the buildpack transformation.
- `bin/compile BUILD_DIR CACHE_DIR ENV_DIR`
  - $1: BUILD_DIR
    - App's location
  - $2: CACHE_DIR
    - Cache between builds
  - $3: ENV_DIR
    - Directory with a file for each of the app’s configuration variables
    - Config vars are available as environment variables (Node.js = process.env.\*)
    - The config vars are not set as environment variables at this time, to allow `export` none, all or parts of the app config vars available in ENV_DIR.
    - Also, it makes it easier for buildpacks to correctly handle config vars with multi-line values
    ```
    export_env_dir() {
        env_dir=$1
        acceptlist_regex=${2:-''}
        denylist_regex=${3:-'^(PATH|GIT_DIR|CPATH|CPPATH|LD_PRELOAD|LIBRARY_PATH)$'}
        if [ -d "$env_dir" ]; then
            for e in $(ls $env_dir); do
                echo "$e" | grep -E "$acceptlist_regex" | grep -qvE "$denylist_regex" && export "$e=$(cat $env_dir/$e)"
                :
            done
        fi
    }
    ```
- The contents of CACHE_DIR will be persisted between builds.
  - You can cache the results of long processes like dependency resolution here to speed up future builds
- All output received on stdout from this script will be displayed to the user and stored with the build
- The application in BUILD_DIR along with all changes made by the compile script will be packaged into a slug
- Additional environment variables

  - STACK
    - ???
  - SOURCE_VERSION
    - “version” of the source code currently being built (For git-push builds, this is the git commit SHA-1 hash)
  - Style
    - display the versions of the software being used.
    ```
    -----> Main actions are prefixed with a 6-character arrow
            Additional information is indented to align
    ```
  - Side effects
    - Avoid generating side effects during the build (do not perform database migrations in the compile script)

## bin/release

- `bin/release BUILD_DIR`
- This script will only be run if present.
- This script returns a YAML formatted hash with two keys:
  - `addons` is a list of default addons to install.
  - `default_process_types` is a hash of default Procfile entries.

```
#!/bin/sh

cat << EOF
---
addons:
  - heroku-postgresql
default_process_types:
  web: bin/node server.js
EOF
```

# Bash help

- Bash docs:

  - https://tldp.org/LDP/abs/html/internalvariables.html

- Environment variables used in the `bin/compile` script

| EnvVar     | Who? | What?                                              |
| ---------- | ---- | -------------------------------------------------- |
| `$SECONDS` | Bash | The number of seconds the script has been running. |

- Functions used in the `bin/compile` script

| EnvVar               | Where?        | What?                                                        |
| -------------------- | ------------- | ------------------------------------------------------------ |
| `header`             | lib/common.sh | echo with arrow (`----->`)                                   |
| `log`                | lib/common.sh | echo with spaces                                             |
| `export_env`         | lib/stdlib.sh | Exports the environment variables defined in given directory |
| `setup_dirs`         | bin/compile   | Exports PATH                                                 |
| `install_sfdx_cli`   | bin/compile   | `npm install sfdx-cli --global` and `sfdx -v`                |
| `fix_sfdx_cli`       | bin/compile   | Fixes issue with `legacyClientId` in `authInfo.js:64`        |
| `install_ETCopyData` | bin/compile   | `npm install etcopydata@2.0.3 --global` and `sfdx plugins`   |

- Variables
  - VAR=${[number]:-default}
    - `VAR1=${1:-8}`
    - ${1:-8} is a positional parameters $1.
    - These hold the values passed to the script (or shell function) on the command line.
    - If they are not set or empty, the variable substitutions will use the default value of 8
  -
