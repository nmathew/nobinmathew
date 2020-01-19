---
author: Cloud Fundoo
comments: true
date: 2012-05-18 04:09:26+00:00
layout: post
slug: minor-page-faults-and-dynamic-memory-allocation-in-linux
title: Minor Page faults and dynamic memory allocation in Linux
wordpress_id: 154
tags:
- glibc
- malloc
- mmap
- pagefault
---

There is a very good [paper](http://www.usenix.org/publications/library/proceedings/als01/full_papers/ezolt/ezolt.pdf) written by Philip Ezolt, this paper takes the reader for a step by step analysis of a problem where malloc is causing large number of [minor page faults](http://en.wikipedia.org/wiki/Page_fault). If you want to understand what is written here, you must read that paper.

In that author is using following code snippet to reproduce the issue.

{% highlight c %}
int main(int argc, char *argv[])
{
    char *ptr;
    int count;

    count = atoi(argv[1]) * 1024;

    while(1)
    {
        ptr = malloc(count);
        free(ptr);
    }
    return 0;
}
{% endhighlight %}

You run the application like ./test_malloc 128 where 128KB is the M_MMAP_THRESHOLD, use sar(System Activity Report tool) like sar -B 1 1000, where 1 is the time interval and 1000 is the count. But when you see the sar output you won’t see much changes in the minor page faults count, surprised? Don’t worry, this is because the current [glibc's](http://sourceware.org/git/?p=glibc.git;a=tree;h=b34e12e22c00d74ee549ae9ac304f64d1d6374d5;hb=b34e12e22c00d74ee549ae9ac304f64d1d6374d5http://) malloc implementation changed a lot from 2001. See [Wolfram Gloger’s](http://www.malloc.de/en/) page and Arjan’s [patch](http://sourceware.org/ml/libc-alpha/2006-03/msg00033.html).

So how we can reproduce the issue again, see the code snippet below

{% highlight c %}
int main(int argc, char *argv[])
{
    char *ptr;
    int count;

    count = atoi(argv[1]) * 1024;

    while(1)
    {
        ptr = malloc(count);
        free(ptr);
        count += (100*1024);
    }
    return 0;
}
{% endhighlight %}

where we increment the chunk size by 100KB, this way we can reproduce the minor page faults issue.

This is because current glibc defines M_MMAP_THRESHOLD as a dynamically varying quantity between DEFAULT_MMAP_THRESHOLD_MIN(128KB) and DEFAULT_MMAP_THRESHOLD_MAX(32MB) in 32 bit systems. This simple algorithm wants to achieve two things.

1) Create minimum fragmentation
2) use mmap() for long lived allocations
3) temporary allocations should use brk().

Major difference between brk() and mmap() is that, the chunk got using mmap() cann’t be recycled i.e. it cann’t be used for future allocations, you must give back to the system using munmap(), while chunk allocated using brk() can be used for future allocations. mmap() allocations are quite expensive.

Now coming back to the algorithm, we have two main things, long lived ones and temporary ones, when we allocate a chunk greater than M_MMAP_THRESHOLD for the first time we will consider it as long lived ones and will use mmap() for allocating that chunk. If we free this chunk immediately then glibc will update the M_MMAP_THRESHOLD and will consider future allocations of that chunk size as temporary ones and will use brk() for allocating it. Once it is allocated using brk(), that chunk can be recycled.This is for chunk sizes between between DEFAULT_MMAP_THRESHOLD_MIN and DEFAULT_MMAP_THRESHOLD_MAX. If chunk size is greater than DEFAULT_MMAP_THRESHOLD_MAX then glibc will always consider it as long lived ones and will use mmap() for allocating the chunk.

We can use mallopt() call to tune M_MMAP_THRESHOLD, DEFAULT_MMAP_THRESHOLD_MIN and DEFAULT_MMAP_THRESHOLD_MAX. If the application set the M_MMAP_THRESHOLD value explicitly then dynamic behaviour of M_MMAP_THRESHOLD will be switched off.
