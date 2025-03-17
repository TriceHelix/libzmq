cmake_minimum_required(VERSION 3.28 FATAL_ERROR)

project(
    zmq_tests
    LANGUAGES CXX
)

message(STATUS "Configuring ZMQ unit tests")

set(tests
    test_ancillaries
    test_system
    test_pair_inproc
    test_pair_tcp
    test_reqrep_inproc
    test_reqrep_tcp
    test_hwm
    test_hwm_pubsub
    test_reqrep_device
    test_sub_forward
    test_invalid_rep
    test_msg_flags
    test_msg_ffn
    test_connect_resolve
    test_immediate
    test_last_endpoint
    test_term_endpoint
    test_router_mandatory
    test_probe_router
    test_stream
    test_stream_empty
    test_stream_disconnect
    test_disconnect_inproc
    test_unbind_wildcard
    test_ctx_options
    test_ctx_destroy
    test_security_no_zap_handler
    test_security_null
    test_security_plain
    test_security_zap
    test_iov
    test_spec_req
    test_spec_rep
    test_spec_dealer
    test_spec_router
    test_spec_pushpull
    test_req_correlate
    test_req_relaxed
    test_conflate
    test_inproc_connect
    test_issue_566
    test_shutdown_stress
    test_timeo
    test_diffserv
    test_connect_rid
    test_xpub_nodrop
    test_pub_invert_matching
    test_setsockopt
    test_sockopt_hwm
    test_heartbeats
    test_atomics
    test_bind_src_address
    test_capabilities
    test_metadata
    test_router_handover
    test_srcfd
    test_stream_timeout
    test_xpub_manual
    test_xpub_topic
    test_xpub_welcome_msg
    test_xpub_verbose
    test_base85
    test_bind_after_connect_tcp
    test_sodium
    test_monitor
    test_socket_null
    test_reconnect_ivl
    test_reconnect_options
    test_tcp_accept_filter
    test_mock_pub_sub
)

if(${HAVE_IPC})
    list(
        APPEND tests
        test_ipc_wildcard
        test_pair_ipc
        test_reqrep_ipc
        test_rebind_ipc
    )
endif()

if(NOT ${WIN32})
    list(
        APPEND tests
        test_security_gssapi
        test_socks
        test_connect_null_fuzzer
        test_bind_null_fuzzer
        test_connect_fuzzer
        test_bind_fuzzer
        test_proxy
        test_proxy_hwm
        test_proxy_single_socket
        test_proxy_steerable
        test_proxy_terminate
        test_getsockopt_memset
        test_filter_ipc
        test_stream_exceeds_buffer
        test_router_mandatory_hwm
        test_use_fd
        test_zmq_poll_fd
        test_many_sockets
    )

    if(${HAVE_FORK})
        list(APPEND tests test_fork)
    endif()

    if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
        list(APPEND tests test_abstract_ipc)
    endif()
endif()

if(${ZMQ_ENABLE_DRAFTS})
    list(
        APPEND
        tests
        test_poller
        test_thread_safe
        test_client_server
        test_timers
        test_radio_dish
        test_scatter_gather
        test_dgram
        test_app_meta
        test_router_notify
        test_xpub_manual_last_value
        test_peer
        test_msg_init
        test_channel
        test_hello_msg
        test_disconnect_msg
        test_hiccup_msg
        test_zmq_ppoll_fd
        test_xsub_verbose
        test_pubsub_topics_count
    )

    if(${HAVE_FORK})
        list(APPEND tests test_zmq_ppoll_signals)
    endif()

    if(${HAVE_BUSY_POLL})
        list(APPEND tests test_busy_poll)
    endif()
endif()

if(${ZMQ_ENABLE_WS})
  list(APPEND tests test_ws_transport)
endif()

if(${WIN32})
    add_definitions(-D_WINSOCK_DEPRECATED_NO_WARNINGS)
    link_libraries(ws2_32)
endif()

add_library(
    unity STATIC
    "${PROJECT_SOURCE_DIR}/../external/unity/unity.c"
    "${PROJECT_SOURCE_DIR}/../external/unity/unity.h"
    "${PROJECT_SOURCE_DIR}/../external/unity/unity_internals.h"
)

set_target_properties(unity PROPERTIES PUBLIC_HEADER "${PROJECT_SOURCE_DIR}/../external/unity/unity.h")
target_compile_definitions(unity PUBLIC "UNITY_USE_COMMAND_LINE_ARGS" "UNITY_EXCLUDE_FLOAT")
target_include_directories(unity PUBLIC "${PROJECT_SOURCE_DIR}/../external/unity")

set(
    testutil_sources
    testutil.cpp
    testutil.hpp
    testutil_monitoring.cpp
    testutil_monitoring.hpp
    testutil_security.cpp
    testutil_security.hpp
    testutil_unity.cpp
    testutil_unity.hpp
)

add_library(testutil STATIC ${testutil_sources})
target_link_libraries(testutil libzmq ${OPTIONAL_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} unity)

if(${MSVC})
    if(${MSVC_VERSION} LESS 1700)
        set_source_files_properties("${CMAKE_CURRENT_LIST_DIR}/../external/unity/unity.c" PROPERTIES LANGUAGE CXX)
    endif()

    if(${MSVC_VERSION} LESS 1600)
        target_compile_definitions(unity PUBLIC "UNITY_EXCLUDE_STDINT_H")
    endif()
endif()

# add include dirs for all targets
include_directories("${ZeroMQ_SOURCE_DIR}/../include" "${ZeroMQ_BINARY_DIR}")

if(${WIN32})
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
endif()

foreach(test ${tests})
    add_executable(${test} "${PROJECT_SOURCE_DIR}/${test}.cpp")

    set_target_properties(
        ${test} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED TRUE
        RUNTIME_OUTPUT_DIRECTORY "$<TARGET_FILE_DIR:libzmq>"
    )

    target_link_libraries(${test} testutil)

    if(${CMAKE_SYSTEM_NAME} STREQUAL "QNX")
        target_link_libraries(${test} socket)
        target_link_libraries(${test} m)
    endif()

    add_test(
        NAME ${test}
        WORKING_DIRECTORY "$<TARGET_FILE_DIR:libzmq>"
        COMMAND "$<TARGET_FILE:${test}>"
    )

    set_tests_properties(${test} PROPERTIES TIMEOUT 10)
    set_tests_properties(${test} PROPERTIES SKIP_RETURN_CODE 77)
endforeach()

# override timeout for these tests
set_tests_properties(test_heartbeats PROPERTIES TIMEOUT 60)

if(${WIN32} AND ${ZMQ_ENABLE_DRAFTS})
  set_tests_properties(test_radio_dish PROPERTIES TIMEOUT 30)
endif()

set_tests_properties(test_security_zap PROPERTIES TIMEOUT 60)
set_tests_properties(test_reconnect_ivl PROPERTIES TIMEOUT 15)

list(LENGTH tests num_tests)
message(STATUS "Created ${num_tests} unit tests")
