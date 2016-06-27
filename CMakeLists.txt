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
# WITH_SSL_INCLUDE
#    CFLAGS := $(CFLAGS) -I$(WITH_SSL_INCLUDE)
# WITH_SSL_LIB
#    LDFLAGS := $(LDFLAGS) -L$(WITH_SSL_LIB)
# SSLLIBS
#    SSLLIBS = -lssl -lcrypto

# On Linux XMails expcts
#add_definitions(-DOPENCV_VERSION=${OpenCV_VERSION})
#add_definitions( -D__LINUX__)
add_definitions( 
    -D__UNIX__ -D__LINUX__ -D_REENTRANT=1 -D_THREAD_SAFE=1 -DHAS_SYSMACHINE 
        -D_GNU_SOURCE -D_LARGEFILE64_SOURCE -D_POSIX_PTHREAD_SEMANTICS -DSYS_HAS_SENDFILE )
#include_directories( ${PROJECT_SOURCE_DIR} )
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )


find_package(OpenSSL REQUIRED)
message("OPENSSL_INCLUDE_DIR: ${OPENSSL_INCLUDE_DIR}")
message("OPENSSL_LIBRARIES: ${OPENSSL_LIBRARIES}")
message("OPENSSL_ROOT_DIR: ${OPENSSL_ROOT_DIR}")
include_directories(${OPENSSL_INCLUDE_DIR})
message("LIBS: ${LIBS}")
set(LIBS ${LIBS} ${OPENSSL_LIBRARIES})
message("LIBS: ${LIBS}")

find_package (Threads)
set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})
message("LIBS: ${LIBS}")

set( MAINSRC MainLinux.cpp )
set( SYSSRCS SysDepLinux.cpp SysDepUnix.cpp )

# On x86_64 it is actually 
#	/usr/include/x86_64-linux-gnu/sys/eventfd.h
# /usr/include/sys/eventfd.h
#    SYSSRCS := $(SYSSRCS) SysOsEventfd_eventfd.cpp
#    CFLAGS := $(CFLAGS) -DHAS_EVENTFD
#    SYSSRCS := $(SYSSRCS) SysOsEventfd_pipe.cpp
set( SYSSRCS ${SYSSRCS} SysOsEventfd_eventfd.cpp )
add_definitions( -DHAS_EVENTFD )


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
#target_link_libraries(${PROJECT_NAME} ${OPENSSL_LIBRARIES})
#add_executable( CtrlClnt )
#add_executable( XMCrypt )
#add_executable( MkUsers )
#add_executable( sendmail )

