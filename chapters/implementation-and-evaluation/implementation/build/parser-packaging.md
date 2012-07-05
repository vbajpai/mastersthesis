Parser Directory Organization:  
Source code has been moved to `parser/src/`, and documentation has been moved to `parser/docs/`.  
Queries and Examples traces have been moved to `parser/examples/`

Parser Installation is now Painless:  
Apparatenly there is a bug in `numexpr` which imports externals dependencies in its egg files.  
As a result, installation from a requirements file fails [1].

To circumvent the issue, I have created a custom Makefile that virtually creates an additional
preprocessing pass to install `numexpr`'s dependencies before going forward with installation from `requirements.txt`.

	make: numexpr
	        (pip install -r requirements.txt)
	
	numexpr: numpy
	        (pip install numexpr==2.0.1)
	
	numpy: cython
	        (pip install numpy==1.6.1)
	
	cython:
	        (pip install Cython==0.15.1)
	
	clean:
	        rm -f -r build/
	        rm -f -r src/*.pyc
	        rm -f -r flowy-run/
	        rm -f -r parsetab.py parser.out
	        rm -f -r examples/output.h5
	        
A detailed `README.md` file has been included to provide build instructions with `virtualenvwrapper` and usage.


Resources:

[1] [[stackoverflow]: pip fails to install packages from requirements.txt &rarr;](http://stackoverflow.com/questions/11015692/pip-fails-to-install-packages-from-requirements-txt)