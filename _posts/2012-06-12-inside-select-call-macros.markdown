---
author: Cloud Fundoo
comments: true
date: 2012-06-12 05:10:21+00:00
layout: post
slug: inside-select-call-macros
title: Inside select() call macros
wordpress_id: 218
tags:
- I/O
- Linux
- Non Blocking
- poll
- select()
---

I was looking at the IO multiplexing calls for writing an IO library, select() has some macros for setting, clearing the file descriptors in file descriptor sets. While looking a for a method to optimize the walk through fd sets, i looked at the implementation of those macros.

Let's go through the implementation of FD_SET macro in Linux, other macros are similar.

See both /usr/include/i386-linux-gnu/sys/select.h and /usr/include/i386-linux-gnu/bits/select.h, FD_SET is defined as below.
{% highlight c %}

#define    FD_SET(fd, fdsetp)    __FD_SET(fd, fdsetp)
#define	__FD_SET(d, set)	\
		((void)(__FDS_BITS(set)[__FD_ELT (d)] |= __FD_MASK(d)))
#define	__FDS_BITS(set)		((set)->__fds_bits)
#define	__FD_ELT(d)			((d)/__NFDBITS)
#define	__FD_MASK(d)		((__fd_mask) 1 << ((d) % __NFDBITS))

{% endhighlight %}

Other definitions are below.

{% highlight c %}

typedef	long int __fd_mask;

#define	__NFDBITS			(8*(int) sizeof(__fd_mask))

typedef struct 
{
	__fd_mask __fds_bits[__FD_SETSIZE/__NFDBITS];
} fd_set;

{% endhighlight %}

In a 32 bit Linux system, __FD_SETSIZE is defined as 1024, i.e. usually maximum allowed file descriptors for a process, see "ulimit -a"

See both /usr/include/i386-linux-gnu/sys/typesizes.h and /usr/include/linux/posix_types.h

{% highlight c %}

#define __FD_SETSIZE	1024

{% endhighlight %}

On 32bit Linux __NFDBITS is 32 since 8*sizeof(long int) is 8*4 i.e. 32.

So we can rewrite the FD_SET macro as below.

{% highlight c %}

#define	__FD_SET(d, set)	\
	((void)(((set)->fds_bits)[((d)/ __NFDBITS)] |= ((__fd_mask) 1 << ((d) % __NFDBITS))))

/** On 32bit Linux **/
#define	__FD_SET(d, set)	\
	((void)(((set)->fds_bits)[((d)/ 32)] |= ((__fd_mask) 1 << ((d) % 32))))

{% endhighlight %}
 
fd_set has an array of 32 32bit long ints, so ((set)->fds_bits)[((d)/ 32)] finds array element where new fd falls, then ((__fd_mask) 1 << ((d) % 32))) finds the bit corresponding to new fd in that long int array element.

So FD_SET sets the bit corresponding to new descriptor in a bit mask of size 1024.
