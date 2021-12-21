# How We Use NPM for Frontend Development

We use npm to manage packages for frontend development. Currently Vercel does not support node 16, so we are on node version 14, which is locked in the `package.json` file. However, we strive to use the latest LTS version, when our infrastructure supports it.

## NVM

If developing on several frontends, that use different versions of node, we recommend you install [nvm](https://github.com/nvm-sh/nvm) for `bash` shell or [nvm.fish](https://github.com/jorgebucaran/nvm.fish) for `fish` shell.
With NVM, changing versions is as easy as writing `nvm use lts` or `nvm use 14`. Keep in mind though, that global NPM packages will be installed with the node version you are currently on.
