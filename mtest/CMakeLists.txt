# SPDX-License-Identifier: GPL-3.0-only
# MuseScore-CLA-applies
#
# MuseScore
# Music Composition & Notation
#
# Copyright (C) 2021 MuseScore BVBA and others
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 3 as
# published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.

# set(CMAKE_VERBOSE_MAKEFILE ON)
enable_testing()

if (MSVC)
      project(mtest)
      set (PROJECT_BINARY_DIR ${PROJECT_BINARY_DIR}/..)
      set (PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/..)
      include(FindStaticLibrary)
endif (MSVC)

include_directories(
      ${PROJECT_BINARY_DIR}
      ${PROJECT_SOURCE_DIR}
      ${PROJECT_SOURCE_DIR}/thirdparty
      ${PROJECT_SOURCE_DIR}/framework/global
      ${PROJECT_SOURCE_DIR}/mu4
      )

if (USE_SYSTEM_FREETYPE)
      include_directories(${FREETYPE_INCLUDE_DIRS})
else (USE_SYSTEM_FREETYPE)
      include_directories(${PROJECT_SOURCE_DIR}/thirdparty/freetype/include)
endif (USE_SYSTEM_FREETYPE)

if (MSVC)
      set(_all_h_file "${PROJECT_SOURCE_DIR}/all.h")
endif (MSVC)

QT5_ADD_RESOURCES(qrc_files ${PROJECT_SOURCE_DIR}/mtest/mtest.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Leland.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Bravura.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Campania.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Free.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_FreeSerif.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Gootville.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_MScore.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_MuseJazz.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Smufl.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Tabulature.qrc
      ${PROJECT_SOURCE_DIR}/framework/fonts/fonts_Petaluma.qrc
)

add_library (testResources STATIC
      ${qrc_files}
      )

set (IMPORTEXPORT_DIR ${PROJECT_SOURCE_DIR}/mu4/importexport/internal)
include(${IMPORTEXPORT_DIR}/bb/bb.cmake)
include(${IMPORTEXPORT_DIR}/capella/capella.cmake)
include(${IMPORTEXPORT_DIR}/musicxml/musicxml.cmake)
include(${IMPORTEXPORT_DIR}/guitarpro/guitarpro.cmake)
include(${IMPORTEXPORT_DIR}/ove/ove.cmake)
include(${IMPORTEXPORT_DIR}/midiimport/midiimport.cmake)
include(${IMPORTEXPORT_DIR}/braille/braille.cmake)

set (SOURCE_LIB
      testutils.cpp
      mtestglobals.cpp

      ${BB_SRC}
      ${CAPELLA_SRC}
      ${MUSICXML_SRC}
      ${BRAILLE_SRC}
      ${GUITARPRO_SRC}
      ${OVE_SRC}
      ${MIDIIMPORT_SRC}

      ${PROJECT_SOURCE_DIR}/audio/exports/exportmidi.cpp
      ${PROJECT_SOURCE_DIR}/audio/exports/exportmidi.h
      ${PROJECT_SOURCE_DIR}/mscore/preferences.cpp
      ${PROJECT_SOURCE_DIR}/mscore/shortcut.cpp
      ${PROJECT_SOURCE_DIR}/mscore/stringutils.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/rtf2html/fmt_opts.cpp        # Required by capella.cpp and capxml.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/rtf2html/rtf2html.cpp        # Required by capella.cpp and capxml.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/rtf2html/rtf_keyword.cpp     # Required by capella.cpp and capxml.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/rtf2html/rtf_table.cpp       # Required by capella.cpp and capxml.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/beatroot/Agent.cpp           # Required by importmidi.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/beatroot/AgentList.cpp       # Required by importmidi.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/beatroot/BeatTracker.cpp     # Required by importmidi.cpp
      ${PROJECT_SOURCE_DIR}/thirdparty/beatroot/Induction.cpp       # Required by importmidi.cpp
      ${PROJECT_SOURCE_DIR}/mscore/extension.cpp # required by zerberus tests
      omr
      )

if (MSVC)
    add_library(
      testutils STATIC
	  ${_all_h_file}
	  ${PCH}
	  ${SOURCE_LIB}
	  )
else (MSVC)
    add_library(
      testutils STATIC
	  ${SOURCE_LIB}
	  )
endif (MSVC)

# testutils version that is linked to a full mscoreapp target
add_library(testutils_mscoreapp STATIC
      testutils.cpp
      mscoreappresources.cpp
      )

if (MSVC)
      target_sources(testutils_mscoreapp PRIVATE
            ${_all_h_file}
            ${PCH}
            )
endif (MSVC)

target_link_libraries(testutils_mscoreapp mscoreapp)

add_executable(
      mtest
      mtest.cpp
      )

target_link_libraries(
      mtest
      ${QT_LIBRARIES}
      )

if (USE_SYSTEM_FREETYPE)
      target_link_libraries(mtest freetype)
else (USE_SYSTEM_FREETYPE)
      target_link_libraries(mtest mscore_freetype)
endif (USE_SYSTEM_FREETYPE)

set(CMAKE_CXX_FLAGS         "${CMAKE_CXX_FLAGS} ${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}")

string(REPLACE ";" ";-I" INC "${QT_INCLUDES}")

if (APPLE AND (CMAKE_VERSION VERSION_LESS "3.5.0"))
      set(TESTUTILS_COMPILE_FLAGS "-include all.h -D TESTROOT=\\\\\"${PROJECT_SOURCE_DIR}\\\\\" -g -Wall -Wextra")
else (APPLE AND (CMAKE_VERSION VERSION_LESS "3.5.0"))
      if (MSVC)
            set(TESTUTILS_COMPILE_FLAGS "${PCH_INCLUDE}  /D TESTROOT=\\\"${PROJECT_SOURCE_DIR}\\\"")
      else (MSVC)
            set(TESTUTILS_COMPILE_FLAGS "-include all.h -D TESTROOT=\\\"${PROJECT_SOURCE_DIR}\\\" -g -Wall -Wextra")
      endif (MSVC)
endif (APPLE AND (CMAKE_VERSION VERSION_LESS "3.5.0"))

set_target_properties(testutils PROPERTIES COMPILE_FLAGS "${TESTUTILS_COMPILE_FLAGS}")
set_target_properties(testutils_mscoreapp PROPERTIES COMPILE_FLAGS "${TESTUTILS_COMPILE_FLAGS}")

if (NOT MSVC)
add_custom_target(report
      COMMAND ant -f ${PROJECT_SOURCE_DIR}/mtest/build.xml -Droot.dir=${PROJECT_BINARY_DIR}/mtest report
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/mtest"
      )

add_custom_target(reporthtml
      COMMAND ant -f ${PROJECT_SOURCE_DIR}/mtest/build.xml -Droot.dir=${PROJECT_BINARY_DIR}/mtest reporthtml
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/mtest"
      )
endif (NOT MSVC)

subdirs (
        libmscore/all_elements
        libmscore/compat114
        libmscore/compat206
#        libmscore/album            # obsolete
        libmscore/barline
        libmscore/beam
        libmscore/breath
        libmscore/box
        libmscore/chordsymbol
        libmscore/clef
        libmscore/clef_courtesy
        libmscore/concertpitch
        libmscore/copypaste
        libmscore/copypastesymbollist
        libmscore/dynamic
        libmscore/durationtype
        libmscore/earlymusic
        libmscore/element
        libmscore/exchangevoices
        libmscore/hairpin
        libmscore/implode_explode
        libmscore/instrumentchange
        libmscore/join
        libmscore/keysig
        libmscore/layout
        libmscore/links
        libmscore/parts
        libmscore/measure
        libmscore/midi                 # one disabled
#        libmscore/midimapping # TODO: compiles but mostly fails
        libmscore/note
        libmscore/readwriteundoreset
        libmscore/remove
        libmscore/repeat
        libmscore/rhythmicGrouping
        libmscore/selectionfilter
        libmscore/selectionrangedelete
        libmscore/unrollrepeats
        libmscore/spanners
        libmscore/split
        libmscore/splitstaff
        libmscore/timesig
        libmscore/tools                # Some tests disabled
        libmscore/transpose
        libmscore/tuplet
#        libmscore/text        work in progress...
        libmscore/utils
#        mscore/workspaces    Not worked on CI, because "Could not initialize GLX"
#        mscore/palette       Not worked on CI, because "Could not initialize GLX"
        importmidi
        capella
        biab
        musicxml
        braille
        guitarpro
        scripting
        stringutils
#        testoves
        testscript
        )

if (NOT MSVC)
install(FILES
      ../share/styles/chords_std.xml
      ../share/styles/chords_jazz.xml
      ../share/styles/chords.xml
      ../share/styles/stdchords.xml
      ../share/styles/jazzchords.xml
      ../share/styles/cchords_muse.xml
      ../share/styles/cchords_nrb.xml
      ../share/styles/cchords_rb.xml
      ../share/styles/cchords_sym.xml
      DESTINATION ${PROJECT_BINARY_DIR}/mtest/styles
      )
endif (NOT MSVC)

