# this will look for MAPL and all the other stuff we don't needd
#add_subdirectory(gocart)

list(APPEND _gocart_srcs gocart/Process_Library/gsd_chem_constants.F90
                         gocart/Process_Library/gsd_chem_config.F90
                         gocart/Process_Library/plume_data_mod.F90
                         gocart/Process_Library/plume_rise_mod.F90
                         gocart/Process_Library/plume_scalar_mod.F90
                         gocart/Process_Library/plume_zero_mod.F90
                         gocart/Process_Library/Chem_MieTableMod2G.F90
                         gocart/Process_Library/GOCART2G_Process.F90)
add_library(gocart ${_gocart_srcs})

target_compile_definitions(gocart PRIVATE "HAS_NETCDF3")
set_target_properties(gocart PROPERTIES Fortran_MODULE_DIRECTORY
                                        ${CMAKE_CURRENT_BINARY_DIR}/mod)
target_include_directories(gocart PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/mod>
                                         $<INSTALL_INTERFACE:mod>)
target_link_libraries(gocart PRIVATE NetCDF::NetCDF_Fortran)

install(TARGETS gocart
        EXPORT gocart-targets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION lib
)
# Export our configuration
install(EXPORT gocart-targets
        FILE gocart-config.cmake
        DESTINATION lib/cmake
)
