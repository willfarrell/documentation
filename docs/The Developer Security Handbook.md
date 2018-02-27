# The Developer Security Handbook (WIP)
v2018.02.13 By @willfarrell

## Terminal
### ssh (Secure Shell)
Every device gets its own ssh keys.
- Generate keys https://gist.github.com/willfarrell/e9b7553367f5edca0ac7e0b8e9647a04

#### File Structure
```bash
.ssh
|-- config
|-- config.d
|   |-- git
|   |-- company-env		# multiple following this format
|-- id_ecdsa
|-- id_ecdsa.pub
|-- id_ec25519
|-- id_ec25519.pub
|-- id_rsa
|-- id_rdsa.pub
|-- known_hosts			# auto generated
```

#### ~/.ssh/config
```
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
  RSAAuthentication yes
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
- use for email and signing git commits
  - TODO finsh gist
  - https://blog.eleven-labs.com/en/openpgp-almost-perfect-key-pair-part-1/
  - https://alexcabal.com/creating-the-perfect-gpg-keypair/
  - https://spin.atomicobject.com/2013/11/24/secure-gpg-keys-guide/
  - Add `export GPG_TTY=$(tty)` to your bashrc file

### git (English slang for `a stupid person`) 
```bash
git config --global user.name "FULL NAME"
git config --global user.email "EMAIL@example.com"

gpg --list-secret-keys --keyid-format LONG
git config --global user.signingkey SEC_LONG_VALUE
```

## Secret Management (WIP)
- HashiCorp Vault
- docker secret

## Key Management (WIP)
### AWS Credentials

**Naming convention:** `${company}-${env}`

## Code

### Quality Assurance (QA)

#### GitHub Branch Enforcement 
Basically enable everything for `master` and `develop`.
- [x] Protect this branch
  - [x] Require pull request reviews before merging
     - [x] Dismiss stale pull request approvals when new commits are pushed
     - [x] Require review from Code Owners
  - [x] Require status checks to pass before merging
     - [x] Require branches to be up to date before merging
  - [x] Require signed commits
  - [x] Include administrators
  
#### git
##### Branching
- Standard: [@nvie/git-flow](https://github.com/nvie/gitflow) [@atlasian/git-flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- Install: `brew install git-flow`
 
![git-flow Diagram](https://wac-cdn.atlassian.com/dam/jcr:61ccc620-5249-4338-be66-94d563f2843c)
  
##### Commits
- Standard: [Conventional Commits](https://conventionalcommits.org/)
  - PreReq Reading: [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
- Enforce: [`commitlint`](http://marionebl.github.io/commitlint/#/)
 
```json
{
  "devDependencies": {
    "husky":"*",
    "@commitlint/cli":"*",
    "@commitlint/config-conventional":"*"
  },
  "scripts": {
    "commitmsg": "commitlint -e $GIT_PARAMS"
  },
  "commitlint": {
    "extends":["@commitlint/config-conventional"]
    "rules":{
      "scope-enum":[1, "always",[***ADD YOUR SCOPES HERE***,"security","packaging","changelog"]]
    }
  }
}
```

![Git Commit (https://xkcd.com/1296/)](https://imgs.xkcd.com/comics/git_commit.png)

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
##### formatting & linting 
- [`prettier`](https://prettier.io) - formatting 
- [`standard`](https://standardjs.com) - linting
- [`prettier-standard`](https://github.com/sheerun/prettier-standard) - bring them together

```json
{
  "devDependencies": {
  	"lint-staged":"*",
  	"husky":"*",
    "prettier":"*",
    "standard":"*",
    "prettier-standard":"*"
  },
  "scripts": {
    "precommit":"lint-staged"
  },
  "lint-staged": {
    "src/**/*.js": ["prettier-standard", "git add"],
    "src/**/*.json": ["prettier --write", "git add"]
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


#### Static Application Security Testing (SAST)
- [SonarQube](https://www.sonarqube.org)
- [kiuwan](https://www.kiuwan.com/code-security-sast/)

#### Dependencies
##### Tips
- lock to a certain version, update manually

##### No Unused Packages
- [`depcheck`](https://github.com/depcheck/depcheck)
- [`npm-check`](https://github.com/dylang/npm-check)

##### Version Up to Date
- [David DM](https://david-dm.org) - OpenSource SaaS
- [Greenkeeper](https://greenkeeper.io) - SaaS
- [`npm outdated`](https://docs.npmjs.com/cli/outdated)
- [`npm-check`](https://github.com/dylang/npm-check)
- [`version-check`](https://github.com/iamssen/version-check)

##### Security vulnerabilities
- [snyk](https://snyk.io) - SaaS
- [Node Security Platform](https://nodesecurity.io) - SaaS
- [GitHub](https://help.github.com/articles/about-security-alerts-for-vulnerable-dependencies)
- [`retire`](http://retirejs.github.io/retire.js/) - OpenSource

##### Licensing 
- [FOSSA](https://fossa.io) - SaaS 
- [`license-to-fail`](https://www.npmjs.com/package/license-to-fail) - CI error if whitelist not met
- [`npm-license-crawler`](https://github.com/mwittig/npm-license-crawler) - compile list of licenses that need to be displayed
- [License Zero](https://licensezero.com) - get / buy missing 

##### Missing
Sometimes dependencies go missing (ie. [left-pad](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm), [2018-02](https://status.npmjs.org/incidents/36j9bnllqtnj), [2018-01](https://status.npmjs.org/incidents/41zfb8qpvrdj)).
To prevent this, get a private repo that proxies and caches packages
- [`sinopia`](https://github.com/rlidwka/sinopia) - OpenSource
- [npm Enterprise](https://www.npmjs.com/enterprise) - SaaS
- [JFrog](https://www.jfrog.com/confluence/display/RTF/Npm+Registry) - SaaS
- Others SaaS offering: gemfurry, cloudsmith

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

### IDE (WIP)
#### IntelliJ IDEA
- [Prettier](https://prettier.io/docs/en/webstorm.html)
- [Standard](https://blog.jetbrains.com/webstorm/2017/04/using-javascript-standard-style/)
- [SonarLint](https://www.sonarlint.org/intellij/) (SonarQube)
  - Requires config w/ server

#### Sublime Text 3 (ST3)

## Continuous Integration / Continuous Deployment (WIP)

- Travis CI
- Circle CI
- Jenkins
  - Codeship

## APIGateway
- [OpenAPI](https://www.openapis.org) - definition (basically swagger under the hood)
- [jsonapi](http://jsonapi.org) - response format
- [apidocs](http://apidocjs.com) - documentation

## Infrastructure
- [Terraform](https://www.terraform.io)
- [`terragrunt`](https://github.com/gruntwork-io/terragrunt)


## Logging (WIP)
- OWASP A10:2017

### What
- request id
- session / user id
- timestamp
- level
- error code - where in the code
- stack trace


## Monitoring (WIP)

### Web Application Security
- [`sonarwhal`](https://sonarwhal.com) - best practices
- TLS - ssllabs.com by Qualys
- OWASP Scanning - Qualys w/ Selenium
- Secure Headers - securityheaders.io
- Tips:
  - API Gateway should have `/ping` and/or `/health` for testing
  - `?healthcheck` arg to flag for internal health checks

![Exploits of a Mom (https://xkcd.com/327/)](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)

### Availability
- endpoint 
  - up time
  - error rates
- function error rate

## References
- [OWASP 2010 Top 10](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/owasptop10/OWASP%20Top%2010%20-%202010.pdf)
- [OWASP 2013 Top 10](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/owasptop10/OWASP%20Top%2010%20-%202013.pdf)
- [OWASP 2017 Top 10](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)