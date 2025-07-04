cmake_minimum_required(VERSION 3.5)
project(barony)

if (NOT CMAKE_BUILD_TYPE)
	set (CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

if (DEFINED ENV{STEAMWORKS_ENABLED})
	set (STEAMWORKS_ENABLED $ENV{STEAMWORKS_ENABLED})
endif()

if (NOT DEFINED STEAMWORKS_ENABLED)
	set (STEAMWORKS_ENABLED 0)
	option(STEAMWORKS "Enable Steamworks" OFF)
	if (STEAMWORKS)
		set (STEAMWORKS_ENABLED 1)
	endif()
endif()

if (DEFINED ENV{OPTIMIZATION_LEVEL})
	set (OPTIMIZATION_LEVEL $ENV{OPTIMIZATION_LEVEL})
else()
	set (OPTIMIZATION_LEVEL "-O3")
endif()

if (DEFINED ENV{EOS_ENABLED})
	set (EOS_ENABLED $ENV{EOS_ENABLED})
endif()

if (NOT DEFINED EOS_ENABLED)
	set (EOS_ENABLED 0)
	option(EOS "Enable Epic Online Services" OFF)
	if (EOS)
		set (EOS_ENABLED 1)
	endif()
	set(BUILD_ENV_PR "" CACHE STRING "EOS product id")
	set(BUILD_ENV_SA "" CACHE STRING "EOS sandbox")
	set(BUILD_ENV_DE "" CACHE STRING "EOS deployment")
	set(BUILD_ENV_CC "" CACHE STRING "EOS cc")
	set(BUILD_ENV_CS "" CACHE STRING "EOS cs")
	set(BUILD_ENV_GSE "" CACHE STRING "EOS gse")
endif()

if (DEFINED ENV{PLAYFAB_ENABLED})
    set (PLAYFAB_ENABLED $ENV{PLAYFAB_ENABLED})
endif()

if (NOT DEFINED PLAYFAB_ENABLED)
    set (PLAYFAB_ENABLED 0)
    option(PLAYFAB "Enable PlayFab support" OFF)
    if (PLAYFAB)
        set (PLAYFAB_ENABLED 1)
    endif()
    set(BUILD_ENV_PFTID "" CACHE STRING "PlayFab TID")
    set(BUILD_ENV_PFHID "" CACHE STRING "PlayFab HID")
endif()

if (DEFINED ENV{THEORAPLAYER_ENABLED})
    set (THEORAPLAYER_ENABLED $ENV{THEORAPLAYER_ENABLED})
endif()

if (NOT DEFINED THEORAPLAYER_ENABLED)
    set (THEORAPLAYER_ENABLED 0)
    option(THEORAPLAYER "Enable TheoraPlayer for video playback" OFF)
    if (THEORAPLAYER)
        set (THEORAPLAYER_ENABLED 1)
    endif()
endif()

if (DEFINED ENV{CURL_ENABLED})
    set (CURL_ENABLED $ENV{CURL_ENABLED})
endif()

if (NOT DEFINED CURL_ENABLED)
    set (CURL_ENABLED 0)
    option(CURL "Enable CURL support" OFF)
    if (CURL)
        set (CURL_ENABLED 1)
    endif()
endif()

if (DEFINED ENV{OPUS_ENABLED})
    set (OPUS_ENABLED $ENV{OPUS_ENABLED})
endif()

if (NOT DEFINED OPUS_ENABLED)
    set (OPUS_ENABLED 0)
    option(OPUS "Enable OPUS support" OFF)
    if (OPUS)
        set (OPUS_ENABLED 1)
    endif()
endif()

if (PLAYFAB_ENABLED AND NOT CURL_ENABLED)
	message(FATAL_ERROR "CURL must be enabled if PlayFab is enabled.")
endif()

# EOS & PlayFab Tokens

if (DEFINED ENV{BUILD_ENV_PR} AND NOT DEFINED BUILD_ENV_PR) # TODO: We should make all environment variable checks in this file not overwrite whatever was explicitly passed in on the command line...
	set (BUILD_ENV_PR $ENV{BUILD_ENV_PR})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_PR}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_PR must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_SA} AND NOT DEFINED BUILD_ENV_SA)
	set (BUILD_ENV_SA $ENV{BUILD_ENV_SA})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_SA}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_SA must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_DE} AND NOT DEFINED BUILD_ENV_DE)
	set (BUILD_ENV_DE $ENV{BUILD_ENV_DE})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_DE}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_DE must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_CC} AND NOT DEFINED BUILD_ENV_CC)
	set (BUILD_ENV_CC $ENV{BUILD_ENV_CC})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_CC}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_CC must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_CS} AND NOT DEFINED BUILD_ENV_CS)
	set (BUILD_ENV_CS $ENV{BUILD_ENV_CS})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_CS}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_CS must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_GSE} AND NOT DEFINED BUILD_ENV_GSE)
	set (BUILD_ENV_GSE $ENV{BUILD_ENV_GSE})
endif()
if (EOS_ENABLED AND "${BUILD_ENV_GSE}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_GSE must be specified if EOS is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_PFTID} AND NOT DEFINED BUILD_ENV_PFTID)
	set (BUILD_ENV_PFTID $ENV{BUILD_ENV_PFTID})
endif()
if (PLAYFAB_ENABLED AND "${BUILD_ENV_PFTID}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_PFTID must be specified if PlayFab is enabled.")
endif()

if (DEFINED ENV{BUILD_ENV_PFHID} AND NOT DEFINED BUILD_ENV_PFHID)
	set (BUILD_ENV_PFHID $ENV{BUILD_ENV_PFHID})
endif()
if (PLAYFAB_ENABLED AND "${BUILD_ENV_PFHID}" STREQUAL "")
	message(FATAL_ERROR "BUILD_ENV_PFHID must be specified if PlayFab is enabled.")
endif()

if (DEFINED ENV{EDITOR_ENABLED})
	set (EDITOR_ENABLED $ENV{EDITOR_ENABLED})
else()
	set (EDITOR_ENABLED 1)
endif()

if (DEFINED ENV{GAME_ENABLED})
	set (GAME_ENABLED $ENV{GAME_ENABLED})
else()
	set (GAME_ENABLED 1)
endif()

if (DEFINED ENV{BARONY_WIN32_LIBRARIES})
	set (BARONY_WIN32_LIBRARIES $ENV{BARONY_WIN32_LIBRARIES})
endif()

if (NOT "${BARONY_WIN32_LIBRARIES}" STREQUAL "")
	list(APPEND CMAKE_PREFIX_PATH $ENV{BARONY_WIN32_LIBRARIES})
endif(NOT "${BARONY_WIN32_LIBRARIES}" STREQUAL "")

if (DEFINED ENV{BARONY_DATADIR}) # Used by visual studio to set the current working directory when debugging the barony executables.
	set (BARONY_DATADIR $ENV{BARONY_DATADIR})
endif()

#FMOD can be disabled in one of two ways:
# * Not having FMOD installed will automatically disable FMOD (and, consequently, the game's sound engine as a whole).
# * Running cmake with -DFMOD_ENABLED=OFF
set(FMOD 0) #Acts as an alias to FMOD_FOUND
option(FMOD_ENABLED "Use the FMOD Ex library in the sound engine" ON)
set(OPENAL 0) #Acts as an alias to OPENAL_FOUND
option(OPENAL_ENABLED "Use the OpenAL library in the sound engine" OFF)
set(TREMOR 0) #Acts as an alias to TREMOR_FOUND
option(TREMOR_ENABLED "Use Tremor instead of libvorbis for OpenAL" OFF)

if(FMOD_ENABLED AND OPENAL_ENABLED)
	set(FMOD_ENABLED OFF)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

set(CMAKE_OSX_ARCHITECTURES x86_64)

if (EDITOR_ENABLED)
	set(EDITOR_EXE_NAME "editor" CACHE STRING "Editor executable name")
endif()
option(DATA_DIR "Points to install data directory" FALSE)

if (STEAMWORKS_ENABLED)
	message("Building with steamworks")
else()
	message("Building without steamworks")
endif()

if (EOS_ENABLED)
	message("Building with EOS")
else()
	message("Building without EOS")
endif()

if (PLAYFAB_ENABLED)
	message("Building with PlayFab")
else()
	message("Building without PlayFab")
endif()

if (THEORAPLAYER_ENABLED)
    message("Building with TheoraPlayer")
else()
    message("Building without TheoraPlayer")
endif()

if (CURL_ENABLED)
	message("Building with CURL")
else()
	message("Building without CURL")
endif()

if (OPUS_ENABLED)
	message("Building with OPUS")
else()
	message("Building without OPUS")
endif()

# OS/Platform Flags
set(Apple 0)
set(Windows 0)
set(Linux 0)
set(Bsd 0)
set(Haiku 0)
set(PANDORA 0)
set(NOT_DWORD_DEFINED 0)
option(PANDORA_ENABLED "Build for the OpenPandora" OFF)
if(PANDORA_ENABLED)
	set(PANDORA 1)
endif()

# Set platform-dependent compiler & linker flags
if( NOT WIN32 )
  if(PANDORA)
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -ffast-math -funroll-loops -fstrict-aliasing -mcpu=cortex-a8 -mfpu=neon -mfloat-abi=softfp -g -fuse-ld=gold -fuse-linker-plugin")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} ${OPTIMIZATION_LEVEL} -ffast-math -funroll-loops -fstrict-aliasing -mcpu=cortex-a8 -mfpu=neon -mfloat-abi=softfp -g -fuse-ld=gold -fuse-linker-plugin -fsingle-precision-constant")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${OPTIMIZATION_LEVEL} -ffast-math -funroll-loops -fstrict-aliasing -mcpu=cortex-a8 -mfpu=neon -mfloat-abi=softfp -g -fuse-ld=gold -fuse-linker-plugin -fsingle-precision-constant")
  else()
	set(DISABLED_WARNINGS "-Wno-pedantic -Wno-empty-body -Wno-string-plus-int -Wno-parentheses -Wno-format-security -Wno-multichar")
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${DISABLED_WARNINGS} -pg -ffast-math -funroll-loops -fstrict-aliasing")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${DISABLED_WARNINGS} -Wno-inconsistent-missing-override -pg")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} ${DISABLED_WARNINGS} ${OPTIMIZATION_LEVEL} -ffast-math -funroll-loops -fstrict-aliasing")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${DISABLED_WARNINGS} -Wno-inconsistent-missing-override ${OPTIMIZATION_LEVEL}")
  endif()
  if (APPLE)
	set (Apple 1)
	set (NOT_DWORD_DEFINED 1)
	#set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -x objective-c")
	#set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -x objective-c")
	set(MACOSX_BUNDLE true)
	set(MACOSX_BUNDLE_BUNDLE_NAME Barony)
	#TODO: Make version a define which is also set in src/Config.h
	set(MACOSX_BUNDLE_INFO_STRING Barony 2.0.2)
	set(MACOSX_BUNDLE_LONG_VERSION 2.0.2)
	set(MACOSX_BUNDLE_SHORT_VERSION_STRING 2.0.2)
	set(MACOSX_BUNDLE_BUNDLE_VERSION 2.0.2)
	set(MACOSX_BUNDLE_ICON_FILE "barony.icns")
	#set(MACOSX_BUNDLE_ICON_FILE "game.icns")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -std=c++17 -stdlib=libc++ -I/opt/local/include/")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -std=c++17 -stdlib=libc++ -I/opt/local/include/")
  else()
	set (Apple 0)
	set (Linux 1) #TODO: Should not assume this? What if BSD/DOS/whatever-have-you?
	if (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD|NetBSD|DragonFly") # OpenBSD is purposely discarded at it lacks for now current exe path retrieval feature
	   set(Bsd 1)
	endif()
	if (${CMAKE_SYSTEM_NAME} MATCHES "Haiku")
	   set(Haiku 1)
	endif()
	set (NOT_DWORD_DEFINED 1)
	if (NOT DISABLE_MALIGN_DOUBLE) #Only compile with -malign-double if not told to disable. Pass via command line flags.
	  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -malign-double")
	  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -malign-double")
	endif()
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -rdynamic")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -std=c++17")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -std=c++17")
	if (CMAKE_COMPILER_IS_GNUCXX)
		set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -Wl,-rpath='$ORIGIN'")
		set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wl,-rpath='$ORIGIN'")
		set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -Wl,-rpath='$ORIGIN'")
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wl,-rpath='$ORIGIN'")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath='$ORIGIN'")
	endif()
  endif()

	if (ARCHITECTURE) # TODO: Support this on Windows too.
		set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -mcpu=${ARCHITECTURE}")
		set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_DEBUG} -mcpu=${ARCHITECTURE}")
	endif()
else( NOT WIN32 )
	# These look like mingw to me?
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -pg ${OPTIMIZATION_LEVEL} -ffast-math -funroll-loops -malign-double -fstrict-aliasing -mwindows")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} ${OPTIMIZATION_LEVEL} -ffast-math -funroll-loops -malign-double -fstrict-aliasing -mwindows")
	# We should be using Visual Studio flags...or better yet, providing an option to select which you'd prefer.
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MP")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MP ")
	set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS")
  set (Windows 1)
endif( NOT WIN32 )

if (APPLE)
	set ( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -LD_RUNPATH_SEARCH_PATHS=@loader_path/../Frameworks -rpath @loader_path/../Frameworks")
endif()
set ( CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}")
set ( CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS}")

cmake_policy(SET CMP0072 NEW) # use new opengl libs on linux

IF (APPLE)
  #include_directories(/usr/X11R6/include/)
  #link_directories(/usr/X11R6/lib)
  #FIND_LIBRARY(OpenGL_LIBRARY OpenGL )
  #SET(EXTRA_LIBS ${OpenGL_LIBRARY} X11)
  FIND_LIBRARY(OpenGL_LIBRARY OpenGL )
  SET(EXTRA_LIBS ${OpenGL_LIBRARY})

  #set(CMAKE_LIBRARY_PATH "/Library/Frameworks/")
  find_package(SDL2 REQUIRED)
  find_package(SDL2_image REQUIRED)
  find_package(SDL2_net REQUIRED)
  find_package(SDL2_ttf REQUIRED)
  find_package(IOKit REQUIRED)
ELSE (APPLE)
	find_package(OpenGL REQUIRED)
	IF(NOT WIN32)
		INCLUDE(FindPkgConfig)
		PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
		PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)
		PKG_SEARCH_MODULE(SDL2NET REQUIRED SDL2_net)
		PKG_SEARCH_MODULE(SDL2TTF REQUIRED SDL2_ttf)
	ELSE(NOT WIN32)
		find_package(SDL2 REQUIRED)
		find_package(SDL2_image REQUIRED)
	ENDIF(NOT WIN32)
	find_package(SDL2_net REQUIRED) # ...why the duplication?
	find_package(SDL2_ttf REQUIRED)
ENDIF (APPLE)
find_package(PhysFS REQUIRED)
find_package(RapidJson REQUIRED)
INCLUDE_DIRECTORIES(${RAPID_JSON_INCLUDE_DIR})

if (STEAMWORKS_ENABLED)
  find_package(NFD REQUIRED)
  INCLUDE_DIRECTORIES(${NFD_INCLUDE_DIR})
  LINK_DIRECTORIES(${NFD_LIBRARY})
ENDIF (STEAMWORKS_ENABLED)

INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2_INCLUDE_DIR} ${SDL2IMAGE_INCLUDE_DIRS} ${SDL2_IMAGE_INCLUDE_DIR} ${SDL2NET_INCLUDE_DIR} ${SDL2_NET_INCLUDE_DIRS} ${SDL2TTF_INCLUDE_DIR}  ${SDL2_TTF_INCLUDE_DIRS} ${PHYSFS_INCLUDE_DIR})
LINK_DIRECTORIES(${SDL2_LINK_DIRS} ${SDL2_LINK_DIR} ${SDL2IMAGE_LINK_DIRS} ${SDL2_IMAGE_LINK_DIR} ${SDL2NET_LINK_DIR} ${SDL2_NET_LINK_DIRS} ${SDL2TTF_LINK_DIR}  ${SDL2_TTF_LINK_DIRS} ${PHYSFS_LINK_DIR})

# if (NOT WIN32)
# else()
#   INCLUDE_DIRECTORIES(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS})
# endif()
if (OPENAL_ENABLED)
  find_package(OpenAL)
  if(TREMOR_ENABLED)
	find_package(TREMOR)
  else()
	find_package(OGGVORBIS)
  endif()
  if (OPENAL_FOUND AND ((TREMOR_ENABLED AND TREMOR_FOUND) OR ((NOT TREMOR_ENABLED) AND OGGVORBIS_FOUND)))
	INCLUDE_DIRECTORIES(${OPENAL_INCLUDE_DIR})
	if(TREMOR_ENABLED)
	  set(TREMOR 1)
	  INCLUDE_DIRECTORIES(${TREMOR_INCLUDE_DIR})
	else()
	  INCLUDE_DIRECTORIES(${OGG_INCLUDE_DIR} ${VORBIS_INCLUDE_DIR})
	endif()
	set(OPENAL 1)
  endif()
endif()

set(IMGUI 1)

find_package(Threads REQUIRED)
if (FMOD_ENABLED)
	find_package(FMOD REQUIRED)
	if (FMOD_FOUND)
		set(FMOD 1)
	endif()
endif()
if(NOT APPLE)
  #Linux needs these two.
  #Macs have their own special line.
  find_package(PNG REQUIRED)
elseif (APPLE)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FMOD_CXX_FLAGS}")
  MESSAGE("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
  set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${FMOD_LINK_FLAGS}")
  MESSAGE("CMAKE_EXE_LINKER_FLAGS: ${CMAKE_EXE_LINKER_FLAGS}")
endif()

if(STEAMWORKS_ENABLED)
  find_package(STEAMWORKS REQUIRED)
  include_directories(${STEAMWORKS_INCLUDE_DIR})
endif()

if(EOS_ENABLED)
  find_package(EOS REQUIRED)
  include_directories(${EOS_INCLUDE_DIR})
endif()

if (PLAYFAB_ENABLED)
  find_package(PLAYFAB REQUIRED)
  include_directories(${PLAYFAB_INCLUDE_DIR})
  link_directories(${PLAYFAB_LIBRARY})
  find_package(JsonCPP REQUIRED)
  INCLUDE_DIRECTORIES(${JSONCPP_INCLUDE_DIR})
  LINK_DIRECTORIES(${JSONCPP_LIBRARY})
endif()

if (THEORAPLAYER_ENABLED)
  find_package(TheoraPlayer REQUIRED)
  include_directories(${THEORAPLAYER_INCLUDE_DIR})
  link_directories(${THEORAPLAYER_LIBRARIES})
  find_package(OGGVORBIS REQUIRED)
  include_directories(${OGG_INCLUDE_DIR} ${VORBIS_INCLUDE_DIR})
  link_directories(${OGGVORBIS_LIBRARIES})
endif()

if (CURL_ENABLED)
  find_package(OpenSSL REQUIRED)
  include_directories(${OPENSSL_INCLUDE_DIR})
  link_directories(${OPENSSL_LIBRARY})
  find_package(CURL REQUIRED)
  include_directories(${CURL_INCLUDE_DIR})
  link_directories(${CURL_LIBRARIES})
endif()

if (OPUS_ENABLED)
  find_package(Opus REQUIRED)
  include_directories(${OPUS_INCLUDE_DIR})
  link_directories(${OPUS_LIBRARIES})
endif()

# Add include directories
#include_directories(${GLM_INCLUDE_DIR})
if( NOT WIN32 )
 	include_directories(${THREADS_INCLUDE_DIR})
endif()
if(NOT APPLE)
 	include_directories(${PNG_INCLUDE_DIR})
endif()
if (FMOD_FOUND)
	include_directories(${FMOD_INCLUDE_DIR})
endif()

# Add the source directory
#file(GLOB_RECURSE SOURCE_FILES src/*.c src/*.h) # Can't do this because it'd then compile barony and editor together. Which just won't work.
#file(GLOB_RECURSE BOOK_FILES books/*)
#file(GLOB_RECURSE IMAGE_FILES images/*)
#file(GLOB_RECURSE ITEM_FILES items/*)
#file(GLOB_RECURSE MAP_FILES maps/*)
#file(GLOB_RECURSE MODEL_FILES models/*)
#file(GLOB_RECURSE MUSIC_FILES music/*)
#file(GLOB_RECURSE SOUND_FILES sound/*)

#add_subdirectory(books)
 
configure_file ( "${PROJECT_SOURCE_DIR}/src/Config.hpp.in" "${PROJECT_SOURCE_DIR}/src/Config.hpp")

message("***************************")
message("Debug flags: ")
message("${CMAKE_C_FLAGS_DEBUG}")
message("${CMAKE_CXX_FLAGS_DEBUG}")
#-g -pg -ffast-math -funroll-loops -fstrict-aliasing
message("Release flags: ")
message("${CMAKE_C_FLAGS_RELEASE}")
message("${CMAKE_CXX_FLAGS_RELEASE}")
#-O3 -DNDEBUG -pg -ffast-math -funroll-loops -fstrict-aliasing
message("***************************")


add_subdirectory(src)
if( WIN32 )
	set(GAME_RESOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/barony.rc")
	set(EDITOR_RESOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/editor.rc")
endif()

#ADD_DEFINITONS(${PNG_DEFINITIONS})
if (APPLE)
  set(COPY_FRAMEWORKS ${PROJECT_SOURCE_DIR}/../Frameworks/SDL.framework ${PROJECT_SOURCE_DIR}/../Frameworks/SDL_image.framework ${PROJECT_SOURCE_DIR}/../Frameworks/SDL_mixer.framework ${PROJECT_SOURCE_DIR}/../Frameworks/SDL_net.framework)
  message("{COPY_FRAMEWORKS}: ${COPY_FRAMEWORKS}")
  if (FMOD_ENABLED)
	SET_SOURCE_FILES_PROPERTIES(libfmod.dylib PROPERTIES MACOSX_PACKAGE_LOCATION MacOS) # TODO: Use ${FMOD_LIBRARY} instead?
  endif()
  #add_executable(barony MACOSX_BUNDLE OSX/SDLmain.m ${GAME_SOURCES} ${COPY_FRAMEWORKS} ${BOOK_FILES} ${IMAGE_FILES} ${ITEM_FILES} ${MAP_FILES} ${MODEL_FILES} ${MUSIC_FILES} ${SOUND_FILES} libfmodex.dylib)
  #add_executable(barony MACOSX_BUNDLE OSX/SDLmain.m ${GAME_SOURCES} ${COPY_FRAMEWORKS} libfmodex.dylib /opt/local/lib/libpng16.16.dylib)
  #add_executable(barony MACOSX_BUNDLE OSX/SDLmain.m ${GAME_SOURCES} ${MACOSX_BUNDLE_ICON_FILE} libfmodex.dylib /opt/local/lib/libpng16.16.dylib)
  if (GAME_ENABLED)
	add_executable(barony MACOSX_BUNDLE ${GAME_SOURCES} ${MACOSX_BUNDLE_ICON_FILE} ${PROJECT_SOURCE_DIR}/libpng16.16.dylib)
  #add_executable(barony OSX/SDLmain.m ${GAME_SOURCES})
  #add_executable(barony ${GAME_SOURCES})
  #SET_SOURCE_FILES_PROPERTIES(${COPY_FRAMEWORKS} PROPERTIES MACOSX_PACKAGE_LOCATION Frameworks)
  #SET_SOURCE_FILES_PROPERTIES(${BOOK_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/books/)
  #SET_SOURCE_FILES_PROPERTIES(${IMAGE_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/images/)
  #SET_SOURCE_FILES_PROPERTIES(${ITEM_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/items/)
  #SET_SOURCE_FILES_PROPERTIES(${MAP_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/maps/)
  #SET_SOURCE_FILES_PROPERTIES(${MODEL_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/models/)
  #SET_SOURCE_FILES_PROPERTIES(${MUSIC_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/music/)
  #SET_SOURCE_FILES_PROPERTIES(${SOUND_FILES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources/sound/)
	set_source_files_properties(${GAME_SOURCES} PROPERTIES COMPILE_FLAGS "-x objective-c++")
  endif(GAME_ENABLED)
  SET_SOURCE_FILES_PROPERTIES(${PROJECT_SOURCE_DIR}/libpng16.16.dylib PROPERTIES MACOSX_PACKAGE_LOCATION MacOS)
  SET_SOURCE_FILES_PROPERTIES(${COPY_RESOURCES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  SET_SOURCE_FILES_PROPERTIES(${MACOSX_BUNDLE_ICON_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
  #set_source_files_properties(${GAME_SOURCES} PROPERTIES COMPILE_FLAGS "-stdlib=libc++")
else()
  if (GAME_ENABLED)
	add_executable(barony ${GAME_SOURCES} ${GAME_RESOURCE_FILE})
  endif (GAME_ENABLED)
endif(APPLE)

if (GAME_ENABLED)
  if(WIN32)
    # Only for MinGW...
	# target_link_libraries(barony -lmingw32)
	# target_link_libraries(barony -lSDL2main)
	# target_link_libraries(barony -lSDL2)
	# target_link_libraries(barony -lSDL2_net)
	# target_link_libraries(barony -lSDL2_image)
	# target_link_libraries(barony -lSDL2_ttf)
	# if(STEAMWORKS_ENABLED)
	#   #target_link_libraries(barony -lsteamworks_cwrapper -lstdc++)
	#   target_link_libraries(barony -lsteam_api)
	# endif()
	# if(EOS_ENABLED)
	#   target_link_libraries(barony -lsteam_api) #TODO: Fix for Windows.
	# endif()
	set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT barony) # Otherwise, hitting "debug" will default to "ALL_BUILD" :|
	if (DEFINED BARONY_DATADIR)
		set_target_properties(barony PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "\$(BARONY_DATADIR)/")
	endif()
	target_link_libraries(barony ${SDL2_LIBRARIES} ${SDL2_LIBRARY} ${SDL2IMAGE_LIBRARIES} ${SDL2_IMAGE_LIBRARIES} ${SDL2_NET_LIBRARIES} ${SDL2_TTF_LIBRARIES} ${SDL2TTF_LIBRARY})
	if (STEAMWORKS_ENABLED)
		target_link_libraries(barony ${STEAMWORKS_LIBRARY})
		target_link_libraries(barony ${NFD_LIBRARY})
	endif()
	if (EOS_ENABLED)
		target_link_libraries(barony ${EOS_LIBRARIES})
	endif()
	if (PLAYFAB_ENABLED)
	    target_link_libraries(barony ${PLAYFAB_LIBRARY})
	    target_link_libraries(barony ${JSONCPP_LIBRARY})
	endif()
	if (THEORAPLAYER_ENABLED)
	    target_link_libraries(barony ${THEORAPLAYER_LIBRARIES} ${OGGVORBIS_LIBRARIES})
	endif()
	if (OPUS_ENABLED)
	    target_link_libraries(barony ${OPUS_LIBRARIES})
	endif()
	if (CURL_ENABLED)
	  target_link_libraries(barony ${CURL_LIBRARIES} ${OPENSSL_LIBRARIES})
	  #target_link_libraries(barony OpenSSL::applink)
	  #target_link_libraries(barony OpenSSL::SSL)
	  #target_link_libraries(barony OpenSSL::Crypto)
	endif()
  else(WIN32)
	if (CURL_ENABLED)
	  target_link_libraries(barony ${CURL_LIBRARIES} ${OPENSSL_LIBRARIES})
	  #target_link_libraries(barony OpenSSL::applink)
	  #target_link_libraries(barony OpenSSL::SSL)
	  #target_link_libraries(barony OpenSSL::Crypto)
	endif()
	if (THEORAPLAYER_ENABLED)
	    target_link_libraries(barony ${THEORAPLAYER_LIBRARIES} ${OGGVORBIS_LIBRARIES})
	endif()
	if (OPUS_ENABLED)
	    target_link_libraries(barony ${OPUS_LIBRARIES})
	endif()
	if (PLAYFAB_ENABLED)
	  target_link_libraries(barony ${PLAYFAB_LIBRARY})
	  target_link_libraries(barony ${JSONCPP_LIBRARY})
	endif()
	if( ("${CMAKE_SIZEOF_VOID_P}" EQUAL "8") AND (NOT APPLE) )
	  # 64 bit
	  target_link_libraries(barony -L/usr/lib64 -lstdc++)
	endif()
	target_link_libraries(barony ${SDL2_LIBRARIES} ${SDL2_LIBRARY} ${SDL2IMAGE_LIBRARIES} ${SDL2_IMAGE_LIBRARIES} ${SDL2_NET_LIBRARIES} ${SDL2_TTF_LIBRARIES} ${SDL2TTF_LIBRARY})
	if (STEAMWORKS_ENABLED)
	  target_link_libraries(barony ${STEAMWORKS_LIBRARY})
	  target_link_libraries(barony ${NFD_LIBRARY})
	endif()
	if (EOS_ENABLED)
	  target_link_libraries(barony ${EOS_LIBRARIES})
	endif()
	if (APPLE)
	  target_link_libraries(barony ${PROJECT_SOURCE_DIR}/libpng16.16.dylib) #Wait...what? if(APPLE) in if(WIN32)? What was I thinking back then, haha.
	endif()
	if (${CMAKE_SYSTEM_NAME} MATCHES "BSD|DragonFly" OR ${CMAKE_SYSTEM_NAME} MATCHES "Haiku")
	  # For backtrace
	  find_path(EXECINFO_INC NAMES execinfo.h)
	  find_library(EXECINFO_LIB NAMES execinfo)
	  include_directories(${EXECINFO_INC})
	  target_link_libraries(barony ${EXECINFO_LIB})
	endif()
	#if(LINUX)
		target_link_libraries(barony -lpthread)
	#endif(LINUX)
  endif()
  target_link_libraries(barony ${OPENGL_LIBRARIES})
  target_link_libraries(barony ${THREADS_LIBRARIES}) # TODO: Not sure what this is doing, since it doesn't appear to be working in Linux!
  target_link_libraries(barony -lm) # TODO: In Visual Studio, I bet this is something else, right?
  if (NOT CMAKE_SYSTEM_NAME MATCHES "Haiku")
	  target_link_libraries(barony -lc)
  endif()
  if (FMOD_FOUND)
    target_link_libraries(barony ${FMOD_LIBRARY})
  endif()
  target_link_libraries(barony ${PHYSFS_LIBRARY})
  if(NOT APPLE)
	#Remember, Mac isn't using find_package for PNG.
	target_link_libraries(barony ${PNG_LIBRARY})
  endif()
  if (APPLE)
	target_link_libraries(barony ${IOKit_LIBRARY})
  endif()
  # We need to link to Winsock if we're on Windows
  if(WIN32)
	target_link_libraries(barony wsock32 ws2_32)
	#target_link_libraries(barony -lpng -lfmodex) Mingw, not Visual Studio.
  endif()
  target_link_libraries(barony ${EXTRA_LIBS}) #Apple needs this for OpenGL to work.

  if (OPENAL)
	target_link_libraries(barony ${OPENAL_LIBRARY})
	if(TREMOR_ENABLED)
	  target_link_libraries(barony ${TREMOR_LIBRARY})
	else()
	  target_link_libraries(barony ${VORBISFILE_LIBRARY} ${OGG_LIBRARY})
	endif()
  endif()
endif(GAME_ENABLED)

set(BASE_DATA_DIR "./" CACHE INTERNAL "Base data dir")

if (NOT APPLE AND UNIX)
  include(GNUInstallDirs)
  if (DATA_DIR)
	set(BASE_DATA_DIR "${CMAKE_INSTALL_PREFIX}")
  endif()
  message(STATUS "Base data directory ${BASE_DATA_DIR}")

  if (GAME_ENABLED)
	install(TARGETS barony
	RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}"
	COMPONENT Runtime
	)
  endif(GAME_ENABLED)
else()
  if (GAME_ENABLED)
	install(TARGETS barony
	  RUNTIME DESTINATION bin COMPONENT Runtime BUNDLE DESTINATION Resources
	)
  endif(GAME_ENABLED)
endif()

add_definitions(-DBASE_DATA_DIR=\"${BASE_DATA_DIR}\")

if (EDITOR_ENABLED)
  if (APPLE)
	#add_executable(editor OSX/SDLmain.m ${EDITOR_SOURCES})
	#set(MACOSX_BUNDLE_ICON_FILE "${PROJECT_SOURCE_DIR}/editor.icns")
	#set(MACOSX_BUNDLE_ICON_FILE "editor.icns")
	#add_executable(editor MACOSX_BUNDLE OSX/SDLmain.m ${EDITOR_SOURCES} ${COPY_FRAMEWORKS} libfmodex.dylib  /opt/local/lib/libpng16.16.dylib)
	#add_executable(editor MACOSX_BUNDLE OSX/SDLmain.m ${EDITOR_SOURCES} libfmodex.dylib  /opt/local/lib/libpng16.16.dylib)
	set_source_files_properties("editor.icns" PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
	add_executable(${EDITOR_EXE_NAME} MACOSX_BUNDLE "editor.icns" ${EDITOR_SOURCES} ${PROJECT_SOURCE_DIR}/libpng16.16.dylib)
	#SET_SOURCE_FILES_PROPERTIES(${MACOSX_BUNDLE_ICON_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
	set_source_files_properties(${EDITOR_SOURCES} PROPERTIES COMPILE_FLAGS "-x objective-c++")
  else()
	if (WIN32)
		# add_executable(${EDITOR_EXE_NAME} EXCLUDE_FROM_ALL ${EDITOR_SOURCES} ${EDITOR_RESOURCE_FILE})
		add_executable(${EDITOR_EXE_NAME} ${EDITOR_SOURCES} ${EDITOR_RESOURCE_FILE})
		set_target_properties(${EDITOR_EXE_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "\$(BARONY_DATADIR)/")
	else()
		add_executable(${EDITOR_EXE_NAME} ${EDITOR_SOURCES} ${EDITOR_RESOURCE_FILE})
	endif()
  endif()
  target_compile_definitions(${EDITOR_EXE_NAME} PUBLIC EDITOR) # So that shared source files know this is the editor they're dealing with. # TODO: Try using this to disable some things we don't want in the editor?

  if (THEORAPLAYER_ENABLED)
    target_link_libraries(${EDITOR_EXE_NAME} ${THEORAPLAYER_LIBRARIES} ${OGGVORBIS_LIBRARIES})
  endif()
  if (OPUS_ENABLED)
    target_link_libraries(${EDITOR_EXE_NAME} ${OPUS_LIBRARIES})
  endif()
  if (CURL_ENABLED)
	target_link_libraries(${EDITOR_EXE_NAME} ${CURL_LIBRARIES} ${OPENSSL_LIBRARIES})
	#target_link_libraries(${EDITOR_EXE_NAME} OpenSSL::applink)
	#target_link_libraries(${EDITOR_EXE_NAME} OpenSSL::SSL)
	#target_link_libraries(${EDITOR_EXE_NAME} OpenSSL::Crypto)
  endif()
  target_link_libraries(${EDITOR_EXE_NAME} ${SDL2_LIBRARIES} ${SDL2_LIBRARY} ${SDL2IMAGE_LIBRARIES} ${SDL2_IMAGE_LIBRARIES} ${SDL2_NET_LIBRARIES} ${SDL2_TTF_LIBRARIES} ${SDL2TTF_LIBRARY})
  if(STEAMWORKS_ENABLED)
	target_link_libraries(${EDITOR_EXE_NAME} ${STEAMWORKS_LIBRARY})
    target_link_libraries(${EDITOR_EXE_NAME} ${NFD_LIBRARY})
  endif()
  if(WIN32)
	if (EOS_ENABLED)
	  target_link_libraries(${EDITOR_EXE_NAME} ${EOS_LIBRARY}) #TODO: Fix for Windows.
	endif()
  else()
	if( ("${CMAKE_SIZEOF_VOID_P}" EQUAL "8") AND (NOT APPLE))
	  # 64 bit
	  target_link_libraries(${EDITOR_EXE_NAME} -L/usr/lib64 -lstdc++)
	endif()
	#if(LINUX)
		target_link_libraries(${EDITOR_EXE_NAME} -lpthread)
	#endif(LINUX)
  endif()
  target_link_libraries(${EDITOR_EXE_NAME} ${OPENGL_LIBRARIES})
  target_link_libraries(${EDITOR_EXE_NAME} ${THREADS_LIBRARIES})
  target_link_libraries(${EDITOR_EXE_NAME} -lm) # TODO: Only for Mingw?
  if (NOT CMAKE_SYSTEM_NAME MATCHES "Haiku")
	  target_link_libraries(${EDITOR_EXE_NAME} -lc)
  endif()
  # We need to link to Winsock if we're on Windows
  if (FMOD_FOUND)
    target_link_libraries(${EDITOR_EXE_NAME} ${FMOD_LIBRARY})
  endif()
  if(NOT APPLE)
	#Remember, Mac isn't using find_package for FMOD and PNG.
	target_link_libraries(${EDITOR_EXE_NAME} ${PHYSFS_LIBRARY})
	target_link_libraries(${EDITOR_EXE_NAME} ${PNG_LIBRARY})
  endif()
  if (APPLE)
	target_link_libraries(${EDITOR_EXE_NAME} ${PHYSFS_LIBRARY})
	target_link_libraries(${EDITOR_EXE_NAME} ${IOKit_LIBRARY})
	if (EOS_ENABLED)
	  target_link_libraries(${EDITOR_EXE_NAME} ${EOS_LIBRARY})
	endif()
  endif()
  if(WIN32)
	target_link_libraries(${EDITOR_EXE_NAME} wsock32 ws2_32)
  endif()
  if (APPLE)
	if (FMOD_ENABLED)
	  target_link_libraries(${EDITOR_EXE_NAME} ${FMOD_LIBRARY}) #Finally manually link fmod for mac.
	endif()
  endif()
  target_link_libraries(${EDITOR_EXE_NAME} ${EXTRA_LIBS}) #Apple needs this for OpenGL to work.
  if (OPENAL)
	target_link_libraries(${EDITOR_EXE_NAME} ${OPENAL_LIBRARY})
	if(TREMOR_ENABLED)
	  target_link_libraries(${EDITOR_EXE_NAME} ${TREMOR_LIBRARY})
	else()
	  target_link_libraries(${EDITOR_EXE_NAME} ${VORBISFILE_LIBRARY})
	endif()
  endif()
endif(EDITOR_ENABLED)

# Various install targets
# 1) <TODO: insert proper description here...I think they're being used by the BSD maintainer(s)?>.
if (NOT APPLE AND UNIX)
  if (EDITOR_ENABLED)
	install(TARGETS ${EDITOR_EXE_NAME}
	  RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}" COMPONENT Runtime
	)
  endif()

  install(DIRECTORY
	${CMAKE_SOURCE_DIR}/books
	${CMAKE_SOURCE_DIR}/data
	${CMAKE_SOURCE_DIR}/fonts
	${CMAKE_SOURCE_DIR}/images
	${CMAKE_SOURCE_DIR}/items
	${CMAKE_SOURCE_DIR}/lang
	${CMAKE_SOURCE_DIR}/maps
	${CMAKE_SOURCE_DIR}/models
	${CMAKE_SOURCE_DIR}/music
	${CMAKE_SOURCE_DIR}/sound
	DESTINATION "${CMAKE_INSTALL_PREFIX}"
  )

  install(FILES
  	${CMAKE_SOURCE_DIR}/steam_appid.txt
  	${CMAKE_SOURCE_DIR}/gamecontrollerdb.txt
  	${CMAKE_SOURCE_DIR}/playernames-female.txt
  	${CMAKE_SOURCE_DIR}/playernames-male.txt
	#TYPE INCLUDE # If your CMAKE is stupid like mine and it complains about "install FILES given no DESTINATION!", you can tell it to shove it by uncommenting this line.
  	DESTINATION "${CMAKE_INSTALL_PREFIX}"
  )
endif()
# 2) Windows install targets
if (WIN32)
	if (DEFINED BARONY_DATADIR)
		# THIS IS AN UTTER BODGE TO STOP VISUAL STUDIO/CMAKE FROM TRYING TO CREATE BARONY/BIN IN C:/PROGRAM FILES
		# ...there's probably a better way to do this
		set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR})
		# TODO: How to force Visual Studio to run the INSTALL target before you hit the "play" button to run the debugger?
		# Maybe...custom commands or targets?

		# 2.a) Copy over lang/en.txt over to the data directory, if it exists.
		install(FILES lang/en.txt DESTINATION ${BARONY_DATADIR}/lang/ CONFIGURATIONS ${CMAKE_CONFIGURATION_TYPES})

		# 2.b) Do the same for the executables, why not?
		install(TARGETS barony DESTINATION ${BARONY_DATADIR}/ CONFIGURATIONS ${CMAKE_CONFIGURATION_TYPES}) # TODO: To escape or to not escape these?
		install(TARGETS ${EDITOR_EXE_NAME} DESTINATION ${BARONY_DATADIR}/ CONFIGURATIONS ${CMAKE_CONFIGURATION_TYPES})
	endif()
endif()

if (APPLE)
#  SET_TARGET_PROPERTIES(
#    barony
#    PROPERTIES
#    RESOURCE ${BOOK_FILES} ${IMAGE_FILES} ${ITEM_FILES} ${MAP_FILES} ${MODEL_FILES} ${MUSIC_FILES} ${SOUND_FILES}
#  )
endif()
