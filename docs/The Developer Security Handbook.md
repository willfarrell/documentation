# The Developer Security Handbook (WIP)
v2018.03.14 By @willfarrell

## Security Policy as Code
- https://www.conjur.org/blog/2018/03/06/security-as-first-class-citizen.html

## Terminal
### ssh (Secure Shell)
Every device gets its own ssh keys.
- Generate keys https://gist.github.com/willfarrell/e9b7553367f5edca0ac7e0b8e9647a04

TODO: otfe of ssh keys on sleep

#### File Structure
```bash
.ssh
|-- config
|-- config.d
|   |-- git
|   |-- company-env		# multiple following this format
|   |-- radius			# for WiFi
|   |-- retropi			# Who doesn't have one
|-- id_ecdsa
|-- id_ecdsa.pub
|-- id_ecdsa256.pub		# from sekey
|-- id_ec25519
|-- id_ec25519.pub
|-- id_rsa
|-- id_rdsa.pub
|-- known_hosts			# auto generated
```

#### ~/.ssh/config
```
# TODO add in why for each
Host *
  ControlMaster auto
  ControlPath /tmp/ssh_%r@%h:%p
  ServerAliveInterval 120
  ServerAliveCountMax 120
  TCPKeepAlive no
  ForwardAgent yes
  PermitLocalCommand yes
  VisualHostKey yes
  GSSAPIAuthentication no
  KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
  PubkeyAuthentication yes
  HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ssh-ed25519,ssh-rsa,ecdsa-sha2-nistp256,ecdsa-sha2-nistp521
  Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
  MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
  UseRoaming no

Include config.d/*
```

#### ~/.ssh/config.d/git
```
### Bitbucket ###
# Only RSA is allowed

Host bitbucket.org
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/id_rsa

### GitLab ###

Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_rsa

### GitHub ###

Host github.com
  HostName github.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25512
```

#### Sample Bastion Host Proxy
```
### Company Name (cn) ###

# ssh -N cn-proxy-test
Host cn-proxy-test
  HostName #.#.#.#
  IdentityFile ~/.ssh/id_ed25512
  User USERNAME
  ControlPath /tmp/ssh_sn-proxy-demo
  LocalForward 3307 mysql-test.*****.us-east-1.rds.amazonaws.com:3306
  LocalForward 6378 redis-test.*****.0001.use1.cache.amazonaws.com:6379

Host cn-bastion-test
  HostName #.#.#.#
  IdentityFile ~/.ssh/id_ed25512
  User USERNAME
  ControlPath /tmp/ssh_sn-bastion-test

Host cn-test-*
  ProxyCommand ssh -W %h:%p sn-bastion-demo
  IdentityFile ~/.ssh/id_ed25512
  User USERNAME
  
# Add hosts behind bastion here
Host cn-test-web-1
  HostName 10.0.0.1
```

### gpg (GNU Privacy Guard) (WIP)
Use for encrypting emails / files and signing git commits

1. Open `GPG Keychain`
1. Generate new master key and encryption subkey, then press `???`
   - Name: Your Name
   - Email: email@example.com
   - Password: ************
   - Confirm: ************
   - Key type: RSA and RSA (default)
   - Length: 4096
   - Key expires: true
   - Expiration date: ~1 year
1. Select the key you just created and click `Details`
1. Toggle to `Subkeys` and press `+`
1. Generate a signing subkey, then press `Generate Key`
   - Key type: RSA (sign only)
   - Length: 4096
   - Key expires: true
   - Expiration date: ~1 year
1. Generate a [authentication subkey](https://github.com/drduh/YubiKey-Guide#authentication-key) in Terminal `gpg --expert --edit-key $KEYID`
1. Click `Export`, Save public key
1. Click `Export`, Save secret key
1. Backup your private key
1. Upload your public key
   - [GitHub](https://help.github.com/articles/adding-a-new-gpg-key-to-your-github-account/)
   - [MIT](https://pgp.mit.edu)
   - [PGP Global Directory](https://keyserver.pgp.com)

TODO: otfe of private keys on sleep

### YubiKey (WIP)
Have a YubiKey?

TODO:
- yubikey for ssh - [YubiKey-Guide](https://github.com/drduh/YubiKey-Guide)
- yubikey for git
- yubikey for otfe
- yubikey for email?

### git (English slang for `a stupid person`)
```bash
#~/.gitconfig

[alias]
  co = checkout
  ci = commit
  st = status
  br = branch
  hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short

```

```bash
git config --global user.name "FULL NAME"
git config --global user.email "EMAIL@example.com"

# GPG - https://help.github.com/articles/telling-git-about-your-gpg-key/
gpg --list-secret-keys --keyid-format LONG
# Copy >>VALUE<< from `sec   rsa4096/>>912C0E0667AB2369<< 2018-02-28 ...`
git config --global commit.gpgsign true
git config --global user.signingkey ${VALUE}

git config --global --list
```

#### CI Variables
```bash
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [ "${GIT_BRANCH}" == "HEAD" ]; then export GIT_BRANCH=${CI_BRANCH}; fi
export GIT_COMMIT_ID=$(git rev-parse HEAD)
export GIT_LATEST_TAG=$(git describe --abbrev=0 --tags)
export GIT_COMMIT_TAG=$(git name-rev --tags --name-only $(git rev-parse HEAD))

gitflowEnv () {
  if [ "${GIT_BRANCH}" == "master" ] && [ "${GIT_LATEST_TAG}" == "${GIT_COMMIT_TAG}" ]; then
      echo "production"
  elif [[ "${GIT_BRANCH}" == "release/*" ]] && [ "${GIT_LATEST_TAG}" == "${GIT_COMMIT_TAG}" ]; then
    echo "staging"
  elif [[ "${GIT_BRANCH}" == "release/*" ]]; then
    echo "testing"
  elif [ "${GIT_BRANCH}" == "develop" ]; then
    echo "development"
  else
    echo "... skipping"
    echo "Commit: ${GIT_COMMIT_ID}; Branch: ${GIT_BRANCH}; Tag: ${GIT_LATEST_TAG} vs. ${GIT_COMMIT_TAG}"
    exit 1
  fi
}
echo "Deploying to $(gitflowEnv)"
ENVIRONMENT=$(gitflowEnv)
```

## Infrastructure (AWS)
EU General Data Protection Regulation (GDPR) - https://segment.com/blog/segment-and-the-gdpr/
- https://segment.com/blog/secure-access-to-100-aws-accounts/

### Credentials
Don't use `[default]`, this forces the use of the `--profile` arg so you're consciously choosing the account you're interfacing with.

- [Gruntworks aws-auth](https://github.com/gruntwork-io/module-security-public/tree/master/modules/aws-auth)

### Structure
```
root 				# IAM Users, Billing
|-- operations 		# CloudTrial, GuardDuty, TerraForm State, Vault
|-- forensics		# Empty, to be used when forensics need to be done
|-- production		# Production [master tag]
|-- staging			# External Testing [{hotfix,release}/* tag]
|-- testing			# Internal Testing [{hotfix,release}/* commit]
|-- development		# Developer Sandbox [develop commit]
```

## Secret Management (WIP)
- HashiCorp Vault
- docker secret

## Key Management (WIP)


**Naming convention:** `${company}-${env}`

## Code

### Documentation
#### README 
- https://gist.github.com/PurpleBooth/109311bb0361f32d87a2

TODO add in .github templates/samples

### Standards
[![Standards - Fortunately, the charging one has been solved now that we've all standardized on mini-USB. Or is it micro-USB? Shit.](https://imgs.xkcd.com/comics/standards.png)](https://xkcd.com/927/)

- [ISO 8601]() Timestamp
- [JSON Schema]() Form and API sanitization and validation

### Approved npm (WIP)
ajv, moment, lodash

### Quality Assurance (QA)

#### GitHub Branch Enforcement 
Basically enable everything for `master` and `develop`.
- [x] Protect this branch
  - [x] Require pull request reviews before merging
     - [x] Dismiss stale pull request approvals when new commits are pushed
     - [x] Require review from Code Owners
     - [ ] Restrict who can dismiss pull request reviews
  - [x] Require status checks to pass before merging
     - [x] Require branches to be up to date before merging
  - [ ] Restrict who can push to this branch
  - [x] Require signed commits
  - [x] Include administrators
  
#### git
[![Git - If that doesn't fix it, git.txt contains the phone number of a friend of mine who understands git. Just wait through a few minutes of 'It's really pretty simple, just think of branches as...' and eventually you'll learn the commands that will fix everything.](https://imgs.xkcd.com/comics/git.png)](https://xkcd.com/1597/)
##### Branching
- Standard: [@nvie/git-flow](https://github.com/nvie/gitflow) [@atlasian/git-flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- Install: `brew install git-flow-avh`
- Commands: [Cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/)
 
![git-flow Diagram](https://wac-cdn.atlassian.com/dam/jcr:61ccc620-5249-4338-be66-94d563f2843c)

##### Commits
- Standard: [Conventional Commits](https://conventionalcommits.org/)
  - PreReq Reading: [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

```
{type:build,chore,ci,docs,feat,fix,perf,refactor,revert,style,test}[({optional scope:security,packaging,changelog,__custom__})]: description

[optional body]

[optional footer]
```

- TODO add details for body - links to blogs/code researched

- Enforce: [`commitlint`](http://marionebl.github.io/commitlint/#/)
- Enforce Scopes: [`lerna-scopes`](https://github.com/marionebl/commitlint/tree/master/%40commitlint/config-lerna-scopes) from [`lerna`](https://lernajs.io)

```
$ npm i -D husky @commitlint/{cli,config-conventional,config-lerna-scopes}
```
```json
{
  "devDependencies": {
    "husky":"*",
    "@commitlint/cli":"*",
    "@commitlint/config-conventional":"*",
    "@commitlint/config-lerna-scopes":"*"
  },
  "scripts": {
    "commitmsg": "commitlint -e $GIT_PARAMS"    
  },
  "commitlint": {
    "extends":["@commitlint/config-conventional","@commitlint/config-lerna-scopes"]
  }
}
```

[![Git Commit - Merge branch 'asdfasjkfdlas/alkdjf' into sdkjfls-final](https://imgs.xkcd.com/comics/git_commit.png)](https://xkcd.com/1296/)

##### Versioning:
- Standard: [Semantic Versioning](https://semver.org) 

Updates       | [`npm version`](https://docs.npmjs.com/cli/version) | [`semantic-release`](https://www.gitbook.com/book/semantic-release/semantic-release/details) | [`standard-version`](https://github.com/conventional-changelog/standard-version)*
--------------|---|---|---
**Version**   | Y | Y | Y
**CHANGELOG** | N | Y | Y
**npm**       | Y | Y | N

* Preferred option, `standard-version`, allows review of CHANGELOG before publishing.

```json
{
  "devDependencies": {
    "standard-version":"*"
  },
  "scripts": {
    "release": "standard-version"
  }
}
```

#### Code Analysis
TODO research `npx standard-damn-it` - setup repo

##### formatting & linting 
- [`prettier`](https://prettier.io) - formatting 
- [`standard`](https://standardjs.com) - linting
- [`prettier-standard`](https://github.com/sheerun/prettier-standard) - bring them together
- [`terraform fmt`](https://www.terraform.io/docs/commands/fmt.html) - formatting for terraform files

```
$ npm i -D husky lint-staged prettier standard
```
```json
{
  "devDependencies": {
  	"lint-staged":"*",
  	"husky":"*",
    "prettier":"*",
    "standard":"*"
  },
  "scripts": {
    "precommit":"lint-staged",
    "lint": "standard src/**/*.{js,json}",
  },
  "lint-staged": {
    "src/**/*.{js,json}": [ 
      "prettier --write",
      "standard --fix", 
      "git add" 
    ]
  }
}
```

##### Catch large jumps in code size 
- [`size-limit`](https://www.npmjs.com/package/size-limit)

```json
{
  "devDependencies": {
  	"size-limit":"*"
  },
  "scripts": {
    "size":"size-limit"
  },
  "size-limit": [
    { "path":"dist/**/*.js", "limit":"100 KB" }
  ]
}
```
##### Secrets & Tokens
Ensure Secrets and Token are not committed
- [gitleaks](https://github.com/zricethezav/gitleaks)
- [How to remove them safely](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)

#### Static Application Security Testing (SAST)
- [SonarQube](https://www.sonarqube.org)
- [kiuwan](https://www.kiuwan.com/code-security-sast/)
- [`eslint-plugin-security`](https://github.com/nodesecurity/eslint-plugin-security)

#### Dependencies
- [bithound](https://www.bithound.io)

##### Tips
- pinned to a certain version, update manually

##### No Unused Packages
- [`depcheck`](https://github.com/depcheck/depcheck)
- [`npm-check`](https://github.com/dylang/npm-check)

##### Version Up to Date
- [David DM](https://david-dm.org) - OpenSource SaaS
  - [`david`](https://github.com/alanshaw/david) - OpenSource
- [Greenkeeper](https://greenkeeper.io) - SaaS
- [Gemnasium](https://gemnasium.com) - SaaS
- [`npm outdated`](https://docs.npmjs.com/cli/outdated)
- [`npm-check`](https://github.com/dylang/npm-check)
- [`version-check`](https://github.com/iamssen/version-check)

##### Security vulnerabilities
- [snyk](https://snyk.io) - SaaS
- [Node Security Platform](https://nodesecurity.io) - SaaS
  - [`nsp`](https://github.com/nodesecurity/nsp) - OpenSource
- [David DM](https://david-dm.org)
- [GitHub](https://help.github.com/articles/about-security-alerts-for-vulnerable-dependencies)
- [`retire`](http://retirejs.github.io/retire.js/) - OpenSource

##### Licensing (WIP)
- [FOSSA](https://fossa.io) - SaaS 
- [`licenser-checker`](https://www.npmjs.com/package/license-checker) - Scans repository
- [`npm-license`](https://www.npmjs.com/package/npm-license) - iterative scan of repo
- [`license-to-fail`](https://www.npmjs.com/package/license-to-fail) - CI error if whitelist not met
- [`npm-license-crawler`](https://github.com/mwittig/npm-license-crawler) - compile list of licenses that need to be displayed
- [License Zero](https://licensezero.com) - get / buy missing 
- [Creative Commons](https://creativecommons.org/publicdomain/zero/1.0/) - No License
- [API Commons](http://apicommons.org/)

##### Malware (WIP)
- [Sophos](https://www.sophos.com/en-us/security-news-trends/security-trends/malicious-javascript.aspx)
- [Hash Registry](https://www.npmjs.com/package/malware-hash-registry)

##### Missing
Sometimes dependencies go missing (ie. [left-pad](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm), [2018-02](https://status.npmjs.org/incidents/36j9bnllqtnj), [2018-01](https://status.npmjs.org/incidents/41zfb8qpvrdj)).
To prevent this, get a private repo that proxies and caches packages
- [`sinopia`](https://github.com/rlidwka/sinopia) - OpenSource
- [npm Enterprise](https://www.npmjs.com/enterprise) - SaaS
- [JFrog](https://www.jfrog.com/confluence/display/RTF/Npm+Registry) - SaaS
- Others SaaS offering: gemfurry, cloudsmith

- [`local-npm`](https://addyosmani.com/blog/using-npm-offline/)

##### Reputation
Use popular well known deps
- downloads /day, /month, year
- github stats
- dependents
- badges: for topics above

### Testing
- Angular - protractor
- ReactJS - jest & enzyme with polyfills
- javascript - mocha & chai
- End-to-End (e2e) - [`nightwatch`](http://nightwatchjs.org) / `TestCafe`
  - local
  - BrowserStack
  - ci
  - webappscan

#### Tips
- If adding a unique identifier to a DOM element use `data-test="name"`.
- Testing: https://en.wikipedia.org/wiki/Software_testing

#### Coverage
- [`istanbul`](https://github.com/gotwarlost/istanbul)
- [Coveralls](https://coveralls.io) - SaaS
- [SonarSource](https://www.sonarsource.com) - SaaS
- [Code Climate](https://codeclimate.com) - SaaS

### Anti-Virus (WIP)
- [ClamAV](https://www.clamav.net) - [clamav.js](https://www.npmjs.com/package/clamav.js), [s3-antivirus](https://www.npmjs.com/package/s3-antivirus), [clamscan](https://www.npmjs.com/package/clamscan)

### IDE (WIP)
#### IntelliJ IDEA
- [Prettier](https://prettier.io/docs/en/webstorm.html)
- [Standard](https://blog.jetbrains.com/webstorm/2017/04/using-javascript-standard-style/)
- [SonarLint](https://www.sonarlint.org/intellij/) (SonarQube)
  - Requires config w/ server

#### Sublime Text 3 (ST3)
- [prettier-standard](https://github.com/sheerun/prettier-standard#sublime-text-3)
- [Prettier](https://packagecontrol.io/packages/JsPrettier)
- [SonarLint]() - needs link

#### Atom
- [Prettier](https://github.com/prettier/prettier-atom)
- [Standard]() - needs link
- [SonarLint]() - needs link

## Continuous Integration / Continuous Deployment (WIP)

- Travis CI
- Circle CI
- Jenkins
  - Codeship

## Code Docs
- [README Template](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2)
- [documentation.js](http://documentation.js.org)
- [jsdoc](http://usejsdoc.org)

## API Gateway
- [OpenAPI](https://www.openapis.org) - definition (basically swagger under the hood)
- [jsonapi](http://jsonapi.org) - response format
- [apidocs](http://apidocjs.com) - documentation

### Middleware
- query string parse (req)
- body parse (req)
- cors (res)
- language header (req/res)
- formatting (res)
- authorization (req)
- cache header (res)
- rate limiting (req)
- sanitization & validation (req/res)
  - security scan (xss, sql injection)
- csrf

### Schema
- ajv - json-schema
- json-schema-to-graphql-types

### Graphql
- https://github.com/aws-samples/aws-mobile-appsync-events-starter-react

## Infrastructure (WIP)
- [Terraform](https://www.terraform.io)
- [`terragrunt`](https://github.com/gruntwork-io/terragrunt)

### AWS
- sub accounts

## Logging (WIP)
- OWASP A10:2017

### What
- timestamp (taken care of by aws)
- request id (unique identifier for the event, from aws)
- version (the version of the code)
- severity (info, warn, error)
- message (human readable message)
- meta (optional json object of data that is needed

ex `${request_id} v${version} [${severity}] "${message}" ${meta?}`

## Monitoring (WIP)

### Web Application Security
- https://blog.risingstack.com/node-js-security-checklist/
#### DNS
- http://dnscheck.pingdom.com / http://dnscheck.iis.se

- [ ] DNSSEC
- [ ] DANE - https://github.com/vdukhovni/danecheck
- [ ] CAA
- [ ] SPF (if MX)

#### TLS
- hstspreload.org
- ssllabs.com by Qualys (API) [node](https://github.com/keithws/node-ssllabs)
- [ ] nopn-revoked TLS certificates

#### Headers
- securityheaders.io [php]
- https://observatory.mozilla.org (API) [python](https://github.com/mozilla/http-observatory)

#### Cookies
- https://observatory.mozilla.org

- Flags:
  - [ ] secure
  - [ ] httpOnly
- Scope
  - [ ] domain
  - [ ] path
  - [ ] expires
- [ ] csrf

#### Spiders
- ZAProxy Basic
- ZAProxy API via OpenAPI.json
- Qualys w/ Selenium (https://www.qualys.com/documentation/)
- [`sonarwhal`](https://sonarwhal.com) - best practices
- [tinfoilsecurity](https://www.tinfoilsecurity.com)
- https://jdow.io/blog/2018/03/18/web-application-penetration-testing-methodology/

#### Other
- Malware

#### Tips
  - API Gateway should have `/ping` and/or `/health` for testing
  - `?healthcheck` arg to flag for internal health checks

[![Exploits of a Mom - Her daughter is named Help I'm trapped in a driver's license factory.](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)](https://xkcd.com/327/)

### Availability
- endpoint 
  - up time
  - error rates
- function error rate

## References
- [OWASP 2010 Top 10](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/owasptop10/OWASP%20Top%2010%20-%202010.pdf)
- [OWASP 2013 Top 10](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/owasptop10/OWASP%20Top%2010%20-%202013.pdf)
- [OWASP 2017 Top 10](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)