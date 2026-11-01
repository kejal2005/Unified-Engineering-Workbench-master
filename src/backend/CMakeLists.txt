add_library(workbench_core
    src/core/error.cpp
    src/core/config.cpp
    src/core/logger.cpp
    src/core/cancel_token.cpp
)

target_include_directories(workbench_core PUBLIC
    include
    ${CMAKE_SOURCE_DIR}/third_party
)

add_executable(workbench_server
    src/main.cpp
)

target_link_libraries(workbench_server PRIVATE
    workbench_core
)

if(BUILD_TESTING)
    add_executable(workbench_core_tests
        tests/core/core_tests.cpp
    )
    target_link_libraries(workbench_core_tests PRIVATE
        workbench_core
    )
    add_test(NAME core_tests COMMAND workbench_core_tests)
endif()

