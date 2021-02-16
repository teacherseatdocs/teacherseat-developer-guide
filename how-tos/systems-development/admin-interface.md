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
 
 THe version is important because it determine what will get compiled when the interface is used by CodeBuild. It's necessary to keep this upto date for everything to work properly.
</details>
