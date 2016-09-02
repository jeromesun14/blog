ARM 平台 abort 退出后堆栈打不全
================================

ARM 平台 assert 退出后堆栈打不出来的原因是：调用了abort()函数，abort是一个没有返回的函数，只会被调用一次，运行时不会去保存相关的寄存器值。

改进方向：
* 追踪邮件列表发现08年后的abort()函数可能有优化。这是一个方向，优化abort函数，以使得调用abort函数的源能打出来。
* 改 assert 函数，让它以其他 signal 退出，比如通过改写 0 地址内容产生 signal 11 退出。

链接：http://comments.gmane.org/gmane.comp.gdb.devel/24018

>> Your implementation of abort does not save a return address, so GDB
>> can't display it.  I believe tehis is a known limitation of the ARM
>> GCC port.
>
> GCC should really not do this.  People are almost guaranteed to want
> to be able to see a backtrace from abort(3).
>
> I suppose it optimizes away the instructions to save the return
> address, because abort() is marked with __attribute__(noreturn).  But
> that means there is very little point in actually doing that
> optimization since __attribute__(noreturn) implies that the function
> will only be called once!  I suppose there are some space savings but
> are they really significant?
>>  Joe> There are several effects from "noreturn".  We would want some
>>  Joe> of these effects for "abort", but not others, to get debuggable
>>  Joe> code without degrading compile-time warnings.
>>
>> So the issue is that two unrelated features are currently combined in
>> a single attribute:
>> 
>> 1. This function doesn't return, do the right thing with warnings in 
>>    the caller of this function.
>> 
>> 2. Don't bother saving registers when calling this function because it
>>    won't return so the registers aren't needed afterwards.
>> 
>> The issue is that #2 doesn't apply to "abort" because the registers
>> ARE needed afterwards -- at debug time.
