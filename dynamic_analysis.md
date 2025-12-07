# Talk: Dynamic Program Analysis for Fun and Profit
[https://www.linuxfoundation.org/webinars/dynamic-program-analysis-for-fun-and-profit]

- analysis of the properties of running program
- properties: bugs, performance, code coverage, call graph, data flow
- static analysis: analyze properties of program code (type, syntax, etc.)
- True positive: report a real bug
- False positve: it is not a bug but get reported

```C
void func() {
    char* p = malloc(10);
    p[20] = 1; // OOB (out-of-bounds)
}
```
This is harder.
```C
void func(int index) {
    char* p = malloc(10);
    p[index] = 1; // OOB (out-of-bounds)
}

char* str = &buffer[offset];
int index = atoi(str);
func(index);
```

- static & dynamic analysis is complementary.
- dynamic has no false positive.
- challenge of dynamic analysis: getting the coverage
    - test, fuzzing, development ...

- CONFIG_DEBUG_LIST=y
```C 
#ifdef CONFIG_DEBUG_LIST
    // <some checks>
#endif
```
- FORTIFY_INLINE
- BUILD_BUG_ON(condition)
- WARN_ON(condition)

```
$ scripts/decode_stacktrace.sh vmlinux < crash.log
```
will help change the memory addres (hex) to file:function:lineno

# KASan
- No false positive
- Out-of-Bounds, Use-after-free: usually nothing happens. You just corrupt some memory
- KASan = BOOM the program when these happen
- mechanism: shadow memory (every 8 align bytes (64bit) has + 1 shadow byte)
- shadow byte encodes how many bad memory bytes [good good ... good5 bad bad] <- shadow = [2]
- if all bad, shadow shows -1
- [physical memory] [vmalloc] [KASAN shadow] [...] [.text]
- There is a mapping from kernel address to shadow byte: Shadow = KAddr/8 + offset (simple)
- Red-zone around heap object [heap obj shadow heap obj shadow ...]. If kernel tries to access the shadow (redzone), errors
- Quarantine for heap objects: kfree() [ ] kmalloc() -> Quarantine = delay reuse (mark this address as bad-to-use for a while)
- compiler instrumentation: shadow = ...; if (*shadow) throw error; // expect shadow to be zero
