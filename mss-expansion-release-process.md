# MSS Expansion Release Process
Think of this process as a conveyor belt that is always moving forward.
Sometimes it will stop briefly, but it never moves backwards.

NOTE: When we refer to "deploying to PRODUCITON" we mean we are deploying to the
three production level environments, ETRAIN (aka ESAND), TRAIN, and PROD.

```
approved PRs -> QA artifact -> REF -> PRODUCTION artifact -> STAGE -> PRODUCTION
                                |                              |
                         swarm on issues                 sanity check
```

- Pull Requests are approved, but are NOT merged on approval.

- When QA requests a new build, the approved pull requests are merged in.

  - If there are _more_ approved pull requests than QA's capacity, they are
    merged based on order of priority.

  - If there are _less_ approved pull requests than QA's capacity, they are
    merged from the bottom up.

  - As each PR is merged in, it should change staus in Jira to TESTING/QA.

  - Definition of an approved pull request:

    - There must be at least 2 approvals and one must be from a Tech Lead or
      Architect. Right now, that means AD Slaton, James Drenter, Matt
      Crutchfield or James Young.

    - Any linked pull request must also be approved.

- After the last pull request merged in has built successfully on Codeship, the
  QA artifact will be deployed to the REF environment for QA to test.  An email
  will be sent detailing the version of package, where it was deployed, and what
  tickets and pull requests are in the build.  The email should look **exactly**
  like this:

```
To: Thomas.White@turner.com, LaToya.Brown@turner.com, JayeshK@hcl.com,
    Sharon.Wright@turner.com, Timothy.D.Smith@turner.com,
    Karen.Price@turner.com, Jason.Shemper@turner.com, Gavin.George@turner.com,

Cc: James.Drenter@turner.com, AD.Slaton@turner.com, James.Young@turner.com
    Tony.Thoensen@turner.com, Ian.Moraes@turner.com,
    Melissa.Koehler@turner.com, Matthew.Drooker@turner.com

Subject: Deployed to REF, cnn-michonne-app-1.11.0-2387018.qa, pal-server-1.9.0-2387032.qa

Deployed to REF

cnn-michonne-app-1.11.0-2387018.qa

- AVNGRS-1303 (pull request #1823)
- AVNGRS-1300 (pull request #1822)
- AVNGRS-1260 (pull request #1820)
- AVNGRS-1272 (pull request #1821)
- AVNGRS-1268 (pull request #1803)


pal-server-1.9.0-2387032.qa

- AVNGRS-1306 (pull request #953)
```

- QA will test the QA artifact on REF and compile a list of issues.  As each
  ticket passes testing, it should change state in Jira to PASSED QA.

- The list will be reviewed by QA and PM teams and a decision will be made on
  what, if anything, is REQUIRED to be fixed.

  - IF there is an issue that is not REQURIED to be fixed for this immediate
    release, then a new ticket will be created to fix the specific problem.

- When the build is approved, a release will be created.

  - The Release Manager will branch `develop` into `release/[version-number]`,
    bump the version, then merge into `master`.  This will create the PRODUCTION
    artifact.

- When the PRODUCTION artifact has built successfully on Codeship, it will be
  deployed to the STAGE environment.  An email will be sent, just like the one
  above with updated information in it.

- QA will do a sanity check on STAGE.  This is the EXACT SAME BUILD that was
  just tested on REF.  The difference here is now there is real production data.
  This should not take more than 15 minutes.

- When the build is approved, the PRODUCTION artifact will be deployed to
  PRODUCTION.  This means it will be deployed to ETRAIN (ESAND), TRAIN, and
  PROD.

- Each ticket that was deployed will be moved to CLOSED/OTHER with a comment of
  "RELEASED".

- The cycle repeats.  By this time there should be more pull requests approved.


## Commands to run in Terminal

```shell
# Go to repo
$ cd cnn-michonne-app

# Make sure you are on the `develop` branch
$ git checkout develop

# Make sure you have pruned your local repository
$ git fetch -p

# Make sure you have the most current `develop` branch
$ git pull origin develop

# Switch to the `master` branch
$ git checkout master

# Make sure you have the most current `master` branch
$ git pull origin master

# Switch to the `develop` branch
$ git checkout develop

# See what version the code is currently at
$ grep version package.json

# Create a `release` branch for the next version
$ git checkout -b release/1.10.0

# Bump the version on the `release` branch
$ vim package.json

# Check in the version bump
$ git commit -am '1.10.0'

# Diff `release` to `develop`, the only change should be the `package.json` file
$ git diff develop

# Diff `release` to `master`, there should be lots of changes
$ git diff master

# Switch to the `master` branch
$ git checkout master

# Merge `release` into `master`
$ git merge --no-ff release/1.10.0

# Push `master` upstream, this will create a PRODUCTION artifact
$ git push origin master

# Tag the release with the version
$ git tag 1.10.0

# Push the tag upstream
$ git push origin 1.10.0

# Switch to the `develop` branch
$ git checkout develop

# Merge `master` into `develop`, locally
$ git merge --no-ff --no-commit master

# Check that ONLY the `package.json` is changing
$ git status

# Commit the merge changes
$ git commit -v

# Push `develop` upstream, this will create a new QA artifact.
$ git push origin develop

# Go do the same thing for pal-server
```
