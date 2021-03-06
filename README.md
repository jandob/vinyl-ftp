# vinyl-ftp

Blazing fast vinyl adapter for FTP.
Supports parallel transfers, conditional transfers, buffered or streamed files, and more.
Often performs better than your favorite desktop FTP client.

## Usage

Nice and gulpy deployment task:

```javascript
var gulp = require( 'gulp' );
var gutil = require( 'gulp-util' );
var ftp = require( 'vinyl-ftp' );

gulp.task( 'deploy', function() {

	var conn = ftp.create( {
		host:     'mywebsite.tld',
		user:     'me',
		password: 'mypass',
		parallel: 10,
		log: gutil.log
	} );

	var globs = [
		'src/**',
		'css/**',
		'js/**',
		'fonts/**',
		'index.html'
	];

	// using base = '.' will transfer everything to /public_html correctly
	// turn off buffering in gulp.src for best performance

	return gulp.src( globs, { base: '.', buffer: false } )
		.pipe( conn.newer( '/public_html' ) ) // only upload newer files
		.pipe( conn.dest( '/public_html' ) );

} );
```

Without Gulp:

```javascript
var fs = require( 'vinyl-fs' );
var ftp = require( 'vinyl-ftp' );

var conn = new ftp( /* ... */ );

fs.src( [ './src/**' ], { buffer: false } )
	.pipe( conn.dest( '/dst' ) );
```

## API

`var ftp = require( 'vinyl-ftp' )`

### ftp.create( config )

Return a new `vinyl-ftp` instance with the given config. Config options:

- __host:__       FTP host,     default is localhost
- __user:__       FTP user,     default is anonymous
- __pass[word]:__ FTP password, default is anonymous@
- __port:__       FTP port,     default is 21
- __log:__        Log function, default is null
- __timeOffset:__ Offset server time by this number of minutes, default is 0
- __parallel:__   Number of parallel transfers, default is 3
- __maxConnections:__ Maximum number of connections, should be greater or
equal to "parallel". Default is 5, or the parallel setting, if any.
Don't worry about setting this too high, vinyl-ftp
recovers from "Too many connections" errors nicely.
- __keep:__       Keep connections alive when streams end, default is false
Remember to have your last stream { keep: false } so
remaining FTP connections are closed on end.
- __reload:__     Clear caches before (each) stream

You can override `parallel`, `keep` and `reload` per stream in their `options`.

<hr>

`var conn = ftp.create( config )`

### conn.src( globs[, options] )

Returns a vinyl file stream that emits remote files matched by the given
globs.
The remote files have a `file.ftp` property containing remote information.
Possible options:

- __cwd:__ Set as file.cwd, default is `/`.
- __base:__ Set as file.base, default is glob beginning. This is used to determine the file names when saving in .dest().
- __since:__ Only emit files modified after this date.
- __buffer:__ Should the file be buffered (complete download) before emitting? Default is true.
- __read:__ Should the file be read? Default is true. False will emit null files.

Glob-related options are documented at [minimatch](https://www.npmjs.com/package/minimatch).

<hr>

### conn.dest( remoteFolder[, options] )

Returns a transform stream that transfers input files to a remote folder.
All directories are created automatically.
Passes input files through.

### conn.mode( remoteFolder, mode[, options] )

Returns a transform stream that sets remote file permissions for each file.
`mode` must be a string between '0000' and '0777'.

### conn.newer( remoteFolder[, options] )

Returns a transform stream which filters the input for files
which are newer than their remote counterpart.

### conn.differentSize( remoteFolder[, options] )

Returns a transform stream which filters the input for files
which have a different file size than their remote counterpart.

### conn.newerOrDifferentSize( remoteFolder[, options] )

See above.

### conn.filter( remoteFolder, filter[, options] )

Returns a transform stream that filters the input using a callback.
The callback should be of this form:

```javascript
function( localFile, remoteFile, callback ) {

	// localFile and remoteFile are vinyl files.
	// Check remoteFile.ftp for remote information.
	// Decide wether localFile should be emitted and call callback with boolean.
	// callback is a function( error, emit )

	callback( null, emit );

}
```

### conn.delete( path, cb )

Deletes a file.

### conn.rmdir( path, cb )

Removes a directory, recursively.


## Todo

- implement `watch( globs[, opt, cb] )`
