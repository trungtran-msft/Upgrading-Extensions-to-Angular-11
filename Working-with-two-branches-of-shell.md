# Switching development environments

There are a lot of environmental changes in the new shell. One such change is the use of node 12.18.3 from the previous 10.22.0 version. 
These versions are incompatible and you must toggle your global version to run build commands in each environment. 

For this, we suggest using Node Version Manager:
https://github.com/coreybutler/nvm-windows

Follow the instructions to install nvm-windows on your machine.

Once installed, you can prepare your environment by running these commands:
```
nvm install 12.18.3
nvm use 12.18.3
npm i -g gulp-cli
npm i -g @angular/cli
npm i -g vsts-npm-auth
npm i -g typescript

nvm install 10.22.0
nvm use 10.22.0
npm i -g gulp-cli
npm i -g @angular/cli
npm i -g vsts-npm-auth
npm i -g typescript
```

This will setup your node environment for development with both the new and old versions of Angular.

## Toggling node version:

- The ```nvm list``` command can be used to list installed node versions

- The ```nvm use <version>``` command can be used to quickly switch between node versions.

**Example:**
```
PS C:\git\sme\msft-sme-shell> nvm list

  * 12.18.3 (Currently using 64-bit executable)
    10.22.0
PS C:\git\sme\msft-sme-shell> nvm use 10.22.0
Now using node v10.22.0 (64-bit)
```

You can find a full index list of which Node | Angular | Typescript versions go together here: [Node - Angular compatibility index](https://gist.github.com/LayZeeDK/c822cc812f75bb07b7c55d07ba2719b3).

**Note**: All versions mentioned are for current upgrade cycle at time of writing (upgrading from Angular 7 to Angular 11).

## Restoring npm-authentication
Following the process above, you will lose all global node settings including your vsts authentication. 

To restore vsts authentication run this at the root of any repo:
``` vsts-npm-auth -config .npmrc ```

# Other development activities

- Sideloading shell is not be effected
- Sideloading extensions is not effected
- using copyTarget, be aware of which shell branch you are in. Only use this command in the 2.0 branch if the extension you are working with is also upgraded to angular 11.
- Upgrading repos. If the repo has been upgraded to angular 11, then use the latest 2.x.0 version of shell libraries. Otherwise continue to use the latest 1.x.0 version.

\*You can tell if a repo is upgraded by looking at the package.json file.
