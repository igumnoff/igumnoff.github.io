---
layout: post
title: "How I Added 250 GB to My VPS Disk for €4 a Month Using S3 and ext4"
date: 2026-11-18 12:00:00 +0100
categories: [devops, docker, storage]
---

I recently ran into a shortage of disk space on my VPS. I currently rent a VPS for €7 a month with this configuration: 4 vCPUs, 8 GB of RAM, and a 120 GB NVMe drive. It has 50 GB of free space left. But I need to run a Docker container whose image is 76 GB. I could upgrade my current VPS to get a 240 GB disk, but that would cost €30 a month. That’s steep. I noticed that my hosting provider offers 250 GB of S3 storage for just €4 a month, accessible over its internal network. So I started wondering how I could turn it into a filesystem and have Docker store files there.

First, I tried rclone. It lets you create a FUSE filesystem from your S3 storage. To speed things up, you can cache frequently used files on a local SSD using the --vfs-cache-max-size flag. Unfortunately, Docker uses overlay2, which requires filesystem operations that rclone mount does not guarantee: atomic renames, correct chmod and chown behavior, hard links, and extended attributes. In short, a full POSIX filesystem is needed.

And I found a solution: s3backer. It has 600+ stars on GitHub and provides a full-fledged ext4 filesystem. I set a 20 GB cache on my local SSD using the --blockCacheSize option, mounted it at /mnt/s3, and reconfigured Docker to store images, containers, and volumes there. As a result, the most frequently used files are cached locally instead of being pulled over the network from S3. What do you think? Expanding my virtual machine’s disk space by 250 GB for €4 a month? Pretty great, if you ask me!

By the way, here’s a summary table of the available options:

| Solution | How it works | Pros | Cons |
| --- | --- | --- | --- |
| **s3backer + ext4** | S3 as a block device | Full POSIX filesystem, Docker compatibility, caching | Complex setup, latency, risk of filesystem corruption |
| **rclone mount** | FUSE over S3 | Simple setup, local cache, support for multiple cloud providers | Incomplete POSIX compatibility, some operations are unsupported |
| **s3fs** | FUSE over S3 | Simple mounting, familiar file operations | Slow metadata operations, POSIX limitations |
| **JuiceFS** | Distributed filesystem on top of S3 | POSIX, caching, high performance | Requires separate metadata storage |
| **S3QL** | Filesystem on top of S3 | POSIX, compression, deduplication, encryption | Extra overhead, complex recovery |

Of course, s3backer is far from the only way to turn S3 into a filesystem. There are at least two other interesting projects: JuiceFS and S3QL. Both run on top of S3, support POSIX operations and local caching, but they seemed like overkill for my use case.

JuiceFS is a fairly powerful distributed filesystem. It supports multiple servers working concurrently, caching, POSIX, and much more. But there’s a catch: in addition to S3 itself, it requires a separate metadata store. This could be Redis, PostgreSQL, MySQL, or even a local SQLite database. The last option doesn’t require a separate service, but you still have to keep the metadata database safe and back it up. Overall, it’s an interesting solution, especially if you have a server cluster. But I only have one VPS, and I didn’t want to set up extra infrastructure just to store Docker images.

S3QL is another interesting option. It supports POSIX, local caching, compression, deduplication, and encryption. It doesn’t need a separate database server: metadata is stored in SQLite and synchronized with the object store. But these extra features come at the cost of added complexity. Deduplication requires computing hashes, compression uses CPU time, and recovering the filesystem after a failure may require checking and restoring metadata with specialized tools. These aren’t critical drawbacks, but I didn’t see any particular advantages for my use case, either.

The key difference lies in the approach: JuiceFS and S3QL implement filesystems themselves using FUSE, while s3backer provides virtual block storage on top of which you can create a standard ext4 filesystem. As a result, Docker works with the familiar Linux filesystem, and s3backer translates block operations into requests to S3. No separate metadata databases, no deduplication, and no extra layers of file management.

Of course, this setup has its downsides too. Random reads and writes can be heavily affected by S3 latency, and losing the connection to the storage can bring the containers to a halt. A serious failure could also corrupt the filesystem. So I wouldn’t use this setup for critical databases. But for storing large Docker images that are downloaded once and then mostly read, it seemed perfectly reasonable—especially when S3 is available over the hosting provider’s local network. In short, I wanted an extra 250 GB for €4 a month without building a distributed storage system. From that perspective, s3backer seemed like the best fit.
