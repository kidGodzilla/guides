As you're developing your Ember app, you'll likely run into common scenarios that aren't addressed by Ember itself,
such as authentication or using SASS for your stylesheets.
Ember CLI provides a common format called _Ember Addons_ for distributing reusable libraries
to solve these problems.
Additionally, you may want to make use of front-end dependencies like a CSS framework
or a JavaScript datepicker that aren't specific to Ember apps.
Ember CLI supports installing these packages through the standard _Bower package manager_.

## Addons

Ember Addons are installed using NPM (e.g. `npm install --save-dev ember-cli-sass`).
Addons may bring in other dependencies by modifying your project's `bower.json` file automatically.

You can find listings of addons on [Ember Observer](http://emberobserver.com).

## Installing an Ember Addon for our application

For our sample application, `Ember Rentals`, we have created an ember addon containing all of the stylesheets and images you will need for the project.

To install this addon, navigate to your ember-cli project and type:

```shell
ember install ember-tutorial
```

Which will return:

```text
Installed packages for tooling via npm.
Installed addon package.
```

This will add `app.css` to your project, which will be added to the build pipeline, and eventually appear in the `vendor.css` portion of your application. It will also add the NPM package `ember-tutorial` to your `package.json`, which will ensure you always have the latest version of the addon when building your project.

Since this was added to `vendor.css`, you will not see this in your app directory, and can continue to use `app/styles/app.css` to add custom styles to your ember application.