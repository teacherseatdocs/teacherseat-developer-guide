# Multi-engine Setup

This setup guide is to help you understand how to work with a multi-engine service in your development enviroment.


## Setup TeacherSeat Directories and Pull Repositories 

The web-application is composed of multiple frontend and backends

Create the following directory structure:

```
~/Sites/
├── teacherseat
    ├── ts_frontend/
    |   ├── admin/ 
    |   └── student/
    └── ts_backend/
        ├── admin/ 
        └── student/ 
```

Run the following commands. 
```
mkdir -p ~/Sites/teacherseat/ts_frontend/admin
mkdir -p ~/Sites/teacherseat/ts_frontend/student
mkdir -p ~/Sites/teacherseat/ts_backend/admin
mkdir -p ~/Sites/teacherseat/ts_backend/student
```

A Rails application will need the following Core engines installed:

- `ts_admin_dsh` - Dashboard Service (the homepage dashboard admin's first see when they login)
- `ts_admin_iam` — Identity and Access Management (how we manage users and permissions)
- `ts_admin_sys` — Systems Manager (how we register systems)

We'll need to clone the repos:

```
cd ~/Sites/teacherseat/ts_backend/admin
git clone https://github.com/teacherseat/ts_admin_dsh
git clone https://github.com/teacherseat/ts_admin_iam
git clone https://github.com/teacherseat/ts_admin_sys
```

Most admin engines will have a corresponding frontend that is an isolate javascript application:

- `ts_admin_dsh` -> `ts_ui_admin_dsh`
- `ts_admin_iam` -> `ts_ui_admin_iam`
- `ts_admin_sys` -> `ts_ui_admin_sys`

We'll need to clone the repos:

```
cd ~/Sites/teacherseat/ts_frontend/admin
git clone https://github.com/teacherseat/ts_ui_admin_dsh
git clone https://github.com/teacherseat/ts_ui_admin_iam
git clone https://github.com/teacherseat/ts_ui_admin_sys
```

> The example is only show the Admin Systems, its the same process for Student Systems.

## Configuring Backend for Development

In your Gemfile you can then reference them via path for local development:

```
gem 'ts_admin_dsh'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_dsh'
gem 'ts_admin_iam'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_iam'
gem 'ts_admin_sys'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_sys'
bundle install
```

You can now change code within these engines and they should reflect in real time in the running development app.
In some cases you need to stop and restart your Rails app since Rails does not dynamically reload all classes and modules eg. RAILS_ROOT/lib

## Configuring Frontend for Development

All the frontend code is compiled directly into the the RAILS_ROOT/public/{javascripts,stylesheets}`

### Building individual frontends

Each frontend locally requiries `development.env` to be configured within the project.
You need to configure two values: OUTPUT_PATH

```
OUTPUT_PATH=/Users/andrewbrown/Sites/exampro-projects/exampro/public
```

When working with multiple projects you can pass `TS_PROFILE` enviroment variable to specific a different configuration file eg.

`TS_PROFILE=exp npm run watch` would load a configuration file called `development.exp.env`

### Recursively building frontends 

Using the node package `concurrently` we can build all our desired frontends in a single command.

Add a package.json in the `ts_frontend`

```
touch ~/Sites/teacherseat/ts_frontend
```

Add the following contents and configure based on your use case

```json
{
  "name": "teacherseat-recursive",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "directories": {
    "lib": "lib",
    "test": "test"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch-exp-admin-iam"  : "cd admin/ts_ui_admin_iam && TS_PROFILE=exp npm run watch",
    "watch-exp-admin-pay"  : "cd admin/ts_ui_admin_pay && TS_PROFILE=exp npm run watch",
    "watch-exp-admin-sys"  : "cd admin/ts_ui_admin_sys && TS_PROFILE=exp npm run watch",
    "watch-exp-admin-dsh"  : "cd admin/ts_ui_admin_dsh && TS_PROFILE=exp npm run watch",
    "watch-exp-student_pay": "cd student/ts_ui_student_pay && TS_PROFILE=exp npm run watch",
    "watch-exp-student_tmi": "cd student/ts_ui_student_tmi && TS_PROFILE=exp npm run watch",
    "watch-exp": "concurrently npm:watch-exp-*",

    "watch-wcd-admin-iam"  : "cd admin/ts_ui_admin_iam && TS_PROFILE=wcd npm run watch",
    "watch-wcd-admin-sys"  : "cd admin/ts_ui_admin_sys && TS_PROFILE=wcd npm run watch",
    "watch-wcd-admin-dsh"  : "cd admin/ts_ui_admin_dsh && TS_PROFILE=wcd npm run watch",
    "watch-wcd": "concurrently npm:watch-wcd-*",
  },
  "repository": {
    "type": "git",
    "url": ""
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": ""
  },
  "homepage": "",
  "dependencies": {
    "concurrently": "^5.3.0"
  }
}
```

And then install the dependenices 

```
cd ~/Sites/ts_frontend
npm install
```

Then you can build for your target project. For ExamPro it would be:

```
npm run watch-exp
```
