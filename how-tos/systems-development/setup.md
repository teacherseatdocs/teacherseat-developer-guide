# Multi-engine Setup

This setup guide is to help you understand how to work with a multi-engine service in your development enviroment.


## Setup TeacherSeat Directories

The web-application is composed of multiple frontend and backends

Create the following directory structure:

```
~/Sites/
├── teacherseat
    └── ts_frontend/
    └── ts_backend/
```

Run the following commands. 
```
mkdir ~/Sites
mkdir ~/Sites/teacherseat
mkdir ~/Sites/teacherseat/ts_frontend
mkdir ~/Sites/teacherseat/ts_backend
```

A Rails application will need the followin Core engines installed:

- `ts_admin_dsh` - Dashboard Service (the homepage dashboard admin's first see when they login)
- `ts_admin_iam` — Identity and Access Management (how we manage users and permissions)
- `ts_admin_sys` — Systems Manager (how we register systems)

We'll need to clone the repos:

```
cd ~/Sites/teacherseat/ts_frontend
git clone https://github.com/teacherseat/ts_admin_dsh
git clone https://github.com/teacherseat/ts_admin_iam
git clone https://github.com/teacherseat/ts_admin_sys
```

In your Gemfile you can then reference them via path for local development:

```
gem 'ts_admin_dsh'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_dsh'
gem 'ts_admin_iam'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_iam'
gem 'ts_admin_sys'  , path: '/Users/andrewbrown/Sites/teacherseat/ts_backend/admin/ts_admin_sys'
bundle install
```
