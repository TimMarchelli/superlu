set(headers
    supermatrix.h
    slu_Cnames.h
    slu_dcomplex.h
    slu_scomplex.h
    slu_util.h
    superlu_enum_consts.h
    ${CMAKE_CURRENT_BINARY_DIR}/superlu_config.h
)

set(source_property_change_list)

set(sources
  superlu_timer.c
  util.c
  memory.c
  get_perm_c.c
  mmd.c
  sp_coletree.c
  sp_preorder.c
  sp_ienv.c
  relax_snode.c
  heap_relax_snode.c
  colamd.c
  ilu_relax_snode.c
  ilu_heap_relax_snode.c
  mark_relax.c
  mc64ad.c
  qselect.c
  input_error.c
)

list(APPEND source_property_change_list superlu_timer.c)

if(enable_single)
  list(APPEND headers
    slu_sdefs.h
  )

  list(APPEND sources
    slacon2.c
    smach.c
    sgssv.c
    sgssvx.c
    ssp_blas2.c
    ssp_blas3.c
    sgscon.c
    slangs.c
    sgsequ.c
    slaqgs.c
    spivotgrowth.c
    sgsrfs.c
    sgstrf.c
    sgstrs.c
    scopy_to_ucol.c
    ssnode_dfs.c
    ssnode_bmod.c
    spanel_dfs.c
    spanel_bmod.c
    sreadhb.c
    sreadrb.c
    sreadtriple.c
    sreadMM.c
    scolumn_dfs.c
    scolumn_bmod.c
    spivotL.c
    spruneL.c
    smemory.c
    sutil.c
    smyblas2.c
    sgsisx.c
    sgsitrf.c
    sldperm.c
    ilu_sdrop_row.c
    ilu_ssnode_dfs.c
    ilu_scolumn_dfs.c
    ilu_spanel_dfs.c
    ilu_scopy_to_ucol.c
    ilu_spivotL.c
    sdiagonal.c
  )
  list(APPEND source_property_change_list smach.c)
endif()

if(enable_double)
  list(APPEND headers
    slu_ddefs.h
  )

  list(APPEND sources
    dlacon2.c
    dmach.c
    dgssv.c
    dgssvx.c
    dsp_blas2.c
    dsp_blas3.c
    dgscon.c
    dlangs.c
    dgsequ.c
    dlaqgs.c
    dpivotgrowth.c
    dgsrfs.c
    dgstrf.c
    dgstrs.c
    dcopy_to_ucol.c
    dsnode_dfs.c
    dsnode_bmod.c
    dpanel_dfs.c
    dpanel_bmod.c
    dreadhb.c
    dreadrb.c
    dreadtriple.c
    dreadMM.c
    dcolumn_dfs.c
    dcolumn_bmod.c
    dpivotL.c
    dpruneL.c
    dmemory.c
    dutil.c
    dmyblas2.c
    dgsisx.c
    dgsitrf.c
    dldperm.c
    ilu_ddrop_row.c
    ilu_dsnode_dfs.c
    ilu_dcolumn_dfs.c
    ilu_dpanel_dfs.c
    ilu_dcopy_to_ucol.c
    ilu_dpivotL.c
    ddiagonal.c
    dGetDiagU.c
  )
  list(APPEND source_property_change_list dmach.c)
endif()

if(enable_complex)
  list(APPEND headers
    slu_cdefs.h
  )

  list(APPEND sources
    clacon2.c
    scsum1.c
    icmax1.c
    scomplex.c
    cgssv.c
    cgssvx.c
    csp_blas2.c
    csp_blas3.c
    cgscon.c
    clangs.c
    cgsequ.c
    claqgs.c
    cpivotgrowth.c
    cgsrfs.c
    cgstrf.c
    cgstrs.c
    ccopy_to_ucol.c
    csnode_dfs.c
    csnode_bmod.c
    cpanel_dfs.c
    cpanel_bmod.c
    creadhb.c
    creadrb.c
    creadtriple.c
    creadMM.c
    ccolumn_dfs.c
    ccolumn_bmod.c
    cpivotL.c
    cpruneL.c
    cmemory.c
    cutil.c
    cmyblas2.c
    cgsisx.c
    cgsitrf.c
    cldperm.c
    ilu_cdrop_row.c
    ilu_csnode_dfs.c
    ilu_ccolumn_dfs.c
    ilu_cpanel_dfs.c
    ilu_ccopy_to_ucol.c
    ilu_cpivotL.c
    cdiagonal.c
  )
endif()

if(enable_complex16)
  list(APPEND headers
    slu_zdefs.h
  )

  list(APPEND sources
    zlacon2.c
    dzsum1.c
    izmax1.c
    dcomplex.c
    zgssv.c
    zgssvx.c
    zsp_blas2.c
    zsp_blas3.c
    zgscon.c
    zlangs.c
    zgsequ.c
    zlaqgs.c
    zpivotgrowth.c
    zgsrfs.c
    zgstrf.c
    zgstrs.c
    zcopy_to_ucol.c
    zsnode_dfs.c
    zsnode_bmod.c
    zpanel_dfs.c
    zpanel_bmod.c
    zreadhb.c
    zreadrb.c
    zreadtriple.c
    zreadMM.c
    zcolumn_dfs.c
    zcolumn_bmod.c
    zpivotL.c
    zpruneL.c
    zmemory.c
    zutil.c
    zmyblas2.c
    zgsisx.c
    zgsitrf.c
    zldperm.c
    ilu_zdrop_row.c
    ilu_zsnode_dfs.c
    ilu_zcolumn_dfs.c
    ilu_zpanel_dfs.c
    ilu_zcopy_to_ucol.c
    ilu_zpivotL.c
    zdiagonal.c
  )
endif()

# disable optimization on some files
foreach(src ${source_property_change_list})
  if(MSVC)
    set_source_files_properties(${src} PROPERTIES COMPILE_FLAGS /Od)
  else()
    set_source_files_properties(${src} PROPERTIES COMPILE_FLAGS -O0)
  endif()
endforeach()

# add library
add_library(superlu ${sources})

# add public propogation to external BLAS and compile definition
if(BLAS_FOUND)
  target_link_libraries(superlu PUBLIC ${BLAS_LIB})
  target_compile_definitions(superlu PRIVATE USE_VENDOR_BLAS)

# private link to local BLAS
else()
  target_link_libraries(superlu PRIVATE ${BLAS_LIB})
endif()

# add METIS
if(METIS_FOUND)
  target_include_directories(superlu PUBLIC ${METIS_INCLUDE})
  target_link_libraries(superlu PUBLIC ${METIS_LIB})
endif()

if (NOT WIN32)
  target_link_libraries(superlu PUBLIC m)
endif()

target_include_directories(superlu PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:include>
)

set_target_properties(superlu PROPERTIES
  VERSION ${PROJECT_VERSION} 
  SOVERSION ${VERSION_MAJOR}
)

# create an alias
add_library(superlu::superlu ALIAS superlu)

# install suplerlu
install(TARGETS superlu
  EXPORT superluTargets
  RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
  LIBRARY DESTINATION ${INSTALL_LIBRARY_DIR}
  ARCHIVE DESTINATION ${INSTALL_LIBRARY_DIR}
  INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}
)

install(FILES ${headers}
  DESTINATION ${INSTALL_INCLUDE_DIR}
)

# generate config files
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  superluConfigVersion.cmake
  VERSION ${PACKAGE_VERSION}
  COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
  superluConfig.cmake.in superluConfig.cmake
  INSTALL_DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/superluConfig.cmake"
)

# install configuration files
install(EXPORT superluTargets
  FILE superluTargets.cmake
  NAMESPACE superlu::
  DESTINATION ${INSTALL_CONFIG_DIR}
)

install(
  FILES
      "${CMAKE_CURRENT_BINARY_DIR}/superluConfig.cmake"
      "${CMAKE_CURRENT_BINARY_DIR}/superluConfigVersion.cmake"
  DESTINATION ${INSTALL_CONFIG_DIR}
)
