project(datareftool CXX)

SET(SRC about_window.cpp plugin.cpp viewer_window.cpp viewer_window_command_button.cpp allrefs.cpp dataref.cpp commandref.cpp dataref_files.cpp string_util.cpp find_datarefs_in_files.cpp prefs.cpp logging.cpp search.cpp ../lib/glew/glew.c)
SET(HDR about_window.h plugin.h viewer_window.h viewer_window_command_button.h allrefs.h dataref.h commandref.h ref.h dataref_files.h string_util.h clipboard.h find_datarefs_in_files.h prefs.h logging.h search.h ../lib/glew/glew.h)

SET(SRC_PLUGIN_EXAMPLE plugin_custom_dataref.cpp)
SET(SRC_PLUGIN_CRASHER plugin_crasher.cpp)


if(APPLE)
	list(APPEND SRC "clipboard_mac.mm")
elseif(WIN32)
	list(APPEND SRC "clipboard_win.cpp")
else()
	list(APPEND SRC "clipboard_linux.cpp")
endif()

SET(ALL_SRC ${SRC} ${HDR})
LIST(SORT ALL_SRC)

SOURCE_GROUP(src FILES ${ALL_SRC})

if(APPLE)
	add_executable(plugin ${ALL_SRC})
	add_executable(plugin_custom_dataref ${SRC_PLUGIN_EXAMPLE})
	add_executable(plugin_crasher ${SRC_PLUGIN_CRASHER})
elseif(UNIX)
	add_library(plugin SHARED ${ALL_SRC})
	add_library(plugin_custom_dataref SHARED ${SRC_PLUGIN_EXAMPLE})
	add_library(plugin_crasher SHARED ${SRC_PLUGIN_CRASHER})
elseif(WIN32)
	add_library(plugin MODULE ${ALL_SRC})
	add_library(plugin_custom_dataref MODULE ${SRC_PLUGIN_EXAMPLE})
	add_library(plugin_crasher MODULE ${SRC_PLUGIN_CRASHER})
	SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
	SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /MD")
	SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
endif()

#compiler flags
if(WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCHECK_RESULT_USED=_Check_return_")
	add_definitions(-DNOMINMAX -DGLEW_STATIC -D_SCL_SECURE_NO_WARNINGS)
else()
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra --std=c++14")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCHECK_RESULT_USED=\"__attribute__ ((warn_unused_result))\"")
	if(APPLE)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wmost")
	endif()
endif()

#libraries
set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_DEBUG ON)
if(WIN32)
	set(Boost_USE_STATIC_RUNTIME OFF)
	add_definitions( -DBOOST_ALL_NO_LIB ) # disable auto linking
endif()

find_package(Boost 1.54.0 REQUIRED COMPONENTS filesystem system iostreams)
include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
target_link_libraries(plugin PUBLIC ${Boost_LIBRARIES})

if(UNIX)
	include_directories(SYSTEM "../lib/xplane_sdk/CHeaders/XPLM/" "../lib/xplane_sdk/CHeaders/Widgets/" "../lib/xplane_sdk/CHeaders/Wrappers/")
endif(UNIX)
if(APPLE)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -arch x86_64")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -arch x86_64")
endif(APPLE)
if(WIN32)
	include_directories(SYSTEM "..\\lib\\xplane_sdk\\CHeaders\\XPLM" "..\\lib\\xplane_sdk\\CHeaders\\Widgets" "..\\lib\\xplane_sdk\\CHeaders\\Wrappers")
endif(WIN32)

# X-Plane stuff
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DXPLM200=1 -DXPLM210=1 -DXPLM300=1 -DXPLM301=1")
if(APPLE)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DAPL=1 -DIBM=0 -DLIN=0")
elseif(UNIX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DAPL=0 -DIBM=0 -DLIN=1")
elseif(WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DAPL=0 -DIBM=1 -DLIN=0 -D_CRT_SECURE_NO_WARNINGS /bigobj")
endif()

#linking
if(APPLE)
	set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ../lib/xplane_sdk/Libraries/Mac/)
	FIND_LIBRARY(XPLM_LIBRARY XPLM ../lib/xplane_sdk/Libraries/Mac/)
	FIND_LIBRARY(XPWIDGETS_LIBRARY XPWidgets ../lib/xplane_sdk/Libraries/Mac/)
	target_link_libraries(plugin PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY})
	target_link_libraries(plugin_custom_dataref PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY})
	target_link_libraries(plugin_crasher PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY})
	target_link_libraries(plugin PUBLIC m)
elseif(UNIX)
	target_link_libraries(plugin PUBLIC m)
#	target_link_libraries(plugin "-lGL" "-lGLU")
elseif(WIN32)
	set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "..\\lib\\xplane_sdk\\Libraries\\Win\\")
	FIND_LIBRARY(XPLM_LIBRARY XPLM_64 "..\\lib\\xplane_sdk\\Libraries\\Win\\")
	FIND_LIBRARY(XPWIDGETS_LIBRARY XPWidgets_64 "..\\lib\\xplane_sdk\\Libraries\\Win\\")
	target_link_libraries(plugin PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY} OpenGL32.lib)
	target_link_libraries(plugin_custom_dataref PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY})
	target_link_libraries(plugin_crasher PUBLIC ${XPLM_LIBRARY} ${XPWIDGETS_LIBRARY})
endif()

#output
SET_TARGET_PROPERTIES(plugin PROPERTIES PREFIX "")
SET_TARGET_PROPERTIES(plugin PROPERTIES SUFFIX "")
SET_TARGET_PROPERTIES(plugin_custom_dataref PROPERTIES PREFIX "")
SET_TARGET_PROPERTIES(plugin_custom_dataref PROPERTIES SUFFIX "")
SET_TARGET_PROPERTIES(plugin_crasher PROPERTIES SUFFIX "")
if(APPLE)
	set(PLUGIN_OUTPUT_FILENAME "mac.xpl")
	SET_TARGET_PROPERTIES(plugin PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}  -fPIC -fvisibility=hidden -bundle -framework OpenGL -framework Cocoa")
	SET_TARGET_PROPERTIES(plugin_custom_dataref PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}  -fPIC -fvisibility=hidden -bundle")
	SET_TARGET_PROPERTIES(plugin_crasher PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}  -fPIC -fvisibility=hidden -bundle")
elseif(UNIX)
	set(PLUGIN_OUTPUT_FILENAME "lin.xpl")
	SET_TARGET_PROPERTIES(plugin PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -rdynamic -nodefaultlibs -undefined_warning -fPIC -fvisibility=hidden")
	SET_TARGET_PROPERTIES(plugin_custom_dataref PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -rdynamic -nodefaultlibs -undefined_warning -fPIC -fvisibility=hidden")
	SET_TARGET_PROPERTIES(plugin_crasher PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -rdynamic -nodefaultlibs -undefined_warning -fPIC -fvisibility=hidden")
elseif(WIN32)
	set(PLUGIN_OUTPUT_FILENAME "win.xpl")
endif()


set_target_properties(plugin PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool" )
set_target_properties(plugin PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool" )
set_target_properties(plugin PROPERTIES OUTPUT_NAME "${PLUGIN_OUTPUT_FILENAME}")

set_target_properties(plugin_custom_dataref PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool_custom_dataref_example" )
set_target_properties(plugin_custom_dataref PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool_custom_dataref_example" )
set_target_properties(plugin_custom_dataref PROPERTIES OUTPUT_NAME "${PLUGIN_OUTPUT_FILENAME}")

set_target_properties(plugin_crasher PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool_crasher" )
set_target_properties(plugin_crasher PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/datareftool_crasher" )
set_target_properties(plugin_crasher PROPERTIES OUTPUT_NAME "${PLUGIN_OUTPUT_FILENAME}")
