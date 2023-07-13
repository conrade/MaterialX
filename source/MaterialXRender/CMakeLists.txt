cmake_policy(SET CMP0074 NEW)
include_directories(
    ${EXTERNAL_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/../)

file(GLOB_RECURSE materialx_source "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")
file(GLOB_RECURSE materialx_inlined "${CMAKE_CURRENT_SOURCE_DIR}/*.inl")
file(GLOB_RECURSE materialx_headers "${CMAKE_CURRENT_SOURCE_DIR}/*.h*")

if(NOT MATERIALX_BUILD_OIIO)
    list(REMOVE_ITEM materialx_source "${CMAKE_CURRENT_SOURCE_DIR}/OiioImageLoader.cpp")
endif()

assign_source_group("Source Files" ${materialx_source})
assign_source_group("Source Files" ${materialx_inlined})
assign_source_group("Header Files" ${materialx_headers})

if(UNIX)
    add_compile_options(-Wno-unused-function)
endif()

add_library(MaterialXRender ${materialx_source} ${materialx_headers} ${materialx_inlined})

add_definitions(-DMATERIALX_RENDER_EXPORTS)

target_link_libraries(
    MaterialXRender
    MaterialXGenShader
    ${CMAKE_DL_LIBS})

if(MATERIALX_BUILD_OIIO)
    set(OPENIMAGEIO_ROOT_DIR ${MATERIALX_OIIO_DIR})
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/External/OpenImageIO")
    find_package(OpenImageIO REQUIRED)
    if(OPENIMAGEIO_FOUND)
        include_directories(${OPENIMAGEIO_INCLUDE_DIR})
        target_link_libraries(MaterialXRender 
            ${OPENIMAGEIO_LIBRARIES}
            /rel/rez/dwa/openimageio/2.3.20.0.x.0.3.0.0/os-CentOS-7/cpp_std-17.0/boost-1.73.0.x.2/openexr-2.5.7.x.0/python-3.7.7.x/zlib-1.2.8.x.2/qt-5.15.2.x.3/opencolorio-2.0.2.x.0/freetype-2.8.1.x/lib64/libOpenImageIO_Util.so
        )
        # Also needed by MaterialXTest:
        set(OPENIMAGEIO_FOUND "${OPENIMAGEIO_FOUND}" PARENT_SCOPE)
        set(OPENIMAGEIO_INCLUDE_DIR "${OPENIMAGEIO_INCLUDE_DIR}" PARENT_SCOPE)
        set(OPENIMAGEIO_LIBRARIES "${OPENIMAGEIO_LIBRARIES}" PARENT_SCOPE)
        set(OPENIMAGEIO_LIBRARY_DIRS "${OPENIMAGEIO_LIBRARY_DIRS}" PARENT_SCOPE)
        set(OPENIMAGEIO_ROOT_DIR "${OPENIMAGEIO_ROOT_DIR}" PARENT_SCOPE)

        find_package(OpenEXR REQUIRED)
        target_link_libraries(MaterialXRender OpenEXR::OpenEXR)
    endif()
endif()

set_target_properties(
    MaterialXRender PROPERTIES
    OUTPUT_NAME MaterialXRender${MATERIALX_LIBNAME_SUFFIX}
    COMPILE_FLAGS "${EXTERNAL_COMPILE_FLAGS}"
    LINK_FLAGS "${EXTERNAL_LINK_FLAGS}"
    INSTALL_RPATH "${MATERIALX_SAME_DIR_RPATH}"
    VERSION "${MATERIALX_LIBRARY_VERSION}"
    SOVERSION "${MATERIALX_MAJOR_VERSION}")

install(TARGETS MaterialXRender
    EXPORT MaterialX
    ARCHIVE DESTINATION ${MATERIALX_INSTALL_LIB_PATH}
    LIBRARY DESTINATION ${MATERIALX_INSTALL_LIB_PATH}
    RUNTIME DESTINATION bin)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/"
    DESTINATION ${MATERIALX_INSTALL_INCLUDE_PATH}/MaterialXRender/ MESSAGE_NEVER
    FILES_MATCHING
    PATTERN "*.h*"
    PATTERN "*.inl")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/MaterialXRender.pdb"
    DESTINATION "${MATERIALX_INSTALL_LIB_PATH}/" OPTIONAL)
