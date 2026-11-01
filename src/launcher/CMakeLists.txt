add_executable(Launch WIN32
    launcher_main.cpp
)

target_include_directories(Launch PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

if(MSVC)
    target_link_libraries(Launch PRIVATE
        Shell32
        User32
        Advapi32
    )
endif()

