---
title: "2019 03 25 Can Bitcoin Reach Exascale"
date: 2019-03-12T15:57:48+08:00
Categories: []
Tags: []
draft: true
---

https://medium.com/@thepiratewhocantbenamed/my-thoughts-on-your-thoughts-17474d800dda

https://moosefs.com/faq/
"In our environment (ca. 1 PiB total space, 36 million files, 6 million folders distributed on 38 million chunks on 100 machines) the usage of chunkserver CPU (by constant file transfer) is about 15-30% and chunkserver RAM usually consumes in between 100MiB and 1GiB (dependent on amount of chunks on each chunk server). The master server consumes about 50% of modern 3.3 GHz CPU (ca. 5000 file system operations per second, of which ca. 1500 are modifications) and 12GiB RAM. CPU load depends on amount of operations and RAM on the total number of files and folders, not the total size of the files themselves. The RAM usage is proportional to the number of entries in the file system because the master server process keeps the entire metadata in memory for performance. HHD usage on our master server is ca. 22 GB."

