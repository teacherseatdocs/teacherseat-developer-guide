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
