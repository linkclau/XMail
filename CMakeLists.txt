cmake_minimum_required (VERSION 2.8)
project(XMail)
# The version number.
set (XMail_VERSION_MAJOR 1)
set (XMail_VERSION_MINOR 27)


# Options from the Maekfile
# XMAIL_FILE_OFF_BITS)", "")
#    CFLAGS := $(CFLAGS) -D_FILE_OFFSET_BITS=64
#    CFLAGS := $(CFLAGS) -D_FILE_OFFSET_BITS=$(XMAIL_FILE_OFF_BITS)
#

# On Linux XMails expcts
add_definitions( -DHAS_SYSMACHINE)
#add_definitions( -D__UNIX__ 
#  -D_REENTRANT=1 -D_THREAD_SAFE=1 
#  -D_GNU_SOURCE -D_LARGEFILE64_SOURCE -D_POSIX_PTHREAD_SEMANTICS )
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )


find_package(OpenSSL REQUIRED)
include_directories(${OPENSSL_INCLUDE_DIR})
set(LIBS ${LIBS} ${OPENSSL_LIBRARIES})

find_package (Threads REQUIRED)
set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})

INCLUDE (CheckIncludeFiles)
# usage: CHECK_INCLUDE_FILES (<header> <RESULT_VARIABLE> )
check_include_files(kvm.h HAS_KVM_H )

message( "CMAKE_SYSTEM_NAME ${CMAKE_SYSTEM_NAME}" )
IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	add_definitions( -D__BSD__)
	add_definitions( -D__DARWIN__)
	add_definitions( -D_REENTRANT=1 -D_THREAD_SAFE=1 -D_POSIX_PTHREAD_SEMANTICS )
	## I don't like this 
	add_definitions( -D__DARWIN_10_5__)
	set( MAINSRC MainBSD.cpp )
	set( SYSSRCS SysDepBSD.cpp SysDepUnix.cpp )
	set(LIBS ${LIBS} -lc )
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	add_definitions( -D__LINUX__)
	set( MAINSRC MainLinux.cpp )
	set( SYSSRCS SysDepLinux.cpp SysDepUnix.cpp )
	# Actually XMail has a bug. If SYS_HAS_SENDFILE is not defined
	# SysSendFile will be defined twice 
	check_function_exists (sendfile SYS_HAS_SENDFILE)
	if (SYS_HAS_SENDFILE)
	  #message( "SYS_HAS_SENDFILE: ${HAS_SYSMACHINE}" ) 
	  add_definitions( -DSYS_HAS_SENDFILE )
	endif(SYS_HAS_SENDFILE)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

include (CheckFunctionExists)
check_function_exists (eventfd HAS_EVENTFD)
if (HAS_EVENTFD)
  set( SYSSRCS ${SYSSRCS} SysOsEventfd_eventfd.cpp )
  add_definitions( -DHAS_EVENTFD )
else()
  set( SYSSRCS ${SYSSRCS} SysOsEventfd_pipe.cpp )
endif(HAS_EVENTFD)


set( COMMONSRCS 
Base64Enc.cpp
BuffSock.cpp
Errors.cpp
MD5.cpp
MiscUtils.cpp
SSLBind.cpp
SSLMisc.cpp
StrUtils.cpp
SysDepCommon.cpp
)


set( SVRSRCS ${MAINSRC} 
    CTRLSvr.cpp 
    DNS.cpp DNSCache.cpp 
    ExtAliases.cpp FINGSvr.cpp MailConfig.cpp MailSvr.cpp 
    Maildir.cpp MailDomains.cpp 
    LMAILSvr.cpp AliasDomain.cpp POP3GwLink.cpp 
    POP3Svr.cpp POP3Utils.cpp PSYNCSvr.cpp 
    ResLocks.cpp SList.cpp SMAILSvr.cpp TabIndex.cpp SMAILUtils.cpp 
    SMTPSvr.cpp SMTPUtils.cpp 
    ShBlocks.cpp MessQueue.cpp QueueUtils.cpp 
    SvrUtils.cpp UsrMailList.cpp UsrAuth.cpp 
    UsrUtils.cpp Filter.cpp SSLConfig.cpp 
    Hash.cpp Array.cpp 
)

set( CCLNSRCS CTRLClient.cpp )

add_executable( MkMachDep MkMachDep.cpp )

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  COMMAND MkMachDep > ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  DEPENDS MkMachDep
)

set(LIBS ${LIBS} -ldl )

add_library(common_objects OBJECT ${COMMONSRCS} 
	${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h)
add_library(system_objects OBJECT ${SYSSRCS} 
    ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h )

# MAILSVR
add_executable( XMail ${SVRSRCS} 
    $<TARGET_OBJECTS:system_objects> $<TARGET_OBJECTS:common_objects> )
target_link_libraries(XMail ${LIBS} )
add_executable( CtrlClnt CTRLClient.cpp 
    $<TARGET_OBJECTS:system_objects> $<TARGET_OBJECTS:common_objects> )
target_link_libraries(CtrlClnt ${LIBS} )
add_executable( XMCrypt XMCrypt.cpp )
add_executable( MkUsers MkUsers.cpp )
add_executable( sendmail SendMail.cpp )

