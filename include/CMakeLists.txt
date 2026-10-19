add_custom_command(
    OUTPUT amgcl_c_di.h
    COMMAND ${CMAKE_COMMAND} -D inputfile=${CMAKE_CURRENT_SOURCE_DIR}/amgcl_c_tv_ti.h.in
                             -D BLOCKSIZES="${BLOCKSIZES}"
                             -D outputfile=${CMAKE_BINARY_DIR}/include/amgcl_c_di.h
                             -D TVTI="DI"
                             -D TV_TI="di"
                             -D TV="double"
                             -D TI="int"
                             -P ${CMAKE_SOURCE_DIR}/cmake/specialize.cmake
                             DEPENDS amgcl_c_tv_ti.h.in
                             )

add_custom_command(
  OUTPUT amgcl_c_dl.h
  COMMAND ${CMAKE_COMMAND} -D inputfile=${CMAKE_CURRENT_SOURCE_DIR}/amgcl_c_tv_ti.h.in
                           -D BLOCKSIZES="${BLOCKSIZES}"
                           -D outputfile=${CMAKE_BINARY_DIR}/include/amgcl_c_dl.h
                           -D TVTI="DL"
                           -D TV_TI="dl"
                           -D TV="double"
                           -D TI="${INT64_T}"
                           -P ${CMAKE_SOURCE_DIR}/cmake/specialize.cmake
                           DEPENDS amgcl_c_tv_ti.h.in
                           )
                         
add_custom_command(
  OUTPUT amgcl_c.h
  COMMAND ${CMAKE_COMMAND} -E cat ${HEADERS} > amgcl_c.h
  DEPENDS ${HEADERS}
  )

# Consumers include <amgcl_c/amgcl_c.h>; the installation puts the header at
# ${CMAKE_INSTALL_INCLUDEDIR}/amgcl_c/amgcl_c.h. Mirror that layout into the
# build tree (the exported BUILD_INTERFACE include dir is this
# ${CMAKE_CURRENT_BINARY_DIR}) so downstream projects can also build against
# an uninstalled amgcl_c build registered in the user package registry.
add_custom_command(
  OUTPUT amgcl_c/amgcl_c.h
  COMMAND ${CMAKE_COMMAND} -E make_directory amgcl_c
  COMMAND ${CMAKE_COMMAND} -E copy amgcl_c.h amgcl_c/amgcl_c.h
  DEPENDS amgcl_c.h
  )

add_custom_target(amgcl_c_h DEPENDS amgcl_c.h amgcl_c/amgcl_c.h)
