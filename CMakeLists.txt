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
elseif(WIN32)
	set(CMAKE_C_FLAGS         "-Wall")
	# set(CMAKE_C_FLAGS_DEBUG   "")
	set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
endif()


if(APPLE)
	include_directories(/usr/local/include)
	link_directories(/usr/local/lib)
	include_directories(/usr/local/opt/openssl/include)
	link_directories(/usr/local/opt/openssl/lib)
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
elseif(WIN32)
endif()

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

if(APPLE)
	add_folder_files_to_group("./src/config/darwin" "config")
	add_folder_files_to_group("./src/core/threads" "core\\\\threads")
	add_folder_files_to_group("./src/os/unix/threads" "os\\\\unix\\\\threads")
	add_folder_files_to_group("./src/os/unix" "os\\\\unix")
	add_folder_files_to_group("./src/os/unix/darwin" "os\\\\unix\\\\darwin")
	add_files_to_group("event\\\\modules"
		./src/event/modules/ngx_select_module.c
		./src/event/modules/ngx_poll_module.c
		./src/event/modules/ngx_kqueue_module.c
	)
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
	add_folder_files_to_group("./src/config/linux" "config")
	add_folder_files_to_group("./src/core/threads" "core\\\\threads")
	add_folder_files_to_group("./src/os/unix/threads" "os\\\\unix\\\\threads")
	add_folder_files_to_group("./src/os/unix" "os\\\\unix")
	add_folder_files_to_group("./src/os/unix/linux" "os\\\\unix\\\\linux")
	add_files_to_group("event\\\\modules"
		./src/event/modules/ngx_select_module.c
		./src/event/modules/ngx_poll_module.c
		./src/event/modules/ngx_epoll_module.c
	)
elseif(WIN32)
	add_folder_files_to_group("./src/os/win32" "os\\\\win32")
	include_directories(./src/event/modules)
	add_files_to_group("event\\\\modules"
		./src/event/modules/ngx_win32_select_module.c
		./src/event/modules/ngx_win32_poll_module.c
		./src/event/modules/ngx_iocp_module.c
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
elseif(WIN32)
	target_link_libraries(${PROJECT_NAME} pcre z)
endif()

function(add_nginx_module module_dir)
	include_directories(./src/${module_dir})
	aux_source_directory(./src/${module_dir} MODULE_SRC_)
	source_group(src FILES ${MODULE_SRC_})
	set(MODULE_NAME_ ${module_dir}_module)
	add_library(${MODULE_NAME_} MODULE ${MODULE_SRC_})
	if(APPLE)
		target_link_options(${MODULE_NAME_} PUBLIC -Wl,-undefined,dynamic_lookup)
	endif(APPLE)
	set_target_properties(${MODULE_NAME_} PROPERTIES
		OUTPUT_NAME "ngx_${module_dir}_module"
		PREFIX ""
		POSITION_INDEPENDENT_CODE ON
	)
	install(TARGETS ${MODULE_NAME_}
		RUNTIME DESTINATION sbin
		LIBRARY DESTINATION modules
		ARCHIVE DESTINATION lib
	)
endfunction(add_nginx_module)

include_directories(
	./src/mail/imap
	./src/mail/pop3
	./src/mail/smtp
)
add_nginx_module(mail)
add_nginx_module(stream)

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
	target_link_libraries(${MODULE_NAME_} ${ARGN})
	install(TARGETS ${MODULE_NAME_}
		RUNTIME DESTINATION sbin
		LIBRARY DESTINATION modules
		ARCHIVE DESTINATION lib
	)
endfunction(add_http_module)

add_http_module(xslt xslt_filter xml2 xslt exslt)
add_http_module(image image_filter gd)
add_http_module(geoip geoip GeoIP)

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

install(TARGETS ${PROJECT_NAME}
	RUNTIME DESTINATION sbin
	LIBRARY DESTINATION modules
	ARCHIVE DESTINATION lib
)
install(DIRECTORY conf DESTINATION .)
install(DIRECTORY etc/docs/html DESTINATION .)
install(DIRECTORY etc/logs DESTINATION .)
