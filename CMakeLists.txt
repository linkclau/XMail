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
add_definitions( -D__LINUX__)
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

set( MAINSRC MainLinux.cpp )
set( SYSSRCS SysDepLinux.cpp SysDepUnix.cpp )

include (CheckFunctionExists)
check_function_exists (eventfd HAS_EVENTFD)
if (HAS_EVENTFD)
  set( SYSSRCS ${SYSSRCS} SysOsEventfd_eventfd.cpp )
  add_definitions( -DHAS_EVENTFD )
else()
  set( SYSSRCS ${SYSSRCS} SysOsEventfd_pipe.cpp )
endif(HAS_EVENTFD)

# Ectually XMail has a bug. If SYS_HAS_SENDFILE is not defined
# SysSendFile will be defined twice 
check_function_exists (sendfile SYS_HAS_SENDFILE)
if (SYS_HAS_SENDFILE)
  #message( "SYS_HAS_SENDFILE: ${HAS_SYSMACHINE}" ) 
  add_definitions( -DSYS_HAS_SENDFILE )
endif(SYS_HAS_SENDFILE)


set( SVRSRCS ${MAINSRC} ${SYSSRCS} 
    SysDepCommon.cpp BuffSock.cpp CTRLSvr.cpp 
    DNS.cpp DNSCache.cpp Errors.cpp 
    ExtAliases.cpp FINGSvr.cpp MailConfig.cpp MailSvr.cpp 
    Maildir.cpp MailDomains.cpp MD5.cpp 
    MiscUtils.cpp LMAILSvr.cpp AliasDomain.cpp POP3GwLink.cpp 
    POP3Svr.cpp POP3Utils.cpp PSYNCSvr.cpp 
    ResLocks.cpp SList.cpp SMAILSvr.cpp TabIndex.cpp SMAILUtils.cpp 
    SMTPSvr.cpp SMTPUtils.cpp 
    ShBlocks.cpp StrUtils.cpp MessQueue.cpp QueueUtils.cpp 
    SvrUtils.cpp UsrMailList.cpp UsrAuth.cpp 
    UsrUtils.cpp Base64Enc.cpp Filter.cpp SSLBind.cpp SSLConfig.cpp 
    Hash.cpp Array.cpp SSLMisc.cpp
)


set( CCLNSRCS ${SYSSRCS} 
    SysDepCommon.cpp Base64Enc.cpp BuffSock.cpp StrUtils.cpp 
    MD5.cpp MiscUtils.cpp 
    CTRLClient.cpp Errors.cpp SSLBind.cpp SSLMisc.cpp
)



add_executable( MkMachDep MkMachDep.cpp )

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  COMMAND MkMachDep > ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  DEPENDS MkMachDep
)


# MAILSVR
add_executable( XMail ${SVRSRCS} ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h )
target_link_libraries(XMail ${LIBS} -ldl)
add_executable( CtrlClnt ${CCLNSRCS} )
target_link_libraries(CtrlClnt ${LIBS} -ldl)
#add_executable( XMCrypt )
#add_executable( MkUsers )
#add_executable( sendmail )

