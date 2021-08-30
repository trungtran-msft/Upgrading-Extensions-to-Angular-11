#Overview

Upgrading each extension repo is the next step after we've verified that shell is in a ready state. The process is very similar and this guide will outline the general steps.

# Preliminary steps

0. Create a `features/ng11` branch in the repo.
1. Update `version.json` version to `(N+1).0.0`. At time of writing we'll be updating from 1.x.x to 2.0.0.
2. In a Powershell console, make sure to switch to the respective Node version ([Working with two branches of shell](https://microsoft.visualstudio.com/SME/_wiki/wikis/SME.wiki/60269/Working-with-two-branches-of-shell?anchor=switching-development-environments) - At time of writing `nvm use 12.18.3`). Close the terminal for this change to take effect.
3. Cleanup `node_modules` folder to avoid npm conflicts.

# Automated process
## Prerequisites

4. Run `npm config set @msft-sme:registry https://microsoft.pkgs.visualstudio.com/_packaging/SME/npm/registry` to add the @msft-sme scope to your global npm registry
5. Download and install WAC CLI tools by running `npm install -g @msft-sme/tools` if you have not already done so (note: must be at least version 2.54.0 or higher)

## Upgrade

6. At the root level of the repo, run `wac upgrade --audit=false` (Note: If upgrading an extension that uses the SDK, perform step 6.b.ii. first).
a. If working on an extension repository that is consumed by other extensions (e.g. `msft-sme-file-explorer`), 
include the `--library` flag as well.
____i. If the library flag was used, edit the `name` property in `src/package.json` to something unique to the extension (e.g. `@msft-sme/event-viewer-lib` for `msft-sme-event-viewer`).
b. If working on an extension repository that uses the SDK, specify the `--experimental` and `--internalOverride` flags. These additional manual steps will also be necessary: 
____i. Do a find and replace all on `@msft-sme` to `@microsoft/windows-admin-center-sdk`.
____ii. Ensure the .npmrc file contains only this content: 
`@dna:registry=https://microsoft.pkgs.visualstudio.com/_packaging/SME/npm/registry`
`always-auth=true`
____iii. Navigate to `node_modules/@microsoft/windows-admin-center-sdk/angular/angular.d.ts` and change the reference paths at the top of the file to use `../core` instead of `../../../core`. (Note: This is a temporary work around that will only work when building locally, we're currently working on a long term fix for this issue).

7. There will likely be unresolved errors returned, these will either need to be resolved here or during the [Build steps](https://microsoft.visualstudio.com/SME/_wiki/wikis/SME.wiki/69810/Upgrading-extensions?anchor=build-steps) section below. Once ready, proceed to Build steps.
8. [Conditional] If the extension repo has dependencies on any other `@msft-sme` package besides shell you will have to manually pick the new 2.x.x version for that one (e.g. `msft-sme-certificate-manager` has a dependency on `msft-sme-event-viewer`. The automated tools will **not** update `msft-sme-event-viewer` version, it has to be manually updated.)
Also be sure to specify the '/dist' folder level on any imports from extensions, any lower or higher level imports won't work (e.g. `import { foobar } from '@msft-sme/event-viewer'` would need to be changed to `import { foobar } from '@msft-sme/event-viewer/dist'`.) 
9. Open `app-routing.module.ts` and change any appRoutes that have the format `./folder-name/file-name#ModuleClass` to `() => import('./folder-name/file-name').then(m => m.ModuleClass)`. If there are any other `routing.module.ts` files they will also need to be updated in this way.
10. Open `main.ts` and check if it previously contained `websocket: true` under `CoreEnvironment.initialize`. If so, re-add it in the same location.
11. Remove `UpgradeAudit.txt` file. It's auto-generated for your reference but doesn't need to go in the repo.

# Manual process
These steps are only needed if automation is not used, if using the WAC CLI to perform the upgrade you can skip this section.

## Libraries upgrade
4. Figure out the correct versions for the libraries:
    a. Open Powershell
    b. Create a new Angular project using Angular CLI: `ng new test-project`, it will prompt you to answer how you want to configure the project - select yes to strict type checking and routing, SASS for the styles format. Wait for it to get generated.
    c. Open the `package.json` that was created in the `test-project`
    d. Go through the list and compare with `package.json` for the extension
    e. For each package that's in both - update the extension's version to the one from the `test-project`
    f. Add each package that's in the `test-project` but not in the extension's `package.json`
    g. For each package that's in the extension's `package.json` but not in the `test-project` - take it on a case to case basis when deciding which version to update it to.
**Note**: Ignore updating `@msft-sme` dependencies until we have a published shell `v2.0`.

5. Add `.browserslistrc` file to the root of the project: [Process viewer libraries upgrade commit](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2F.browserslistrc)
6. Update the `tsconfig.json` structure:
    a. Create a `tsconfig.base.json` for Angular compilation which all other `tsconfigs` will extend from. [tsconfig.base.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Ftsconfig.base.json)
    b. Update the root `tsconfig.json` for compilation of `gulpfile.ts` **only**. : [tsconfig.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Ftsconfig.json)
Angular and gulp have different module systems now and gulp compilation will be looking for `tsconfig.json`, this is not configurable
    c. Update `tsconfig.app.json` and `tsconfig.spec.json` to extend `tsconfig.base.json`. [tsconfig.app.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fsrc%2Ftsconfig.app.json)

7. Update Angular builder `angular.json` from `"@angular-builders/custom-webpack:browser"` to `"@angular-devkit/build-angular:browser"`: [angular.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fangular.json)

8. Review and update `tslint.json` if needed: [tslint.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/cee753f90a7bd1400c5a89235b0661c18975ce75?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Ftslint.json)

9. `npm install` to install the newest packages. `package-lock.json` will get updated when this is complete.
10. Stage and commit all these changes, including the `package-lock.json` file generated from the previous step.

## Update build system
11. Change how things are imported
12. Update how modules get exported
13. Summary of changes: [gulpfile.ts changes](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/f7e3527b7819340b00c2b29fc27bd1c9b3436635?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fgulpfile.ts)
14. Modify inline compile flow for compatibility: [compile.ts, copy.ts, and index.ts changes](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/09e29082daa3d60e96b1460e43da1b35071a80c4?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fgulpfile.ts%2Fcommon%2Fcompile.ts)
15. Upgrade from karma-coverage-instanbul-reporter to karma-coverage:
i. Remove karma-coverage-instanbul-reporter package: [package.json](https://microsoft.visualstudio.com/SME/_git/msft-sme-file-explorer/commit/5821479dc2d9aff0cc1452fd02e6249f052ab595?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fpackage.json)
ii. Modify karma.conf.js to use karma-coverage: [karma.conf.js](https://microsoft.visualstudio.com/SME/_git/msft-sme-file-explorer/commit/5821479dc2d9aff0cc1452fd02e6249f052ab595?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fsrc%2Fkarma.conf.js)
iii. Modify .gitignore to exclude "src/coverage" folder generated on build: [.gitignore](https://microsoft.visualstudio.com/SME/_git/msft-sme-file-explorer/commit/5539e3c8db031f45f15f3af39a803272f5f13845?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2F.gitignore)

## Clean up polyfills
16. Remove polyfills that are no longer needed: [polyfills.ts](https://microsoft.visualstudio.com/SME/_git/msft-sme-process-viewer/commit/09e29082daa3d60e96b1460e43da1b35071a80c4?refName=refs%2Fheads%2Ffeatures%2Fng11&path=%2Fsrc%2Fpolyfills.ts)

# Remove library build
0. If working on an extension repo that is not consumed by another extension, the library build is not necessary. Later versions of the automation tools will not include the library build configuration by default, but for any repository that has already been upgraded the configuration must be removed manually if not needed.
1. Install the most recent version of the `@msft-sme/tools` package globally and re-run `wac upgrade --audit=false` in order to include the most recent gulp files.
2. Under the `src` folder, remove the `tsconfig.lib.json`, `tsconfig.lib.prod.json`, `ng-package.json`, `package.json`, and `public_api.ts` files.
3. Under the `src\app` folder, remove `index.ts` if it exsits (there may be other index.ts files throughout the repository, you can remove these if you wish as they are also no longer necessary, but they won't cause any issues).
4. In `angular.json` at the root of the repository, remove the `module-lib` project configuration.

# Build steps
1. At this point the extension repo is ready to be built. Run `gulp build`
2. Watch out for any linting and compilation errors.
3. Fix and repeat from 1.
4. When all build errors are fixed - commit your changes.

## Difficult to diagnose errors
- **NG6002: Appears in the NgModule.imports of AppModule, but could not be resolved to an NgModule class**
This type of error occurs at build time, typically before the upgraded repository has been successfully built at least once. To resolve, run "ng serve --prod", after which these errors should no longer appear when building.

- ![image.png](/.attachments/image-caed7de1-29f0-47fe-ba03-326fef78a31d.png)
This error occurs during the inlineCompile step of "gulp build" and occurs as the result of a mismatch in versions between the @types/jasmine package downloaded and what the @types/jasminewd2 package requires. This can be resolved by removing the @types/jasminewd2 package (if possible, otherwise a different solution should be found and documented).


# Run steps
1. Sideload the extensions with `gulp serve --port <port> --prod --aot`
2. In the browser, look for any runtime issues with the extension that you can find:
    a. Extension page(s) not loading
    b. Elements missing from the extension page(s)
    c. Console errors
    d. Anything else that looks off or behaves strange
3. Fix any runtime issues that you have discovered.
4. When the extension has been stabilized, commit your changes.

# Creating main branch
0. Ensure that you are ready to complete the upgrade process and everything is working as expected in the feature branch
1. Create a new branch named "main" in the repository
2. Create a PR from the features/ng11 branch into main
3. Navigate to [Repository settings](https://microsoft.visualstudio.com/SME/_settings/repositories) in Azure DevOps and select the relevant repository
4. At the bottom of the page that appears add main to searchable branches
5. At the top of the page, select the "Policies" tab, scroll down to "Branch Policies" and select main
6. Turn on the "Require minimum number of reviewers" policy and set the number to 2
7. Turn on the "Check for linked work items" policy and set it to "Required"
8. Turn on the "Check for comment resolution" policy and set it to "Required"
9. Turn on the "Limit merge types" policy and select "Rebase and fast-forward" and "Squash merge"
10. Stop here until the PR is completed. Once completed, first complete the steps in the "Configuring Build Pipelines" section before proceeding.
11. Next to the Build Validation section click the plus button on the right
12. For Build pipeline, search for the relevant pull request pipeline created from the .yml file
13. Ensure "Automatic" is selected for Trigger
14. Ensure "Required" is selected for Policy requirement
15. Ensure "Immediately when main is updated" for Build expiration
16. Click "Save" at the bottom
17. Next to the Automatically include reviewers section click the plus button on the right
18. For Reviewers, search "WAC Team" and select the WAC Team that comes up with "[SME]" underneath it
19. Ensure "Required" is selected for Policy requirement and set it to 2
19. Unselect "Allow requestors to approve their own changes"
![image.png](/.attachments/image-b4c6a6f3-cd71-4f21-bd82-4824ea1f5aca.png)
20. Click "Save" at the bottom
21. Navigate back to pipelines, select the CI pipeline you created from the .yml file and click "Run pipeline" in the top right. Make sure it completes successfully or troubleshoot any issues.
22. Go to gateway repo. Create a user branch and add a reference to main branch in [New-Report.ps1](https://microsoft.visualstudio.com/SME/_git/gateway/commit/f0b5a18f34ef5a9f63015a7afd8a70d356a33477?refName=refs%2Fheads%2Fmaster). Start a PR for this change.
23. Go to the extension repo > Branches, select the three dots on `main` and set it as compare branch and default branch.
![image.png](/.attachments/image-855b5461-f337-4065-91a9-770167931ae8.png)
24. Congratulations, you just successfully upgraded an extension!



# Configuring Build Pipelines
1. Navigate to [Pipelines](https://microsoft.visualstudio.com/SME/_build?view=folders&treeState=XGFkbWluLWNlbnRlciRcYWRtaW4tY2VudGVyXHRvb2xz) in Azure DevOps
2. Click the 3 dots to the right of the folder for the relevant extension under the path "admin-center/tools/"
3. Create a new folder and call it "master-old". Move all current pipelines to this folder.
![image.png](/.attachments/image-94c372e4-8a49-4301-94b0-fbf88936b147.png)
3. Select "New pipeline"
4. In the first step of the menu that opens, select "Azure Repos Git"
5. Select the relevant repository from the list that appears
6. On the next page, select Existing Azure Pipelines YAML file
7. Select main for Branch and then any of the .yml files shown for Path (you will repeat these steps for the others later)
8. Press continue and on the next page click the dropdown next to "Run" (don't click run) and then "Save"
9. On the next page, click the 3 dots at the top right and select "Rename/move"
10. Rename the pipeline to something more descriptive (e.g. "CI - process-viewer", by default DevOps will have used the repository name)
11. Repeat these steps for the remaining .yml files in the repository
