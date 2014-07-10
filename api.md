gulp API docs
=====================

### gulp.src(globs[, options])

输入一个通配符并且代表了一个文件结构。能够被输送到插件。

_Takes a glob and represents a file structure. Can be piped to plugins._

```
gulp.src('./client/templates/*.jade')
    .pipe(jade())
    .pipe(minify())
    .pipe(gulp.dest('./build/minified_templates'));
```

__通配符__参见 [node-glob syntax](https://github.com/isaacs/node-glob) 或者直接是一个文件路径。

_`glob` refers to [node-glob syntax](https://github.com/isaacs/node-glob) or it can be a direct file path._

#### globs

Type: `String` or `Array`

需要读取的通配符或者一组通配符。

_Glob or globs to read._

#### options

Type: `Object`

通过[glob-stream](https://github.com/wearefractal/glob-stream) 传递给 [node-glob](https://github.com/isaacs/node-glob) 的配置。

_Options to pass to [node-glob](https://github.com/isaacs/node-glob) through [glob-stream](https://github.com/wearefractal/glob-stream)_

`glup` 添加了两个新增配置项到[node-glob](https://github.com/isaacs/node-glob)和[glob-stream](https://github.com/wearefractal/glob-stream)所支持的配置项中。

_gulp adds two additional options in addition to the options supported by node-glob and glob-stream:_

#### options.buffer

Type: `Boolean` Default: `true`

设置为`false` `file.contents`将以`stream`的形式返回而不是`buffer`。这对操作打文件非常有帮助。__注意：__ 插件可以不支持 `streams`。

_Setting this to `false` will return `file.contents` as a stream and not buffer files. This is useful when working with large files. Note: Plugins may not implement support for streams._

#### options.read

Type: `Boolean` Default: `true`

设置这个配置为`false`  `file.contents` 将会返回`null`，并且不会读取这个文件。

_Setting this to `false` will return `file.contents` as null and not read the file at all._


### gulp.dest(path)

输送的目的地并且写入文件。在次触发所有数据可以被输送到多个目录中去。目录如果不存在将会被创建。

_Can be piped to and it will write files. Re-emits all data passed to it so you can pipe to multiple folders. Folders that don't exist will be created._

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

注意：你的任务将会在所有的依赖执行完成之后执行？请确保所有的依赖正确的使用了异步执行提示：输入到一个回调或者返回一个`promise` 或者 `event stream`。

_Note: Are your tasks running before the dependencies are complete? Make sure your dependency tasks are correctly using the async run hints: take in a callback or return a promise or event stream._

#### fn

执行任务操作的方法。一般来说，采取`gulp.src().pipe(someplugin)`的形式。

_The function that performs the task's operations. Generally this takes the form of `gulp.src().pipe(someplugin)`._

#### 异步任务支持 _Async task support_

任务可以是异步的，如果`fn`按照下面的方法来实现：

_Tasks can be made asynchronous if its `fn` does one of following:_

##### 接受一个回调 _Accept a callback_

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

##### 返回一个流 _Return a stream_

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

__注意：__ 默认情况下，任务的执行有最大并发数。例如：它将会启动所有的任务并且什么也不等待。如果你想创建一个队列，来让任务按照特定顺序执行，你需要做两件事情：

_Note: By default, tasks run with maximum concurrency -- e.g. it launches all the tasks at once and waits for nothing. If you want to create a series where tasks run in a particular order, you need to do two things:_

* 提供一个提示来告诉它任务什么时候完成

* _give it a hint to tell it when the task is done,_

* 并且提示它一个任务依赖另一个任务的完成。

* _and give it a hint that a task depends on completion of anthoer._

例如，假设你有两个任务，"one" 和 "two" 想要按照一下顺序来执行：

_For there examples, let's presume you have two tasks, "one" and "two" that you specifically want to run in this order:_

1. 在任务"one"中添加一个提示告诉它任务什么时候完成。输入到回调或者或者在完成的地方调用它或者返回一个`promise`或者`stream` 引擎需要等待解决或者各自结束。
 
_1. In task "one" you add a hint to tell it when the task is done. Either take in a callback and call it where you're done or return a promise or stream that the engine should wait to resolve or end respectively._

2. 在任务"two" 中添加一个提示告诉引擎他需要等待第一个任务的结束。

_2. In task "two" you add a hint  telling the  engine that it depends on completion of the first task._

所以这个列子应该像这样：

_So this example would look like this:_

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

监控文件并在文件发生变化时做处理。此方法返回一个[`EventEmitter`](./eventemitter.md)来触发`change`事件。

_Watch files and do something when a file changes. This always returns an EventEmitter that emits `change` events._

### gulp.watch(glob[,opts], tasks)

#### glob

类型：`String` 或者 `Array`

单一通配符或者一组通配符，指明监控哪些文件的变化。

_A single glob or array of globs that indicate which files to watch for changes._

#### opts

类型：`Object`

传递给[`gaze`](https://github.com/shama/gaze)的配置

_Options, that are passed to [`gaze`](https://github.com/shama/gaze)._

#### tasks

类型：`Array`

当一个文件变化时运行通过 `gulp.task()`添加的任务名。

_Names of task(s) to run when a file changes, added with `gulp.task()`_

```
var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File '+event.path+' was '+event.type+', running tasks...');
});
```

### gulp.watch(glob[,opts, cb])

#### glob

类型：`String` or `Array`

单一通配符或者一组通配符指明那些文件需要监控变化。

_A single glob or array of globs that indicate which files to watch for changes._

#### opts

类型：`Object`

传递给[`gaze`](https://github.com/shama/gaze)的配置

_Options, that are passed to [`gaze`](https://github.com/shama/gaze)._

#### cb(event)

类型：`Function`

每次文件变化所执行的回调

```
gulp.watch('js/**/*.js', function(event) {
  console.log('File '+event.path+' was '+event.type+', running tasks...');
});
```

回调将会被传递一个描述了此次变化的`event`对象：

_The callback will be passed an `object`, `event`, that describes the change:_

##### event.type

类型：`String`

发生变化的类型，`added`, `changed` 或者 `deleted`.

_The type of change that occurred, either `added`, `changed` or `deleted`._

##### event.path

触发事件的文件路径.

_The path to the file that triggered the event._