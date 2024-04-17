# For VS2019 and Xcode 12+ support.
cmake_minimum_required(VERSION 3.19 FATAL_ERROR)

# Custom config
#set(C_PROJECT_NAME "mDNSResponder")

if(CMAKE_SYSTEM_NAME STREQUAL "Android")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -g  -fno-strict-aliasing  -D_GNU_SOURCE  -DHAVE_IPV6  -DHAVE_LINUX  -DNOT_HAVE_SA_LEN  -DPLATFORM_NO_RLIMIT  -DTARGET_OS_LINUX  -DUSES_NETLINK  -DMDNS_DEBUGMSGS=0 -W  -Wall  -Wextra  -Wno-array-bounds  -Wno-pointer-sign  -Wno-unused  -Wno-unused-parameter  -Werror=implicit-function-declaration")
endif()
#
# Sources
#

#
# mdns service
# TODO: mdns library mDNSResponderDLL.dll/libmdns.so
#

# mDNSCore
#set(MDNSCORE_SOURCES
#    fontawesome/iconhelper.h
#    fontawesome/iconhelper.cpp
#)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(MDNSCORE_SOURCES ${MDNSCORE_SOURCES} 
        util/mousetap/winmousetap.h
        util/mousetap/winmousetap.cpp
        util/winutils.h
        util/winutils.cpp
    )
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(MDNSCORE_SOURCES ${MDNSCORE_SOURCES} 
        mDNSCore/anonymous.c
        mDNSCore/CryptoAlg.c
        mDNSCore/DNSCommon.c
        mDNSCore/DNSDigest.c
		mDNSCore/mDNS.c
		mDNSCore/uDNS.c
    )
endif()
source_group(mDNSCore FILES ${MDNSCORE_SOURCES})

# mDNSShared
#set(MDNSSHARED_SOURCES
#    fontawesome/iconhelper.h
#    fontawesome/iconhelper.cpp
#)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(MDNSSHARED_SOURCES ${MDNSSHARED_SOURCES} 
        util/mousetap/winmousetap.h
        util/mousetap/winmousetap.cpp
        util/winutils.h
        util/winutils.cpp
    )
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(MDNSSHARED_SOURCES ${MDNSSHARED_SOURCES} 
		mDNSShared/dnssd_ipc.c
		mDNSShared/GenLinkedList.c
        mDNSShared/mDNSDebug.c
		mDNSShared/PlatformCommon.c
		mDNSShared/uds_daemon.c
    )
endif()
source_group(mDNSShared FILES ${MDNSSHARED_SOURCES})

# mDNSPosix
set(MDNSPOSIX_SOURCES
    mDNSPosix/mDNSPosix.c
    mDNSPosix/mDNSUNP.c
    mDNSPosix/PosixDaemon.c
)
source_group(mDNSPosix FILES ${MDNSPOSIX_SOURCES})


# all sources
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	set(MDNSD_PROJECT_SOURCES
		${QC_FONTAWESOME_SOURCES}
		${QC_UIBASE_SOURCES}
		${QC_UI_SOURCES}
		${QC_UTIL_SOURCES}
		${QC_MAIN_SOURCES}
		${QC_GROUP_CONTROLLER}
		${QC_PLANTFORM_SOURCES}
		${QC_AUDIO_SOURCES}
	)
    set(QC_RUNTIME_TYPE WIN32)
	add_executable(${MDNSD_PROJECT_NAME} ${QC_RUNTIME_TYPE} ${QC_PROJECT_SOURCES})
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
	set(MDNSD_PROJECT_SOURCES
		${MDNSCORE_SOURCES}
		${MDNSSHARED_SOURCES}
		${MDNSPOSIX_SOURCES}
	)
	set(MDNSD_PROJECT_NAME mdnsd)
    add_executable(${MDNSD_PROJECT_NAME} ${MDNSD_PROJECT_SOURCES})
    
    target_include_directories(${MDNSD_PROJECT_NAME} PRIVATE mDNSCore mDNSShared mDNSPosix)
endif()



#
#dns_sd client library
#
# dns_sd
#set(DNSSD_SOURCES
#    fontawesome/iconhelper.h
#    fontawesome/iconhelper.cpp
#)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(DNSSD_SOURCES ${DNSSD_SOURCES} 
        util/mousetap/winmousetap.h
        util/mousetap/winmousetap.cpp
        util/winutils.h
        util/winutils.cpp
    )
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(DNSSD_SOURCES ${DNSSD_SOURCES} 
        mDNSShared/dnssd_clientlib.c
        mDNSShared/dnssd_clientstub.c
		mDNSShared/dnssd_ipc.c
		mDNSShared/Java/JNISupport.c
    )
	set(DNSSD_PROJECT_NAME jdns_sd)
	
endif()
source_group(dns_sd FILES ${DNSSD_SOURCES})


add_library(${DNSSD_PROJECT_NAME} SHARED ${DNSSD_SOURCES})

if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    target_include_directories(${DNSSD_PROJECT_NAME} PRIVATE mDNSCore mDNSShared mDNSShared/Java/ mDNSPosix)
endif()
