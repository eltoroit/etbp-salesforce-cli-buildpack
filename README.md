# buildpack for sfdx cli

## Original Notes

I took this from the official buildpack, forked for stability and removed jq because I don't use it in node (node does javacript just fine)

## Usage

To use `sfdx`, you simply need to export the appropriate paths:

```
export PATH="$BUILD_DIR/vendor/sfdx/cli/bin:$PATH"
```

The `$BUILD_DIR` is the path where your apps source is stored on the Heroku dyno.

- forked and cleaned from [mshanemc/salesforce-cli-buildpack](https://github.com/mshanemc/salesforce-cli-buildpack)

## 2021-11-23

- Added ETCopyData version 0.6.6-Beta

## 2022-07-26

- Updated ETCopyData version to the latest

## 2022-08-21

- Modified a file in the SFDX file set to fix an issue with creating scratch orgs based on a session Id.
