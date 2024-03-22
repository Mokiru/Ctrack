```c++
//malloc-based allocator 通常比 default alloc速度慢
//一般而言是thread-safe，并且对于空间的运用比较高效（efficient）
//以下是第一级配置器
//注意，无“template类型参数”，至于“非类型参数”inst，则完全没有用处
template<int inst>
class __malloc_alloc_template {
private:
    //以下函数将用来处理内存不足的情况
    //oom:out of memory
    static void *oom_malloc(size_t);
    static void *oom_realloc(void*, size_t);
    static void (*__malloc_alloc_oom_handler) ();
public:
    static void* allocate(size_t n) {
        void* result = malloc(n); //第一级配置器直接使用malloc()
        //以下无法满足需求时，改用oom_malloc
        if (0 == result) result = oom_malloc(n);
        return result;
    }
    static void* deallocate(void* p, size_t /* n */) {
        free(p); //第一级配置器直接使用free
    }
    static void* reallocate(void* p, size_t /* old_sz */, size_t new_sz) {
        void* result = realloc(p, new_sz);
        //以下无法满足需求时，改用oom_realloc()
        if (0 == result) result = oom_realloc(p, new_sz);
        return result;
    }
    //以下仿真c++的set_new_handler()
    static void (*set_malloc_handler(void(*f)()))() {
        void (*old)() = __malloc_alloc_oom_handler;
        __malloc_alloc_oom_handler = f;
        return (old);
    }
};
//malloc_alloc out-of-memory handling
//初值为0，待客端设定
template<int inst>
void (*__malloc_alloc_template<inst>::__malloc_alloc_oom_handler)() = 0;

template<int inst>
void *__malloc_alloc_template<inst>::oom_realloc(void *p, size_t n) {
    void (*my_malloc_handler)();
    void *result;
    for (;;) {//不断尝试释放、配置、再释放、再配置。。。
        my_malloc_handler = __malloc_alloc_oom_handler;
        if (0 == my_malloc_handler) {__THROW_BAD_ALLOC;}
        (*my_malloc_handler)(); //调用处理例程，企图释放内存
        result = realloc(p, n);//再次尝试配置内存
        if (result) return (result);
    }
}

//以下直接将参数inst指定为0
type __malloc_alloc_template<0> malloc_alloc;
```