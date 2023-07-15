# 简易C++线程池

一个非常简易的C++线程池。仍在继续学习博大精深的C++多线程和并发，希望未来随着学习进步持续改进。

## 解析
### 构造函数
1. emplace_back()类似于push_back(),但是效率更高

2. 中间部分代码在独立作用域中，是因为离开作用域会自动释放锁

3. condition_variable是用来阻塞当前进程的，它的函数 `wait( std::unique_lock<std::mutex>& lock, Predicate pred );``
要求等待同时还要满足后面的pred条件，等价于
```
while (!pred())
{
    wait(lock);
}
```

### submit函数
1. 注意看，这是可变长参数模板的用法。

2. 写  -> std::future<decltype(f(args...))>  的目的是为了明确指定  submit  函数的返回类型。 std::future  是一个模板类，它的模板参数类型需要明确指定

3. 通过  decltype(f(args...)) ，我们可以获得调用  f  函数并传递  args  参数后的返回类型。其实就是f的返回类型。

4. make_shared函数创建了一个智能指针，保证这个task在submit函数以外依然能存活并发挥作用。因此task是一个shared_ptr。

5. std::package_task把一个可调用对象包装起来，并允许非即时获取其结果。它类似于std::function,但是会自动把结果传递给一个std::future对象。

6. std::bind用来把一个可调用对象绑定到一组参数上，std::bind的第一个参数是可调用对象，后面的参数是可调用对象的参数。std::bind的返回值也是一个可调用对象。

7. std::forward  是一个用于完美转发的函数模板，用于在泛型代码中保持参数的值类别（左值或右值）。在这里， std::forward<F>(f)  保持  f  的值类别，并将其传递给  std::bind  函数；而  std::forward<Args>(args)...  则保持每个参数的值类别，并将它们传递给  std::bind  函数。

8. [task]() { (*task)(); }是一个 lambda 表达式，表示一个匿名函数。它的含义是创建一个函数对象，该函数对象接受一个指向函数的指针作为参数，并调用该函数。 
在这个 lambda 表达式中， [task]  是捕获列表，用于指定在 lambda 表达式中引用的外部变量。 (*task)()  是函数调用语法，表示调用指针  task  所指向的函数。 
因此，这一行代码的意思是创建一个 lambda 函数对象，它接受一个指向函数的指针作为参数，并调用该函数。


9. condition.notify_one()通知一个正在等待的线程，condition.notify_all()通知所有线程。