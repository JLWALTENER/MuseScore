# SPDX-License-Identifier: GPL-3.0-only
# MuseScore-Studio-CLA-applies
#
# MuseScore Studio
# Music Composition & Notation
#
# Copyright (C) 2021 MuseScore Limited
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

set(MODULE_TEST iex_midi_tests)

set(MODULE_TEST_SRC
    ${PROJECT_SOURCE_DIR}/src/engraving/tests/utils/scorerw.cpp
    ${PROJECT_SOURCE_DIR}/src/engraving/tests/utils/scorerw.h
    ${PROJECT_SOURCE_DIR}/src/engraving/tests/utils/scorecomp.cpp
    ${PROJECT_SOURCE_DIR}/src/engraving/tests/utils/scorecomp.h

    ${CMAKE_CURRENT_LIST_DIR}/environment.cpp
    #${CMAKE_CURRENT_LIST_DIR}/testbase.cpp
    #${CMAKE_CURRENT_LIST_DIR}/testbase.h
    ${CMAKE_CURRENT_LIST_DIR}/midiimport_tests.cpp
    #${CMAKE_CURRENT_LIST_DIR}/midiexport_tests.cpp doesn't compile and needs actualization
)

set(MODULE_TEST_LINK
    engraving
    iex_midi
    )

set(MODULE_TEST_DATA_ROOT ${CMAKE_CURRENT_LIST_DIR})

include(SetupGTest)
