#mak

A sophisticated build api for javascript and typescript

Progress: I have to do other things right now but here's the idea. I did [a graph API for javascript](https://github.com/makoConstruct/Gnods) which we can use though



###concept

Mak is inspired by gulp, but replaces streams with graph nodes representing stages. By using graph analysis instead of simple, autonomous streams, mak is able to centralize planning and control over its bodily systems, ascending to unprecedented levels of electronic intelligence.

Here is a depiction of mak that will be at once familiar and alien to gulp users:

```typescript
import {mak, write, dir, file} from 'mak'
import {tsc} from 'mak-ts'
import {unmodulizer} from 'mak-unmodulizer'

mak.task('project', ['pack'], (pack)=> write(pack, dir('assets/')) )

mak.task('pack', ['compile typescript'], (tscresults)=> unmodulizer(tscresults) )

mak.task('compile typescript', ()=>
	tsc({
		preserveConstEnums: true,
		target: 'ES5',
		sourceMap: true,
		module: 'commonjs',
		inputFiles: file('mem.ts')
	})
)
```

As you can see, the nerves cross inter-task barriers. There is a reason for this, but I feel the tasks are getting in the way here. Let's phrase this differently so that you can see the structure of the build more clearly.

```typescript
import {mak, write, dir, file} from 'mak'
import {tsc} from 'mak-ts'
import {unmodulizer} from 'mak-unmodulizer'

mak.task('project', ()=>
	write(
		unmodulizer(
			tsc({
				preserveConstEnums: true,
				target: 'ES5',
				sourceMap: true,
				module: 'commonjs',
				inputFiles: file('mem.ts')
			})
		),
		dir('assets/')
	)
}
```

The role of the task is not to run the compilation stages, but to describe them. The overarching system then decides which parts of it need to run. It is supplied with a lot of information.

You will also note that there is no `.pipe()`. `.pipe()` is actually stupid, as it assumes that each task will only take one input, or that the meaning of the input should always be the same in each case, or obvious. Data flow systems don't really work that way. In mak, each of a task's inputs are expected to be named and documented. It turns out, if we dig, we find that we already have a very robust syntax that allows specifying processes that take multiple inputs, it was invented a long time ago and is already ubiquitous in modern languages. The terminology preferred by the mak project for processes that take one or more inputs is "mergy box", but the standard term is "function", and we will use this in documentation instead, to avoid unnecessary obfuscation.

A rundown of what happens here:

* `mak.file` resolves a data source ./mem.ts as an input. The node notes the file's modification date.

* `tsc` composes a task node that relates inputFiles (mem.ts) as one of its dependencies. The system will decides that tsc's task node inherits its recency from that of the `mak.file` node, which is the modification date of the file.

* `unmodulizer` creates a task node that, again, links to the node before it as a dependency. `unmodulizer` is for packing/browserification btw. I was thinking of listing a 'webpack' node but I'm not sure how compatible webpack is with this technology, and I'm fairly sure it's not compatible conceptually.

* `mak.dir` returns an end node, then `mak.write` wires the result of `unmodulizer` to it.

* the `mak.task` method now receives its completed task graph. It analyses it (this is simple enough. It just has to ???????? until all nodes have been ???????'d.), and by comparing the dependencies of the `unmodulizer` branch to the contents of the `assets/` dir, is able to compare their modification dates with the freshness of the data that would be produced by the `unmodulizer` function. If the files in the dir are fresher, it knows that there is no need to actually run the `unmodulizer` branch and reproduce the data, as it would be no different than what is already there. Otherwise, if the files in the dir are older, then the tasks run, each one pulling data from the last. There need be no intermediary files, as the metadata carried along with the compilation stages can be extensive enough for ~~mergy boxes~~ functions like webpack to know what to do with it.

```
> mak project
Dest files more recent than source files. Nothing to do.
Build successful
```
