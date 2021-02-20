<h1 id='table-of-contents'>TABLE OF CONTENTS</h1>

- [MonoRepo Frontend Initializers](#monorepo-frontend-initializers)
  - [ts_ui_admin_ten](#ts_ui_admin_ten)
    - [Folders and Files](#folders-and-files)
    - [Webpack](#webpack)
      - [Loading an External Library](#loading-an-external-library)
      - [webpack.dev.js](#webpackdevjs)
      - [webpack.prod.js](#webpackprodjs)
    - [JavaScripts](#javascripts)
      - [Services](#services)
      - [Components](#components)
      - [Views](#views)
      - [Routes](#routes)
  - [ts_ui_admin](#ts_ui_admin)
    - [Load Initializers](#load-initializers)
    - [Shared Api](#shared-api)
    - [Dead End](#dead-end)
    - [Not Found](#not-found)
  - [Project Initializer](#project-initializer)
    - [MOUNT_PATH](#mount_path)
    - [Loading Initializers](#loading-initializers)
    - [Config](#config)

# MonoRepo Frontend Initializers

[Go Back to Developer Guide](../../README.md)

The mono-repo is an app that acts as a container for the frontends and backends.

Within this application we configure out custom frontend initializers to load third-party dependencies.

In this example we are going to create `ts_ui_admin_ten` initializer

## ts_ui_admin_ten

### Folders and Files

[Go Back to Contents](#table-of-contents)

As a convention we are going to follow the following structure.

```Bash
  mkdir images
  mkdir javascripts
  mkdir javascripts/services
  mkdir javascripts/views
  mkdir javascripts/components
  mkdir javascripts/views/tenants
  mkdir stylesheets
  touch images/.keep
  touch javascripts/app.coffee
  touch javascripts/services/api.coffee
  touch javascripts/services/filters.coffee
  touch javascripts/components/tenants.coffee
  touch javascripts/views/tenants/index.coffee
  touch stylesheets/style.sass
  touch webpack.dev.js
  touch webpack.prod.js
```

```Bash
  ts_ui_admin_ten
  ├─ buildspec.yaml
  ├─ development.pa.env
  ├─ development.wcd.env
  ├─ images
  │  └─ .keep
  ├─ index.js
  ├─ javascripts
  │  ├─ app.coffee
  │  ├─ components
  │  │  └─ tenants.coffee
  │  ├─ services
  │  │  ├─ api.coffee
  │  │  └─ filters.coffee
  │  └─ views
  │     └─ tenants
  │        └─ index.coffee
  ├─ package-lock.json
  ├─ package.json
  ├─ stylesheets
  │  └─ style.sass
  ├─ webpack.dev.js
  └─ webpack.prod.js
```

### Webpack

[Go Back to Contents](#table-of-contents)

Each namespace has 2 configuration files:

- `webpack.dev.js`
- `webpack.prod.js`

Both configuration are very similar, the only difference is that the `production` has the `TerserPlugin` to minifier (compress) the file.

```JavaScript
  // Only required in webpack.prod.js

  const minifier = new TerserPlugin({
    terserOptions: {
      mangle: true,
      toplevel: true,
      keep_fnames: false,
      keep_classnames: true
    }
  })
```

- **NOTE:** In the `Terser` configuration, we are not compressing the classnames (`keep_classnames: true`) because in our **css** we use the actual class name to apply the styles.

#### Loading an External Library

[Go Back to Contents](#table-of-contents)

**minicss object**

- In our `webpack` configuration file, we need to define a `minicss` object, this object is responsible for mapping/including all `css/sass` files that we want to use.
- As you can see we are chaining the `css-loader` with the `sass-loader` to immediately apply all styles to the DOM.

  - **MiniCssExtractPlugin** (`mini-css-extract-plugin`) extracts `CSS` into separate files. It creates a `CSS` file per `JS` file which contains `CSS`. It supports On-Demand-Loading of `CSS` and SourceMaps.
  - In our case we are including `ts_ui_admin_ten/stylesheets` path and the `ts_ui_admin_ten/node_modules/ts_ui_admin/stylesheets` path (**dev dependencies** that we installed in our `package.json`)

  ```JavaScript
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
  ```

**module.exports**

- In `module.exports` we are going to export the configuration to be compiled with webpack.
- Two important things are required we need to check before executing the webpack

  1. Check the entry point

     - Check if the `entry` object is pointing to the right `key/value` pair
     - In this case we are creating the `ts_ui_admin_ten`, so our keys should follow the following format `stylesheets/ts_ui_admin_ten` and `javascripts/ts_ui_admin_ten`

       ```JavaScript
         'javascripts/ts_ui_admin_ten' : path.resolve(__dirname, 'javascripts/app.coffee')
         //           |                      |
         //           |                      └── value (path_to/namespace/javascripts/app.coffee)
         //           └── key (folder_type/namespace)
       ```

       ```JavaScript
         entry: {
           'stylesheets/ts_ui_admin_ten' : path.resolve(__dirname, 'stylesheets/style.sass'),
           'javascripts/ts_ui_admin_ten' : path.resolve(__dirname, 'javascripts/app.coffee')
         },
       ```

  2. Check the alias

     - Check if the alias is mapping to the correct path
     - These alias we are going to use to import shared components in our CoffeeScripts files

       ```JavaScript
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
       ```

#### webpack.dev.js

[Go Back to Contents](#table-of-contents)

Final `ts_ui_admin_ten/webpack.dev.js` structure

```JavaScript
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
      'stylesheets/ts_ui_admin_ten' : path.resolve(__dirname, 'stylesheets/style.sass'),
      'javascripts/ts_ui_admin_ten' : path.resolve(__dirname, 'javascripts/app.coffee')
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
      new MiniCssExtractPlugin({ filename: '[name].css'}),
      new webpack.DefinePlugin({
        "process.env": JSON.stringify(dotenv.parsed)
      })
    ],
    node: { __dirname: false },
    module: { rules: [coffee, minicss, file] }
  }
```

#### webpack.prod.js

[Go Back to Contents](#table-of-contents)

Final `ts_ui_admin_ten/webpack.prod.js` structure

```JavaScript
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
      'stylesheets/ts_ui_admin_ten' : path.resolve(__dirname, 'stylesheets/style.sass'),
      'javascripts/ts_ui_admin_ten' : path.resolve(__dirname, 'javascripts/app.coffee')
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

### JavaScripts

#### Services

[Go Back to Contents](#table-of-contents)

In `javascripts/services/api.coffee`

- Create a custom `API` class that extends from `ApiBase`
- Our new class will be responsible for making our API calls, before each call we need to define a few information:

  - namespace
  - headers
  - resources

  ```Coffee
    import { ApiBase } from 'dilithium-js'

    class Api extends ApiBase
      namespace: "#{window.MOUNT_PATH}/ten/api"
      headers:
        'X-CSRF-Token': =>
          document.getElementsByName('csrf-token')[0].content
      resources:
        tenants: true

    api = new Api()
    export default api
  ```

In `javascripts/services/filters.coffee`

- We don't need to change anything, just use the default configuration

  ```Coffee
    import { trust } from 'dilithium-js'
    import { DateTime, Duration } from 'luxon'

    export commify = (input) ->
      return unless input
      input.toString().replace /\B(?=(\d{3})+(?!\d))/g, ','

    export iso_date = (input) ->
      return unless input
      DateTime.fromFormat(input, "yyyy-MM-dd hh:mm ZZZ").toFormat("yyyy-MM-dd")

    export highlight = (input,term) ->
      return unless input
      regex = new RegExp "(#{term})", 'ig'
      input = input.replace regex, "<mark>$1</mark>"
      trust input

    export posts_datetime = (input) ->
      return unless input
      if DateTime.fromFormat(input,"yyyy-MM-dd HH:mm ZZZ").hasSame(DateTime.local(), 'day')         then DateTime.fromFormat(input, "yyyy-MM-dd HH:mm ZZZ").toFormat("h:mm a")
      else if DateTime.fromFormat(input, "yyyy-MM-dd HH:mm ZZZ").hasSame(DateTime.local(), 'year')  then DateTime.fromFormat(input, "yyyy-MM-dd HH:mm ZZZ").toFormat("MMM dd")
      else
        DateTime.fromFormat(input, "yyyy-MM-dd HH:mm ZZZ").toFormat("MMM dd yyyy")

    export time_ago = (original_datetime)->
      return '' unless original_datetime
      luxon_time = DateTime.fromFormat(original_datetime, "yyyy-MM-dd HH:mm ZZZ").toMillis()
      diff = DateTime.local().toMillis() - luxon_time
      duration = Duration.fromMillis(diff)
      days = duration.as('days').toString().split('.')[0]
      weeks = duration.as('weeks').toString().split('.')[0]
      hours = duration.as('hours').toString().split('.')[0]
      if weeks > 0
        "#{weeks} weeks ago"
      else if days > 0
        "#{days} days ago"
      else
        "#{hours} hours ago"

    export seconds_difference = (old_date, new_date)->
      return '' unless old_date
      return '' unless new_date
      old = DateTime.fromFormat(old_datetime, "yyyy-MM-dd HH:mm ZZZ").toMillis()
      current = DateTime.fromFormat(new_date, "yyyy-MM-dd HH:mm ZZZ").toMillis()
      diff = current - old
      duration = Duration.fromMillis(diff)
      duration.as('seconds').toString().split('.')[0]

    export date_after = (compare_date)->
      return false unless compare_date
      compare = DateTime.fromFormat(compare_date, "yyyy-MM-dd").toMillis()
      diff = DateTime.local().toMillis() - compare
      if diff > 0
        return true
      else
        return false

    export friendly_duration = (start_datetime,end_datetime)->
      return '' unless start_datetime
      return '' unless end_datetime
      sdt = DateTime.fromFormat(start_datetime, "yyyy-MM-dd HH:mm ZZZ")
      edt = DateTime.fromISO(end_datetime)
      ms = edt.diff(sdt,'milliseconds')
      DateTime.local().minus(ms, 'milliseconds').toRelative().replace(/ ago/,'')

    export strip_html = (html) ->
      tmp = document.createElement('DIV')
      tmp.innerHTML = html
      tmp.textContent or tmp.innerText or ''

    export move = (arr, old_index, new_index)->
      if new_index >= arr.length
        k = new_index - (arr.length)
        while k-- + 1
          arr.push undefined
      arr.splice new_index, 0, arr.splice(old_index, 1)[0]
      this

    export sort_position = (dir,collection,model) ->
      last = collection.length-1
      i    = collection.indexOf model
      switch  dir
        when 'up'   then move collection, i, i-1 unless i is 0
        when 'down' then move collection, i, i+1 unless i is last
  ```

#### Components

[Go Back to Contents](#table-of-contents)

In `javascripts/components/tenants.coffee`

- Create a basic `Tenant` component

  ```Coffee
    import { m, trust, stream, Component} from 'dilithium-js'
    import { highlight, iso_date } from 'services/filters'

    export default class Tenant extends Component
      expects:
        model: true
        search: true
      render:=>
        m 'tr.t.tenant',
          m 'td.t.one.name', highlight @model.name, @search()
  ```

#### Views

[Go Back to Contents](#table-of-contents)

In `javascripts/views/tenants/index.coffee`

- Import our `Tenant` component
- Create a new `View` (`TenantIndex`)
- Update the variables/properties to `tenants`, `tenant` or `Tenant` where applicable

```Coffee
  import { m, Link, rout, param, stream, View } from 'dilithium-js'
  import Api from 'services/api'
  import Header from 'shared/components/header'
  import Paginate from 'shared/components/paginate'
  import Search from 'shared/components/shared'
  import Tenant from ' components/tenant'

  export default class TenantIndex extends View
    body:
      view: 'list'
    events:
      'tenants/index': 'success'
      'tenants/index#err': 'err'
    constructor:(args)->
      super args
      @search = stream param('search')
      @filter = param('filter')
      @pagination = {}
      @pagination.page = parseInt(param('page')) if param('page')
      @pagination.search = @search()
    reindex:(skip_loading=false)=>
      if skip_loading
        @loading = false
      else
        @loading = true
      attrs = {}
      attrs.filter = @filter if @filter
      attrs.page = @pagination.page
      attrs.search = @search() if @search()
      Api.tenants.index(attrs, {extract: @headers})
    oncreate:(vnode)->
      tippy '.tippy', arrow: true
    onupdate:(vnode, old)->
      tippy '.tippy', arrow: true
    headers:(xhr)=>
      @pagination = JSON.parse(xhr.getResponseHeader('X-Pagination'))
      JSON.parse xhr.responseText
    heading:=>
      m '.page_heading',
        m Search, pagination: @pagination, reindex: @reindex, search: @search
        m '.spacer'
        m Paginate, pagination: @pagination, reindex: @reindex
        m '.clear'
    thead:=>
      m '.tenants'
        m 'thead.t'
          m 'td.t.one.system', 'System'
          m 'td.t.one.name', 'Name'
          m 'td.t.one.desc', 'Description'
    list:=>
      m '.groups',
        m 'table.t',
          for tenant in @model
            m Tenant model: tenant, search: @search

    rend er:=>
      return unless @model
      m 'main',
        m Header
        m 'article',
          m '.content_wrap',
            @heading()
            m '.content',
              @list()
```

#### Routes

[Go Back to Contents](#table-of-contents)

In `ts_ui_admin_ten/javascripts/app.coffee`

- We need to import `route` from `dilithium-js`
- Import [Deadend](#dead-end) and [Notfound](#not-found) from our `shared/views` (`ts_ui_admin` library)
- Import `SharedApi` from our `shared/services` (`ts_ui_admin` library)

  - [SharedApi](#shared-api) will `GET` our initial data
  - Then we re-assign the same window variable `_init` with the incoming data
  - Then we pass `_init` object to our [load_initializer](#load-initializers) to be loaded

  ```Coffee
    import { route } from 'dilithium-js'
    import Deadend  from 'shared/views/deadend'
    import Notfound from 'shared/views/notfound'

    import TenantsIndex from 'views/tenants/index'

    import SharedApi from 'shared/services/api'
    import load_initializers from 'shared/services/load_initializers'

    SharedApi.users.init {}, success: (data)=>
      window._init = data
      load_initializers _init

      routes =
        "#{window.MOUNT_PATH}": Deadend
        "#{window.MOUNT_PATH}/ten/tenants": TenantsIndex
        '/:any...' : Notfound

      route.prefix = ''
      route document.body, window.MOUNT_PATH, routes
  ```

  - Naming convention, every system has a 3 letters abbreviation

    ```Coffee
      "#{window.MOUNT_PATH}/ten/tenants": TenantsIndex
      #                      |      |           |
      #                      |      |           └── View
      #                      |      └── sub-system
      #                      └── abbreviation
    ```

## ts_ui_admin

[Go Back to Contents](#table-of-contents)

- [ts_ui_admin Repo](https://github.com/teacherseat/ts_ui_admin)
- The `ts_ui_admin` is a sharable library where we can define common components (`sass/css` and `CoffeeScript/JavaScript`).
- The idea is to have a sharable library where we can use with multiple frontends/namespaces.

- In the current version `1.3.2` we have available the following components:

  ```Bash
    .
    ├── javascripts
    │   ├── components
    │   │   ├── account_dropdown.coffee
    │   │   ├── form_header.coffee
    │   │   ├── header.coffee
    │   │   ├── paginate.coffee
    │   │   ├── popup_delete.coffee
    │   │   ├── search.coffee
    │   │   ├── services_dropdown.coffee
    │   │   ├── services_search.coffee
    │   │   └── support_dropdown.coffee
    │   ├── services
    │   │   ├── api.coffee
    │   │   ├── filters.coffee
    │   │   └── load_initializers.coffee
    │   └── views
    │       ├── deadend.coffee
    │       └── notfound.coffee
    ├── stylesheets
    │   └── shared
    │       ├── breakpoints.sass
    │       ├── default.sass
    │       ├── delete_popup.sass
    │       ├── field.sass
    │       ├── form_header.sass
    │       ├── form.sass
    │       ├── header.sass
    │       ├── index.sass
    │       ├── markdown.sass
    │       ├── notfound.sass
    │       ├── page_heading.sass
    │       ├── popup.sass
    │       ├── show.sass
    │       └── table.sass
    ├── index.js
    └── package.json
  ```

### Load Initializers

[Go Back to Contents](#table-of-contents)

In `ts_ui_admin/javascripts/services/load_initializers.coffee`

- The `load_initializers` will receive the incoming data (`_init` object)
- Then will call the `console.debug` so we can see what kind of data is being loaded to our window

```Coffee
  # merges two objects eg. merge {}, {}
  merge = (xs...) ->
    if xs?.length > 0
      tap {}, (m) -> m[k] = v for k, v of x for x in xs
  tap = (o, fn) -> fn(o); o

  export default load_initializers = (_init) ->
    options = {}
    for init in window.initializers
      console.debug("load_initializers:initialize_#{init}")
      outputted_options = window["initialize_#{init}"](_init,options)
      options = merge options, outputted_options
```

### Shared Api

[Go Back to Contents](#table-of-contents)

In `ts_ui_admin/javascripts/services/api.coffee`

- The `api.coffee` is responsible for getting the initial data

  ```Coffee
    import { ApiBase } from 'dilithium-js'

    class SharedApi extends ApiBase
      namespace: "#{window.MOUNT_PATH}/iam/api"
      headers:
        'X-CSRF-Token': =>
          document.getElementsByName('csrf-token')[0].content
      resources:
        users:
          collection:
            init: 'get'
        permissions:
          collection:
            header: 'get'

    api = new SharedApi()
    export default api
  ```

### Dead End

[Go Back to Contents](#table-of-contents)

In `ts_ui_admin/javascripts/views/deadend.coffee`

- All what `deadend` does is to redirect to `home` page if page doesn't exist

```Coffee
  import { m, View } from 'dilithium-js'
  # Needed to a Component for the default route.
  export default class Deadend extends View
    constructor:(args)->
      super(args)
      window.location.href = "#{window.MOUNT_PATH}/dsh/home"
    view:(vnode)->
      m 'div'
```

### Not Found

[Go Back to Contents](#table-of-contents)

In `ts_ui_admin/javascripts/views/notfound.coffee`

- The `notfound` is rendered if the user try to access a route that doesn't exist

```Coffee
  import { m, View } from 'dilithium-js'
  export default class Notfound extends View
    render:=>
      m 'main',
        m '.logo'
        m '.line'
        m '.error', '404'
        m 'h1', 'Page Not Found.'
        m 'p.msg',
          "You are seeing this page because you've tried to view a page which does not exist."
```

## Project Initializer

[Go Back to Contents](#table-of-contents)

Every project will have a namespace called `_initializers`
In `_initializers` folder we have common things that are loaded before any other script.
The `_initializers` namespace has the following structure:

```Bash
  .
  ├── javascripts
  │   ├── logrocket.coffee
  │   ├── mount_path.coffee
  │   ├── rollbar.coffee
  │   └── stripe.coffee
  ├── development.env
  ├── development.env.example
  ├── package-lock.json
  ├── package.json
  ├── webpack.dev.js
  └── webpack.prod.js
```

### MOUNT_PATH

[Go Back to Contents](#table-of-contents)

In `your_rails_project/_initializers/javascripts/mount_path.coffee`

- The `mount_path.coffee` is responsible for appending the `MOUNT_PATH` to our window. This way we can access/know where to mount page

```Coffee
  # Do to race case we need to define the outside of the initializer
  window.MOUNT_PATH = '/admin'

  window.initialize_mount_path = (_init,options={}) ->
    window.MOUNT_PATH = '/admin'
    return {}
```

### Loading Initializers

[Go Back to Contents](#table-of-contents)

In `your_rails_project/app/views/layouts/dilithium.html.haml`

- As we said before, all the initializers are loaded before any other script.
- In the end of the file we can find the `initializer` import and below that we have our `scripts` import (`include_js_tag`)

  ```Ruby
    - if admin_namespace?
      - initializers = JSON.parse(File.open(Rails.root.join('initializers.json')).read)['exp_admin']
      = javascript_tag "window.initializers = #{initializers.to_json}"
      - initializers.each do |initializer|
        = include_js_initializer_tag initializer
      = include_js_tag 'pa_admin'
    - else
      - initializers = JSON.parse(File.open(Rails.root.join('initializers.json')).read)['exp_student']
      = javascript_tag "window.initializers = #{initializers.to_json}"
      - initializers.each do |initializer|
        = include_js_initializer_tag initializer
      = include_js_tag 'pa_student'
  ```

### Config

[Go Back to Contents](#table-of-contents)

In `your_rails_project/initializers.json`

- We can define all the [initializers](#project-initializer) that we want to load
- In our case we need to add `ts_ui_admin_ten`

  - our `ts_ui_admin_ten` will load the `admin_nav`, `rollbar` and [mount_path](#mount_path)

  ```JSON
    {
      "ts_ui_admin_dsh"  : ["admin_nav","rollbar","mount_path"],
      "ts_ui_admin_iam"  : ["admin_nav","rollbar","mount_path"],
      "ts_ui_admin_sys"  : ["admin_nav","rollbar","mount_path"],
      "ts_ui_admin_ten"  : ["admin_nav","rollbar","mount_path"],
      "pa_admin"         : ["admin_nav","rollbar","mount_path"],
      "pa_student"       : ["rollbar","logrocket"]
    }
  ```
