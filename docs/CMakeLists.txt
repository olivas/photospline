
set(SPHINX_EXTRA_OPTIONS "-N;-a;-E" CACHE STRING "Sphinx options")

find_program(SPHINX_BUILD sphinx-build)
find_program(DOXYGEN doxygen)

if(SPHINX_BUILD AND DOXYGEN)
  EXECUTE_PROCESS (COMMAND ${PYTHON_EXECUTABLE} -c
    "import breathe"
    RESULT_VARIABLE _breathe_notfound
    OUTPUT_QUIET ERROR_QUIET)
    if (NOT ${_breate_notfound} EQUAL 0)
      set(MAYBE_BREATHE "")
    else()
      set(MAYBE_BREATHE "'breathe'")
    endif()
endif()

if(SPHINX_BUILD)
  message("-- + sphinx-build found, building sphinx documentation")
  set(SPHINX_PAPER_SIZE "letter" CACHE STRING "Size of paper when generating latex/pdf of documentation.  Options are 'letter' or 'a4'")
  set(SPHINX_EXTRA_OPTIONS "" CACHE STRING "Extra options to pass to sphinx")

  add_custom_target(html
    COMMAND mkdir -p ${CMAKE_BINARY_DIR}/docs ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/doctrees 
    COMMAND 
    ${SPHINX_BUILD} ${SPHINX_EXTRA_OPTIONS} 
    -b html 
    -d ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/doctrees 
    -D latex_paper_size=${SPHINX_PAPER_SIZE} 
    source
    ${CMAKE_BINARY_DIR}/docs
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Generating docs to ${CMAKE_BINARY_DIR}/docs")

  add_custom_target(latex
    COMMAND mkdir -p ${CMAKE_BINARY_DIR}/latex ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/doctrees
    COMMAND ${SPHINX_BUILD} 
    ${SPHINX_EXTRA_OPTIONS} 
    -b latex 
    -d ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/doctrees 
    -D latex_paper_size=${SPHINX_PAPER_SIZE} 
    source
    ${CMAKE_BINARY_DIR}/latex
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Generating latex to ${CMAKE_BINARY_DIR}/latex")

  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/conf.py.in ${CMAKE_CURRENT_SOURCE_DIR}/source/conf.py
    @ONLY)
else()
  message("-- - sphinx-build *not* found, disabling build of sphinx documentation.")
endif()

if(DOXYGEN)
  add_custom_target(doxygen
    COMMAND doxygen doxyfile
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/docs
  )
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doxyfile.in ${CMAKE_BINARY_DIR}/docs/doxyfile
    @ONLY)
  if(SPHINX_BUILD)
    add_dependencies(html doxygen)
    add_dependencies(latex doxygen)
  endif()
endif()