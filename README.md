bareos-libcloud
===

`bareos-libcloud` is a python-based plugin for Bareos.
It extends the base fonctionnalities with "object storage" support.

Based on Apache's libcloud, it supports many providers. The full list can be found here: https://libcloud.readthedocs.io/en/latest/storage/supported_providers.html
However, no test have been made for any providers, except Ceph's rgw. Please open an issue if needed.

Supported features
---
- Full backup and incremental backup are fully supported.
- Restore are supported locally, not in-place (you cannot restore some files to an s3 bucket, for instance). If you need such feature, please open an issue to discuss your use-case, and I will add it :)
- Accurate mode is supported too, based on mtime and size.
- ACLs, lifecycle configuration, additionnal metadata etc are **not** backupped.

Required packages
---

- python-libcloud (https://github.com/apache/libcloud).
- standard python-legacy installation (the plugin should run on Python3, Bareos seems not)
- standard Bareos installation (tested on 16.2.6)

*Important*: the last version of libcloud, v2.2.1, has a bug regarding object streaming (object is filled in memory, not really streamed, which is an issue for large object, as you will encounted memory overrun). It is fixed on the trunk branch.

Installation
---
Simply copy `*.py` to `/usr/lib/bareos/plugins`


Configuration
---

Basic configuration: backup bucket 'test' on a Ceph's RGW:
```
Plugin = "python:module_path=/usr/lib/bareos/plugins:module_name=bareos-fd-rgw:provider=S3_RGW:key=the_key:secret=the_secret:host=your_hostname:buckets_include=test"
```

Details:

`provider`: Which provider shall be used. This is a mandatory option, with no default value.  
Possible values can be found here : https://libcloud.readthedocs.io/en/latest/storage/supported_providers.html (see "Provider Constant")  
`buckets_include`: A comma-separated list of buckets to backup. If this option is specified, no other bucket will be backed. If a bucket from this list does not exists, it is simply ignored.  
`buckets_exclude`: Same as `buckets_include`, but reversed: buckets from this list will not be backed up.  
`nb_prefetcher`: Number of prefetcher process to spawn. Default to 24. Somehow, you cannot spawn more than 768 processes (probably strictly lower than 1024, I have not tried, nor search out where does this limitation come from). See more below about performance and memory usage.  
`queue_size`: Number of file to be processed by the plugin. This can be prefetched file (with in-memory content) or non-prefetchable (large) file. See more below about performance and memory usage.  
`prefetch_size`: Size of a 'small' object, in byte. Any object smaller will be prefetched. Default to 10485760 (10MB). See more below about performance and memory usage.  
`debug`: A boolean (True/true or False/false). If True or true, the plugin will log many debugging information via syslog. Default False.  


By default, all
All options are passed to libcloud. Thus, you can modify any option (including mandatories options, like `host`, `key` and `secret`).  
To see which options are supported by libcloud, check out their documentation (for instance, Ceph's RGW: https://libcloud.readthedocs.io/en/latest/apidocs/libcloud.storage.drivers.html#libcloud.storage.drivers.rgw.S3RGWStorageDriver , Google's GCS: https://libcloud.readthedocs.io/en/latest/apidocs/libcloud.storage.drivers.html#libcloud.storage.drivers.google_storage.GoogleStorageDriver)


Performance, CPU & memory usage
---

`bareos-libcloud` performs well, thanks to a largely multi-process design.  
Object storage is based on HTTP, and serializing each HTTP request makes the whole process slow, especially for small files.  
To counter this, small files are prefetched by a fleet of worker (called "prefetcher") : thus, when Bareos wants to backup a small file, its data is already fetched and available in memory.  
This allow a huge processing speed-up.  

Three options control this behavior : `nb_prefetcher`, the number of `prefetcher` that will gather small file in the background. `prefetch_size`, which defines what is a "small" file. `queue_size`, the maximum number of prefetched objects.  

Increasing `nb_prefetcher` is a win-move : more worker, more parallels HTTP-request, less downtime on the Bareos's side (time without data to backup).  

When `queue_size` is full, prefetchers that have downloaded an object will block until a free slot is available. The larger the queue, the better (but increasing `nb_prefetcher` is more efficient).  

As you have probably already find out, this design will eat a lot of memory. By "a lot", I mean "an infinite number of TB of memory, if you want to".  
To avoid memory overrun, you must configure these three options, based on the physical memory you can use. Else, some processes (probably prefetchers) will be killed by the oom killer. The file processed by that prefetcher **will** be lost (not present in the backup). Avoid that !  

The worst case scenario, concerning memory usage, is based on the following formula:  
```
worst_memory_usage_in_bytes = prefetch_size * (queue_size + nb_prefetcher)

# With the default values:
worst_memory_usage_in_bytes = 10485760 * (1000 + 24)
worst_memory_usage_in_bytes = 10737418240 bytes
worst_memory_usage = 10 GBytes
```

Performance data
---

A couple of performance sample are available. See [performance-sample.md](performance-sample.md)


Known bug
---

Libcloud currently cannot handle "wild" object name. It uses urllib.urlencode() where is should not. In short, if you have exotic characters, these objects will not be backed up (a message will be logged).  
A bug report is filled, no bug fix is released yet.



Feedbacks, feature requests or bug reports are welcome !
