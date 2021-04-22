# How to create a TeacherSeat Admin System

## 0. Naming

A System / subsystem need to follow the naming schema to avoid conflicts with other future systems: 

```
<provider>_<namespace>_<system>_<subsystem>
```

We also have to take account realworld limits when naminig our system:



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
teacherseat_admin_team_insights_organizations
```

So we can instead name it:

```
ts_admin_tmi_organizations
```

## 1. Setup

### 1.1 Generate a Rails Engine
Systems are [mountable Rails Engines](https://guides.rubyonrails.org/engines.html) configured to a specific standard.
To create an Admin System you will prepend your engine name with admin namsepace `<provider>_admin_<system>`

```
rails plugin new ts_admin_iam --mountable
```

> The reason we create a `mountable` engine instead of a `full` engine is because we want our engine to be completely isolate so if we include it in our coddebase it acts like an isolate service or can be deployed on its on.

Fill in the required information for author, email, homepage, summary, description, license in the generated `ts_admin_iam.gemspec` file

```rb
  spec.name        = "ts_admin_iam"
  spec.version     = TsAdminIam::VERSION
  spec.authors     = ["TeacherSeat"]
  spec.email       = ["andrew@teacherseat.com"]
  spec.homepage    = "https://www.teacherseat.com/engines/ts_admin_iam"
  spec.summary     = "TeacherSeat Admin Identity and Access Management System"
  spec.description = "The management of users and the access to resources via permissions, policies, groups, and roles"
  spec.license     = "MIT"
```
### 1.3 Create TeacherSeat System Configuration File

In the root of your project you need to create a `teacherseat.json`

```json
{
  "author": "TeacherSeat",
  "name": "TeacherSeat.Admin.IAM",
  "friendly_name": "Identity Management System",
  "description": "Identity and Access Management System",
  "version" : "1.0.0"
}
```

This file is used to register the system for you use in the Admin Systems Management.

Verify the version number of the `version.rb` is corrected in the generated folder `lib/ts_admin_iam/`

```rb
module TsAdminIam
  VERSION = '1.0.0'
end
```
### 1.4 Configure Querylet and Querylet Rails

- [Querylet](https://github.com/teacherseat/querylet) is a gem we use to write raw queries.
- [Querylet Rails](https://github.com/teacherseat/querylet-rails) is a gem that has Rails Concerns files for our BaseController and ApplicationRecord

Open up your gemspec file eg. admin_iam.gemspec and add the following:

```rb
  spec.add_dependency "rails", "~> 6.1.1"
  spec.add_dependency "querylet-rails"
```

Create a queries directory under your System's app directory

```
mkdir app/queries
touch app/queries/.keep
```

Create an Api::BaseController and include QueryletRails::Controller::Queryable

```
mkdir app/controllers/ts_admin_iam/api
touch app/controllers/ts_admin_iam/api/base_controller
```

```rb
require 'querylet_rails/controller/queryable'
require_dependency "ts_admin_iam/application_controller"
class TsAdminIam::Api::BaseController < ApplicationController
  include QueryletRails::Controller::Queryable
end
```

> Note that we have a `require_dependency`, this is to ensure our engine loads the correct application_controller from the engine and not from our parent app

Modify app/controllers/ts_admin_iam/application_controller.rb

```rb
require 'teacherseat_permissions/controller/admin/base/permissible'
module TsAdminIam
  class ApplicationController < ActionController::Base
    include TeacherseatPermissions::Controller::Admin::Base::Permissible
    
    protect_from_forgery with: :exception
  end
end
```

Modify your ApplicationRecord to include QueryletRails::Model::Queryable

```rb
require 'querylet_rails/controller/queryable'
module TeacherseatAdminIam
  class ApplicationRecord < ActiveRecord::Base
    self.abstract_class = true
    include QueryletRails::Model::Queryable

    def self.query_root
      TsAdminIam::Engine.root
    end
  end
end
```

> self.query_root is an class method we can override for querylet-rails to tell querylet-rails where are app/queries directory is located. This ensure it loads the queries directory in our engine and not from the parent's app/queries directory

## Install and Configure TeacherseatPermissions

Every Admin System has to handle permissions. Update your `teacherseat.json` to have a section for permissions

```json
{
  "author": "TeacherSeat",
  "name": "TeacherSeat.Admin.IAM",
  "friendly_name": "Identity Management System",
  "description": "Identity and Access Management System",
  "version" : "1.0.0",
  "permissions": [
    {"name": "TeacherSeat::Admin::IAM::Policies::Create", "Be able to create a new policy" }
  ]
}
```
If the engine requires an admin menu option, or needs to be searchable in the admin panel add this information in the `teacherseat.json`

```json
{
  "author": "TeacherSeat",
  "name": "TeacherSeat.Admin.IAM",
  "friendly_name": "Identity Management System",
  "description": "Identity and Access Management System",
  "version" : "1.0.0",
  "permissions": [
    {"name": "TeacherSeat::Admin::IAM::Policies::Create", "Be able to create a new policy" }
  ],
  "admin": {
    "search": [
       {"permission_handle": "users"       , "permission": "TeacherSeat:Admin:IAM:Admins:List"      , "icon": "fal fa-user"         , "path": "/iam/users"       , "name": "Admins"      , "description": "Users that have access to your admin panel" },
       {"permission_handle": "roles"       , "permission": "TeacherSeat:Admin:IAM:Roles:List"       , "icon": "fal fa-user"         , "path": "/iam/roles"       , "name": "Roles"       , "description": "Roles associate a set of permissions to an indentity" },
       {"permission_handle": "permissions" , "permission": "TeacherSeat:Admin:IAM:Permissions:List" , "icon": "fal fa-shield-check" , "path": "/iam/permissions" , "name": "Permissions" , "description": "Permissions specifc access to system actions" },
       {"permission_handle": "policies"    , "permission": "TeacherSeat:Admin:IAM:Policies:List"    , "icon": "fal fa-lock"         , "path": "/iam/policies"    , "name": "Polices"     , "description": "Policies defines a set of permissions" }
    ],
    "browse": [
      {"position": 100, "category": "Identity and Access", "name": "Admins"      , "permission": "TeacherSeat:Admin:IAM:Admins:List"       , "path": "/iam/users"},
      {"position": 200, "category": "Identity and Access", "name": "Roles"       , "permission": "TeacherSeat:Admin:IAM:Roles:List"       , "path": "/iam/roles"},
      {"position": 300, "category": "Identity and Access", "name": "Policies"    , "permission": "TeacherSeat:Admin:IAM:Policies:List"    , "path": "/iam/policies"},
      {"position": 400, "category": "Identity and Access", "name": "Permissions" , "permission": "TeacherSeat:Admin:IAM:Permissions:List" , "path": "/iam/permissions"}
    ],
    "categories": [
      {"position": 100, "name": "Identity and Access", "icon": ""}
    ]
  }
}
```

Modify your Api::BaseController and include Teacherseat::Controller::Admin::Api::Permissible

```rb
require 'querylet_rails/controller/queryable'
require 'teacherseat_permissions/controller/admin/api/permissible'
require_dependency "ts_admin_iam/application_controller"

class TeacherseatAdminIam::Api::BaseController < ApplicationController
  include QueryletRails::Controller::Queryable
  include TeacherseatPermissions::Controller::Admin::Api::Permissible
end
```

In the main app, update the `routes.rb` to mount the engine in the appropriate namespace

```rb
namespace :admin, path: '/admin' do
    # Admin Engines [Start]
    mount TsAdminIam::Engine => '/iam'
    mount TsAdminSys::Engine => '/sys'
    mount TsAdminDsh::Engine => '/dsh'
    mount TsAdminRes::Engine => '/res'
    mount TsAdminTen::Engine => '/ten'
    mount TsAdminEva::Engine => '/eva'
    mount TsAdminStu::Engine => '/stu'
    mount TsAdminCfg::Engine => '/cfg'
    # Admin Engines [End]
  end
```

Include the new engine in the `Gemfile` by specifying your local development path

```rb
gem 'ts_admin_iam'  , path: '/Users/denniswork/Desktop/teacherseat/ts_backend/admin/ts_admin_iam'
```

Run a bundle install
