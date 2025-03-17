cmake_minimum_required(VERSION 3.28 FATAL_ERROR)

project(
    ZeroMQ
    LANGUAGES C CXX
)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

include(CheckCSourceRuns)
include(CheckCXXSymbolExists)
include(CheckIncludeFiles)
include(CheckStructHasMember)

include(TestZMQVersion)
include(ZMQSourceRunChecks)

option(ZMQ_SHARED "Build ZMQ as a dynamic library" TRUE)
option(ZMQ_ENABLE_DRAFTS "Build and install draft classes and methods" FALSE)
option(ZMQ_ENABLE_WS "Enable WebSocket transport" ${ZMQ_ENABLE_DRAFTS})
option(ZMQ_ENABLE_RADIX_TREE "Use radix tree implementation to manage subscriptions" ${ZMQ_ENABLE_DRAFTS})
option(ZMQ_WITH_MILITANT "Enable militant assertions" FALSE)
option(ZMQ_PEDANTIC "Enable pedantic compiler warnings" FALSE)
option(ZMQ_BUILD_TESTS "Build unit tests." FALSE)

if("${CMAKE_CURRENT_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_BINARY_DIR}")
    message(
        FATAL_ERROR
        "CMake generation is not allowed within the source directory! \
    Remove the CMakeCache.txt file and try again from another folder, e.g.: \
    \
      rm CMakeCache.txt \
      mkdir cmake-make \
      cd cmake-make \
      cmake ..")
endif()

if(NOT DEFINED CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

set(
    API_POLLER "" CACHE STRING
    "Choose polling system for zmq_poll(er)_*. Valid values are
    poll or select [default=poll unless POLLER=select]"
)

set(
    POLLER "" CACHE STRING
    "Choose polling system for I/O threads. Valid values are
    kqueue, epoll, devpoll, pollset, poll or select [default=autodetect]"
)

set(
    sources
    "address.cpp"
    "channel.cpp"
    "client.cpp"
    "clock.cpp"
    "ctx.cpp"
    "curve_mechanism_base.cpp"
    "curve_client.cpp"
    "curve_server.cpp"
    "dealer.cpp"
    "devpoll.cpp"
    "dgram.cpp"
    "dist.cpp"
    "endpoint.cpp"
    "err.cpp"
    "fq.cpp"
    "io_object.cpp"
    "io_thread.cpp"
    "ip.cpp"
    "ipc_address.cpp"
    "ipc_connecter.cpp"
    "ipc_listener.cpp"
    "kqueue.cpp"
    "lb.cpp"
    "mailbox.cpp"
    "mailbox_safe.cpp"
    "mechanism.cpp"
    "mechanism_base.cpp"
    "metadata.cpp"
    "msg.cpp"
    "mtrie.cpp"
    "norm_engine.cpp"
    "object.cpp"
    "options.cpp"
    "own.cpp"
    "null_mechanism.cpp"
    "pair.cpp"
    "peer.cpp"
    "pgm_receiver.cpp"
    "pgm_sender.cpp"
    "pgm_socket.cpp"
    "pipe.cpp"
    "plain_client.cpp"
    "plain_server.cpp"
    "poll.cpp"
    "poller_base.cpp"
    "polling_util.cpp"
    "pollset.cpp"
    "proxy.cpp"
    "pub.cpp"
    "pull.cpp"
    "push.cpp"
    "random.cpp"
    "raw_encoder.cpp"
    "raw_decoder.cpp"
    "raw_engine.cpp"
    "reaper.cpp"
    "rep.cpp"
    "req.cpp"
    "router.cpp"
    "select.cpp"
    "server.cpp"
    "session_base.cpp"
    "signaler.cpp"
    "socket_base.cpp"
    "socks.cpp"
    "socks_connecter.cpp"
    "stream.cpp"
    "stream_engine_base.cpp"
    "sub.cpp"
    "tcp.cpp"
    "tcp_address.cpp"
    "tcp_connecter.cpp"
    "tcp_listener.cpp"
    "thread.cpp"
    "trie.cpp"
    "radix_tree.cpp"
    "v1_decoder.cpp"
    "v1_encoder.cpp"
    "v2_decoder.cpp"
    "v2_encoder.cpp"
    "v3_1_encoder.cpp"
    "xpub.cpp"
    "xsub.cpp"
    "zmq.cpp"
    "zmq_utils.cpp"
    "decoder_allocators.cpp"
    "socket_poller.cpp"
    "timers.cpp"
    "config.hpp"
    "radio.cpp"
    "dish.cpp"
    "udp_engine.cpp"
    "udp_address.cpp"
    "scatter.cpp"
    "gather.cpp"
    "ip_resolver.cpp"
    "zap_client.cpp"
    "zmtp_engine.cpp"
    # headers
    "address.hpp"
    "array.hpp"
    "atomic_counter.hpp"
    "atomic_ptr.hpp"
    "blob.hpp"
    "channel.hpp"
    "client.hpp"
    "clock.hpp"
    "command.hpp"
    "compat.hpp"
    "condition_variable.hpp"
    "config.hpp"
    "ctx.hpp"
    "curve_client.hpp"
    "curve_client_tools.hpp"
    "curve_mechanism_base.hpp"
    "curve_server.hpp"
    "dbuffer.hpp"
    "dealer.hpp"
    "decoder.hpp"
    "decoder_allocators.hpp"
    "devpoll.hpp"
    "dgram.hpp"
    "dish.hpp"
    "dist.hpp"
    "encoder.hpp"
    "endpoint.hpp"
    "err.hpp"
    "fd.hpp"
    "fq.hpp"
    "gather.hpp"
    "generic_mtrie.hpp"
    "generic_mtrie_impl.hpp"
    "gssapi_client.hpp"
    "gssapi_mechanism_base.hpp"
    "gssapi_server.hpp"
    "i_decoder.hpp"
    "i_encoder.hpp"
    "i_engine.hpp"
    "i_mailbox.hpp"
    "i_poll_events.hpp"
    "io_object.hpp"
    "io_thread.hpp"
    "ip.hpp"
    "ipc_address.hpp"
    "ipc_connecter.hpp"
    "ipc_listener.hpp"
    "kqueue.hpp"
    "lb.hpp"
    "likely.hpp"
    "macros.hpp"
    "mailbox.hpp"
    "mailbox_safe.hpp"
    "mechanism.hpp"
    "mechanism_base.hpp"
    "metadata.hpp"
    "msg.hpp"
    "mtrie.hpp"
    "mutex.hpp"
    "norm_engine.hpp"
    "null_mechanism.hpp"
    "object.hpp"
    "options.hpp"
    "own.hpp"
    "pair.hpp"
    "peer.hpp"
    "pgm_receiver.hpp"
    "pgm_sender.hpp"
    "pgm_socket.hpp"
    "pipe.hpp"
    "plain_client.hpp"
    "plain_common.hpp"
    "plain_server.hpp"
    "platform.hpp"
    "poll.hpp"
    "poller.hpp"
    "poller_base.hpp"
    "polling_util.hpp"
    "pollset.hpp"
    "precompiled.hpp"
    "proxy.hpp"
    "pub.hpp"
    "pull.hpp"
    "push.hpp"
    "radio.hpp"
    "random.hpp"
    "raw_decoder.hpp"
    "raw_encoder.hpp"
    "raw_engine.hpp"
    "reaper.hpp"
    "rep.hpp"
    "req.hpp"
    "router.hpp"
    "scatter.hpp"
    "secure_allocator.hpp"
    "select.hpp"
    "server.hpp"
    "session_base.hpp"
    "signaler.hpp"
    "socket_base.hpp"
    "socket_poller.hpp"
    "socks.hpp"
    "socks_connecter.hpp"
    "stdint.hpp"
    "stream.hpp"
    "stream_engine_base.hpp"
    "stream_connecter_base.hpp"
    "stream_connecter_base.cpp"
    "stream_listener_base.hpp"
    "stream_listener_base.cpp"
    "sub.hpp"
    "tcp.hpp"
    "tcp_address.hpp"
    "tcp_connecter.hpp"
    "tcp_listener.hpp"
    "thread.hpp"
    "timers.hpp"
    "trie.hpp"
    "udp_address.hpp"
    "udp_engine.hpp"
    "v1_decoder.hpp"
    "v1_encoder.hpp"
    "v2_decoder.hpp"
    "v2_encoder.hpp"
    "v3_1_encoder.hpp"
    "v2_protocol.hpp"
    "vmci.hpp"
    "vmci_address.hpp"
    "vmci_connecter.hpp"
    "vmci_listener.hpp"
    "windows.hpp"
    "wire.hpp"
    "xpub.hpp"
    "xsub.hpp"
    "ypipe.hpp"
    "ypipe_base.hpp"
    "ypipe_conflate.hpp"
    "yqueue.hpp"
    "zap_client.hpp"
    "zmtp_engine.hpp"
)

if(${ZMQ_ENABLE_WS})
    list(
        APPEND sources
        "ws_address.cpp"
        "ws_connecter.cpp"
        "ws_decoder.cpp"
        "ws_encoder.cpp"
        "ws_engine.cpp"
        "ws_listener.cpp"
        # headers
        "ws_address.hpp"
        "ws_connecter.hpp"
        "ws_decoder.hpp"
        "ws_encoder.hpp"
        "ws_engine.hpp"
        "ws_listener.hpp"
        "ws_protocol.hpp"
    )
endif()

set(sources_full "")
foreach(file ${sources})
    list(APPEND sources_full "${PROJECT_SOURCE_DIR}/src/${file}")
endforeach()
set(sources ${sources_full})

if(${ZMQ_SHARED})
    set(zmq_lib_type SHARED)
else()
    set(zmq_lib_type STATIC)
endif()

add_library(
    libzmq
    ${zmq_lib_type}
    ${sources}
)

set_target_properties(
    libzmq PROPERTIES
    C_STANDARD 11
    CXX_STANDARD 11
    C_STANDARD_REQUIRED TRUE
    CXX_STANDARD_REQUIRED TRUE
    RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}"
    PDB_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}"
)

target_compile_definitions(
    libzmq PRIVATE
    _REENTRANT
    _THREAD_SAFE
    ZMQ_USE_CV_IMPL_STL11 # use C++11 std::condition_variable_t implementation
    ZMQ_CUSTOM_PLATFORM_HPP
)

if(NOT ${ZMQ_SHARED})
    # technically only required on Windows
    target_compile_definitions(libzmq PUBLIC ZMQ_STATIC)
endif()

if(${ZMQ_ENABLE_DRAFTS})
    target_compile_definitions(libzmq PUBLIC ZMQ_BUILD_DRAFT_API)
endif()

if(${ZMQ_ENABLE_WS})
    target_sources(
        libzmq PRIVATE
        "${PROJECT_SOURCE_DIR}/external/sha1/sha1.c"
        "${PROJECT_SOURCE_DIR}/external/sha1/sha1.h"
    )

    target_compile_definitions(
        libzmq PRIVATE
        ZMQ_HAVE_WS
        ZMQ_USE_BUILTIN_SHA1
    )
endif()

if(${ZMQ_ENABLE_RADIX_TREE})
    target_compile_definitions(libzmq PRIVATE ZMQ_USE_RADIX_TREE)
endif()

if(${ZMQ_WITH_MILITANT})
    target_compile_definitions(libzmq PRIVATE ZMQ_ACT_MILITANT)
endif()

set(HAVE_IPC FALSE)
if(${WIN32})
    set(POLLER "epoll")

    target_compile_definitions(
        libzmq PUBLIC
        ZMQ_HAVE_WINDOWS
    )

    target_compile_definitions(
        libzmq PRIVATE
        _WIN32_WINNT=0x0A00 # require Windows 10
        FD_SETSIZE=16384 # max sockets
        _CRT_SECURE_NO_WARNINGS
        _WINSOCK_DEPRECATED_NO_WARNINGS
    )

    check_include_files("winsock2.h;afunix.h" HAVE_IPC)
    if(${HAVE_IPC})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_IPC)
        check_struct_has_member("struct sockaddr_un" sun_path "winsock2.h;afunix.h" HAVE_STRUCT_SOCKADDR_UN)
        if(${HAVE_STRUCT_SOCKADDR_UN})
            target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_STRUCT_SOCKADDR_UN)
        endif()
    endif()
else()
    target_compile_definitions(
        libzmq PRIVATE
        ZMQ_HAVE_IPC
        ZMQ_HAVE_STRUCT_SOCKADDR_UN
    )

    check_cxx_symbol_exists(pselect sys/select.h HAVE_PSELECT)
    if(${HAVE_PSELECT})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_PPOLL)
    endif()
endif()

if(NOT ${MSVC})
    if("${POLLER}" STREQUAL "")
        check_cxx_symbol_exists(kqueue "sys/types.h;sys/event.h;sys/time.h" HAVE_KQUEUE)
        if(${HAVE_KQUEUE})
            set(POLLER "kqueue")
        endif()
    endif()

    if("${POLLER}" STREQUAL "")
        check_cxx_symbol_exists(epoll_create sys/epoll.h HAVE_EPOLL)
        if(${HAVE_EPOLL})
            set(POLLER "epoll")
            check_cxx_symbol_exists(epoll_create1 sys/epoll.h HAVE_EPOLL_CLOEXEC)
            if(${HAVE_EPOLL_CLOEXEC})
                target_compile_definitions(libzmq PRIVATE ZMQ_IOTHREAD_POLLER_USE_EPOLL_CLOEXEC)
            endif()
        endif()
    endif()

    if("${POLLER}" STREQUAL "")
        check_include_files("sys/devpoll.h" HAVE_DEVPOLL)
        if(${HAVE_DEVPOLL})
            set(POLLER "devpoll")
        endif()
    endif()

    if("${POLLER}" STREQUAL "")
        check_cxx_symbol_exists(pollset_create sys/pollset.h HAVE_POLLSET)
        if(${HAVE_POLLSET})
            set(POLLER "pollset")
        endif()
    endif()

    if("${POLLER}" STREQUAL "")
        check_cxx_symbol_exists(poll poll.h HAVE_POLL)
        if(${HAVE_POLL})
            set(POLLER "poll")
        endif()
    endif()
endif()

if("${POLLER}" STREQUAL "")
    if(${WIN32})
        set(HAVE_SELECT TRUE)
    else()
        check_cxx_symbol_exists(select sys/select.h HAVE_SELECT)
    endif()

    if(${HAVE_SELECT})
        set(POLLER "select")
    else()
        message(FATAL_ERROR "Could not autodetect polling method")
    endif()
endif()

if(${POLLER} STREQUAL "kqueue"
    OR ${POLLER} STREQUAL "epoll"
    OR ${POLLER} STREQUAL "devpoll"
    OR ${POLLER} STREQUAL "pollset"
    OR ${POLLER} STREQUAL "poll"
    OR ${POLLER} STREQUAL "select")
    message(STATUS "Using polling method in I/O threads: ${POLLER}")
    string(TOUPPER ${POLLER} UPPER_POLLER)
    target_compile_definitions(libzmq PRIVATE ZMQ_IOTHREAD_POLLER_USE_${UPPER_POLLER})
else()
    message(FATAL_ERROR "Invalid polling method")
endif()

if(${POLLER} STREQUAL "epoll" AND ${WIN32})
    target_sources(
        libzmq PRIVATE
        "${PROJECT_SOURCE_DIR}/src/epoll.cpp"
        "${PROJECT_SOURCE_DIR}/src/epoll.hpp"
    )

    if(${WIN32})
        message(STATUS "Including wepoll")
        target_sources(
            libzmq PRIVATE
            "${PROJECT_SOURCE_DIR}/external/wepoll/wepoll.c"
            "${PROJECT_SOURCE_DIR}/external/wepoll/wepoll.h"
        )
    endif()
endif()

if("${API_POLLER}" STREQUAL "")
    if(POLLER STREQUAL "select")
        set(API_POLLER "select")
    else()
        set(API_POLLER "poll")
    endif()
endif()

message(STATUS "Using polling method in zmq_poll(er)_* API: ${API_POLLER}")
string(TOUPPER ${API_POLLER} UPPER_API_POLLER)
target_compile_definitions(libzmq PRIVATE ZMQ_POLL_BASED_ON_${UPPER_API_POLLER})

# TODO: determine real cache line size dynamically
set(CACHELINE_SIZE 64)
message(STATUS "Using ${CACHELINE_SIZE} bytes alignment for lock-free data structures")
target_compile_definitions(libzmq PRIVATE ZMQ_CACHELINE_SIZE=${CACHELINE_SIZE})

if(NOT ${MSVC})
    check_include_files(ifaddrs.h HAVE_IFADDRS)
    check_include_files(sys/uio.h HAVE_UIO)
    check_include_files(sys/eventfd.h HAVE_EVENTFD)

    if(${HAVE_IFADDRS})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_IFADDRS)
    endif()
    if(${HAVE_UIO})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_UIO)
    endif()
    if(${HAVE_EVENTFD})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_EVENTFD)
        if(NOT ${CMAKE_CROSSCOMPILING})
            zmq_check_efd_cloexec()
            if(${HAVE_EVENTFD_CLOEXEC})
                target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_SOCK_CLOEXEC)
            endif()
        endif()
    endif()
endif()

if(${WIN32})
    set(HAVE_IF_NAMETOINDEX TRUE)
    set(HAVE_BUSY_POLL FALSE)

    target_link_libraries(
        libzmq PUBLIC
        ws2_32
        iphlpapi
    )
else()
    check_cxx_symbol_exists(if_nametoindex net/if.h HAVE_IF_NAMETOINDEX)
    check_cxx_symbol_exists(SO_PEERCRED sys/socket.h HAVE_SO_PEERCRED)
    check_cxx_symbol_exists(LOCAL_PEERCRED sys/socket.h HAVE_LOCAL_PEERCRED)
    check_cxx_symbol_exists(SO_BUSY_POLL sys/socket.h HAVE_BUSY_POLL)

    if(${HAVE_SO_PEERCRED})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_SO_PEERCRED)
    endif()
    if(${HAVE_LOCAL_PEERCRED})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_LOCAL_PEERCRED)
    endif()
    if(${HAVE_BUSY_POLL})
        target_compile_definitions(libzmq PRIVATE ZMQ_HAVE_BUSY_POLL)
    endif()
endif()

if(${HAVE_IF_NAMETOINDEX})
    target_compile_definitions(libzmq PRIVATE HAVE_IF_NAMETOINDEX)
endif()

if(${MSVC})
    set(HAVE_STRNLEN TRUE)
    set(HAVE_FORK FALSE)
else()
    check_cxx_symbol_exists(clock_gettime time.h HAVE_CLOCK_GETTIME)
    check_cxx_symbol_exists(fork unistd.h HAVE_FORK)
    check_cxx_symbol_exists(gethrtime sys/time.h HAVE_GETHRTIME)
    check_cxx_symbol_exists(mkdtemp "stdlib.h;unistd.h" HAVE_MKDTEMP)
    check_cxx_symbol_exists(accept4 sys/socket.h HAVE_ACCEPT4)
    check_cxx_symbol_exists(strnlen string.h HAVE_STRNLEN)
    if(${HAVE_CLOCK_GETTIME})
        target_compile_definitions(libzmq PRIVATE HAVE_CLOCK_GETTIME)
    endif()
    if(${HAVE_FORK})
        target_compile_definitions(libzmq PRIVATE HAVE_FORK)
    endif()
    if(${HAVE_GETHRTIME})
        target_compile_definitions(libzmq PRIVATE HAVE_GETHRTIME)
    endif()
    if(${HAVE_ACCEPT4})
        target_compile_definitions(libzmq PRIVATE HAVE_ACCEPT4)
    endif()
endif()

if(${HAVE_STRNLEN})
    target_compile_definitions(libzmq PRIVATE HAVE_STRNLEN)
endif()

if(${ZMQ_PEDANTIC})
    if(${MSVC})
        target_compile_options(libzmq PRIVATE /W4)
    else()
        target_compile_options(libzmq PRIVATE -wpedantic)
    endif()
endif()

if(NOT ${MSVC})
    # Compilation checks
    zmq_check_pthread_setname()
    zmq_check_pthread_setaffinity()

    # Execution checks
    if(NOT ${CMAKE_CROSSCOMPILING})
        zmq_check_sock_cloexec()
        zmq_check_o_cloexec()
        zmq_check_so_bindtodevice()
        zmq_check_so_keepalive()
        zmq_check_so_priority()
        zmq_check_tcp_keepcnt()
        zmq_check_tcp_keepidle()
        zmq_check_tcp_keepintvl()
        zmq_check_tcp_keepalive()
        zmq_check_getrandom()
    endif()
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux"
    OR ${CMAKE_SYSTEM_NAME} STREQUAL "GNU/kFreeBSD"
    OR ${CMAKE_SYSTEM_NAME} STREQUAL "GNU/Hurd"
    OR "${CYGWIN}")
    target_compile_definitions(libzmq PRIVATE _GNU_SOURCE)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "FreeBSD")
    target_compile_definitions(libzmq PRIVATE __BSD_VISIBLE)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "NetBSD")
    target_compile_definitions(libzmq PRIVATE _NETBSD_SOURCE)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "OpenBSD")
    target_compile_definitions(libzmq PRIVATE _OPENBSD_SOURCE)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "SunOS")
    target_compile_definitions(libzmq PRIVATE _PTHREADS)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "HP-UX")
    target_compile_definitions(libzmq PRIVATE _POSIX_C_SOURCE=200112L)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
    target_compile_definitions(libzmq PRIVATE _DARWIN_C_SOURCE)
endif()

if(${MSVC})
    # Suppress linker warnings caused by #ifdef omission of file content.
    target_link_options(libzmq PRIVATE "/ignore:4221")

    if(${ZMQ_SHARED})
        # Whole Program Optimization flags. http://msdn.microsoft.com/en-us/magazine/cc301698.aspx
        #
        # "Finally, there's the subject of libraries. It's possible to create .LIB
        # files with code in its IL form. The linker will happily work with these
        # .LIB files. Be aware that these libraries will be tied to a specific
        # version of the compiler and linker. If you distribute these libraries,
        # you'll need to update them if Microsoft changes the format of IL in a
        # future release."
        #
        # /GL and /LTCG can cause problems when libraries built with different
        # versions of compiler are later linked into an executable while /LTCG is active.
        # https://social.msdn.microsoft.com/Forums/vstudio/en-US/5c102025-c254-4f02-9a51-c775c6cc9f4b/problem-with-ltcg-when-building-a-static-library-in-vs2005?forum=vcgeneral
        #
        # For this reason, enable only when building a "Release" (e.g. non-DEBUG) DLL.
        if(NOT "${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
            target_compile_options(libzmq PRIVATE /GL /LTCG)
        endif()
    endif()
endif()

if(${ZMQ_SHARED} AND ${WIN32})
    # export symbols
    target_compile_definitions(libzmq PRIVATE DLL_EXPORT)
endif()

if(${ZMQ_BUILD_TESTS})
    enable_testing()
    add_subdirectory(tests)
endif()
