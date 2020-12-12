# How to create a TeacherSeat Admin System


## Create a Mountable Rails Engine

Systems are [mountable Rails Engines](https://guides.rubyonrails.org/engines.html) configured to a specific standard.
To create an Admin System name your engine admin_<system>

```
rails plugin new admin_iam --mountable
```

> The reason we create a `mountable` engine instead of a `full` engine is because we want our engine to be completely isolate so if we include it in our coddebase it acts like an isolate service
