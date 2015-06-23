# quick-sip
Gulp build process tasks that provide watchify and sass compilation.

## Project Layout
Here is an example project hierarchy that will work with the default settings:

```
project
  |-- package.json
  |-- gulpfile.js
  |-- app
       |-- app.js
       |-- app.scss
       |
       |-- scripts
       |      |-- chairModel.js
       |      |-- chairView.js
       |
       |-- styles
       |      |-- _chair.scss
       |      |-- _engraved.scss
       |
       |-- index.html
```

After running the build task, a "dist" directory will be created:
```
project
   | ...
   |-- dist
        |-- app.js
        |-- app.js.map
        |-- app.css
        |
        |-- index.html
```
**app.js** is the browserified application that app.js completely specifies.
**app.js.map** is the source map for the browserified app.js bundle.
**app.scss** is compiled by sass into app.css.
**index.html** is copied from the source during the copy-resources task.

## Executing Tasks
Your **gulpfile.js** only needs the following to give you access to the build tasks:
```javascript
var gulp = require('gulp');
var buildProcess = require('quick-sip')(gulp);
```

The gulp tasks you care about are:
- **clean** - Remove the contents of the dist directory
- **build** - Builds the whole application; js, css, and resources.
- **watch** - Sets up watchify for your js, listens for scss changes, and handles other resource changes.
- **copy-resources** - Only copy the resources to dist (non-js and non-scss by default)
- **build-styles** - Only builds the styles to dist
- **build-app** - Only builds the javascript to dist

## Transforming Browserify
Easy!
```javascript
var gulp = require('gulp');
var buildProcess = require('quick-sip')(gulp, {
  browserify: {
    transforms: [
      { transform: 'aliasify', options: { global: true } },
      'hbsfy',
      yourCustomThroughTransform
    ]
  }
});
```

## Defining Resource Exclusion Extensions
Easy!
```javascript
var gulp = require('gulp');
var buildProcess = require('quick-sip')(gulp, {
  copy: {
    excludes: 'js|css|scss|hbs|frag|vert'
  }
});
```

## Configuration Details
The configurations are specified as an optional property when creating the build processes.
```javascript
var gulp = require('gulp');
var buildProcess = require('quick-sip')(gulp, options);
```
### Options
The options are grouped by task with some top level defaults.  Default values for each option is in ().  Default options are also specified in the [quick-sip/tasks/utils/options.js](tasks/utils/options.js) file.

#### Base Options
##### `taskPrefix` (`''`)
Prefix used to prefix all task names to make them unique in your build.  Not required if you only use quick-sip or if you do not define any overlapping tasks in gulp.

##### `src` (`'app'`)
Default source directory to use, used as the root for each task's default src references (See task specific default options below).

##### `dist` (`'dist'`)
Default distribution directory to use, used as the root for each task's default destination references (See task specific default options below).

#### Clean task options (task name: `options.taskPrefix + 'clean'`):
##### `clean.skip` (`false`)
Whether the clean task should be skipped (`true`) or run (`false`).

##### `clean.dist` (`options.dist`)
Defaults to the dist value in the base options.  This is the directory that clean will delete when run.

#### Browserify task options (task name: `options.taskPrefix + 'build-app'`):
##### `browserify.skip` (`false`)
Whether the browserify (`build-app`) task should be skipped (`true`) or run (`false`).

##### `browserify.root` (`'./' + options.src + '/app'`)
The root javascript file to start browserify.
With no configuration quick-sip will browserify `'[your project]/app/app.js'`.

##### `browserify.transforms` (`[]`)
Collection of transforms to apply when running browserify.  They can either be a value that browserify can consume natively (see [browserify's transform option](https://www.npmjs.com/package/browserify#b-transform-tr-opts)).
Alternatively you can also specify options for the transform using
```javascript
{
  transform: [transform name or function],
  options: [options to pass in for the transform]
}
```

##### `browserify.out` (`'app.js'`)
The name of the file in the `browserify.dist` directory to save the browserified bundle.
With no configuration quick-sip will save the resulting browserified file to `'[your project]/dist/app.js'`.

##### `browserify.failOnError` (`false`)
Whether the browserify task should fail on the first error it encounters (`true`) and error the build or print out all failures it encounters and return success for the overall build.

##### `browserify.debug` (`$.util.env.type !== 'production'`)
Whether to run browserify in debug mode (`true`) or non-debug mode (`false`).  Defaults to checking if the current environment is production (non-debug mode) or some other environment (debug mode).

##### `browserify.dist` (`options.dist`)
The distribution directory for the files generated by the browserify process.
With no configuration quick-sip will save all browserified files to `'[your project]/dist'`.

#### Styles task options (task name: `options.taskPrefix + 'build-styles'`)
##### `styles.skip` (`false`)
Whether the styles (`build-styles`) task should be skipped (`true`) or run (`false`).

##### `styles.includes` (`[]`)
Array of paths to additional locations of files used by the sass plugin (see [node-sass includePaths option](https://github.com/sass/node-sass#includepaths)).

##### `styles.root` (`options.src + '/app.scss'`)
The root file to run the sass on.
With no configuration quick-sip will run sass on `'[your project]/app/app.scss'`.

##### `styles.dist` (`options.dist`)
The destination directory for the resulting .css file.
With no configuration quick-sip will save the css file to `'[your project]/dist/app.css'`.

#### Copy task options (task name: `options.taskPrefix + 'copy-resources'`)
##### `copy.skip` (`false`)
Whether the styles (`copy-resources`) task should be skipped (`true`) or run (`false`).

##### `copy.src` (`options.src`)
The source directory to copy from.  The task will copy all files matching:
```javascript
[
  options.copy.src + '/**/*.*',
  '!' + options.copy.src + '/**/*.+(' + options.copy.excludes + ')'
]
```

##### `copy.excludes` (`'scss'`)
This is a regex that matches any files you want excluded from the copy task.
Example that excludes lots of file types: 'js|css|scss|hbs|frag|vert'

##### `copy.dist` (`options.dist`)
The destination directory for the copied files.
With no configuration quick-sip will save the files to `'[your project]/dist/'`.