Thread Pool
===

Usually a thread pool means
- a queue for callback functions
- producer to add callback
- consumer (waiting threads) to execute callbacks

Simplified implemention
===
[Code](https://github.com/senlinzhan/code-for-blog/blob/master/grpc_dynamic_thread_pool/dynamic_thread_pool.cpp)

Producer
---
``` cpp
void DynamicThreadPool::Add(const std::function<void ()> &callback)
{
    std::lock_guard<std::mutex> lock(mu_);

    // Add works to the callbacks list
    callbacks_.push(callback);

    // Increase pool size or notify as needed
    if (threads_waiting_ == 0)
    {
        // Kick off a new thread
        nthreads_++;
        new DynamicThread(this);
    }
    else
    {
        // unblock a thread
        cv_.notify_one();
    }

    // Also use this chance to harvest dead threads
    if (!dead_threads_.empty())
    {
        ReapThreads(&dead_threads_);
    }
}
```

Consumer
---
```
void DynamicThreadPool::ThreadFunc()
{
    for (;;)
    {
        std::unique_lock<std::mutex> lock(mu_);

        // Wait until work is available or we are shutting down.
        if (!shutdown_ && callbacks_.empty())
        {
            // If there are too many threads waiting, then quit this thread
            if (threads_waiting_ >= reserve_threads_)
            {
                break;
            }

            threads_waiting_++;
            // Wait for producer to push a new callback into the queue.
            // See https://en.cppreference.com/w/cpp/thread/condition_variable/wait.
            cv_.wait(lock);
            threads_waiting_--;
        }
        
        // Drain callbacks before considering shutdown to ensure all work gets completed.
        if (!callbacks_.empty())
        {
            auto cb = callbacks_.front();
            callbacks_.pop();
            lock.unlock();
            cb();            
        }
        else if (shutdown_)
        {
            break;            
        }
    }
}
```
