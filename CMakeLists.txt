cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

project(OpenCTM LANGUAGES C)

add_library(OpenCTM SHARED
    lib/compressMG1.c
    lib/compressMG2.c
    lib/compressRAW.c
    lib/internal.h
    lib/liblzma/Alloc.c
    lib/liblzma/Alloc.h
    lib/liblzma/LzFind.c
    lib/liblzma/LzFind.h
    lib/liblzma/LzHash.h
    lib/liblzma/LzmaDec.c
    lib/liblzma/LzmaDec.h
    lib/liblzma/LzmaEnc.c
    lib/liblzma/LzmaEnc.h
    lib/liblzma/LzmaLib.c
    lib/liblzma/LzmaLib.h
    lib/liblzma/NameMangle.h
    lib/liblzma/Types.h
    lib/openctm.c
    lib/openctm.h
    lib/openctmpp.h
    lib/stream.c
)

target_include_directories(OpenCTM
    PRIVATE
        $<BUILD_INTERFACE:lib lib/liblzma>
)

option(OPENCTM_INSTALL "Add installation rules" ON)
if(OPENCTM_INSTALL)
    include(GNUInstallDirs)

    install(
        FILES lib/openctm.h lib/openctmpp.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/OpenCTM
    )

    install(
        TARGETS OpenCTM
        EXPORT OpenCTMTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
    install(
        EXPORT OpenCTMTargets
        NAMESPACE OpenCTM::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenCTM
    )

    include(CMakePackageConfigHelpers)

    configure_package_config_file(
        cmake/OpenCTMConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/OpenCTMConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenCTM
    )
    install(
        FILES ${CMAKE_CURRENT_BINARY_DIR}/OpenCTMConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenCTM
    )
endif()
