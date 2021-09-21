set(LIB_NAME inotify-cpp)
set(LIB_COMPANY inotify-cpp)
set(LIB_SRCS NotifierBuilder.cpp Event.cpp FileSystemEvent.cpp Inotify.cpp Notification.cpp)
set(LIB_HEADER
        include/inotify-cpp/NotifierBuilder.h
        include/inotify-cpp/Event.h
        include/inotify-cpp/FileSystemEvent.h
        include/inotify-cpp/Inotify.h
        include/inotify-cpp/Notification.h)

cmake_minimum_required(VERSION 3.8)
project(${LIB_NAME} VERSION 0.2.0)
include(GNUInstallDirs)

macro(configure_target target)
        set_target_properties(${target} PROPERTIES
                LINKER_LANGUAGE CXX
                OUTPUT_NAME ${LIB_NAME}
                SOVERSION ${PROJECT_VERSION}
        )

        install(TARGETS ${target} EXPORT ${target}Targets
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
                INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
        install(EXPORT ${target}Targets
                DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME}
                FILE ${target}Targets.cmake
                NAMESPACE ${LIB_COMPANY}::)

        target_compile_features(${target} PUBLIC cxx_std_11)
        target_include_directories(${target} PUBLIC
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                $<INSTALL_INTERFACE:include>)

endmacro(configure_target target)

macro(configure_target_interface_lib target)

        install(TARGETS ${target} 
                EXPORT ${target}Targets
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
                INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
        install(EXPORT ${target}Targets
                DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${target}
                FILE ${target}Targets.cmake
                NAMESPACE ${LIB_COMPANY}::)

        target_include_directories(${target} INTERFACE
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                $<INSTALL_INTERFACE:include>)
        #target_link_libraries(${target} PUBLIC inotify-filesystem-adapter)
endmacro(configure_target_interface_lib)

# add export for adapter
install(TARGETS inotify-filesystem-adapter EXPORT inotify-filesystem-adapterExport)
install(EXPORT inotify-filesystem-adapterExport DESTINATION ${CMAKE_INSTALL_LIBDIR})
configure_target_interface_lib(inotify-filesystem-adapter)

# library definition
if(BUILD_SHARED_LIBS)
        add_library(${LIB_NAME}-shared SHARED ${LIB_SRCS} ${LIB_HEADER})
        list(APPEND PACKAGE_LIBRARIES ${LIB_NAME})

        configure_target(${LIB_NAME}-shared)
        if(NOT BUILD_STATIC_LIBS)
                add_library(${LIB_COMPANY}::${LIB_NAME} ALIAS ${LIB_NAME}-shared)
        endif()
endif()

if(BUILD_STATIC_LIBS)
        add_library(${LIB_NAME}-static STATIC ${LIB_SRCS} ${LIB_HEADER})
        #list(APPEND PACKAGE_LIBRARIES ${LIB_NAME}-static)

        configure_target(${LIB_NAME}-static)
        if(NOT BUILD_SHARED_LIBS)
                add_library(${LIB_COMPANY}::${LIB_NAME} ALIAS ${LIB_NAME}-static)
        endif()
endif()

if(BUILD_SHARED_LIBS AND BUILD_STATIC_LIBS)
        add_library(${LIB_COMPANY}::${LIB_NAME} ALIAS ${LIB_NAME}-shared)
        list(APPEND PACKAGE_LIBRARIES ${LIB_NAME})
endif()

install(DIRECTORY ${CMAKE_INSTALL_INCLUDEDIR}/${LIB_NAME} DESTINATION include)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(${LIB_NAME}ConfigVersion.cmake
        COMPATIBILITY SameMajorVersion)
configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/cmake/${LIB_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${LIB_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME}
        )
install(FILES "${PROJECT_BINARY_DIR}/${LIB_NAME}ConfigVersion.cmake"
        "${PROJECT_BINARY_DIR}/${LIB_NAME}Config.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME})

export(EXPORT ${PROJECT_NAME}-sharedTargets FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake)

# install(FILES ${LIB_NAME}Config.cmake ${CMAKE_CURRENT_BINARY_DIR}/${LIB_NAME}ConfigVersion.cmake
#         DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME})

# ament_export_libraries(${LIB_NAME})

#################
set(LIB_NAME inotify-filesystem-adapter)
write_basic_package_version_file(${LIB_NAME}ConfigVersion.cmake
        COMPATIBILITY SameMajorVersion)
configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/cmake/${LIB_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${LIB_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME}
        )
# install(FILES ${LIB_NAME}Config.cmake ${CMAKE_CURRENT_BINARY_DIR}/${LIB_NAME}ConfigVersion.cmake
#         DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME})
install(EXPORT ${LIB_NAME}Targets DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME})
install(FILES "${PROJECT_BINARY_DIR}/${LIB_NAME}ConfigVersion.cmake"
              "${PROJECT_BINARY_DIR}/${LIB_NAME}Config.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME})
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ DESTINATION include)

install(FILES ${PROJECT_SOURCE_DIR}/package.xml DESTINATION share/${PROJECT_NAME})


# ament_export_include_directories(${PROJECT_SOURCE_DIR}/include)

# #ament_export_dependencies(${dependencies})
# ament_package()