cmake_minimum_required(VERSION 3.0)
project(nginx
	VERSION 1.18.0
	# DESCRIPTION "Nginx project" # cmake version >= 3.9
	# HOMEPAGE_URL "www.zhyingkun.com"
	LANGUAGES C CXX
)

# The name of the top level project.
message(STATUS "Top level project is ${CMAKE_PROJECT_NAME}")
message(STATUS "CMakeLists.txt for ${PROJECT_NAME}")
message(STATUS "Current OS is ${CMAKE_SYSTEM}")
message(STATUS "CMAKE_INSTALL_PREFIX is ${CMAKE_INSTALL_PREFIX}")

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CMAKE_SKIP_BUILD_RPATH FALSE)
set(CMAKE_SKIP_INSTALL_RPATH FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH FLASE)
# set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib") # must before add_executable


if("${CMAKE_BUILD_TYPE}" STREQUAL "")
	set(CMAKE_BUILD_TYPE "Debug")
endif()
message(STATUS "CMakeLists.txt for ${PROJECT_NAME}")
message(STATUS "CMAKE_BUILD_TYPE is ${CMAKE_BUILD_TYPE}")

if(APPLE)
	set(CMAKE_C_FLAGS         "-Wall -Wextra -Wpointer-arith -Wconditional-uninitialized -Wno-unused-parameter -Wno-deprecated-declarations -Werror")
	set(CMAKE_C_FLAGS_DEBUG   "-g")
	set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(CMAKE_C_FLAGS         "-W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -Wl,-E")
	set(CMAKE_C_FLAGS_DEBUG   "-g")
	set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
endif()


include_directories(/usr/include/libxml2)
if(APPLE)
	include_directories(/usr/local/include)
	link_directories(/usr/local/lib)
	include_directories(/usr/local/opt/openssl/include)
	link_directories(/usr/local/opt/openssl/lib)
endif()


# = Nginx =====================================================================
set(SRC_ALL)

macro(add_folder_files_to_group source_dir group_name)
	set(TMP_SOURCE_)
	set(TMP_SOURCE_H_)
	include_directories(${source_dir})
	aux_source_directory(${source_dir} TMP_SOURCE_)
	file(GLOB TMP_SOURCE_H_ "${source_dir}/*.h")
	source_group(${group_name} FILES ${TMP_SOURCE_} ${TMP_SOURCE_H_})
	list(APPEND SRC_ALL ${TMP_SOURCE_} ${TMP_SOURCE_H_})
endmacro(add_folder_files_to_group)

macro(add_files_to_group group_name)
	source_group(${group_name} FILES ${ARGN})
	list(APPEND SRC_ALL ${ARGN})
endmacro(add_files_to_group)

add_folder_files_to_group("./src/core" "core")
add_folder_files_to_group("./src/event" "event")

add_folder_files_to_group("./src/event/openssl" "event\\\\openssl")
add_folder_files_to_group("./src/core/threads" "core\\\\threads")
add_folder_files_to_group("./src/os/unix/threads" "os\\\\unix\\\\threads")
add_folder_files_to_group("./src/os/unix" "os\\\\unix")

if(APPLE)
	add_folder_files_to_group("./src/config/darwin" "config")
	add_folder_files_to_group("./src/os/unix/darwin" "os\\\\unix\\\\darwin")
	add_files_to_group("event\\\\modules"
		./src/event/modules/ngx_select_module.c
		./src/event/modules/ngx_poll_module.c
		./src/event/modules/ngx_kqueue_module.c
	)
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
	add_folder_files_to_group("./src/config/linux" "config")
	add_folder_files_to_group("./src/os/unix/linux" "os\\\\unix\\\\linux")
	add_files_to_group("event\\\\modules"
		./src/event/modules/ngx_select_module.c
		./src/event/modules/ngx_poll_module.c
		./src/event/modules/ngx_epoll_module.c
	)
endif()

add_folder_files_to_group("./src/http" "http")
add_folder_files_to_group("./src/http/modules" "http\\\\modules")
add_folder_files_to_group("./src/http/v2" "http\\\\v2")

add_executable(${PROJECT_NAME} ${SRC_ALL})

if(APPLE)
	target_link_libraries(${PROJECT_NAME} pcre z ssl crypto)
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
	target_link_libraries(${PROJECT_NAME} pcre z dl pthread crypt ssl crypto)
endif()


# = Mail Module ===============================================================
set(SRC_ALL)

add_folder_files_to_group("./src/mail" "mail")
add_folder_files_to_group("./src/mail/imap" "mail\\\\imap")
add_folder_files_to_group("./src/mail/pop3" "mail\\\\pop3")
add_folder_files_to_group("./src/mail/smtp" "mail\\\\smtp")

add_folder_files_to_group("./src/mail/openssl" "mail\\\\openssl")

add_library(mail_module MODULE ${SRC_ALL})
if(APPLE)
	target_link_options(mail_module PUBLIC -Wl,-undefined,dynamic_lookup)
endif(APPLE)
set_target_properties(mail_module PROPERTIES
	OUTPUT_NAME "ngx_mail_module"
	PREFIX ""
	POSITION_INDEPENDENT_CODE ON
)


# = Stream Module =============================================================
set(SRC_ALL)

add_folder_files_to_group("./src/stream" "stream")

add_folder_files_to_group("./src/stream/openssl" "stream\\\\openssl")

add_library(stream_module MODULE ${SRC_ALL})
if(APPLE)
	target_link_options(stream_module PUBLIC -Wl,-undefined,dynamic_lookup)
endif(APPLE)
set_target_properties(stream_module PROPERTIES
	OUTPUT_NAME "ngx_stream_module"
	PREFIX ""
	POSITION_INDEPENDENT_CODE ON
)


install(TARGETS ${PROJECT_NAME} mail_module stream_module
	RUNTIME DESTINATION sbin
	LIBRARY DESTINATION modules
	ARCHIVE DESTINATION lib
)
install(DIRECTORY conf DESTINATION .)
install(DIRECTORY etc/docs/html DESTINATION .)
install(DIRECTORY etc/logs DESTINATION .)


# = Http Modules ==============================================================
function(add_http_module module_dir module_name)
	include_directories(./src/http/modules/${module_dir})
	aux_source_directory(./src/http/modules/${module_dir} MODULE_SRC_)
	source_group(src FILES ${MODULE_SRC_})
	set(MODULE_NAME_ http_${module_name})
	add_library(${MODULE_NAME_} MODULE ${MODULE_SRC_})
	if(APPLE)
		target_link_options(${MODULE_NAME_} PUBLIC -Wl,-undefined,dynamic_lookup)
	endif(APPLE)
	set_target_properties(${MODULE_NAME_} PROPERTIES
		OUTPUT_NAME "ngx_http_${module_name}_module"
		PREFIX ""
		POSITION_INDEPENDENT_CODE ON
	)
	if(ARGN)
		target_link_libraries(${MODULE_NAME_} ${ARGN})
	endif(ARGN)
	install(TARGETS ${MODULE_NAME_}
		RUNTIME DESTINATION sbin
		LIBRARY DESTINATION modules
		ARCHIVE DESTINATION lib
	)
endfunction(add_http_module)

add_http_module(xslt xslt_filter xml2 xslt exslt)
add_http_module(image image_filter gd)
add_http_module(geoip geoip GeoIP)


# = Stream Modules ============================================================
function(add_stream_module module_dir)
	aux_source_directory(./src/stream/${module_dir} MODULE_SRC_)
	source_group(src FILES ${MODULE_SRC_})
	set(MODULE_NAME_ stream_${module_dir})
	add_library(${MODULE_NAME_} MODULE ${MODULE_SRC_})
	if(APPLE)
		target_link_options(${MODULE_NAME_} PUBLIC -Wl,-undefined,dynamic_lookup)
	endif(APPLE)
	set_target_properties(${MODULE_NAME_} PROPERTIES
		OUTPUT_NAME "ngx_stream_${module_dir}_module"
		PREFIX ""
		POSITION_INDEPENDENT_CODE ON
	)
	target_link_libraries(${MODULE_NAME_} ${ARGN})
	install(TARGETS ${MODULE_NAME_}
		RUNTIME DESTINATION sbin
		LIBRARY DESTINATION modules
		ARCHIVE DESTINATION lib
	)
endfunction(add_stream_module)

add_stream_module(geoip GeoIP)	
