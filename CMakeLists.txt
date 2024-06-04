# For VS2019 and Xcode 12+ support.
cmake_minimum_required(VERSION 3.19 FATAL_ERROR)

# Custom config
#set(C_PROJECT_NAME "mDNSResponder")

project(mDNSResponder)

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")

endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -g  -fno-strict-aliasing  -D_GNU_SOURCE  -DHAVE_IPV6  -DHAVE_LINUX  -DNOT_HAVE_SA_LEN  -DPLATFORM_NO_RLIMIT  -DTARGET_OS_LINUX  -DUSES_NETLINK  -DMDNS_DEBUGMSGS=0 -W  -Wall  -Wextra  -Wno-array-bounds  -Wno-pointer-sign  -Wno-unused  -Wno-unused-parameter  -Werror=implicit-function-declaration")
add_definitions(-DUSE_TCP_LOOPBACK )
endif()

#
# Sources
#

#
# mdns service
# TODO: mdns library mDNSResponderDLL.dll/libmdns.so
#

# mDNSCore
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(MDNSCORE_SOURCES ${MDNSCORE_SOURCES} 
        mDNSCore/anonymous.h
        mDNSCore/anonymous.c
		mDNSCore/CryptoAlg.h
		mDNSCore/CryptoAlg.c
		mDNSCore/DNSCommon.h
		mDNSCore/DNSCommon.c
		mDNSCore/DNSDigest.c
		mDNSCore/mDNSDebug.h
		mDNSCore/mDNSEmbeddedAPI.h
		mDNSCore/mDNS.c
		mDNSCore/uDNS.h
		mDNSCore/uDNS.c
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
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(MDNSSHARED_SOURCES ${MDNSSHARED_SOURCES} 
        mDNSShared/CommonServices.h
        mDNSShared/DebugServices.h
        mDNSShared/DebugServices.c
        mDNSShared/dnssd_ipc.h
        mDNSShared/dnssd_ipc.c
        mDNSShared/GenLinkedList.h
        mDNSShared/GenLinkedList.c
        mDNSShared/mDNSDebug.c
        mDNSShared/uds_daemon.h
        mDNSShared/uds_daemon.c
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

# mDNSWindows
set(MDNSWINDOWS_SOURCES
    mDNSWindows/SystemService/Firewall.cpp
	mDNSWindows/mDNSWin32.h
	mDNSWindows/mDNSWin32.c
	mDNSWindows/Poll.h
	mDNSWindows/Poll.c
	mDNSWindows/SystemService/Resource.h
	mDNSWindows/Secret.h
	mDNSWindows/Secret.c
	mDNSWindows/SystemService/Service.h
	mDNSWindows/SystemService/Service.c
	mDNSWindows/SystemService/main.c
)
source_group(mDNSWindows FILES ${MDNSWINDOWS_SOURCES})
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
set(WINDOWS_MDNSMACOS_SOURCES
	mDNSMacOSX/LegacyNATTraversal.c
)
source_group(mDNSMacOSX FILES ${WINDOWS_MDNSMACOS_SOURCES})
endif()


# mDNSPosix
set(MDNSPOSIX_SOURCES
    mDNSPosix/mDNSPosix.c
    mDNSPosix/mDNSUNP.c
    mDNSPosix/PosixDaemon.c
)
source_group(mDNSPosix FILES ${MDNSPOSIX_SOURCES})


# all sources
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	add_definitions(-D_CONSOLE 
					-DUSE_TCP_LOOPBACK 
					-DTARGET_OS_WIN32 
					-D_WIN32_WINNT=0x0501
					-DWIN32_LEAN_AND_MEAN 
					-DPLATFORM_NO_STRSEP 
					-DPLATFORM_NO_EPIPE 
					-DPLATFORM_NO_RLIMIT 
					-DPID_FILE="" 
					-DUNICODE 
					-D_UNICODE 
					-D_CRT_SECURE_NO_DEPRECATE 
					-D_CRT_SECURE_CPP_OVERLOAD_STANDARD_NAMES=1 
					-D_LEGACY_NAT_TRAVERSAL_ 
					-D_USE_32BIT_TIME_T
	)
	set(MDNSD_PROJECT_SOURCES
		${MDNSCORE_SOURCES}
		${MDNSSHARED_SOURCES}
		${MDNSWINDOWS_SOURCES}
		${WINDOWS_MDNSMACOS_SOURCES}
	)
    set(QC_RUNTIME_TYPE WIN32)
	set(MDNSD_PROJECT_NAME mDNSResponder)
	add_executable(${MDNSD_PROJECT_NAME} ${QC_RUNTIME_TYPE} ${MDNSD_PROJECT_SOURCES})
	
	set_target_properties(${MDNSD_PROJECT_NAME} PROPERTIES LINK_FLAGS "/SUBSYSTEM:CONSOLE")
	
	target_include_directories(${MDNSD_PROJECT_NAME} PRIVATE mDNSCore mDNSShared mDNSWindows mDNSWindows/SystemService)
	target_link_libraries(${MDNSD_PROJECT_NAME} ws2_32 
												iphlpapi 
												netapi32 
												powrprof)
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
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	add_definitions(-D_WINDOWS 
					-D_USRDLL 
					-DNOT_HAVE_SA_LEN 
					-DWIN32_LEAN_AND_MEAN 
					-DUSE_TCP_LOOPBACK
					-D_CRT_SECURE_NO_DEPRECATE 
					-D_CRT_SECURE_CPP_OVERLOAD_STANDARD_NAMES=1 
	)
    set(DNSSD_SOURCES ${DNSSD_SOURCES} 
        mDNSShared/CommonServices.h
        mDNSShared/DebugServices.h
        mDNSShared/DebugServices.c
        mDNSShared/dns_sd.h
		mDNSShared/dnssd_clientlib.c
		mDNSShared/dnssd_clientstub.c
		mDNSShared/dnssd_ipc.h
		mDNSShared/dnssd_ipc.c
		mDNSShared/GenLinkedList.h
		mDNSShared/GenLinkedList.c
		mDNSWindows/DLL/dllmain.c
		mDNSWindows/DLL/dnssd.def
		mDNSWindows/DLL/resource.h
    )
	set(DNSSD_PROJECT_NAME dnssd)
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(DNSSD_SOURCES ${DNSSD_SOURCES} 
        mDNSShared/dnssd_clientlib.c
        mDNSShared/dnssd_clientstub.c
		mDNSShared/dnssd_ipc.c
		mDNSShared/Java/JNISupport.c
    )
	set(DNSSD_PROJECT_NAME dns_sd)
	
endif()
source_group(dns_sd FILES ${DNSSD_SOURCES})

add_library(${DNSSD_PROJECT_NAME} SHARED ${DNSSD_SOURCES})

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	target_include_directories(${DNSSD_PROJECT_NAME} PRIVATE mDNSCore mDNSShared mDNSWindows)
	target_link_libraries(${DNSSD_PROJECT_NAME} ws2_32)
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    target_include_directories(${DNSSD_PROJECT_NAME} PRIVATE mDNSCore mDNSShared mDNSShared/Java/ mDNSPosix)
endif()
