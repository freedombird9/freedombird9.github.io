---
layout: post
title:  "A Simple Launch-and-Go-Away Multiprocessing Architecture"
description: Python: a simple way of launching a task, set a timeout and leave it running or being killed later automatically.
date:   2016-01-22 11:03:44
categories: technology
---
In many situations we may want to launch a background task to do something that is uncontrollable, dangerous and may block the main program for a long time or even *forever*. For example, a FTP uploading or downloading, or a complicated computation that relies on modules of different qualities. For those operations, you don't want to run them in your main process, you want to isolate them as much as you can.

Here I'm only addressing this kind of situation: you launch the task, set a timeout for it, and you are good to go. You don't care if it will succeed or not, you don't retry it on failure and it should be killed automatically if it runs out of time (Why? Because otherwise it will become a zombie and stick there forever and never release any resource).

This design is very useful in situations where you don't care about occasional data loss.

### Thread or Process?
I would use multiprocessing. Not only because a thread can neither be killed in Python, nor can it terminate itself, but also because of the extra overhead on multicore systems, which is very common nowadays, introduced by the infamous GIL.

Regarding the GIL issue, here is a perfect insight: [Inside the Python GIL]. Here is the picture that visualizes this problem very well.
<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-simple-multiprocessing/gil_battle.png" alt="GIL Battle"/>
    <figcaption>GIL Battle</figcaption>
  </figure>
</div>

Put it simply, on a multicore system, a thread will be scheduled to run immediately (on another core) once the OS is signaled that a previously running thread has released its GIL. But when it can start to run, the GIL is already gone. It's because with multiple cores, threads in ready state get scheduled to run on different cores at the same time and they compete for the GIL.

So in the above figure, Thread 2 on CPU 2 gets signaled every time Thread 1 releases GIL, but it cannot start to run before Thread 1 acquires the GIL again due to the signal passing and scheduling overheads.

Be very careful with Python multithreading, especially in multicore systems.

### Architecture
Remember we just want to launch a separate process to run, we don't want our main process to wait for it to finish or to continuously monitor its state. Once the process is forked, our main program shouldn't care about it anymore otherwise it will be slowed down.

How to achieve this? It turns out we can just wrap the worker process in another process which is responsible for the worker's lifecycle. We call it the manager process here. The design looks like this:

<div class='md-10-suffix-1'>
{% highlight Python %}

def worker(queue, queue_manager):
    # Read from the queue
    while True:
        msg = queue.get() # Read from the queue
        if (msg == 'DONE'):
            break
        else:
            '''
               do some work here
            '''

def manager(task_queue):
  '''start a process that launches the worker process'''
    worker_p = Process(target=worker, args=(task_queue, ))
    worker_p.start()
    worker_p.join(90)
    if worker_p.is_alive():
        logger.error('worker process timed out, killing ...')
        worker_p.terminate()

if __name__ == '__main__':
    task_queue = Queue()
    process_list = []
    for i in range(num_workers):
        manager_p = Process(target=manager, args=(task_queue,))
        manager_p.start()
        process_list.append(manager_p)

    add_tasks(task_queue)

{% endhighlight %}
</div>






[Inside the Python GIL]: http://www.dabeaz.com/python/GIL.pdf
