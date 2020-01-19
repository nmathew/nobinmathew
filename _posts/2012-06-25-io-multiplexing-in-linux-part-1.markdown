---
author: Cloud Fundoo
comments: true
date: 2012-06-25 05:36:32+00:00
layout: post
slug: io-multiplexing-in-linux-part-1
title: 'I/O Multiplexing in Linux : Part 1'
wordpress_id: 225
tags:
- epoll
- I/O
- Internals
- kernel
- Linux
- Multiplexing
- Non Blocking
- select()
- server
---

There is lot of discussion going on I/O multiplexing in Linux, select (), poll () and epoll () are the Linux mechanisms for that. I looked at the kernel implementation in order to better understand that, this may help you guys also.
We will start with examples of how to use these interfaces. In all the examples below, open the devices/file in non-blocking mode.

select () example:

{% highlight c %}
fd_set rfds;
FD_ZERO(&rfds);
FD_SET(0, &rfds);	/** checking stdin **/
retval=select(1, &rfds, NULL, NULL, NULL);
if((retval == 1) &&(FD_ISSET(0, &rfds))
	printf(“There is data on stdin\n”);
{% endhighlight %}

poll () example:

{% highlight c %}
struct pollfd rfd;
rfd.fd = 0;		/** checking stdin **/
rfd.events = POLLIN;
retval = poll(&rfd, 1, 0);
if((retval ==1) && (rfd.revents&POLLIN))
	printf(“There is data on stdin\n”);
{% endhighlight %}

epoll () example:

{% highlight c %}
struct epoll_event event2read, eventbuffer;
epfd = epoll_create(1024);
event2read.data.fd = 0		/** checking stdin **/
event2read.events = EPOLLIN|EPOLLET;
epoll_ctl(epfd, EPOLL_CTL_ADD, event2read.data.fd, &event2read);
retval = epoll_wait(epfd, &eventbuffer, 1, 0, NULL);
if((retval == 1) && (eventbuffer.events & EPOLLIN))
	printf(“There is data on stdin\n”);
{% endhighlight %}

Above examples are very simple and incomplete. The picture below will explain basically what happens inside all three interfaces.

[![](http://cloudfundoo.files.wordpress.com/2012/06/io-linux.jpg)](http://cloudfundoo.files.wordpress.com/2012/06/io-linux.jpg)

So why we have three interfaces, if all has almost same implementation. Basic difference comes in the implementation of data structure, where it is maintained (user space/kernel space) and also in the kernel system call interface.

Let’s start with select (), select uses three bit masks, each having size 1024 bits (chunks of (1024/sizeof(unsigned long)), in a 32bit linux machine, it is 32, 32bit unsigned long integers. Three bit masks are used for getting ready for read, ready for write, on exception events of file descriptors. Select () has only one system call select() itself, each time you want to read events on fds, just fill the required bitmasks and call select (), the return from Syscall select () will have bitmasks having bit set for ready descriptors.

Maximum value of fd and maximum number of fds a select () can monitor is 1024 (__FD_SETSIZE), see my previous post [Inside select() call macros](http://cloudfundoo.wordpress.com/2012/06/12/inside-select-call-macros/).

Primary kernel interface for select () call is select system call, which is defined in fs/select.c, select () calls core_sys_select() and then do_select() which is the main handler.

select() and core_sys_select() does nothing much, it copies some arguments from/to kernel/user space and does some error checking. do_select () implements most of the functionality of select(),

do_select() has mainly three for() loops in linux kernel version 3.4.2.

{% highlight c %}
int do_select(int n, fd_set_bits *fds, struct timespec *end_time)
{
      ........
    1)for (;;) {
      unsigned long *rinp, *routp, *rexp, *inp, *outp, *exp;
      ...........
           2)for (i = 0; i < n; ++rinp, ++routp, ++rexp) {
             unsigned long in, out, ex, all_bits, bit = 1, mask, j;
             ...........
                 3)for (j = 0; j < __NFDBITS; ++j, ++i, bit <= n)
                   ..........
                   if (f_op && f_op->poll){
                         wait_key_set(wait, in, out, bit);
                         mask = (*f_op->poll)(file, wait);
                         ........
{% endhighlight %}

[fd_set_bits](http://lxr.free-electrons.com/ident?i=fd_set_bits) *fds parameter to do_select() contains all the input/output bitmasks. In do_select() 1st for() loop goes forever till either any descriptor is ready or timeout occurs, in the 2nd for() loop do_select() goes through each unsigned long int (__NFDBITS) bitmask, and finally sets the corresponding output unsigned long bitmask.

In the 3rd for() loop, it goes through each bit in each unsigned long (__NFDBITS) bitmask. It calls device specific poll () through file operations table corresponding to that fd (fd is set in the bitmask), device poll () will register the process (our application which invoked select() call) for a ready notification on  device specific read/write wait queues. Once device is ready for write/read it will wake up the process, i.e. select() call which is sleeping with a timeout.

If device is ready for read/write while device poll() is called, poll() will anyway add entry on wait queues if timeout is not zero, but this time poll() will return with bitmask indicating device is ready for read/write.

So if at least one fd is ready during the first iteration of 1st for() loop, it will come out of the for() loops, remove itself from all device specific wait queues using poll_freewait() and will return with ready fds bitmask. If none of the fds are ready by first iteration of the 1st for() loop, it will sleep till any fd is ready or till timeout expires, when it wakes up it will loop once again through all the three loops and collect all the ready events and frees itself from all device wait queues and returns with bitmask any ready fds.

Some limitations of select() are, it can handle only a maximum of 1024 fds, which is the maximum default open file descriptors allowed for a process, there are patches available to overcome this. Select bitmasks are not persistent, we need to redo the entire process everytime we call the select(). Management of fds is difficult in the user space.

We will cover poll() and epoll() in the next parts.
