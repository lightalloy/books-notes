# Complete Guide to Rails Performance
## Principles and Tools

*Benchmarking* - comparing small pieces of code in the non-real world situations.
*Profiling* - examining the performance characteristics of an entire, real-world application.

micro-benchmarking - `bechmark-ips`
macro-benchmarking - just unix `time`

> *I will not optimize anything in my application until my metrics tell
me so*

## The Business Case for Performance
> TRAC Research estimates that organizations are losing twice as much revenue from slowdowns as they compared to availability

Every new feature almost always implies the execution of additional code.
Every feature imposes some kind of performance cost. We should not ignore, but quantify them.
E.g. tracking codes should not be added w/o a thought.

> For large (Fortune 500) companies, 1 second in load time equals a 2% change in conversion rate. For medium to small size organizations, 1 second equals a 7-10% change in conversions.

#### Longer Load Times Turn Away Users
Load times have a huge impact on these metrics as well.

#### Mobile and Global Users are Hardest-Hit
Page weight targets for mobile users will need to be about 3x smaller than those for home broadband users.
Large pages are unusable with slow network connections.
> YouTube saw massive increases in traffic from South America, Asia and Africa after reducing page weight by 90% from 1.2MB to 100KB.

#### Bandwidth is an Expense
#### Fast Sites are Cheap Sites

### How To Create a Performance Culture
- Quantify performance costs in dollars, not seconds
- Set a front-end load time budget
Webpagetest.org => take the average of 5 runs
set a budget for several metrics - `DOMContentLoaded` and `window.load`
('Document Complete', 'Fully Loaded', 'start render time')
As a team, agree that if your site exceeds these numbers, you’ll file a bug.
- Set a maximum average response time and/or a maximum 95th percentile response time for your server responses. Agree on that number as a team.
- Set a page weight budget
- Quantify integration costs
Using the studies above, quantify the dollar value or traffic value (in visits/month) of a second of load time to your organization. Use the number to evaluate 3-rd party integrations.
- Add automated performance and page weight tests to your CI.

## Performance Testing
"automated full-stack performance testing" for Ruby webapps => almost nothing!

### Why test performance?

### When should I write a performance test?
Start with the homepage (or another page your users start).
Fullstack performance test.
Other places to test: hot loops, algorithms that'll be executed a lot.

### The benchmarks folder
Common pattern - have a `benchmarks` folder.
Works both for libraries and full apps.
They're not tests but tools for devs to measure the impact of the changes to the code.
`Minitest::Benchmarks` can be a useful tool.

### Performance acceptance tests
Fullstack integration performance tests.
No magic gem!
Set a pass/fail standart according to your CI setup.
Run separately from unit tests.
Hard pass/fail can sometimes be inappropriate.

Database matters! Data should be a copy of production or the same size as production.

### 3rd-Party Services: Blazemeter and Loader.io
### Checklist for Your App
- Run a benchmark on your site locally using
- Set up a performance monitor/tester for your application. Ensure it either runs continiously or after every deploy.

### Lab
Performance testing with Apache Bench (`ab`)
`ab -c 10 -t 60 http://127.0.0.1:3000/`
10 concurrent users, 60 seconds
Compare webrick, thin and puma.

## Profiling with Ruby-Prof, Stackprof and GPerftools
Profilers like Skylight or NewRelic are useful cause they work in production, but they are non-interactive and live in the past.

### Profiling and Benchmarking - A Workflow
- Explore metrics for problems
- Establish a reasonable benchmark
- Profile, and figure out where the time goes
- Experiment with and benchmark alternatives

### When Profilers Lie - Choosing a Profiler Mode
#### CPU - clock counter
modern CPUs do a lot of stepping - they change their clock speed based on the load they're under.
The `sleep` will slow the program, but won't affect the clock counter results.

Use CPU time when you’re interested in seeing the profile without I/O.
Don’t use CPU time when you need really accurate results

#### Wall time
Affected by:
- Other processes (resource contention)
- Network or I/O conditions
Despite its flaws, wall time is usually the mode you’ll want to use.
Don’t use wall time when lots of I/O is involved and is highly variable.

#### Process time
> Process time measures the time used by a process between any two moments. It
is unaffected by other processes concurrently running on the system.

Process time doesn’t include time spent in child processes (`fork`, `spawn` are out).

Process time, if available, is usually a better choice over CPU time.

#### Tracing
Try to measure every method invocation and how long they take, and then aggregate that data across your entire profiling run.
Extremely accurate, but has hight overhead, so is not suitable for production.

#### Sampling
By just randomly “CTRL-C”-ing during program execution and looking at the call stack, congratulations, you’re sampling!
With this method you're taking extremely small sample.
Sampling profilers sample hundreds of times per second, giving us much higher resolution and accuracy than “just halt it!” method.

“numerator” - number
of times this method/line-of-code appeared when we randomly sampled the stack.

denominator - CPU time or wall time, with all the caveats of the above.

In general, use sampling profilers for production, and use aggregating/tracing
profilers for development.

### ruby-prof
ruby-prof works by hooking into MRI directly.
Every time you call a method or something happens in the Ruby VM, ruby-prof gets called and measures how long it took the CPU to do that thing.
The program will run 2-3x slower, so it's hard to use in production.

`ruby-prof` is good for the development.
It'll help you when you need to profile some ruby code outside of a Rack scenario.

#### Quick and dirty profiling
benchmark-ips + ruby-prof

#### Additional profiler modes
Like memory allocation, but these parts of ruby-prof are broken and unmaintained.

### stackprof
Used under the hood by rack-mini-profiler.
It samples, not aggregates, so it can be used for production.
Supports wall and cpu timing modes.
`ruby-prof` usually works more accurately and has most of the features of `stackprof` in the dev environment.

#### gperftools/perftools.rb
- a Ruby front-end for Google's gperftools.
Similar to stackprof, but has great graphic reports.

#### Checklist for Your App
- Use a profiler like `ruby-prof` to diagnose your application's startup time
- Find an algorithm or other "hot" Ruby code in your app. Use a combination of benchmarks to determine which implementation is better.

### Lab: Profiling
Profile a benchmark suite of a `dalli` gem
`ruby-prof test/benchmark_test.rb`

## Profiling Ruby Memory Usage
### ObjectSpace and objspace.so
`ObjectSpace.count_objects`
Returns counts of Ruby primitive types.
You can stop and start gc objects:
```
GC.disable #=> true, GC is now disabled
GC.enable #=> true, GC is now enabled.
GC.start #=> garbage collect RIGHT NOW
```

You can iterate through every live object:
```ruby
puts ObjectSpace.each_object.count #=> 42552
puts ObjectSpace.each_object(Numeric).count #=> 7
puts ObjectSpace.each_object(Complex).count #=> 1
ObjectSpace.each_object(Complex) { |c| puts c } #=> 0+1i
```

Print all active objects by class:
```ruby
ObjectSpace.each_object.
map(&:class).
each_with_object(Hash.new(0)) { |e, h| h[e] += 1 }.
sort_by { |k,v| v }
```
"objspace" is not meant to be used in production.
`ObjectSpace.count_objects_size` - shows you, in bytes, how much memory each type of object is using.  
`ObjectSpace.memsize_of` - shows how much memory each type of object is using.  
`ObjectSpace.memsize_of_all(String)` - get the total memory size of a certain class of objects.

Use for: Play and enhance your knowledge of ruby memory allocation and gc. Explore live objects in your app.

### GC::Profiler
`GC.count` - the number of times the GC has run since the process started, minor + major runs (minor - check only young objects, major - check young and old objects)
`GC.stat` - outputs a detailed hash with some details on garbage collection.

If `old_objects` is gradually increasing over time, it could be a memory leak.

```ruby
GC::Profiler.enable
require 'set'
GC.start
GC::Profiler.report
GC::Profiler.disable
```
"invokes" == `GC.count`

Use for: one of the tools says that gc takes a long time.
### gc_tracer
- extension for `GC::Profiler`
Usage:
- `gem 'gc_tracer', require: 'rack/gc_tracer'`
- insert middleware (to `config.ru`)
`use Rack::GCTracerMiddleware, view_page_path: '/gc_tracer', filename: 'log/gc_tracer'`
- `http://localhost:3000/gc_tracer` - highly detailed information in a tabular format.
Most of the columns is the `GC.stat` log.
You can use a block format `GC::Tracer.start_logging { ... }`, e.g. to get GC information about background jobs.

Use: in development, to get a constant log of garbage collections. Useful for tracking memory leaks.

### derailed_benchmarks
-- a complete benchmarking suite for Rails.
Useful for tracking down memory bloat.
`bundle exec derailed bundle:mem` - checks gems' memory usage.
(it's a static benchmark)

Dynamic benchmarks:
- `derailed exec perf:mem_over_time` - hits your app a bunch of times and outputs total process memory. Increased number => memory leak.
- `derailed exec perf:objects` - hits your app and looks to see where objects are created.
`PATH_TO_HIT=/signup TEST_COUNT=10 derailed exec perf:objects`

Use for: auditing Gemfiles and reducing bloat, find memory leaks.

### memory_profiler
used under-the-hood by `derailed` and `rack-mini-profiler`
Can be used to profile a block of code.
```ruby
require 'memory_profiler'
report = MemoryProfiler.report do
# run your code here
end
report.pretty_print
```
Reports the total amount of memory allocated and retained while it was run.
Retained memory - memory that is used by objects that have survived the garbage collection.

`memory_profiler` works well with C-extensions.

Use for: debugging memory usage of background jobs or other non-Rack-app scenarios.

Checklist for Your App:
- an audit with `derailed-benchmarks`, Substitute or eliminate bloated dependencies.
- use `rack-mini-profiler` + `memory-profiler`


## Rack mini profiler
- a a performance tool for Rack applications, maintained by the
Most important tool for the ruby webapps.

You can use it in production.

```
gem 'rack-mini-profiler'
gem 'flamegraph'
gem 'stackprof'
gem 'memory_profiler'
```

`flamegraph` - super-pretty flame graphs

### The Speed Badge
You'll see:
- how many sql queries
- total request time (nice to see < 50ms) (domcontentloaded + load event)
- % spent in sql
- How long until DOMContentLoaded fires? - between receiving a response and finishing loading all the content. Find if the frontend optimization is needed.
- Are any of the parts of the page taking up an extreme amount of time compared to others (e.g. some partial)

### The Flamegraph
The legend - percentage of the time the request spent inside that stack frame.
Add `pp=flamegraph` to the url.

### GC Profiling
- a set of tools for debugging memory issues live on production.
add `pp=profile-gc` to the url.
You'll get the output of `GC.stat`

Pay attention to any requests that generate abnormally high values here (10+ MB allocated per request, for example).

*ObjectSpace stats*
- look at the numbers of allocated app-specific objects, e.g. 2,000 `Paperclip::Attachment` may be a red flag.

String stats:

### profile-memory
`pp=profile-memory`
Look at allocated and retained (survived the gc) memory amounts.
retained - you may find object leak

Look at allocated/retained by gem.

### Exception Tracing
Exception in ruby is slow. Don't use them for control flow.

### Checklist for Your App
- Set up `rack-mini-profiler` to run in production
- run app in production env locally
- look at the sql queries (same table queries, many queries)
- use `trace-exceptions` to make sure you aren't silently raising and catching any exceptions.


## Little's Law
l = lambda*w
lambda -- the average web request arrival rate (e.g. 1000 requests/second)
w - the average response time of your application in seconds

*The application instance* is the atomic unit of your setup.
Threaded puma => each puma process in an app instance.
Threaded puma + jruby => each thread is an app instance.

Little's Law is only true in the long run.

When to scale: use Little's law. If you're at 25% or less from maximum, you're scaling prematurely.
Spending a large amount of time per-request in the request queue => time to scale.

### Checklist for Your App
Ensure your application instances conform to a reasonable ratio of what Little's law says.
No controller endpoint's average response time should be more than 4 times the overall application's average response time.

Maximize your actual throughput if requests are as close to the median as possible. Push jobs to the background processes (sidekiq, dj).

## Performance Monitoring with New Relic
Transaction - a single response from a controller action.
Real-User Monitoring (also RUM and Browser monitoring) - newrelic will insert some Javascript for you on every page.
Events set include domContentLoaded, domfomplete, requestStart and responseEnd.

Times:
App server avg response time:
< 100ms - fast.  
< 300ms - avg.  
'> 300ms - slow  

Browser avg load time:
< 3 sec - fast  
< 6 sec - Average  
''> 6 sec - Slow  

Graph - figure out how much time goes to what part of the stack.
Typically most of its time is spent in Ruby.

If there's a lot of time in "web external" - there's a controller or view that's waiting, synchronously, on an external API
A lot of time in request queueing => more servers.

Requests per minute/scale.
< 10 => 1 server
10 - 1000 => avg
`> 1000 => High

### Transactions
In the transactions tab, sort by most time consuming.
You can concentrate on just the top 5 or 10 slowest transactions.

### Database
Lots of time in #find, Pay attention to the "time consumption by
caller", search for N+1
SQL - OTHER - don't worry, rails stuff.

### External Services
Make sure that there aren't any external services being pinged during a request
Most of them are not necessary.
Alternative - create a background worker which runs every 5 minutes.

Aggressive timeout or consider coding up a Circuit Breaker.

### Checklist for Your App
- You should be using a performance monitor in production - NewRelic, Skylight, Scout or AppNeta.
- Figure out where your application sits in my performance categories on the frontend and app server
- Use newrelic to find performance issues.

## Skylight
Focus on the catching the worst performance issues instead of the averages.
- Skylight uses logarithmic scales everywhere
- Skylight focuses on 95th percentile times
- Instead of sampling, Skylight aggregates

Only provides information for the 6 hours.

# Module 2: Front-End Optimization
## Chrome Performance panel, Your Front-end Profiler
- server responds with html
- html, css, js are parsed
- html is parsed, rendered and painted
- js is executed

Uncompressed size is important for figuring out how long it will take the client to parse these resources and construct the page.

It records page interactions like VCR (you start/pause).

### Receiving the HTML
1st chunk - server response time + network latency.
Events - receive response, receive data

### Parse HTML
converted -> tokenized -> lexed -> constructed
When a browser sees a js tag, it'll block until the js is loaded and evaluated, cause js can modify the DOM.
With `async` attribute this doesn't happen.
Browsers will not wait on external CSS before continuing past this step.
"Parse HTML" step will reoccur every time the browser has to read new
HTML, e.g. on ajax requests.

### Recalculate Styles
As HTML is to the DOM, so CSS is to the CSSOM.
Css is downloaded and converted -> tokenized -> lexed -> constructed.
First the browser applies the default styles and the html `style` attributes.
Then when the css is loaded, the browser applies it.
These are: `Recalculate Style`  events.

### Layout
This should happen when the browser has all of the DOM and CSSOM in memory.
If you're seeing a lot of "layout" events during a page load, you may be experiencing "layout thrashing".
Usually caused by caused by Javascript messing with the DOM, or using multiple stylesheets.

In the "Layout" step, then, the browser is just calculating what's visible, what isn't, and where it should go on the page.

### DomContentLoaded
This event occurs when the browser has done parsing html and running blocking (non-async) js.
Speed up `DomContentLoaded`:
- make script tags async
- use less complex html markup
- avoid layout thrash (use only one stylesheet!)
- use inline styles in moderation

### Paint
happen when the browser is done rendering and needs to turn the layout into pixels on a screen.

### Parse Author Style Sheet
Waiting for css to download, the CSSOM is re-calculated, we re-render the layout.
Most sites won't benefit from the optimization (inlining css in the page)

### Javascript
the Javascript finish downloading, the js is being parsed and evaluated.
After it: recalculate style again, paint events.
The `load` event fire off. Several events are typically attached to it.
End of the `load` => the page is ready.

### Using Performance Panel to Debug Browser Speed
1) Hard reload & get the fresh data
2) Look at the pie graph and find out what takes a lot of time
3) Reduce Idle - slow server responses, unoptimized assets
4) Reduce loading - less amount of html and css
5) Reduce scripting - (async marketing) scripts, less scripts.
6) Reduce painting - this is hard, may cause a flash of unstyled content.

### Checklist for Your App
- have only 1 remote js and 1 remote css
- use async and defer attributes on every script, you may need to add marketing scripts to your own server and load them in application.js
- minimize js amount, e.g. marketing scripts
- (external) css goes before (external) js
- `$(document).ready` - remove event handlers from it where possible or use a solution that reuses the pages, like turbolinks or single-page-app approach. Maybe you can attach your handlers to `DomContentLoaded` instead?

## The Optimal Head Tag
- Specify content encoding with HTTP headers where possible.
- viewports go after encodings, before the stylesheets
- css goes before js (cause js may block css loading)
- concatenate your css and js into 1 css and 1 js file
- add `async` and `defer` to js, which is not required for the page to render (e.g. marketing stuff)
- manage fonts with css classes (setting wf-loading, wf-active), or the `font-display` css property.

## Resource Hints and Fighting Page Weight
Loading the page:
1) open the connection, do the DNS/TCP/SSL setup.
2) download the html
3) parse the document. If meeting a subresource, open a connection and download it. If the sub-resource is and external script, the parser stops, downloads it, executes, and then moves on.
4) When the parser stops to download an external document, it sends ahead preloader, which will search and download resources if it understands how.

Sub-resource - external js, css, images and more.

### Letting the Preloader do its Job
The parser has to wait for the external script to be loaded and executed, cause it can potentially modify the document, like erase it with `document.write()`.  
Help the preloader:
- Stop inserting scripts with "async" script-injection

Doesn't work with preloaders: iframes, @import (don't use in production), webfonts (use resource hints), html5 audio video (use resource hints)

### HTTP caching
It's often better to concatenate library you use (e.g. jquery) with your js, than use a cdn version.

### Preconnect
`<link rel="preconnect" href="//example.com">`
External scripts loaded with the async injection will need to resolve dns, open a connection, negotiate ssl.
By adding a preconnect tag, we can move this work to the beginning of the page load.
If you change the injection with the simple tag with the async attribute, the browser preloader will pick and download it as soon as possible.

`preconnect` works best with sub resources that are script-injected and for script-injected resources with dynamic URLs.

### Prefetch
`<link rel="prefetch" href="//example.com/some-image.gif">`

Consider using prefetch in any case where you have a good idea what the user might do next.
E.g. image gallery - prefetch the next picture.
You can prefetch the entire pages.

### Prerender
Prerender renders the entire page.
Only use prerender and prefetch where you can be pretty certain a user will actually use those resources on the next navigation.
Prerenders have lower priority and aren't always executed.

### Checklist for the app
- reduce the number of connections
- don't rely on any particular resource being cached (like 3-rd party cdn)
- Use resource hints - especially `preconnect` and `prefetch`

## Turbolinks
Work similar to "Javascript single-page app" paradigm: no full page loads, pushState usage, and AJAX.

pjax - fetches HTML from the server and replaces the content of the container element with this html.
turbolinks - similar but replaces all html.

When using Turbolinks, you don't throw away your entire Javascript runtime on every page.
And don't have to parse and tokenize the CSS and JS ever again, cssom is maintained.

idempotent functions + dom hooks are not a junk drawer

### load is dead, all hail load!
`$(document).ready(function () { ... } );`
You have to use other ways to attach events (jquery `.on` or turbolinks ways)

### Non-RESTful redirects
re-render the updated view instead of redirect.

### Common mistakes
- check if turbolinks is really enabled
- don't append erb pieces, render the whole page for the turbolinks

### Limitations and caveats
Turbolinks doesn't play great with client side JS framework.
Integration testing is a pain.

## WebFonts

### Changing to Google Fonts

## HTTP/2
### HTTP/2 Changes That Benefit Rubyists
- Header Compression
- Multiplexing
  - sending multiple responses to our client over a single connection at the same time.
  - For now you can use an HTTP/2 compatible CDN for serving assets.
- Stream Prioritization
  e.g. ask to load js and css before images.
- Latency Reduction

### How Rails Apps Will Change with HTTP/2
HTTP/2 makes all requests and responses cheaper.
- development mode will get faster
- experiment with more granular http-caching. e.g. separate high-churn files
- experiment with separating js and css for different pages

### How to Take Advantage of HTTP/2 Today
- Move your assets to a HTTP/2 enabled CDN
- Use an HTTP/2 enabled proxy, like NGINX or h20.

## JavaScript
### Proper script tag usage
We can't execute application.js until application.css is downloaded and executed (but can download application.js)

### Don't script inject
Script injection prevents the preloader from doing its job.

### Async defer all the things
`async` executes the script as soon as it is downloaded, `defer` executes it after `DOMContentLoaded` fires

- Use defer for advertisements and other JavaScript that can wait to execute.
- Use async for Javascript that should be executed as soon as possible, like
analytics or user tracking.
- Use neither for Javascript that is required for the page to render properly

- Scripts at the bottom is mostly outdated and a workaround
- Combine scripts (concatenate)

### Common mistakes
- cache DOM lookups
- Minimize DOM size (less DOM elements)
- use SPA framework or Turbolinks

### Don't drop too much in the $(document).ready()
It's better to use event delegators:
```js
$("#myID").on("click", function() {...}); // this version is not async-friendly!
$(document).on("click", "#myID", function() {...});
```

## HTTP Caching
Most useful for:
- static assets
- json apis and ajax endpoints

`Cache-Control: public, max-age=31536000`
public - the resource can be cached by edge or intermediate caches (e.g. cdns)
max-age - tells the browser how long to store the resource, in seconds from now


If the cache is expired, browser will send a new request with `If-Modified-Since` header.
If the resource was modified, the server will send the new resource, if not - it'll respond with 304 Not Modified.

Another way - Entity Tags (ETags). If the resource changes, the ETag changes. Browser will send `If-None-Match:` header.

### Cache-Control directives
- no-store - prevents all caching anywhere (e.g. sensitive information)
- no-cache - will store, but will always revalidate the response from the server.
- public / private: private will prevent from storing on CDN's and intermediates
- max-age - how long to store the resource, in seconds from now
- no-transform - prevent converting content (like images) on the CDN's
- must-revalidate - forces the cache to obey the expiration times in the Cache-Control header

### HTTP Caching and Assets
- serve static files from the /public directory
- use CDN that uses your Rails server as origin

### HTTP Caching and JSON APIs
#### To prevent db queries
Rails has `stale?` and `fresh_when` methods to check.
Use `E-tag` or `Last-Modified` to determine what queries need to be executed.

#### Public and private - controlling cache copies
Rails marks all HTTP cacheable resources in a controller as `private` by default.
Use `no-store` for private, and specify `public` for responses to be cached.

### minimizing churn
Sometimes it's best to break the "one-file" rule. E.g. when one of your files is stagnant (e.g. bootstrap) and the other one has high churn.
Be careful, try to serve library files through http/2 proxy.

# Module 3: Ruby Optimization
RSS (Resident Set Size) - physical RAM pages used by the process (including shared memory)
PSS (Proportional Set Size) - same as RSS, but counts memory pages differently, typically smaller then RSS.

## Memory bloat
### Memory usage by ruby processes
`ps | grep '[r]uby' | awk '{print $1}' | xargs ps -o rss,vsz,nswap`

`GC.stat[:heap_free_slots]`
Large amounts of free heap slots means that something is allocating huge amounts of memory which never gets released.

Temporary large allocations can be caused by:
- opening large files
- large queries
- large webserver responses or requests
- complicated views

If you can't remove/refactor these situations, try streaming approach. E.g. reading a file line by line.

### Oink
Track down controller actions allocating large amounts of memory.

### Gemfile Auditing
`derailed_benchmarks`
`bundle exec derailed bundle:mem`

### jemalloc
Ruby's calls to its memory allocator are abstracted, so you can use any compatible memory allocator.
By default glib'c `malloc` is used.
Use jemalloc - build ruby `--with-jemalloc`

## Memory Leaks
Types of leaks:
- Managed Ruby object leaks (ruby code that leaks)
- C-extension leaks
- Leaks in Ruby itself (the VM)

Memory leak - the memory is allocated and never released. Leak increase memory usage slowly, and never stop growing,
Memory bloat - a process requires a large amount of memory and it's never released cause of continious use.
Bloat occurs quickly, and it eventually levels off.

### Back Off The Memory Pressure To See Clearly
Allowing your Ruby processes to run for at least 24 hours without a restart.
Use only 1 process per server, increase memory (temp), so that the process won't run out of memory.
If you're still seing memory growth after 24 hours, you probably have a memory leak.

### Tools We Can Use To Diagnose Leaks
Have a long-running production memory metric in production.
You should have metrics for at least a week.

### Reproduce Locally
Tool `siege` - the load testing utility.
`siege -c 32 -f urls.txt -t 5M` (urls.txt - the list of urls)

Count objects with GC.stat and ObjectSpace while `siege` is running.
- RSS
- `GC.stat[:heap_live_slots]` says that RSS is increasing.
- `GC.stat[:heap_free_slots]`
- `ObjectSpace.count_objects`

You can track these numbers in a memory-logging thread.

### Zeroing
### Ruby object leak
heap live slots increasing, heap free slots remaining low, RSS increasing => Ruby object leak
Use `memory_profiler` gem with `rack-mini-profiler`

### Known Leaky Gems
https://github.com/ASoftCo/leaky-gems

### Leak in C extension
heap live slots and heap free slots are remaining constant while RSS is increasing => leak in C extension

Hard to battle. Use Heap Dumps, jemalloc Introspection.

### Ruby VM leak
heap live slots and heap free slots are remaining constant while RSS is increasing => Ruby VM leak

### Giving Up: Worker-Killers
Last resort.
`puma_worker_killer`, `unicorn worker killer`
Restarting your application as soon as it starts to use too much memor

## Common ActiveRecord Pitfalls
`Stuff.all.find_each` instead of `Stuff.all.each`  
`in_batches`

### N+1s
`bullet` problems:
- misses a lot of n+1 queries - e.g. those from gems or engines; sometimes can't tell where the n+1 is.
- some of the n+1 are unavoidable, so there's a high chance you'll ignore the warnings
- eager loading is not always appropriate

It's nice to have production-like data on development + read the logs.
Sometimes it's beneficial to switch from the activerecod queries to ruby enumberable methods (need to benchmark).

ActiveRecord instance methods should not use query methods

Select only what you need.

`@posts.first.accessed_fields` in `after_action` will tell what fields were used.
Need array => use `pluck`

Do math in the database.
`average`, `calculate`, `sum`, etc

Use `update_all`, `delete_all`

## Background Jobs
- The action always takes more than your average response time to complete.
- The action contacts an external service over the network.
- The user does not care if the work is completed immediately

### Idempotency
An operation is idempotent if it returns the same result, if executed one or multiple times.

We always must assume that it's possible an enqueued job may be executed twice.

For some jobs you may need `around_perform` block + `with_lock`, e.g. sending email. This will prevent the operation executed twice if they are being executed at the same time?

Little's law:
> Number of workers = Average job execution time * Number of jobs enqueued/sec

Keep jobs small, not only by lines of code, but also by execution time.
Set timeouts aggressively (use those built into the library)

### Reliability
Understand your requirements.
Highly reliable solutions are slow, and extremely fast solutions are not 100% reliable.

### Background Job Processors
- Resque
- Sidekiq
- Sneakers
- Que
- DelayedJob

## Caching in Rails
Recommendations - 1 second "to glass" (from interaction to DOM finished painting)
That means the server response should be less than 300ms

### Profiling Performance
Benchmark before and after, `rack-mini-profiler`
See total time, time spent in views and ar time may be misleading.
Using flamegraph you can see where does every second go.

Benchmark in production mode (locally).

```
export RAILS_ENV=production
rake db:reset
rake assets:precompile
SECRET_KEY_BASE=test rails s
```

Set a goal (desired Maximum Average Response Time)

Use `ab` tool to test the response time.

### Caching techniques
#### Key-based expiration
The cache key changes when the object or template changes.
This technique doesn't actually expire any cache keys - it just leaves them unused.
Cache pushes out unused values when it's out of space.

You can put other objects into cache key:
```ruby
<% cache([current_user, todo]) do %>
... a whole lot of work here ...
<% end %>
```

#### Russian Doll Caching
When the 'inner' cache expires, we also want the outer cache to expire. If the outer
cache expires, though, we don't want to expire the inner caches.

#### Cache backends
##### FileStore
- cheap
- can be shared between processes, but not hosts
- Not LRU (expires by time, not by last recently used time)
- slow(ish)
- may crash Heroky dynos
Use when the load is not high.

##### Memory Store
- fast
- easy to setup
- increase RAM usage
- can't be shared between processes or hosts
Use when you have 1-2 servers, and store small amounts of cached data.

##### Memcache and dalli
##### Redis and redis-store
##### LRURedux

> When using a remote, distributed cache, figure out how long it actually takes to read from the cache

## Slimming down your framework
Rails performs poorly on many benchmarks, but seems to be really fast when used by some sites of the top 1000 (Github, Shopify, Basecamp, etc)

Rails takes several ms longer than rack to handle the request.
But the bottleneck in most end-user experiences on the web lies in the front-end, not the backend.

### Why slow?
Ruby runs more ruby code than other ruby frameworks to handle the request.

in `# rails/railties/lib/rails/all.rb` you can only require what you need, not everything.

#### Rails Isn't Slow - Middleware is Slow
`rake middleware` - get list of middlewares
You can delete middlewares in the `application.rb` like this:
`config.middleware.delete SomeMiddleware`

`Rack::Sendfile:` - used to serve files from responses, not supported by Heroku. Often can be removed.  
Many of the middlewares could be removed by the api-only apps, just read the documentation before and make sure you really don't use it.  

Use `config.api_only` for the api only apps.

Don't log to disk in production (log to STDOUT).

## Exceptions as Flow Control
If exceptions handling controls ordinary execution of your program, then you're using them as flow control.
Exceptions in ruby are slow, so use them only in exceptional circumstances.

Check if you need an exception?
- is this a failure?
- do you throw the exception away?
- can I use throw/catch instead? (it's faster)
- can I find a different http library that doesn't raise exceptions? (typhoues)

Most exceptions should trigger 500. If you return 200, you might not need an exception?

## Webserver Choice
If your app has requests queued and is waiting most of the time, then scaling dynos/hosts will help.  
Scaling increases throughput, not speed.

### How requests get routed to app servers
(heroku)
- load balancer
- heroku router, finds your application's dynos and pass on the request to a dyno; passes the request to the dyno's open TCP socket
Once connected, the socket on the dyno will accept the connection even if the webserver is busy processing other requests.

Attempting to connect to the server is the most critical stage, it differs from server to server.

#### Webrick
A single-process webserver. Will only work on 1 connection at a time, others will wait.
No protection from slow clients.

#### Thin
Thin is an event-driven, single-process web server.
Uses EventMachine.
Multi-threaded.
Thin can deal with slow client requests, but it can't deal with slow application responses or application I/O without a whole lot of custom coding.

#### Unicorn
A single-threaded, multi-process web server.
1 master process + spawned worker processes listening on a single socket.
Worker process accepts the request and waits while it's fully downloaded.
Then releases the socket and processes the request.
You can use NGINX in a custom setup to buffer requests to Unicorn.

#### Phusion Passenger 5
a hybrid model of I/O, multi-process, worker-based. Also includes a buffering reverse proxy.

#### Puma (threaded only)
A multi-threaded, single-process server.
Reactor thread downloads the request, can wait for slow requests asyncronously.
The reactore spawns a new thread for each request, and this thread communicates with your app and process the request.

Puma automatically yields control back to the process when an application thread waits on I/O.
Puma can deal with slow client requests, but it can't deal with slow, CPU-bound application responses.

#### Puma (clustered)
Combines multi-threaded + multi-process model.
In clustered mode puma can deal with both slow requests and slow app responses.

The biggest difference between Ruby application servers is not their speed, but their varying I/O models and characteristics.

#### Queue time
There's not a single queue.
Where the jobs can queue:
- load balancer
- heroku routers
- at the "master process"

### When do I scale app instances?
Scale based on the depth of your job queue.
If you’re not spending a lot of time (>5-10ms of your average server response time) in the request queue, scaling won't be helpful.


## Idiomatically Fast Ruby
Ruby usually has more than 1 way to do the same thing, and often one of them is much slower.
Often the same things are implemented differently under the hood.
But only optimize when you metrics tell you so.

E.g. `loop` is much slower than `while true`

Splatting arguments is slow when passing lots of arguments.
OpenStructs are much slower than Hashes

`Array#bsearch` is much faster than `Array#find(#detect)` for sorted arrays.

Array vs Sets - sets are faster on some operation, but you want to benchmark.
Sets are syntax sugar for Hashes.

`sample` is much faster than `shuffle.first`  
`flat_map` is much faster than `flatten.map`

`Range#cover?` much faster than `Range#include?`

Block arguments are slower than `yield`ing.

`String#tr` is faster than `gsub` (`tr` for single characters, `gsub` for regular expressions)

Where possible, use faster idioms.

## Memory Fragmentation
Ruby can't move the objects in memory, that's why fragmentation happens.
eden pages - pages with at least one live slot

fragmentation measure:
- the number of live slots / the number of slots in all eden pages
- `GC.stat[:heap_eden_pages]/GC.stat[:heap_sorted_length]`
a low percentage indicates heap-page-sized holes.

### Fix 1
Change `MALLOC_ARENA_MAX`. 2-4 is appropriate more most ruby apps.
Reducing the number will reduce memory usage, but has a negative impact to performance.

### Fix 2
Use jemalloc

### Fix 3
Compacting GC

## ActionCable
### Polling
Asking the server for new data every n seconds.

### Long Polling
Asking the server for new data, if there's no new data, open a persistend connection.
There're several techniques, which may involve weird hacks like hidden iframes.

### Server-sent Events (SSEs)
Rails - `ActionController::Live`
Don't work with IE; limited server choice, can't be on Heroku.

### Websockets
Websockets are a stateful connection between a client machine and a server.
Must be maintained by a single instance for the duration of connection.
Each Action Cable server instance listens to a Redis pubsub channel.
When a new message is published, the Action Cable server rebroadcasts that message to all connected clients.
All ActionCable servers are connected to the same redis instance, so everyone gets the data.

Full duplex - simultaneous communication. With websockets client and server can send messages at any time.

### Application Server Configuration

4 fundamental settings:
- Number of child processes
- Number of threads
- Copy-on-write
- Container size

#### Child process count
At least 3 processes per server or container.
That'll help to deal with requests which take a long time.
The maximum is constrained with memory and CPU resources.

Equation:
`(TOTAL_RAM / ( RAM_PER_PROCESS* 1.2))`

To measure ram per process, disable worker killers and wait 12-24 hours, measure with `ps`.
Meausure for 1 worker + 5 threads.

#### Thread count
it’s usually OK to just set 5 threads per application server process.
Be sure to check memory consumption before and after adding threads to the app.

#### Container size
Most Rails applications will need a server with at least 1 GB of RAM (~ 300 mb per process) or more.

### Tuning GC
GC tuning can help reduce certain kinds of memory bloat, and it can reduce
time spent in garbage collection.
The effect of GC tuning here is rather minor (1-5%), and there are better ways to fix the problem.
It's possible to decrease fragmentation.

The second problem we can alleviate with GC tuning is excessive time spent in GC. This will cause a tradeoff between
decreasing CPU time and increasing memory usage.

# Module 4: The Environment

## CDNs
The role of cdn:
- decreasing network latency.
  Cdns use "points of presence" - the content will be delivered from the closest to the user server.
- decrease load on your app
  the server won't spend time on serving assets requests
- cdns can perform modification of your static assets to make them more efficient
  (gzip, lazy loading)
- get some benefits of http/2


> Always using your Rails application as your CDN's origin reduces the different between production and development, making your life easier.

### Common mistakes
- using 3-rd party cdns
  this will lead to more requests, not optimal gzipping, load unneeded parts of the frameworks, and probably you won't get much caching benefits
- s3 is not a cdn
- marking assets as "Do Not Modify!" (do only when needed, e.g. medical images)

Cdn options: cloudfare, amazon cloudfront, azure, CacheFly


## Databases

Indexes:
- foreign keys
- primary keys
- polymorphic associations
- updated_at

`EXPLAIN ANALYZE` - check what is used

### Database Vacuuming
MVCC (multiversion concurrency control).
Instead of locking the db will copy the row and mark it as new data and the old data. When a transaction is completed, the old data is discarded.
But the bits of "old data" are sometimes left behind and not cleaned up properly.

Postgres `autovacuum` is not on by default.

Make sure you have enough concurrent connections available across your application.

For tests the db could be placed into RAM disc.

### JRuby
- JRuby uses more memory at boot, tends to use more memory in the simplest case, though it uses far less memory than MRI as it scales.
- jruby takes longer time to start up
- No access to C extensions (e.g. you can't use nokogiri, cause it uses libxml2)
You'll have to switch to jruby-friendly gems.

Advantages:
- faster after the warmup period
- true parallel threads
- mature garbage collection process
- portability
- access to java ecosystem

In your `Gemfile` you can specify gem groups for both `:ruby` and `jruby` like `platforms :jruby do .. end`

## Alternative Memory Allocators

Alternative - `glibc`, `jemalloc`, `tcmalloc`, `hoard`.
Changing memory allocators may have a small impact on total memory usage and performance.

## SSL
SSL - Secure Socket Layer
TLS - Transportation Layer Security

SSL negotiation:
- open a new tcp connection
- the client sends the list of cipher suites, and info about its ssl capabilities to the server
- the server picks a cipher from a list and sends its ssl certificate back to the client
- now the server and the client agreed upon a common cipher and ssl version, the client sends its key exchange parameters back to the server
- The server processes those key exchange parameters, verifies the message, and returns a 'Finished' message back to the client.
- The client decrypts the Finished message

So there're 2 complete round-trips.

SSL sessions could be cached (e.g. nginx config), but that may complicate load balancing.

## PGBouncer
Heroku has a 500 connection limit, which may easily be exceeded.

Each thread has its own database connection
Be default Rails sets pool size equal to `ENV['RAILS_MAX_THREADS']`, therefore for the threads count.
This problem is frequently solved with connection poolers such as `pgbouncer`.

`default_pool_size` - "the maximum number of server connections allowed"
We want it to be equal to the number of database connections that would need to be open simultaneously from this dyno.

If your app is spending lots of time in the `ActiveRecord::QueryCache` middleware, it means that threads are waiting for a database connection.
It means that you're running out of db connections.

pgbouncer can remove a lot of idle connections from your total connection count, since most of them are not being used at any point in time.

## Easy Mode Stack
CDN - Cloudflare  
Webserver - Nginx  
Application Server - Puma  
Host - Heroku  
Webfonts - Google Fonts  
Ruby Framework - Rails  
HTTP library - Typhoeus  
Database - Postgres  
Cache Backend - Redis  
Background Job Processor - sidekiq  
Perfomance Monitoring - Newrelic  
Performance Testing - Local, with siege, ab, or wrk.  
User Authentication - has_secure_password.  
Memory Allocator - jemalloc  
Ruby Implementation - CRuby  
View Templates - erb  


Questions:
Memory prof:
- what tools for memory profiling do you know?
- how to track memory leaks?
- What tools to use to explore ruby gc and memory allocation?
- What tool to use to audit your gemfile and reduce memory bloat?

- How many units (threads, processes) of the app do you need?
- What's a unit? When it's a thread and when a process?
- How do you determine when to scale?
- как сделать так, чтобы все запросы были близки к м,едианному, какие стандарты?

- When should an app spend most of the time?
- What controllers (newrelic transactions) should you optimize first?
- What to do with external services if an app spends a lot of time for them?

Optimizing front-end:
- What events take place to make the page displayed?
- How to speed up `DomContentLoaded`
- how to speed up things before load?

Optimal head tag:
- What should be the order of css-js-viewport?
- Where to add async-defer attributes?
- How to optimize head tag so that the page would make fewer http-requests?
- How to prevent style-flash (css, fonts)?

Resource hints & page weight
- Why the site is loaded slower than it should theoretically with this page weight and internet speed?
- why parallelizing connections won't help much?
- What's preloader?
- How to add ads scripts? Is it ok to do it via the script-injection?
- What doesn't work with preloader? What can help with it?
- Why is it better to add the library code to the app.js, than load it from a cdn?
- How to make a preloader do its job optimally?
- When to add preconnect?
- when to use dns-prefetch, prefetch, prerender?

Turbolinks:
- What's pjax?
- What's common between pjax and turbolinks?
- What's the difference between turbolinks and spa?
- what's the difference between loading the page w/o and with turbolinks/pjax?
- what's the difference in the load time?
- what rules do you have to consider when writing js while using turbolinks?
- What to do in turbolinks instead of `document.ready`?
- What to do with restful redirects in case of turbolinks?
- Is it ok to append erb pieces while using turbolinks?
- What are limitations of the turbolinks usage?

http/2
- Why it's hard to make rake compatible with http/2?
- Websockets usage when http/2 will be
- What benefits from http/2 will we get?
- What can be done for now to optimize assets loading?
- How will rails apps change with http/2?
- How can server push affect the client receiving assets?

Javascript:
- how to load js while css is being downloaded/executed?
- why shouldn't you use script injection?
- When to use async, defer or neither?
- Why putting scripts at the bottom is an outdated technique?
- What are common mistakes and how to avoid them?

http caching
- why are html responses uncacheable in rails?
- what are ways to validate if the resource has expired?
- what's the difference between using ETags and Last-modified headers?
- what's the difference between `no-store` and `no-cache` directives?
- when to specify cache-control directives explicitly?
- When it's ok to break the one-file (css-js) rule?

Memory bloat
- what's rss and pss, how to track them?
- what can be causing a temporarily large allocation in your app?
- how to track large allocations by your app?
- how can you solve the problem if you can't remove these from your app?
- How to analyze gems memory usage and reduce it?

Memory leaks:
- What's the difference between memory leak and memory bloat?
- Why is it hard to distinguish them in the app?
- How to tell if you have memory bloat or leak (experiment)?
- What tools can you use to detect what causes a memory leak?
- How to determine what type of leak is that?
- What's the last resort?

Common ActiveRecord Pitfalls:
- Why shouldn't you use things like `all.each`? What to use instead?
- What are `bullet` gem problems?
- What other optimizations do you know?

Background Jobs:
- When should a transaction move to the background?
- What operation is called idempotent?
- Why must we assume that the job could be executed twice (or even more)?
- How do you prevent sending email twice if the job is executed 2 times at the same time?
- How to determine number of workers?
- Why shouldn't you care about "exactly once" execution?
- How to care about failures in the background jobs?

Caching:
- What tools can you use to manage performance/response time?
- What env to test locally?
- What is your goal?
- What's the most complicated part of caching?
- How does key-based expiration work?
- When does key-based cache expire?
- How do Russian doll caches expire?
- How to choose the cache store to use?

Slimming down your framework:
- What's usually the bottleneck for the end-user experience?
- How to figure out what parts of the rails could be not loaded (instead of `rails/all`)?
- Why is rails slower than other ruby frameworks (in benchmarks)?
- How to find out what middlewares could be removed?

Exceptions as flow control:
- What's the difference between using exceptions as a control flow and handling exceptions?
- Should you raise an exception if the user mistype credit card information?
- How to check if you need to raise an exception?

Choosing a Webserver:
- When scaling dynos on heroku helps?
- Does scaling increase speed?
- What's the most critical stage of connection?
- Why Webrick works bad with slow clients?
- Which servers are vulnerable for slow processes and why?
- How to solve unicorn's slow request issue?
- What's puma benefits  regarding the slow I/O?
- Where do requests queue, what's queue time?
- What server would you choose for heroku and for your own instance?

Ruby idioms:
- Are sets faster than arrays or not?

Memory Fragmentation
- Though what layers is memory abstracted away from developers?
- How would a long string be created (in memory)?
- Why does memory fragmention happen in ruby?
- How to measure memory fragmentation?
- What are possible fixes for the fragmentation issue?

Websockets:
- What's the difference between polling and long-polling?
- What are disadvantages of the Server-sent events?
- How do websockets and ActionCable work?
- What's full duplex?
- How does ActionCable deal with increased number of connections?

App Server Config
- What are the 4 settings you can tweak to optimize your server configuration?
- Why shoud you have at least 3 processes per server or container?
- How to determine the maximum number of processes per server?
- What amount of RAM should you consider using on a server?

Tuning GC:
- What problems can you solve with GC tuning?
- Will the effect probably be minor or major?

CDNs:
- What's the role of CDN?
- how can cdns modify static assets to make them more efficient?
- how can you prevent the difference between production and dev env?
- why shouldn't you use 3-rd party cdn?
- Is s3 a cdn?
- what are cdn options?

Databases:
- Where will you most definitely need indexes?
- How do you check what indexes are used in a query?
- What's mvcc?
- Why do you need db vacuuming?
- Where to put db to speed up tests?

Jruby:
- What are the disadvantages of using jruby?
- Does jruby use more memory then mri in the simple cases or in the more complex ones?
- What advantages do you get by using jruby?

SSL
- What stands for ssl and tls?
- How does ssl work?
- Are there any ways to make ssl checks faster?

PgBouncer
- What problems do connection poolers solve?
- How to tell you're running out of connections?
