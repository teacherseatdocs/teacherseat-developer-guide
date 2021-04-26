# How to create a TeacherSeat Admin Interface

<h1 id='table-of-contents'>TABLE OF CONTENTS</h1>

- [How to create a TeacherSeat Admin Interface](#how-to-create-a-teacherseat-admin-interface)
  - [0. Naming](#0-naming)
    - [System and Interface](#system-and-interface)
    - [Real-world Limits](#real-world-limits)
  - [1. Directory structure](#1-directory-structure)
    - [Package.json](#packagejson)
    - [Buildspec.json](#buildspecjson)
    - [Development.env](#developmentenv)
    - [Webpack.dev.js & Webpack.prod.js](#webpackdevjs--webpackprodjs)
  
## 0. Naming

### System and Interface
An interface / sub-interface needs to follow the naming schema to avoid conflicts with other future interfaces: 

```
<provider>_ui_<namespace>_<interface>_<sub-interface>
```

Each backend will have a corresponding frontend. The backend and frontend names needs to match.

A frontend interface with the name:
```
<provider>_ui_<namespace>_<interface>_<sub-interface>
```

Would correspond to a system with the name:
```
<provider>_<namespace>_<interface>_<sub-interface>
```

and vice-versa. 

Note that the only difference is the word `ui` in the frontend.

### Real-world Limits
We also have to take into account real-world limits when naming our interface:


| Rule | Length |
|---|---|
| Ruby Gems Name| 64 character name, can't contain hyphens in folder names |
| Github Repos Name | 64 characters |
| NPM Package Name | 124 characters |
| Postgres Table Name | 63 character | 
| DynamoDB Table Name | 255 character | 
| S3 Bucket Name | 255 character, can't contain underscores | 


Take for example the sub-interface which would result in 45 characters:

```
teacherseat_ui_admin_team_insights_organizations
```

So we can instead name it:

```
ts_ui_admin_tmi_organizations
```

## 1. Directory structure

> Tip: It's easier to copy the files from an already existing interface since most of the setup is very similar.

```
~/<provider>_ui_<namespace>_<interface>_<sub-interface>/
  ├── images/
  ├── javascripts/
  |   ├── components/
  |   ├── lib/
  |   ├── models/ 
  |   ├── services/ 
  |   ├── views/
  |   |   └── <sub-interface>/
  |   |       └── index.coffee  
  |   └── app.coffee
  └── stylesheets/   
  |   ├── lib/
  |   ├── <sub-interface>/
  |   |   └── index.sass
  |   └── style.sass
  ├── buildspec.yaml
  ├── development.env
  ├── package.json
  ├── webpack.dev.js
  └── webpack.prod.js
```
### Package.json

Initialize a new npm package

```Bash
  npm init
  package name: "<provider>-ui-<namespace>-<system>-<subsystem>"
  version: "1.0.0" (default)
  description: ""
  entry point: "index.js" (default)
  git repository: ""
  keywords: ""
  author: ""
  license: "ISC" (default)
```


1. Edit name to your interface's name, for example ```ts_ui_admin_sys_organizations```

```
  "name": "<provider>_ui_<namespace>_<system>_<subsystem>",
 ```
> **ATTENTION** the `package name` is `-` (dash) separated, not `_` (underscore).

2. Edit version, for example ```1.2.3```

```
  "version": "major.minor.patch",
 ```
 
The version is important because it determine what will get compiled when the interface is used by CodeBuild. It's necessary to keep this upto date for everything to work properly.


After we generate our default `package.json`, we need to add **build** and **watch** scripts

```JSON
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --mode production     --config webpack.prod.js",
    "watch": "webpack --mode development -w --config webpack.dev.js"
  },
```

- `build` creates the production version
- `watch` creates the development version


**Dependencies**

```Bash
  npm i clipboard
  npm i dilithium-js@<version>
  npm i dotenv
  npm i mini-css-extract-plugin
  npm i rollbar
```

> **ATTENTION** Ensure that dilithium-js is of the same version as other engines.

**Dev Dependencies**

```Bash
  npm i -D actioncable
  npm i -D coffee-loader
  npm i -D coffeescript
  npm i -D css-loader
  npm i -D file-loader
  npm i -D filemanager-webpack-plugin
  npm i -D http-server
  npm i -D inflection
  npm i -D luxon
  npm i -D node-sass
  npm i -D sass-loader
  npm i -D terser
  npm i -D terser-webpack-plugin
  npm i -D uuid
  npm i -D webpack
  npm i -D webpack-cli@~3.3.3
```

> **ATTENTION** Additionally, `ts_ui_admin` has to be added manually to your package.json, under devDependencies.  
> **NOTE** you may need to downgrade uuid to match the version used by dilithium.

```JSON
"ts_ui_admin": "git+ssh://git@github.com:teacherseat/ts_ui_admin.git#semver:1.3.0"
```

**Final `package.json` Structure**

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
</details>

---

### Development.env


```
OUTPUT_PATH=/path/to/public
```
```OUTPUT_PATH``` is the path to the public folder where the output files should be placed.

*See the ```webpack.dev.js``` section for how this file is loaded.* 

---

### Webpack.dev.js & Webpack.prod.js

These are only few differences between these two files.

In the ```webpack.prod.js``` a compressor is used to minify the files.

```js
const minifier = new TerserPlugin({
  terserOptions: {
    mangle: true,
    toplevel: true,
    keep_fnames: false,
    keep_classnames: true
  }
})
```

In the ```webpack.dev.js```, we manually load the ```development.env``` file based on the ```TS_PROFILE``` environment variable that is passed.
```js
let env_filename = ''
console.log('TS_PROFILE')
console.log(process.env.TS_PROFILE)
if (process.env.TS_PROFILE) {
  env_filename = `development.${process.env.TS_PROFILE}.env`
} else {
  env_filename = 'development.env'
}
```

However, for production, only the ```production.env``` file is loaded which is automatically created in the ```buildspec.yaml``` file. 

There is a default library that is shared among all ```teacherseat``` interfaces with base UI components.
```js
'shared/components' : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/components/'),
'shared/services'   : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/services/'),
'shared/views'      : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/views/')
```

These come from the ```ts_ui_admin``` interface. More components will be added soon.

Few changes needs to be made when copying these files over for your admin interface.

In both the ```webpack.dev.js``` and ```webpack.prod.js```, the interface's name needs to be changed.

In the ```module.exports``` function: 

```js
'stylesheets/<provider>_ui_<namespace>_<interface>_<sub-interface>' : path.resolve(__dirname, 'stylesheets/style.sass'),
'javascripts/<provider>_ui_<namespace>_<interface>_<sub-interface>' : path.resolve(__dirname, 'javascripts/app.coffee')
```

**Final `webpack.dev.js` Structure**

<details>
<summary>webpack.dev.js</summary>
  
```js
const path = require('path')
const webpack = require('webpack')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

let env_filename = ''
console.log('TS_PROFILE')
console.log(process.env.TS_PROFILE)
if (process.env.TS_PROFILE) {
  env_filename = `development.${process.env.TS_PROFILE}.env`
} else {
  env_filename = 'development.env'
}
const dotenv = require('dotenv').config({path: path.resolve(__dirname, env_filename) })

const coffee = {
  test: /\.coffee$/, loader: 'coffee-loader'
}

const minicss = {
  test: /\.s[ac]ss$/i,
  use: [
    MiniCssExtractPlugin.loader,
    {
      loader: 'css-loader',
      options: {
        sourceMap: true,
      }
    },
    {
      loader: 'sass-loader',
      options: {
        sourceMap: true,
        sassOptions: {
          includePaths: [
            path.resolve(__dirname, 'stylesheets'),
            path.resolve(__dirname, 'node_modules','ts_ui_admin','stylesheets')
          ]
        } // sassOptions
      } // options
    }
  ]
}

const file = {
  test: /\.(svg|png|jpe?g|gif)$/i,
  loader: 'file-loader',
  options: {
      name: '/[path][name].[ext]',
  },
}

module.exports = {
  mode: 'development',
  target: 'web',
  devtool: 'source-map',
  entry: {
    'stylesheets/ts_ui_admin_iam' : path.resolve(__dirname, 'stylesheets/style.sass'),
    'javascripts/ts_ui_admin_iam.js' : path.resolve(__dirname, 'javascripts/app.coffee')
  },
  output: {
    path    : process.env.OUTPUT_PATH,
    filename: '[name]'
  },
  resolve: {
    extensions: ['.coffee','.js','.json', 'scss'],
    modules: ['node_modules/'],
    alias: {
      components : path.resolve(__dirname, 'javascripts/components/'),
      models     : path.resolve(__dirname, 'javascripts/models/'),
      views      : path.resolve(__dirname, 'javascripts/views/'),
      services   : path.resolve(__dirname, 'javascripts/services/'),
      lib        : path.resolve(__dirname, 'javascripts/lib/'),
      'shared/components' : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/components/'),
      'shared/services'   : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/services/'),
      'shared/views'      : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/views/')
    }
  },
  plugins: [
    new MiniCssExtractPlugin({ filename: '[name].css'}),
    new webpack.DefinePlugin({
      "process.env": JSON.stringify(dotenv.parsed)
    })
  ],
  node: { __dirname: false },
  module: { rules: [coffee, minicss, file] }
}

```
</details>

**Final `webpack.prod.js` Structure**

<details>
<summary>webpack.prod.js</summary>
  
```js
const path = require('path')
const webpack = require('webpack')
const TerserPlugin = require('terser-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const dotenv = require('dotenv').config({path: path.resolve(__dirname, 'production.env') })

const coffee = {
  test: /\.coffee$/, loader: 'coffee-loader'
}

const minicss = {
  test: /\.s[ac]ss$/i,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    {
      loader: 'sass-loader',
      options: {
        sassOptions: {
          includePaths: [
            path.resolve(__dirname, 'stylesheets'),
            path.resolve(__dirname, 'node_modules','ts_ui_admin','stylesheets')
          ]
        } // sassOptions
      } // options
    }
  ]
}

const file = {
  test: /\.(svg|png|jpe?g|gif)$/i,
  loader: 'file-loader',
  options: {
      name: '/[path][name].[ext]',
  },
}

const minifier = new TerserPlugin({
  terserOptions: {
    mangle: true,
    toplevel: true,
    keep_fnames: false,
    keep_classnames: true
  }
})

module.exports = {
  mode: 'production',
  target: 'web',
  entry: {
    'stylesheets/ts_ui_admin_iam' : path.resolve(__dirname, 'stylesheets/style.sass'),
    'javascripts/ts_ui_admin_iam' : path.resolve(__dirname, 'javascripts/app.coffee')
  },
  output: {
    path    : process.env.OUTPUT_PATH,
    filename: '[name].js'
  },
  resolve: {
    extensions: ['.coffee','.js','.json', 'scss'],
    modules: ['node_modules/'],
    alias: {
      components : path.resolve(__dirname, 'javascripts/components/'),
      models     : path.resolve(__dirname, 'javascripts/models/'),
      views      : path.resolve(__dirname, 'javascripts/views/'),
      services   : path.resolve(__dirname, 'javascripts/services/'),
      lib        : path.resolve(__dirname, 'javascripts/lib/'),
      'shared/components' : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/components/'),
      'shared/services'   : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/services/'),
      'shared/views'      : path.resolve(__dirname, 'node_modules/ts_ui_admin/javascripts/views/')
    }
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css'
    }),
    new webpack.DefinePlugin({
      "process.env": JSON.stringify(dotenv.parsed)
    })
  ],
  optimization: {
    minimizer: [minifier],
  },
  node: { __dirname: false },
  module: { rules: [coffee, minicss, file] }
}
```
</details>

--- 

> **!Disclaimer**: In order to get the `Admin` panel running, you need to add the following

**Copy shared libraries**
- Copy `javascripts/lib` and `stylesheets/lib` from other existing interface to your folders
- In `stylesheets/style.sass`, add the shared stylesheets
  ```sass
  @import shared/breakpoints
  @import shared/default
  @import shared/index
  @import shared/show
  @import shared/form
  @import shared/field
  @import shared/form_header
  @import shared/markdown
  @import shared/header
  @import shared/page_heading
  @import shared/popup
  @import shared/table
  @import shared/delete_popup
  @import shared/notfound
  @import shared/components/button
  @import shared/components/button_group
  @import shared/components/button_drop
  @import shared/components/icon
  @import shared/components/layouts
  @import shared/components/breadcrumbs
  @import shared/components/details_table
  @import shared/components/section
  @import shared/components/divider
  @import shared/components/manage_view
  ```


---


### Buildspec.json

The `buildspec.yaml` file is used by `AWS CodeBuild`. The idea of the `buildspec.yaml` is that when we **tag** and **push** the code to GitHub, `CodeBuild` detects that we pushed a new **tag**, then builds out the updated code and saves into a `S3 Bucket` using the version that we defined in our `package.json`

- **Build Steps:**

  1. SSH Keys

     - Create the `.ssh` folder
     - Get the `id_rsa` (private key) from AWS SSM (Systems Manager) Parameter Store and add to our server in `~/.ssh/id_rsa`
     - Change file permission to `400` (Owner can only read)
     - Add github ssh key to `known_hosts`

     ```Yaml
       ## ---- We need the ID RSA so we can npm install ts_ui_admin
       - "[[ -d ~/.ssh ]] || mkdir ~/.ssh"
       - aws ssm get-parameter --name /teacherseat/service-assets/ID_RSA --with-decryption --query Parameter.Value --output text | tee ~/.ssh/id_rsa >/dev/null
       - chmod 400 ~/.ssh/id_rsa
       - touch ~/.ssh/known_hosts
       - "ssh-keygen -F github.com || ssh-keyscan github.com >>~/.ssh/known_hosts"
     ```

  2. Install `ci`

     ```Yaml
       # ----- Install Packages
       # https://docs.npmjs.com/cli/v6/commands/npm-ci
       - npm ci
     ```

  3. Build assets

     - Creates and saves the `OUTPUT_PATH` into `production.env`
     - Run the build
     - Get the `package.json` version, so we can use in the post build step

       ```Yaml
         ## ----- Build
         - cd $CODEBUILD_SRC_DIR
         - echo "OUTPUT_PATH=$CODEBUILD_SRC_DIR" | tee production.env
         - npm run build
         - "VERSION=$(cat package.json | grep version | head -1 | awk -F: '{ print $2 }' | sed 's/[\",]//g' | xargs)"
         - echo "$BUCKET"
         - echo "$NAMESPACE"
         - echo "$VERSION"
       ```

- **Post Build Steps:**

  - Saves the build out **JavaScripts** and **StyleSheets** into a `S3 Bucket` sorted by version

    ```Yaml
      post_build:
        commands:
          - aws s3 cp "$CODEBUILD_SRC_DIR/javascripts/$NAMESPACE.js"  "s3://$BUCKET/$NAMESPACE/$VERSION/$NAMESPACE.$VERSION.js"
          - aws s3 cp "$CODEBUILD_SRC_DIR/stylesheets/$NAMESPACE.css" "s3://$BUCKET/$NAMESPACE/$VERSION/$NAMESPACE.$VERSION.css"
    ```

    ```Bash
      s3://$BUCKET/$NAMESPACE/$VERSION/$NAMESPACE.$VERSION.js

      # s3://teacherseat-service-assets/ts_ui_admin_ten/1.0.0/ts_ui_admin_ten.1.0.0.js
      #                  |                      |         |          |          |    |
      #                  |                      |         |          |          |    └── extension
      #                  |                      |         |          |          └── package.json version
      #                  |                      |         |          └── namespace
      #                  |                      |         └── package.json version
      #                  |                      └── namespace
      #                  └── s3 bucket
    ```


**Final `buildspec.yaml` Structure**

<details>
<summary>buildspec.yaml</summary>
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


#### Server / AWS Deployment

In our **codebase** we have a folder called `aws` that has a bunch of `bash scripts`

- In there we have a script called `aws/deployment/after_install/pull_ts_assets.sh`

  - This scripts is responsible for copying new assets to our public folder (`javascripts` and `stylesheets`)
  - Since we are creating a new asset, we need to add:

    ```Bash
      ...
      ADMIN_TEN=1.0.0 # version 1.0.0 (same package.json version)

      ...
      echo "== s3 copy ts_ui_admin_ten... "
      aws s3 cp s3://$BUCKET/ts_ui_admin_ten/$ADMIN_TEN/ts_ui_admin_ten.$ADMIN_TEN.js  $PUBLIC/javascripts/ts_ui_admin_ten.$ADMIN_TEN.js
      aws s3 cp s3://$BUCKET/ts_ui_admin_ten/$ADMIN_TEN/ts_ui_admin_ten.$ADMIN_TEN.css $PUBLIC/stylesheets/ts_ui_admin_ten.$ADMIN_TEN.css
    ```

  ```Bash
    #!/usr/bin/env bash

    BUCKET=teacherseat-service-assets
    PUBLIC=/home/ec2-user/app/public

    ADMIN_DSH=1.3.1
    ADMIN_SYS=1.2.3
    ADMIN_IAM=1.3.5
    ADMIN_PAY=1.2.3
    ADMIN_TEN=1.0.0

    echo "== s3 copy ts_ui_admin_dsh... "
    aws s3 cp s3://$BUCKET/ts_ui_admin_dsh/$ADMIN_DSH/ts_ui_admin_dsh.$ADMIN_DSH.js  $PUBLIC/javascripts/ts_ui_admin_dsh.$ADMIN_DSH.js
    aws s3 cp s3://$BUCKET/ts_ui_admin_dsh/$ADMIN_DSH/ts_ui_admin_dsh.$ADMIN_DSH.css $PUBLIC/stylesheets/ts_ui_admin_dsh.$ADMIN_DSH.css

    echo "== s3 copy ts_ui_admin_sys... "
    aws s3 cp s3://$BUCKET/ts_ui_admin_sys/$ADMIN_SYS/ts_ui_admin_sys.$ADMIN_SYS.js  $PUBLIC/javascripts/ts_ui_admin_sys.$ADMIN_SYS.js
    aws s3 cp s3://$BUCKET/ts_ui_admin_sys/$ADMIN_SYS/ts_ui_admin_sys.$ADMIN_SYS.css $PUBLIC/stylesheets/ts_ui_admin_sys.$ADMIN_SYS.css

    echo "== s3 copy ts_ui_admin_iam... "
    aws s3 cp s3://$BUCKET/ts_ui_admin_iam/$ADMIN_IAM/ts_ui_admin_iam.$ADMIN_IAM.js  $PUBLIC/javascripts/ts_ui_admin_iam.$ADMIN_IAM.js
    aws s3 cp s3://$BUCKET/ts_ui_admin_iam/$ADMIN_IAM/ts_ui_admin_iam.$ADMIN_IAM.css $PUBLIC/stylesheets/ts_ui_admin_iam.$ADMIN_IAM.css

    echo "== s3 copy ts_ui_admin_pay... "
    aws s3 cp s3://$BUCKET/ts_ui_admin_pay/$ADMIN_PAY/ts_ui_admin_pay.$ADMIN_PAY.js  $PUBLIC/javascripts/ts_ui_admin_pay.$ADMIN_PAY.js
    aws s3 cp s3://$BUCKET/ts_ui_admin_pay/$ADMIN_PAY/ts_ui_admin_pay.$ADMIN_PAY.css $PUBLIC/stylesheets/ts_ui_admin_pay.$ADMIN_PAY.css

    echo "== s3 copy ts_ui_admin_ten... "
    aws s3 cp s3://$BUCKET/ts_ui_admin_ten/$ADMIN_TEN/ts_ui_admin_ten.$ADMIN_TEN.js  $PUBLIC/javascripts/ts_ui_admin_ten.$ADMIN_TEN.js
    aws s3 cp s3://$BUCKET/ts_ui_admin_ten/$ADMIN_TEN/ts_ui_admin_ten.$ADMIN_TEN.css $PUBLIC/stylesheets/ts_ui_admin_ten.$ADMIN_TEN.css
  ```
