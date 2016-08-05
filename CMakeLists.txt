cmake_minimum_required (VERSION 2.8.11 )
project(XMail)
# The version number.
set (XMail_VERSION_MAJOR 1)
set (XMail_VERSION_MINOR 27)


# Options from the Makefile
# XMAIL_FILE_OFF_BITS)", "")
#    CFLAGS := $(CFLAGS) -D_FILE_OFFSET_BITS=64
#    CFLAGS := $(CFLAGS) -D_FILE_OFFSET_BITS=$(XMAIL_FILE_OFF_BITS)
#

#add_definitions( -D__UNIX__ 
#  -D_REENTRANT=1 -D_THREAD_SAFE=1 
#  -D_GNU_SOURCE -D_LARGEFILE64_SOURCE -D_POSIX_PTHREAD_SEMANTICS )
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )


find_package(OpenSSL REQUIRED)
include_directories(${OPENSSL_INCLUDE_DIR})
list(APPEND LIBS ${OPENSSL_LIBRARIES})

find_package (Threads REQUIRED)
list(APPEND LIBS ${CMAKE_THREAD_LIBS_INIT})


set( SYSSRCS SysDep.h SysInclude.h SysLists.h SysMacros.h SysTypes.h CommTypes.h)
message( "CMAKE_SYSTEM_NAME ${CMAKE_SYSTEM_NAME}" )
message( "CMAKE_VS_PLATFORM_NAME ${CMAKE_VS_PLATFORM_NAME}" )
get_cmake_property(_variableNames VARIABLES)
#foreach (_variableName ${_variableNames})
#      message(STATUS "${_variableName}=${${_variableName}}")
#endforeach()
include (CheckIncludeFiles)
include (CheckFunctionExists)

if( UNIX )
  list(APPEND SYSSRCS SysDepUnix.cpp  SysDepUnix.h  SysTypesUnix.h )
  # usage: CHECK_INCLUDE_FILES (<headers> <RESULT_VARIABLE> )
  check_include_files(kvm.h HAS_KVM_H )
  add_definitions(-DHAS_SYSMACHINE)
  list(APPEND SYSSRCS ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h )
endif(UNIX)

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  add_definitions( -D__BSD__)
  add_definitions( -D__DARWIN__)
  #add_definitions( -D_REENTRANT=1 -D_THREAD_SAFE=1 -D_POSIX_PTHREAD_SEMANTICS )
  ## I don't like this 
  add_definitions( -D__DARWIN_10_5__)
  set( MAINSRC MainBSD.cpp )
  list(APPEND SYSSRCS SysDepBSD.cpp SysIncludeBSD.h SysTypesBSD.h )
  list(APPEND LIBS -lc )
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  add_definitions( -D__LINUX__)
  set( MAINSRC MainLinux.cpp )
  list(APPEND SYSSRCS SysDepLinux.cpp SysIncludeLinux.h SysTypesLinux.h )
  # Actually XMail has a bug. If SYS_HAS_SENDFILE is not defined
  # SysSendFile will be defined twice 
  check_function_exists (sendfile SYS_HAS_SENDFILE)
  if (SYS_HAS_SENDFILE)
    add_definitions( -DSYS_HAS_SENDFILE )
  endif(SYS_HAS_SENDFILE)
  check_function_exists (eventfd HAS_EVENTFD)
  list(APPEND LIBS -ldl )
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "SunOS")
  add_definitions( -D__SOLARIS__ )
  list(APPEND LIBS -lsocket )
  set( MAINSRC MainSolaris.cpp )
  list(APPEND SYSSRCS SysDepSolaris.cpp SysIncludeSolaris.h )
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  #add_compile_options( "/nologo" )
  set( MAINSRC MainWin.cpp )
  add_definitions( /D_CRT_SECURE_NO_DEPRECATE /D_CRT_NONSTDC_NO_DEPRECATE )
  list(APPEND SYSSRCS SysDepWin.cpp SysIncludeWin.h  SysTypesWin.h )
  list(APPEND LIBS Ws2_32.lib )
ELSE( ${CMAKE_SYSTEM_NAME} MATCHES "Darwin" )
  # Support Solaris
  message( "${CMAKE_SYSTEM_NAME} is unsupported" FATAL_ERROR )
  #SysIncludeSolaris.h SysTypesSolaris.h 
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

if (HAS_EVENTFD )
  list(APPEND SYSSRCS SysOsEventfd_eventfd.cpp )
  add_definitions( -DHAS_EVENTFD )
else()
  list(APPEND SYSSRCS SysOsEventfd_pipe.cpp )
endif(HAS_EVENTFD)

# Specify headers to include them into the Visual Studio Project
# It would compile as well without specifying them
set( COMMONSRCS 
  Base64Enc.cpp Base64Enc.h
  BuffSock.cpp BuffSock.h 
  Errors.cpp Errors.h
  MD5.cpp MD5.h
  MiscUtils.cpp MiscUtils.h
  SSLBind.cpp SSLBind.h
  SSLMisc.cpp SSLMisc.h
  StrUtils.cpp StrUtils.h 
  SysDepCommon.cpp
  AppDefines.h
)


set( SVRSRCS ${MAINSRC} 
  CTRLSvr.cpp  CTRLSvr.h
  DNS.cpp  DNS.h
  DNSCache.cpp  DNSCache.h
  ExtAliases.cpp  ExtAliases.h
  FINGSvr.cpp  FINGSvr.h
  MailConfig.cpp  MailConfig.h
  MailSvr.cpp  MailSvr.h
  Maildir.cpp  Maildir.h
  MailDomains.cpp  MailDomains.h
  LMAILSvr.cpp  LMAILSvr.h
  AliasDomain.cpp  AliasDomain.h
  POP3GwLink.cpp  POP3GwLink.h
  POP3Svr.cpp  POP3Svr.h
  POP3Utils.cpp  POP3Utils.h
  PSYNCSvr.cpp  PSYNCSvr.h
  ResLocks.cpp  ResLocks.h
  SList.cpp  SList.h
  SMAILSvr.cpp  SMAILSvr.h
  TabIndex.cpp  TabIndex.h
  SMAILUtils.cpp  SMAILUtils.h
  SMTPSvr.cpp  SMTPSvr.h
  SMTPUtils.cpp  SMTPUtils.h
  ShBlocks.cpp  ShBlocks.h
  MessQueue.cpp  MessQueue.h
  QueueUtils.cpp  QueueUtils.h
  SvrUtils.cpp  SvrUtils.h
  UsrMailList.cpp  UsrMailList.h
  UsrAuth.cpp  UsrAuth.h
  UsrUtils.cpp  UsrUtils.h
  Filter.cpp  Filter.h
  SSLConfig.cpp  SSLConfig.h
  Hash.cpp  Hash.h
  Array.cpp  Array.h
  SvrConfig.h SvrDefines.h 
  )


add_executable( MkMachDep MkMachDep.cpp )

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  COMMAND MkMachDep > ${CMAKE_CURRENT_BINARY_DIR}/SysMachine.h
  DEPENDS MkMachDep
)


add_library(system_objects OBJECT ${COMMONSRCS} ${SYSSRCS} )

# MAILSVR
add_executable( XMail ${SVRSRCS} $<TARGET_OBJECTS:system_objects> )
target_link_libraries(XMail ${LIBS} )
add_executable( CtrlClnt CTRLClient.cpp $<TARGET_OBJECTS:system_objects>  )
target_link_libraries(CtrlClnt ${LIBS} )
add_executable( XMCrypt XMCrypt.cpp )
add_executable( MkUsers MkUsers.cpp )
add_executable( sendmail SendMail.cpp )

