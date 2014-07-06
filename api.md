gulp API docs
=====================

### gulp.src(globs[, options])

Takes a glob and represents a file structure. Can be piped to plugins.

```
gulp.src('./client/templates/*.jade')
    .pipe(jade())
    .pipe(minify())
    .pipe(gulp.dest('./build/minified_templates'));
```

`glob` refers to [node-glob syntax](https://github.com/isaacs/node-glob) or it can be a direct file path.

#### globs

Type: `String` or `Array`

Glob or globs to read.

#### options

Type: `Object`

Options to pass to [node-glob](https://github.com/isaacs/node-glob) through [glob-stream](https://github.com/wearefractal/glob-stream)

gulp adds two additional options in addition to the options supported by node-glob and glob-stream:

#### options.buffer

Type: `Boolean` Default: `true`

Setting this to `false` will return `file.contents` as a stream and not buffer files. This is useful when working with large files. __Note:__ Plugins may not implement support for streams.

#### options.read

Type: `Boolean` Default: `true`

Setting this to `false` will return `file.contents` as null and not read the file at all.


### gulp.dest(path)

Can be piped to and it will write files. Re-emits all data passed to it so you can pipe to multiple folders. Folders that don't exist will be created.

```
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```

#### 路径 _path_

类型: `String`

写文件的路径(目录)。

_The path(folder) to write files to._

### gulp.task(name[,deps], fn)

使用[Orchestrator](https://github.com/orchestrator/orchestrator)定义一个任务。

_Define a task using [Orchestrator](https://github.com/orchestrator/orchestrator)_

```
gulp.task('somename', function() {
  // Do stuff
});
```

#### name

任务名称。如果希望在命令行运行任务，则任务名不能包含空格。

_The name of the task. Tasks that you want to run from command line should not have spaces in them._

#### deps

类型：`Array`

一个将要执行的任务列表，并且它们在你的任务开始前结束。

_An array of tasks to be executed and complete before your task will run._

```
gulp.task('mytask', ['array', 'of', 'task', 'names'], function() {
  // Do stuff
});
```
_Note: Are your tasks running before the dependencies are complete? Make sure your dependency tasks are correctly using the async run hints: take in a callback or return a promise or event stream._

#### fn

_The function that performs the task's operations. Generally this takes the form of `gulp.src().pipe(someplugin)`._

#### Async task support

_Tasks can be made asynchronous if its `fn` does one of following:_

##### Accept a callback

```
// Run a command in a shell 
var exec = require('child_process').exec;
gulp.task('jekyll', function(cb) {
  // Build Jekyl
  exec('jekyll build', function(err) { 
    if (err) return cb(err); //return error
    cb(); // finished task
  });
});
```

##### Return a stream

```
gulp.task('somename', function() {
  var stream = gulp.src('./client/**/*.js')
    .pipe(minify())
    .pipe(gulp.dest('/build'));
  return stream;
});
```

##### Return a promise

```
var Q = require('q');

gulp.task('somename', function() {
  var deferred = Q.defer();

  // Do async stuff
  setTimeout(function() {
    deferred.resolve();
  }, 1);

  return deferred.promise;
});
```

__Note:__ By default, tasks run with maximum concurrency -- e.g. it launches all the tasks at once and waits for nothing. If you want to create a series where tasks run in a particular order, you need to do two things:

* give it a hint to tell it when the task is done,

* and give it a hint that a task depends on completion of anthoer.

For there examples, let's presume you have two tasks, "one" and "two" that you specifically want to run in this order:
 
1. In task "one" you add a hint to tell it when the task is done. Either take in a callback and call it where you're done or return a promise or stream that th engine should wait to resolve or end respectively.

2. In task "two" youo add a hint  telling the  engine that it depends on completion of the first task.

So this example would look like this:

```
var gulp = require('gulp');

// takes in a callback so the engine knows when it'll be done
gulp.task('one', function(cb) {
    // do stuff -- async or otherwise
    cb(err); // if err is not null and not undefined, the run will stop, and note that it failed
});

// identifies a dependent task must be complete before this one begins
gulp.task('two', ['one'], function() {
    // task 'one' is done now
});

gulp.task('default', ['one', 'two']);
```

### gulp.watch(glob[,opts], tasks) or gulp.watch(glob[,opts, cb])

Watch files and do something when a file changes. This always return an EventEmitter that emits `change` event.

### gulp.watch(glob[,opts], tasks)

#### glob

类型：`String` 或者 `Array`

A single glob or array of globs that indicate which files to watch for changes.

#### opts

类型：`Object`

传递给[`gaze`](https://github.com/shama/gaze)的配置


#### tasks

类型：`Array`

当一个文件变化时运行通过 `gulp.task()`添加的任务名。

```
var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File '+event.path+' was '+event.type+', running tasks...');
});
```

### gulp.watch(glob[,opts, cb])

#### glob

类型：`String` or `Array`

A single glob or array of globs that indicate which files to watch for changes.

#### opts

类型：`Object`

传递给[`gaze`](https://github.com/shama/gaze)的配置

#### cb(event)

类型：`Function`

每次变化所执行的回调

```
gulp.watch('js/**/*.js', function(event) {
  console.log('File '+event.path+' was '+event.type+', running tasks...');
});
```

回调将会被传递一个对象 `event`, 它描述了变化

##### event.type

类型：`String`

发生变化的类型，`added`, `changed` 或者 `deleted`.

##### event.path

触发事件的文件路径.