# Integrating gulp-rev into your app

Outlined below are two common approaches to integrating an asset manifest like the one **gulp-rev** outputs into an application.

For our examples, we'll assume the following `rev-manifest.json`:

```json
{
  "js/app.js": "js/app-5c41412f.js",
  "js/lib.js": "js/lib-6d94673e.js",
  "css/app.css": "css/app-a4ae3dfa.css",
}
````


## Approach #1 - Generate index.html during build

One approach to working with `rev-manifest.json` is to use a templating language, such as [handlebars](http://handlebarsjs.com), to generate an `index.html` file which contained your fingerprinted files embedded into the page.

The idea is to read in your app's `rev-manifest.json`, and use the non-fingerprinted path to read in the fingerprinted path and inject it into the page. Note, this approach requires the `'compile index.html'` task to be run as part of your build process.

#### `index.hbs`

```html+jinja
<!doctype html>
<html>
	<head>
		<title>My App</title>
		<link rel="stylesheet" href="{{ assetPath "css/app.css" }}">
	</head>
	<body>
		<script src="{{ assetPath "js/lib.js" }}"></script>
		<script src="{{ assetPath "js/app.js" }}"></script>
	</body>
</html>
```

#### `gulpfile.js`

```js
var fs = require('fs');
var gulp = require('gulp');
var handlebars = require('gulp-compile-handlebars');
var rename = require('gulp-rename');

// create a handlebars helper to look up
// fingerprinted asset by non-fingerprinted name
var handlebarOpts = {
	helpers: {
		assetPath: function (path, context){
			return ['/assets', context.data.root[path]].join('/');
		}
	}
};

gulp.task('compile index.html', function () {
	// read in our manifest file
	var manifest = require('path/to/rev-manifest');

	// read in our handlebars template, compile it using
	// our manifest, and output it to index.html
	return gulp.src('index.hbs')
		.pipe(handlebars(manifest, handlebarOpts))
		.pipe(rename('index.html'))
		.pipe(gulp.dest('public'));
});
```


## Approach #2 - AJAX in manifest, inject assets into the page

Another approach would be to make a AJAX request to get the manifest JSON blob, then use the manifest to programmatically find the fingerprinted path to any given asset.

For example, if you wanted to include your JavaScript files into the page:

```js
$.getJSON('/path/to/rev-manifest.json', function (manifest) {
	var s = document.getElementsByTagName('script')[0];

	var assetPath = function (src) {
		src = 'js/' + src + '.js'
		return ['/assets', manifest[src]].join('/');
	};

	['lib', 'app'].forEach(function (src) {
		var el = document.createElement('script');
		el.async = true;
		el.src = assetPath(src);
		s.parentNode.insertBefore(el, s);
	});
})
````

The above example assumes your assets live under `/assets` on your server.
