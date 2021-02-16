# How to create a TeacherSeat Admin Interface

## 0. Naming


An interface / sub-interface needs to follow the naming schema to avoid conflicts with other future interfaces: 

```
<provider>_ui_<namespace>_<system>_<subsystem>
```

Each backend will have a corresponding frontend. The backend and frontend names needs to match.

A frontend interface with the name:
```
<provider>_ui_<namespace>_<system>_<subsystem>
```

Would correspond to a system with the name:
```
<provider>_<namespace>_<system>_<subsystem>
```

and vice-versa. Note that the only difference is the word ```ui``` in the frontend.


We also have to take into account real-world limits when naming our system:



| Rule | Length |
|---|---|
| Ruby Gems Name| 64 character name, can't contain hyphens in folder names |
| Github Repos Name | 64 characters |
| NPM Package Name | 124 characters |
| Postgres Table Name | 63 character | 
| DynamoDB Table Name | 255 character | 
| S3 Bucket Name | 255 character, can't contain underscores | 

Take for example the subsystem which would result in 45 characters:

```
teacherseat_ui_admin_team_insights_organizations
```

So we can instead name it:

```
ts_ui_admin_tmi_organizations
```

## 1. Setup

Tip: It's easier to copy the files from an already existing interface since most of the setup is very similar.


<details>
<summary>package.json</summary>
<p>

```json
{
  "name": "<provider>_ui_<namespace>_<system>_<subsystem>",
  "version": "major.minor.patch",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --mode production     --config webpack.prod.js",
    "watch": "webpack --mode development -w --config webpack.dev.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "actioncable": "^5.2.4-2",
    "coffee-loader": "^0.9.0",
    "coffeescript": "^2.4.1",
    "css-loader": "^4.3.0",
    "file-loader": "^6.1.1",
    "filemanager-webpack-plugin": "^3.0.0-alpha.1",
    "http-server": "^0.12.3",
    "inflection": "^1.12.0",
    "luxon": "^1.25.0",
    "node-sass": "^4.14.1",
    "sass-loader": "^10.0.3",
    "terser": "^4.1.2",
    "terser-webpack-plugin": "^1.3.0",
    "uuid": "^3.3.2",
    "webpack": "^4.33.0",
    "webpack-cli": "^3.3.3",
    "ts_ui_admin": "git+ssh://git@github.com:teacherseat/ts_ui_admin.git#semver:1.3.0"
  },
  "dependencies": {
    "clipboard": "^2.0.6",
    "dilithium-js": "1.1.2",
    "dotenv": "8.2.0",
    "mini-css-extract-plugin": "^1.0.0",
    "rollbar": "^2.19.3"
  }
}
```
</p>

1. Edit name to your interface's name, for example ```ts_ui_admin_sys```

```
  "name": "<provider>_ui_<namespace>_<system>_<subsystem>",
 ```
 
2. Edit version, for example ```1.2.3```

```
  "version": "major.minor.patch",
 ```
 
The version is important because it determine what will get compiled when the interface is used by CodeBuild. It's necessary to keep this upto date for everything to work properly.
</details>

<details>
<summary>buildspec.yaml</summary>


This file is used by CodeBuild to build out the final code. 

<p>

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 12
  build:
    commands:
      ## ---- We need the ID RSA so we can npm install ts_ui_admin
      - "[[ -d ~/.ssh ]] || mkdir ~/.ssh"
      - aws ssm get-parameter --name /teacherseat/service-assets/ID_RSA --with-decryption --query Parameter.Value --output text | tee ~/.ssh/id_rsa >/dev/null
      - chmod 400 ~/.ssh/id_rsa
      - touch ~/.ssh/known_hosts
      - "ssh-keygen -F github.com || ssh-keyscan github.com >>~/.ssh/known_hosts"
      ## ----- Install Packages
      # https://docs.npmjs.com/cli/v6/commands/npm-ci
      - cd $CODEBUILD_SRC_DIR
      - npm ci
      ## ----- Build
      - cd $CODEBUILD_SRC_DIR
      - echo "OUTPUT_PATH=$CODEBUILD_SRC_DIR" | tee production.env
      - npm run build
      - "VERSION=$(cat package.json | grep version | head -1 | awk -F: '{ print $2 }' | sed 's/[\",]//g' | xargs)"
      - echo "$BUCKET"
      - echo "$NAMESPACE"
      - echo "$VERSION"
  post_build:
    commands:
      - aws s3 cp "$CODEBUILD_SRC_DIR/javascripts/$NAMESPACE.js"  "s3://$BUCKET/$NAMESPACE/$VERSION/$NAMESPACE.$VERSION.js"
      - aws s3 cp "$CODEBUILD_SRC_DIR/stylesheets/$NAMESPACE.css" "s3://$BUCKET/$NAMESPACE/$VERSION/$NAMESPACE.$VERSION.css"

```
</p>
</details>

<details>
<summary>development.env</summary>


```
OUTPUT_PATH=/path/to/public
MOUNT_PATH='/example_spot'
```
```OUTPUT_PATH``` is the path to the public folder where the output files should be placed.

```MOUNT_PATH``` is where the admin panel should be placed. 

*See the ```webpack.dev.js``` section for how this file is loaded.* 
<p>

</p>
</details>
