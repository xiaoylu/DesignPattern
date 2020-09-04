resource acquisition is initialization (C++)
===

The basic idea is to represent a resource by a local object (i.e. "resource handle"), so that the local object's destructor will release the resource.

This prevents resource leakage, because the destructor of "resource handle" will always be called (C++). 

```cpp
	class File_handle {
		FILE* p;
	public:
		File_handle(const char* n, const char* a)
			{ p = fopen(n,a); if (p) throw Open_error(errno); }
		File_handle(FILE* pp)
			{ p = pp; if (p) throw Open_error(errno); }

		~File_handle() { fclose(p); }

		operator FILE*() { return p; }

		// ...
	};

	void f(const char* fn)
	{
		File_handle f(fn,"rw");	// open fn for reading and writing
		// use file through f
	}
```

Thanks to garbage collection, Java doesn’t have/need this feature.

Reference
===
[https://www.stroustrup.com/bs_faq2.html#finally](https://www.stroustrup.com/bs_faq2.html#finally)
[https://www.yegor256.com/2017/08/08/raii-in-java.html](https://www.yegor256.com/2017/08/08/raii-in-java.html)
