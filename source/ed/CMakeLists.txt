SET(SOURCE_LIB
        data.cpp
        types.cpp
        build_helper.cpp
)


add_library(ed ${SOURCE_LIB})

SET(BOOST_LIBS ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_FILESYSTEM_LIBRARY}
    ${Boost_SYSTEM_LIBRARY} ${Boost_SERIALIZATION_LIBRARY}
    ${Boost_DATE_TIME_LIBRARY} ${Boost_THREAD_LIBRARY} ${Boost_REGEX_LIBRARY})


FIND_LIBRARY(OSMPBF osmpbf)

add_executable(osm2ed osm2ed.cpp)
target_link_libraries(osm2ed transportation_data_import ed connectors types pq pqxx ${OSMPBF} pb_lib utils
    ${BOOST_LIBS} log4cplus z protobuf)

add_subdirectory(tests)

add_library(transportation_data_import ed_persistor.cpp)
target_link_libraries(transportation_data_import ed fare types pq pqxx data utils ${BOOST_LIBS} log4cplus)

add_executable(gtfs2ed gtfs2ed.cpp)
target_link_libraries(gtfs2ed transportation_data_import connectors)

add_executable(fusio2ed fusio2ed.cpp)
target_link_libraries(fusio2ed transportation_data_import connectors)

add_executable(fare2ed fare2ed.cpp)
target_link_libraries(fare2ed transportation_data_import connectors)

add_executable(ed2nav ed2nav.cpp ed_reader.cpp)
target_link_libraries(ed2nav types connectors pq pqxx data georef routing fare pb_lib
    utils autocomplete ${BOOST_LIBS} log4cplus protobuf)

add_subdirectory(connectors)

add_executable(geopal2ed geopal2ed.cpp)

target_link_libraries(geopal2ed transportation_data_import connectors)
add_executable(poi2ed poi2ed.cpp)

target_link_libraries(poi2ed transportation_data_import connectors)

add_executable(synonym2ed synonym2ed.cpp)
target_link_libraries(synonym2ed transportation_data_import connectors)

SET(ED_TARGETS_TO_INSTALL gtfs2ed osm2ed ed2nav fusio2ed fare2ed geopal2ed poi2ed synonym2ed)
INSTALL_TARGETS(/usr/bin/ ${ED_TARGETS_TO_INSTALL})

# Ed integration tests with docker, build with the docker_test target 
# and not with the default target not to have a strong docker dependency
add_subdirectory(docker_tests EXCLUDE_FROM_ALL)
