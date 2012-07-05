Adding `_GNU_SOURCE` feature test MACRO

`glibc` manual [1] defines `_GNU_SOURCE` as - 

> If you define this macro, everything is included:   
> ISO C89, ISO C99, POSIX.1, POSIX.2, BSD, SVID, X/Open, LFS, and GNU extensions.

Since the `flow-tools` library is based on BSD specific headers.  
Adding `_GNU_SOURCE` brought the features from both world when trying to compile on GNU/Linux.

The order of arguments of `qsort_r(…)` apparantely are different on `glibc` and `BSD` [2]

`glibc:`

	extern void qsort_r (
						 void *__base, 
						 size_t __nmemb, 
						 size_t __size,
						 __compar_d_fn_t __compar, 
						 void *__arg) __nonnull ((1, 4)
						);

`BSD:`

	void qsort_r(
				 void *base, 
				 size_t nel, 
				 size_t width, 
				 void *thunk,
	             int (*compar)(void *, const void *, const void *)
	            );

Therefore the invocation of `qsort_r(…)` [3] had to be wrapped around PLATFORM specific MACROS.

	+#if defined(__APPLE__) || defined(__FreeBSD__)
	   qsort_r(
	           sorted_recordset_ref, 
	           num_filtered_records, 
			   get_grouper_intermediates(
	           (void*)&grouper_ruleset[0]->field_offset2,
	           gtype->qsort_comp
	          );

	+#elif defined(__linux)
	+  qsort_r(
	+          sorted_recordset_ref, 
	+          num_filtered_records, 
	+          sizeof(char **), 
	+          gtype->qsort_comp,
	+          (void*)&grouper_ruleset[0]->field_offset2
	+         );
	+#endif
	
where `gtype->qsort_comp` [3] is -


	 struct grouper_type {

	+#if defined (__APPLE__) || defined (__FreeBSD__)
	   int (*qsort_comp)(
	                     void*                           thunk,
	                     const void*                     e1,
	                     const void*                     e2
	                    );
	-  
	+#elif defined (__linux)
	+  int (*qsort_comp)(
	+                    const void*                     e1,
	+                    const void*                     e2,
	+                    void*                           thunk
	+                   );
	+#endif

	
Resources:

[1] [feature test macros [`glibc` manual] &rarr;](http://www.gnu.org/software/libc/manual/html_node/Feature-Test-Macros.html)

[2] [`qsort_r` argument order [`glibc` mailing list] &rarr;](http://sourceware.org/ml/libc-alpha/2008-12/msg00003.html)

[3] [`grouper` reference &rarr;](http://dl.dropbox.com/u/500389/mthesis/docs-engine/html/grouper_8c.html)

