###### I have a monolith application with rails and react, but it is not using react_on_rails library. The application is running fine with the puma server. Here we are not using Cruby, we are using Jruby 9.2.18.


The problem I faced when I was trying to create the war file.
**What I saw that the size of the war file is 500mb :(. I move inside the war file and found the size of my JS is more than 800mb.
**


I tried to do 

    rails assets:precompile.


On your Rails project especially on the assets:precompile step, or maybe sometimes you see the following error during the assets precompilation:
```FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory```


The solution is to run the command to increase thr memory:
```
export NODE_OPTIONS=--max_old_space_size=4096
```


Here I have increased it to 4 GB. But it took huge time, around 20 mins to execute without any error and with some warnings.  Still there I did no find any improvement in the JS size. Here the new problem is it is taking too long to compile.

When you run rails assets:precompile, it does 2 operations
precompiling the assets using sprockets and second is precompiling the webpacks. If you visit the public folder of your application, you will see 2 new folders named assets and packs.
And the size of the packs folder is larger.

If you want to just get rid of webpacker, run
```
WEBPACKER_PRECOMPILE=false rake assets:precompile
```

This will generate the assets folder under public folder. It should not take time, for my case it executes withing 15 seconds. But the webpacker is not precompiled yet.

So we need to run webpacker stage 
```
rake webpacker:compile
```

There is a way to see the details of webpacker compile command, in order to do that we should use webpack itself instead of webpacker. So, internally it is running the webpack command like this:
```
./bin/webpack --progress
```
This time it executes the operation within 1 min. The folder structure for a react component is like:

```
public/packs/js/component/Alert/
├── Alert-43fcc65488165001f5e0.js
├── Alert-43fcc65488165001f5e0.js.map
├── index-456121eb199bb27d221e.js
└── index-456121eb199bb27d221e.js.map
```

*There should be a time/memory consuming step to generate source-map files. Source-map files help to map a combined/minified file back to an unbuilt state. If you don’t need source-map files on the production server, you can skip generating these files by setting devtool for the webpack/webpacker.
*
I have added the 2 lines in the config/webpack/environment.js
```
...
const devtool = process.env.DEVTOOL;
if (devtool) environment.config.merge({ devtool });
...
module.exports = environment; // this line already exists

```

Then test the implemented code:
```
 DEVTOOL=none ./bin/webpack --progress
```
You should not see the source-map generating step anymore. There is a drastic change in the JS packs folder also. NOw it reduces to around 350mb.
```
public/packs/js/component/Alert/
├── Alert-43fcc65488165001f5e0.js
└── index-456121eb199bb27d221e.js
```

**The size of the war file reduced to 130mb now and it s working. My gutfilling is that it can be reduced more after passing the environment mode in webpacker. I will update here once it is tested.**
