Reverting to a flat source structure.
	
	>>> import this
	The Zen of Python, by Tim Peters
	
	…
	Flat is better than nested.
	…
	
The previous source structure was unnecesarily nested for the simple number of source files that we have.  
Switching to a flat source structure reduced the CMake compilation complexity alot. This is how it looks like now:

	[engine] >> tree
	.
	|-- examples/
	|   `-- trace.ft
	|-- include/
	|   |-- base.h
	|   |-- branch.h
	|   |-- echo.h
	|   |-- errorhandlers.h
	|   |-- filter.h
	|   |-- flowy.h
	|   |-- ftreader.h
	|   |-- grouper.h
	|   |-- groupfilter.h
	|   |-- merger.h
	|   |-- pipeline.h
	|   |-- ungrouper.h
	|   `-- utils.h
	|-- scripts/
	|   |-- queries/
	|   |   |-- build-ftp-tcp-session.py*
	|   |   |-- build-http-octets.py*
	|   |   |-- build-http-tcp-session.py*
	|   |   `-- build-https-tcp-session.py*
	|   `-- generate-functions.py*
	|-- src/
	|   |-- base.c
	|   |-- branch.c
	|   |-- echo.c
	|   |-- errorhandlers.c
	|   |-- filter.c
	|   |-- flowy.c
	|   |-- ftreader.c
	|   |-- grouper.c
	|   |-- groupfilter.c
	|   |-- merger.c
	|   |-- ungrouper.c
	|   `-- utils.c
	|-- CMakeLists.txt
	|-- Doxyfile
	|-- Makefile
	`-- README.md
	
CMake compilation creates a `.build/` and puts the executable binary in `bin/`.   
The auto generated C sources and headers goto `.build/` as well and are automatically included and linked in the final binary.

	# custom command to prepare auto-generated sources
	add_custom_command (
	  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/auto-assign.h
	         ${CMAKE_CURRENT_BINARY_DIR}/auto-assign.c
	         ${CMAKE_CURRENT_BINARY_DIR}/auto-comps.h
	         ${CMAKE_CURRENT_BINARY_DIR}/auto-comps.c
	  COMMAND python ${CMAKE_SOURCE_DIR}/scripts/generate-functions.py
	  COMMENT "Generating: auto-comps{h,c} and auto-assign.{h,c}"
	)


CMake compilation also runs the build query scripts defined in `scripts/queries/` to generate some examples `JSON` queries
and moves them to the `examples/` folder ready to be used by the `bin/engine` binary.

	# custom command to generate examples
	file(GLOB pyFILES ${CMAKE_SOURCE_DIR}/scripts/queries/*.py)
	foreach(pyFILE ${pyFILES})
	  set(query "${pyFILE}_query")
	  add_custom_command (
	    OUTPUT ${query}
	    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/examples/
	    COMMAND python ${pyFILE}
	    COMMENT "Generating: JSON example query using ${pyFILE}"
	  )
	  list(APPEND queryFILES ${query})
	endforeach(pyFILE)

In order to avoid doing - 

	[engine] >> mkdir .build
	[engine] >> cd .build
	[.build] >> cmake ..
	[.build] >> make
	[.build] >> cd ..
	[engine] >> rm -r .build

taking inspiration from Dirk Joel-Luchini Colbry [1], I am using a Makefile that calls CMake to automate this operation.

	make: build/Makefile
		(cd .build; make)
	
	build/Makefile: build
		(cd .build; cmake -D CMAKE_PREFIX_PATH=$(CMAKE_PREFIX_PATH) ..)
	
	build:
		mkdir -p .build
	
Additional targets to `clean` and generate Doxygen documentation `doc` also exist.  
The generated documentation goes in `doc/` and is subsequently deleted by `make clean`.

	doc: Doxyfile
		(mkdir -p doc; doxygen Doxyfile)

	clean:
		rm -f -r .build
		rm -f -r bin
		rm -f -r doc
		rm -f -r examples/*.json
		
Makefile can also take `CMAKE_PREFIX_PATH` as an argument and pass it on to CMake.  
`CMAKE_PREFIX_PATH` is used to supply arbitrary location of external libraries and include PATHS.

Resources:

[1] [CMake Makefile &rarr;](https://wiki.hpcc.msu.edu/display/~colbrydi@msu.edu/2010/08/19/Cmake+Makefile)