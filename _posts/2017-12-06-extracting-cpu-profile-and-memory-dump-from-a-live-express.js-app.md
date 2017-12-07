---
layout: post
date:   2017-10-11 09:00:00
title: "Extracting cpu profile and memory dump from a live Express.js app"
tags: [ express.js, NodeJs, cpu, performance]
image: "/images/2017/12/is-api-memory.png"
published: true
comments: true
---

Still in the same vibe of [How I solved a memory leak in a NodeJS worker](/2017/10/11/memoryleak-in-node-js-q-defer.html), I’m going to describe how I created an entrypoint on Express.js to extract cpu profile logs and heap dump using [v8-profiler](https://github.com/node-inspector/v8-profiler).

The first problem that I’ve found was a segfault when trying to generate the heap dumps, that I promptly noticed in the issue tracker that it is a compatibility bug related to node 8.x. Following a user's suggestion, I replaced the v8-profile by v8-profile-node8 for now. Inspite of the horrible documentation and random v8-profiler is pretty handy.

Here is my code. Enjoy:

```ts
import * as profiler from 'v8-profiler-node8';
if (process.env.ENABLE_PROFILE) {

  router.get('/admin/memory_profile', (_req, res) => {
    var snapshot = profiler.takeSnapshot();
    let contentDisposition = `attachment; filename="${process.pid}-${Date.now()}.heapsnapshot"`
    res.setHeader('Content-type', 'application/pdf');
    res.setHeader('Content-disposition', contentDisposition);
    snapshot.export().pipe(res).on('finish', () => {
      console.log('finished sending dump')
      snapshot.delete()
    })
  })

    // Only allow one capture per thread
  let capturingCpuProfile = false;
  router.get('/admin/cpu_profile', (_req, res) => {
    if (capturingCpuProfile) {
        return res.status(400).send('CPU profile being captured')
    }
    capturingCpuProfile = true

    profiler.startProfiling();

    // Let it capture the cpu log for 30 seconds and then downloads the file
    setTimeout(() => {
      capturingCpuProfile = false
      const profile = profiler.stopProfiling();

      let contentDisposition = `attachment; filename="${process.pid}-${Date.now()}.cpuprofile"`
      res.setHeader('Content-type', 'application/pdf');
      res.setHeader('Content-disposition', contentDisposition);

      profile.export().pipe(res)
      .on('finish', function() {
        profile.delete();
      });
    }, 30000)
  })
}
```

I used that `ENABLE_PROFILE` environment variable because I dont want to let it available on production as default. It's a good idea to prevent unauthorized access too.

After downloading the files, just load them on chrome dev tools. For memory leak debugging, take a look at my [previous post](/2017/10/11/memoryleak-in-node-js-q-defer.html).

## Caveats

- Generating a memory dump uses a lot of memory, so make sure that the process is using less than half of the available memory.
- The memory dump file must have the extension `.heapdump`
- The cpu profile log must have the extension `.cpuprofile`

![Memory consumption](/images/2017/12/is-api-memory.png)
