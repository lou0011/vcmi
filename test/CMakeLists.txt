enable_testing()

set(googleTest_Dir ${CMAKE_CURRENT_SOURCE_DIR}/googletest)
if(EXISTS ${googleTest_Dir})
	set(GTestSrc ${googleTest_Dir}/googletest)
	set(GMockSrc ${googleTest_Dir}/googlemock)
else()
	message(FATAL_ERROR "No googletest src dir found!")
endif()
include_directories(${GTestSrc} ${GTestSrc}/include ${GMockSrc} ${GMockSrc}/include)
include_directories(${CMAKE_HOME_DIRECTORY} ${CMAKE_HOME_DIRECTORY}/include ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_HOME_DIRECTORY}/test)
include_directories(${Boost_INCLUDE_DIRS} ${ZLIB_INCLUDE_DIR})

set(test_SRCS
 		StdInc.cpp
 		main.cpp
 		CMemoryBufferTest.cpp
 		CVcmiTestConfig.cpp
 
 		battle/BattleHexTest.cpp
 		battle/CHealthTest.cpp

 		map/CMapEditManagerTest.cpp
 		map/CMapFormatTest.cpp
 		map/MapComparer.cpp
)

set(test_HEADERS
 		StdInc.h
 
 		CVcmiTestConfig.h
 		map/MapComparer.h
)

assign_source_group(${test_SRCS} ${test_HEADERS})

set(mock_HEADERS
    mock/mock_UnitHealthInfo.h
)

add_subdirectory_with_folder("3rdparty" googletest EXCLUDE_FROM_ALL)

add_executable(vcmitest ${test_SRCS} ${test_HEADERS} ${mock_HEADERS} ${GTestSrc}/src/gtest-all.cc ${GMockSrc}/src/gmock-all.cc)
target_link_libraries(vcmitest vcmi ${RT_LIB} ${DL_LIB})
add_test(vcmitest vcmitest)

vcmi_set_output_dir(vcmitest "")

set_target_properties(vcmitest PROPERTIES ${PCH_PROPERTIES})
cotire(vcmitest)

# Files to copy to the build directory
set(vcmitest_FILES
		testdata/TerrainViewTest.h3m
		testdata/terrainViewMappings.json
		testdata/ObjectPropertyTest/header.json
		testdata/ObjectPropertyTest/objects.json
		testdata/ObjectPropertyTest/surface_terrain.json
		testdata/ObjectPropertyTest/underground_terrain.json
)

foreach(file ${vcmitest_FILES})
	add_custom_command(TARGET vcmitest POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_CURRENT_SOURCE_DIR}/${file}" ${CMAKE_CURRENT_BINARY_DIR}
	)
endforeach()
