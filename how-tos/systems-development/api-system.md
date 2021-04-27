# How to create a TeacherSeat API System

## 0. Naming

A System / subsystem need to follow the naming schema to avoid conflicts with other future systems: 

```
<provider>_<namespace>_<system>_<subsystem>
```

Systems (with the exception of student systems) are always named three letters.
The API and Admin systems will generally match, But its possible for an Admin and System to use multiple apis.

This System is not the same as the actualy API which is a ruby SDK.

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
