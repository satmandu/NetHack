cmake_minimum_required(VERSION 3.15)
project(NetHack VERSION 5.0)

if(CMAKE_BUILD_TYPE)
  message("Build type: ${CMAKE_BUILD_TYPE}.")
endif()

message(STATUS "Building NetHack version: ${NetHack_VERSION}.")

set(HACKDIR
    "$ENV{HOME}/nethackdir.50"
    CACHE STRING "Configuration files for nethack")

message(STATUS "HACKDIR set to: ${HACKDIR}")

include(ExternalProject)

# Playground vars
set(VARDIR ${HACKDIR})
set(INSTDIR ${HACKDIR})

add_compile_definitions(
  GCC_WARN
  NOCLIPPING
  NOMAIL
  NOTPARMDECL
  AVOID_WIN_IOCTL
  SCORE_ON_BOTL
  # ANSI_DEFAULT NO_SIGNAL
  HACKDIR="${HACKDIR}"
  DEFAULT_WINDOW_SYS="tty"
  DLB)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I../include -DNOTPARMDECL")
set(CCFLAGS "-g $ENV{CCFLAGS}" CACHE STRING 'ccflags')
# set(CFLAGS "${CCFLAGS} -I../include -DNOTPARMDECL" CACHE STRING 'cflags')
set(LDFLAGS "" CACHE STRING 'ldflags')
set(NH_SRC ${NetHack_SOURCE_DIR}/src)
set(NH_INC ${NetHack_SOURCE_DIR}/include)
set(NH_DAT ${NetHack_SOURCE_DIR}/dat)
set(NH_UTIL ${NetHack_SOURCE_DIR}/util)
set(NH_DOC ${NetHack_SOURCE_DIR}/doc)

set(NH_SRC_GEN ${NetHack_BINARY_DIR}/src)
set(NH_INC_GEN ${NetHack_BINARY_DIR}/include)
set(NH_DAT_GEN ${NetHack_BINARY_DIR}/dat)
set(NH_UTIL_GEN ${NetHack_BINARY_DIR}/util)

# Lua.
set(LUA_VERSION 5.4.8)

# Prepare Lua source code.
set(LUA_SRC_DIR ${CMAKE_CURRENT_BINARY_DIR}/lua)
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/submodules/lua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# Prepare some variables to build Lua.
# set(LUAMAKEFLAGS "CC='$ENV{CC}' CFLAGS='${CFLAGS}' LDFLAGS='${LDFLAGS}'")
set(LUAMAKEFLAGS 
    CFLAGS='${CFLAGS}' LDFLAGS='${LDFLAGS}')
set(LUA_BUILD_COMMAND
    make ${LUAMAKEFLAGS} a)
set(LUA_LIB_NAME liblua.a)
#if (WIN32)
#set(LUA_BUILD_COMMAND
    #vcvarsall.bat x86_amd64 && cl.exe /MD /O2 /c /DLUA_BUILD_AS_DLL *.c &&
    #del lua.o 2>null && del onelua.o 2>null &&
    #ren lua.obj lua.o && ren onelua.obj onelua.o &&
    #link.exe /DLL /IMPLIB:lua.lib /OUT:lua.dll *.obj)
#set(LUA_LIB_NAME lua.lib)
#elseif (UNIX)
#set(DLLIB "-ldl")
#set(LUAMAKEFLAGS "CC='$ENV{CC}' SYSCFLAGS='${SYSCFLAGS}'")
#set(LUABASE "liblua-${LUA_VERSION}.a")
#set(LUALIB "../lib/lua/${LUABASE}")
#set(LUALIBS "${LUALIB} -lm ${DLLIB}")
#set(LUAHEADERS "lib/lua-${LUA_VERSION}/src")
#set(LUA_BUILD_COMMAND
    #cd ${LUAHEADERS} && make ${LUAMAKEFLAGS} a &&
    #cp liblua.a ${LUA_SRC_DIR}/)
#set(LUA_LIB_NAME liblua.a)
#endif()

set(LUA_INCLUDE ${LUA_SRC_DIR})
set(TOPLUALIB ${LUA_SRC_DIR}/${LUA_LIB_NAME})

# Add target to build Lua.
add_custom_target(build_lua
  COMMAND ${LUA_BUILD_COMMAND}
  WORKING_DIRECTORY ${LUA_SRC_DIR}
  BYPRODUCTS ${TOPLUALIB}
  VERBATIM)

# Add target to generate `nhlua.h`.
add_custom_target(generate_nhlua
  COMMAND ${CMAKE_COMMAND} -E echo "/* nhlua.h - generated file */" >
          ${NH_INC_GEN}/nhlua.h
  COMMAND ${CMAKE_COMMAND} -E echo "#include \"lua.h\"" >> ${NH_INC_GEN}/nhlua.h
  COMMAND ${CMAKE_COMMAND} -E echo "#include \"lualib.h\"" >>
          ${NH_INC_GEN}/nhlua.h
  COMMAND ${CMAKE_COMMAND} -E echo "#include \"lauxlib.h\"" >>
          ${NH_INC_GEN}/nhlua.h
  COMMAND ${CMAKE_COMMAND} -E echo "/*nhlua.h*/" >> ${NH_INC_GEN}/nhlua.h
  VERBATIM)
add_dependencies(generate_nhlua build_lua)

# Add a general target for defining Lua dependency.
add_custom_target(lua_support DEPENDS generate_nhlua)

# Add `util`.
add_subdirectory(util)
add_dependencies(makedefs lua_support)
add_dependencies(dlb lua_support)
add_dependencies(recover lua_support)

# Add `dat`.
add_subdirectory(dat)
add_dependencies(dat lua_support)

set(NETHACK_SRC
    "src/allmain.c"
    "src/alloc.c"
    "src/apply.c"
    "src/artifact.c"
    "src/attrib.c"
    "src/ball.c"
    "src/bones.c"
    "src/botl.c"
    "src/calendar.c"
    "src/cfgfiles.c"
    "src/cmd.c"
    "src/coloratt.c"
    "src/date.c"
    "src/dbridge.c"
    "src/decl.c"
    "src/detect.c"
    "src/dig.c"
    "src/display.c"
    "src/dlb.c"
    "src/do.c"
    "src/do_name.c"
    "src/do_wear.c"
    "src/dog.c"
    "src/dogmove.c"
    "src/dokick.c"
    "src/dothrow.c"
    "src/drawing.c"
    "src/dungeon.c"
    "src/earlyarg.c"
    "src/eat.c"
    "src/end.c"
    "src/engrave.c"
    "src/exper.c"
    "src/explode.c"
    "src/extralev.c"
    "src/files.c"
    "src/fountain.c"
    "src/getpos.c"
    "src/glyphs.c"
    "src/hack.c"
    "src/hacklib.c"
    "src/iactions.c"
    "src/insight.c"
    "src/invent.c"
    "src/isaac64.c"
    "src/light.c"
    "src/lock.c"
    "src/mail.c"
    "src/makemon.c"
    "src/mcastu.c"
    "src/mdlib.c"
    "src/mhitm.c"
    "src/mhitu.c"
    "src/minion.c"
    "src/mklev.c"
    "src/mkmap.c"
    "src/mkmaze.c"
    "src/mkobj.c"
    "src/mkroom.c"
    "src/mon.c"
    "src/mondata.c"
    "src/monmove.c"
    "src/monst.c"
    "src/mplayer.c"
    "src/mthrowu.c"
    "src/muse.c"
    "src/music.c"
    "src/nhlobj.c"
    "src/nhlsel.c"
    "src/nhlua.c"
    "src/nhmd4.c"
    "src/o_init.c"
    "src/objects.c"
    "src/objnam.c"
    "src/options.c"
    "src/pager.c"
    "src/pickup.c"
    "src/pline.c"
    "src/polyself.c"
    "src/potion.c"
    "src/pray.c"
    "src/priest.c"
    "src/quest.c"
    "src/questpgr.c"
    "src/read.c"
    "src/rect.c"
    "src/region.c"
    "src/report.c"
    "src/restore.c"
    "src/rip.c"
    "src/rnd.c"
    "src/role.c"
    "src/rumors.c"
    "src/save.c"
    "src/selvar.c"
    "src/sfbase.c"
    "src/sfstruct.c"
    "src/shk.c"
    "src/shknam.c"
    "src/sit.c"
    "src/sounds.c"
    "src/sp_lev.c"
    "src/spell.c"
    "src/stairs.c"
    "src/steal.c"
    "src/steed.c"
    "src/strutil.c"
    "src/symbols.c"
    "src/sys.c"
    "src/teleport.c"
    "src/timeout.c"
    "src/topten.c"
    "src/track.c"
    "src/trap.c"
    "src/u_init.c"
    "src/uhitm.c"
    "src/utf8map.c"
    "src/vault.c"
    "src/version.c"
    "src/vision.c"
    "src/weapon.c"
    "src/were.c"
    "src/wield.c"
    "src/windows.c"
    "src/wizard.c"
    "src/wizcmds.c"
    "src/worm.c"
    "src/worn.c"
    "src/write.c"
    "src/zap.c"
    "sys/share/pcmain.c"
    "sys/share/pcsys.c"
    "sys/share/pctty.c"
    "sys/share/pcunix.c"
    "sys/share/pmatchregex.c"
    "sys/share/random.c"
    "sys/share/unixtty.c"
    "sys/share/uudecode.c"
    "win/tty/getline.c"
    "win/tty/termcap.c"
    "win/tty/topl.c"
    "win/tty/wintty.c")

# Define sources for curses interface.
set(CURSES_SRC
    "win/curses/cursmain.c"
    "win/curses/curswins.c"
    "win/curses/cursmisc.c"
    "win/curses/cursdial.c"
    "win/curses/cursstat.c"
    "win/curses/cursinit.c"
    "win/curses/cursmesg.c"
    "win/curses/cursinvt.c")

# Define macro to use curses graphics.
add_compile_definitions(CURSES_GRAPHICS)

# Append curses-related sources.
list(APPEND NETHACK_SRC ${CURSES_SRC})

if (UNIX)
    add_compile_definitions(_XOPEN_SOURCE_EXTENDED)
    # Append Unix specific sources.
    set(NETHACK_UNIX_SRC
        "sys/share/posixregex.c"
        "sys/share/ioctl.c"
        "sys/unix/unixunix.c"
        "sys/unix/unixres.c"
        "sys/share/unixtty.c"
        "sys/share/tclib.c"
        "sys/unix/unixmain.c")
    list(APPEND NETHACK_SRC ${NETHACK_UNIX_SRC})
elseif (WIN32)
    add_compile_definitions(WIN32CON SAFEPROCS)
    # Append Windows specific sources.
    set(NETHACK_WIN_SRC
        "sys/share/cppregex.cpp"
        "win/share/safeproc.c"
        "sys/windows/windsys.c"
        "sys/windows/consoletty.c"
        "sys/windows/windmain.c")
    list(APPEND NETHACK_SRC ${NETHACK_WIN_SRC})
endif()

# nethack executable.
add_executable(nethack ${NETHACK_SRC})

# link ncurses
if (WIN32)
    # Add PDCurses library and use it instead of curses.
    set(PDCURSES_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/submodules/pdcurses)
    file(GLOB PDCURSES_SOURCES ${PDCURSES_ROOT_DIR}/pdcurses/*.c)
    file(GLOB PDCURSES_WINDOWS_SOURCES ${PDCURSES_ROOT_DIR}/wincon/*.c)
    add_library(PDcurses STATIC
        ${PDCURSES_SOURCES}
        ${PDCURSES_WINDOWS_SOURCES}
        ${PDCURSES_ROOT_DIR}/wincon/pdcwin.h
    )
    target_include_directories(PDcurses PUBLIC ${PDCURSES_ROOT_DIR})
    target_link_libraries(nethack PDcurses)
    target_link_libraries(nethack Dbghelp)
else()
    target_link_libraries(nethack ncurses)
    target_link_libraries(nethack ncursesw)
endif()

# link other libs
if (WIN32)
target_link_libraries(nethack bcrypt.lib)
else()
target_link_libraries(nethack m)
endif()
target_link_libraries(nethack ${TOPLUALIB} ${CMAKE_DL_LIBS})


# TODO: Add check-dlb, Guidebook
add_dependencies(nethack util dat lua_support)

# add include directories
target_include_directories(
  nethack PUBLIC
  ${NH_INC}
  ${LUA_INCLUDE}
  ${NH_INC_GEN}
  ${CMAKE_CURRENT_SOURCE_DIR}/win/curses
  /usr/local/include)

# post build tasks
if (WIN32)
add_custom_command(TARGET nethack POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy   ${LUA_SRC_DIR}/lua.dll ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy   ${NH_SYS}/windows/sysconf.template ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy   ${NH_DAT}/symbols ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E rename ${CMAKE_CURRENT_BINARY_DIR}/symbols ${CMAKE_CURRENT_BINARY_DIR}/symbols.template
    COMMAND ${CMAKE_COMMAND} -E copy   ${NH_SYS}/windows/nethackrc.template ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy   ${NH_DOC}/Guidebook.txt ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy   ${NH_DAT}/opthelp ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy   ${CMAKE_CURRENT_BINARY_DIR}/dat/nhdat370 ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND ${CMAKE_COMMAND} -E rename ${CMAKE_CURRENT_BINARY_DIR}/dat/nhdat370 ${CMAKE_CURRENT_BINARY_DIR}/dat/nhdat
)
endif()

install(TARGETS nethack DESTINATION ${INSTDIR})
