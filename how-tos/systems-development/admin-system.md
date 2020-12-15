# How to create a TeacherSeat Admin System

## 0. Namining

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
rails plugin new teacherset_admin_iam --mountable
```




> The reason we create a `mountable` engine instead of a `full` engine is because we want our engine to be completely isolate so if we include it in our coddebase it acts like an isolate service or can be deployed on its on.

### 1.3 Create TeacherSeat System Configuration File

In the root of your project you need to create a `teacherseat.json`

```json
{
  "name": "TeacherSeat.Admin.IAM",
  "description": "Indentity and Access Management System",
  "version" : {
    "major": 1,
    "minor": 0,
    "tiny":  0,
    "release" : {
      "type": "alpha",
      "version": 0
    }
  },

}
```

This file is used to register the system for you use in the Admin Systems Management.

### 1.4 Configure Querylet and Querylet Rails

- [Querylet](https://github.com/teacherseat/querylet) is a gem we use to write raw queries.
- [Querylet Rails](https://github.com/teacherseat/querylet-rails) is a gem that has Rails Concerns files for our BaseController and ApplicationRecord

Open up your gemspec file eg. admin_iam.gemspec and add the following:

```rb
  spec.add_dependency "querylet-rails"
```

Create a queries directory under your System's app directory

```
mkdir app/queries
touch app/queries/.keep
```

Create an Api::BaseController and include QueryletRails::Controller::Queryable

```
mkdir app/controllers/api
touch app/controllers/api/base_controller
```

```rb
require 'querylet_rails/controller/queryable'
require_dependency "admin_iam/application_controller"
class AdminIam::Api::BaseController < ApplicationController
  include QueryletRails::Controller::Queryable
end
```

> Note that we have a `require_dependency`, this is to ensure our engine loads the correct application_controller from the engine and not from our parent app

Modify your ApplicationRecord to include QueryletRails::Model::Queryable

```rb
require 'querylet_rails/model/queryable'
module TeacherseatAdminIam
  class ApplicationRecord < ActiveRecord::Base
    self.abstract_class = true
    include QueryletRails::Model::Queryable

    def self.query_root
      AdminIam::Engine.root
    end
  end
end
```

> self.query_root is an class method we can override for querylet-rails to tell querylet-rails where are app/queries directory is located. This ensure it loads the queries directory in our engine and not from the parent's app/queries directory

## Install and Configure TeacherseatPermissions

Every Admin System had to handle permissions. Update your `teacherseat.json` to have a section for permissions

```json
{
  "name": "TeacherSeat.Admin.IAM",
  "description": "Indentity and Access Management System",
  "version" : {
    "major": 1,
    "minor": 0,
    "tiny":  0,
    "release" : {
      "type": "alpha",
      "version": 0
    }
  },
  "permissions": [
    {"name": "TeacherSeat::Admin::IAM::Policies::Create", "Be able to create a new policy" }
  ]
}
```

Modify your Api::BaseController and include Teacherseat::Controller::Permissible

```rb
require 'querylet_rails/controller/queryable'
require 'teacherseat_permissions/controller/permissible'
require_dependency "admin_iam/application_controller"

class TeacherseatAdminIam::Api::BaseController < ApplicationController
  include QueryletRails::Controller::Queryable
  include Teacherseat::Controller::Permissible
end
```

