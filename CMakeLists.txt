project                     (Dionysus)
cmake_minimum_required      (VERSION 2.4)

option                      (logging            "Build Dionysus with logging on"        ON)
# option                      (logging            "Build Dionysus with logging on"        ON)
option                      (counters           "Build Dionysus with counters on"       OFF)
option                      (debug              "Build Dionysus with debugging on"      ON)
option                      (optimize           "Build Dionysus with optimization"      ON)
option                      (use_cgal           "Build examples and python bindings that use CGAL"       ON)
option                      (use_dsrpdb         "Build examples that use DSR-PDB"       OFF)
option                      (use_synaps         "Build examples that use SYNAPS"        OFF)


SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DBOOST_NO_INTRINSIC_INT64_T")

if(logging)
message("logging is ON")
else(logging)
message("logging is OFF")
endif(logging)


# Find everything that's always required
find_package                (Boost REQUIRED COMPONENTS program_options python serialization signals)
find_package                (Doxygen)
if                          (use_dsrpdb)
    find_library            (dsrpdb_LIBRARY             NAMES dsrpdb)
    find_path               (dsrpdb_INCLUDE_DIR         dsrpdb/Protein.h)
endif                       (use_dsrpdb)

# CGAL
if                          (use_cgal)
    find_package            (CGAL QUIET)
endif                       (use_cgal)
#add_definitions             (-DCGAL_NO_ASSERTIONS -DCGAL_NO_PRECONDITIONS)

# SYNAPS
if                          (use_synaps)
    add_definitions         (-DBOOST_UBLAS_TYPE_CHECK=0)
    find_library            (synaps_LIBRARY             NAMES synaps)
    set                     (synaps_libraries           ${synaps_LIBRARY}
                                                        ${gmp_LIBRARY}
                                                        ${gmpxx_LIBRARY})
endif                       (use_synaps)

# Debugging
if                          (debug)
    if                      (optimize)
            set             (cxx_flags                  ${CMAKE_CXX_FLAGS_RELWITHDEBINFO})
    else                    (optimize)
            set             (cxx_flags                  ${CMAKE_CXX_FLAGS_DEBUG})
    endif                   (optimize)
else                        (debug)
    if                      (optimize)
            set             (cxx_flags                  ${CMAKE_CXX_FLAGS_RELEASE})
    else                    (optimize)
            set             (cxx_flags                  ${CMAKE_CXX_FLAGS})
    endif                   (optimize)
endif                       (debug)
add_definitions             (${cxx_flags})

# Logging
if                          (logging)
    find_library            (rlog_LIBRARY               NAMES rlog)
    find_path               (rlog_INCLUDE_DIR           rlog/rlog.h)
    set                     (rlog_INCLUDE_DIR           ${rlog_INCLUDE_DIR})
    add_definitions         (-DLOGGING -DRLOG_COMPONENT=dionysus)
    set                     (libraries                  ${libraries} ${rlog_LIBRARY})
endif                       (logging)

# Counters
if                          (counters)
    add_definitions         (-DCOUNTERS)
endif                       (counters)


# Set includes
include_directories         (${CMAKE_CURRENT_BINARY_DIR}
                             ${CMAKE_CURRENT_SOURCE_DIR}/include
                             ${Boost_INCLUDE_DIR}
                             ${dsrpdb_INCLUDE_DIR}
                             ${cwd_INCLUDE_DIR}
                             ${rlog_INCLUDE_DIR})

# Doxygen (FIXME)
if                          (DOXYGEN_FOUND)
#   add_custom_target       (docs ALL
#                           ${DOXYGEN_EXECUTABLE} Doxyfile
#                           DEPENDS Doxyfile)
endif                       (DOXYGEN_FOUND)

# fixing doxygen
# set (DOXYFILE_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/doc")
set (DOXYFILE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
# if (DOXYGEN_FOUND STREQUAL "NO")
# message("creating ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile ...")
# configure_file(
	# ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Doxyfile.in
	# ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
# )
# add doxygen as target
add_custom_target(
    doxygen COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYFILE_SOURCE_DIR}/Doxyfile
    COMMENT "Writing documentation to ${DOXYFILE_OUTPUT_DIR}..."
)
  
# cleanup $build/api-doc on "make clean"
set_property(DIRECTORY APPEND PROPERTY
    ADDITIONAL_MAKE_CLEAN_FILES doc)

# add doxygen as dependency to doc-target
get_target_property(DOC_TARGET doc TYPE)
if(NOT DOC_TARGET)
    add_custom_target(doc)
endif()
add_dependencies(doc doxygen)




# Process subdirectories
add_subdirectory            (examples)
add_subdirectory            (tests)
add_subdirectory            (tools)
add_subdirectory            (bindings)
