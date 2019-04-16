# Django + Webpack example project

## Build Setup

I used following commands to bootstrap this project.

```sh
mkdir django-webpack
cd django-webpack

pipenv install django django-webpack-loader
pipenv run django-admin startproject django-webpack .
pipenv run python manage.py startapp things

npm install -g vue-cli

vue init webpack vue-test
cd vue-test

npm install webpack-bundle-tracker
```

##### `settings.py`

Add `things` and `webpack_loader` to `INSTALLED_APPS`.

Update static files configurations.

```py
STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")
STATIC_URL = "/static/"

STATICFILES_DIRS = [os.path.join(BASE_DIR, "vue-test", "dist", "static")]

WEBPACK_LOADER = {
    "DEFAULT": {
        "CACHE": not DEBUG,
        "BUNDLE_DIR_NAME": "vue-test/dist/static/",  # must end with slash
        "STATS_FILE": os.path.join(BASE_DIR, "vue-test", "dist", "webpack-stats.json"),
    }
}
```

#### Configure Vue app

##### `vue-test/build/webpack.base.conf.js`

```diff
@@ -4,7 +4,9 @@ const utils = require('./utils')
 const config = require('../config')
 const vueLoaderConfig = require('./vue-loader.conf')

+var BundleTracker = require('webpack-bundle-tracker');
+
 function resolve(dir) {
   return path.join(__dirname, '..', dir)
 }

@@ -88,5 +90,11 @@ module.exports = {
     net: 'empty',
     tls: 'empty',
     child_process: 'empty'
-  }
+  },
+  plugins: [
+    new BundleTracker({
+      path: __dirname,
+      filename: '../dist/webpack-stats.json',
+    }),
+  ]
 }
```


#### Create view

##### `things/views.py`

```py
from django.shortcuts import render
from django.views.generic import TemplateView


class HomeView(TemplateView):
    template_name = "things/home.html"
```

##### `things/templates/things/home.html`

```html
<div id="app"></div>

{% load render_bundle from webpack_loader %}
{% render_bundle 'manifest' %}
{% render_bundle 'vendor' %}
{% render_bundle 'app' %}
```

##### `django_webpack/urls.py`

```diff
 from django.contrib import admin
 from django.urls import path
+from things import views

 urlpatterns = [
+    path("", views.HomeView.as_view()),
     path("admin/", admin.site.urls)
 ]
```

### Build

In one terminal:

```sh
cd vue-test
npm run build
```

In the other terminal:

```sh
pipenv run python manage.py runserver
```