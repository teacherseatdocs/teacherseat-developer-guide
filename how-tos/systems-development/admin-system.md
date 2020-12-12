# How to create a TeacherSeat Admin System


## 1. Setup

### 1.1 Generate a Rails Engine
Systems are [mountable Rails Engines](https://guides.rubyonrails.org/engines.html) configured to a specific standard.
To create an Admin System you will prepend your engine name with admin namsepace `admin_<system>`

```
rails plugin new admin_iam --mountable
```

> The reason we create a `mountable` engine instead of a `full` engine is because we want our engine to be completely isolate so if we include it in our coddebase it acts like an isolate service or can be deployed on its on.

### 1.2 Rename your System

After generating out your project rename the the root folder to the following format `admin.<system>`

```
mv admin_iam admin.iam
```

### 1.3 Create TeacherSeat System Config

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
  }
}
```

This file is used to register the system for you use in the Admin Systems Management.
