enable_testing()
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

include(FindAPR)
find_package(APR REQUIRED)

get_target_property(MAMA_INCLUDES mama INCLUDE_DIRECTORIES)

include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}
	${MAMA_INCLUDES}
)

macro(UnitTestC bin)
	add_executable(${bin}
		${ARGN}
		MainUnitTestC.cpp
	)
	target_link_libraries(${bin} ${GTEST_BOTH_LIBRARIES} mama)
	install(TARGETS ${bin}
			RUNTIME DESTINATION bin
			LIBRARY DESTINATION lib
			ARCHIVE DESTINATION lib)
	if (WIN32 AND MSVC)
		install(FILES $<TARGET_PDB_FILE:${bin}> DESTINATION bin OPTIONAL)
	endif()

    gtest_discover_tests(
        ${bin}
        EXTRA_ARGS -m qpid -p qpidmsg -i Q
        PROPERTIES ENVIRONMENT
            "WOMBAT_PATH=${CMAKE_SOURCE_DIR}/mama/c_cpp/src/examples:${CMAKE_SOURCE_DIR}/mama/c_cpp/src/gunittest/c"
    )
endmacro()

UnitTestC(UnitTestMamaC
	inboxtest.cpp
	iotest.cpp
	mamainternaltest.cpp
	msgutils.cpp
	openclosetest.cpp
	publishertest.cpp
	queuetest.cpp
	dictionarytest.cpp
	subscriptiontest.cpp
	timertest.cpp
	transporttest.cpp
	fieldcache/fieldcachevectortest.cpp
	fieldcache/fieldcachelisttest.cpp
	fieldcache/fieldcachemaptest.cpp
	fieldcache/fieldcachefieldtest.cpp
	fieldcache/fieldcacheiteratortest.cpp
	fieldcache/fieldcacherecordtest.cpp
	fieldcache/fieldcachetest.cpp
)

UnitTestC(UnitTestMamaDateTimeC
	mamadatetime/datetimerangetest.cpp
	mamadatetime/datetimetest.cpp
)

UnitTestC(UnitTestMamaMsgC
	mamamsg/msgatomictests.cpp
	mamamsg/msgcompositetests.cpp
	mamamsg/msgfieldatomictests.cpp
	mamamsg/msgfieldcompositetests.cpp
	mamamsg/msgfieldvectortests.cpp
	mamamsg/msggeneraltests.cpp
	mamamsg/msgiterationtests.cpp
	mamamsg/msgstatustests.cpp
	mamamsg/msgvectortests.cpp
)

UnitTestC(UnitTestMamaPriceC
	mamaprice/pricegeneraltests.cpp
	mamaprice/pricerangetests.cpp
)

UnitTestC(UnitTestMamaMiddlewareC
	middleware/middlewareGeneralTests.cpp
	middleware/middlewareInboxTests.cpp
	middleware/middlewareIoTests.cpp
	middleware/middlewareMsgTests.cpp
	middleware/middlewarePublisherTests.cpp
	middleware/middlewareQueueTests.cpp
	middleware/middlewareSubscriptionTests.cpp
	middleware/middlewareTimerTests.cpp
	middleware/middlewareTransportTests.cpp
)

UnitTestC(UnitTestMamaPayloadC
	payload/fieldatomictests.cpp
	payload/fieldcompositetests.cpp
	payload/fieldvectortests.cpp
	payload/payloadatomictests.cpp
	payload/payloadcompositetests.cpp
	payload/payloadgeneraltests.cpp
	payload/payloadvectortests.cpp
)

if (WIN32)
    set(BATCH_ENV_PATH "${GTEST_ROOT}/bin;${APR_ROOT}/bin;${PROTON_ROOT}/bin;${LIBEVENT_ROOT}/lib;$<TARGET_FILE_DIR:mama>;$<TARGET_FILE_DIR:wombatcommon>;$<TARGET_FILE_DIR:mamabaseimpl>;$<TARGET_FILE_DIR:mamaqpidimpl>;$<TARGET_FILE_DIR:mamaplugindqstrategy>;$<TARGET_FILE_DIR:mamaentnoop>;$<TARGET_FILE_DIR:mamaqpidmsgimpl>")
	if (WITH_MAMDA)
		set(BATCH_ENV_PATH "${BATCH_ENV_PATH};$<TARGET_FILE_DIR:mamda>")
	endif()
	if (WITH_CPP)
		set(BATCH_ENV_PATH "${BATCH_ENV_PATH};$<TARGET_FILE_DIR:mamacpp>")
	endif()
	if (WITH_JAVA)
		set(BATCH_ENV_PATH "${BATCH_ENV_PATH};$<TARGET_FILE_DIR:mamajni>")
	endif()
    file(GENERATE OUTPUT "${CMAKE_BINARY_DIR}/build_dll_path_$<CONFIG>.bat"
	     CONTENT "set PATH=${BATCH_ENV_PATH};%PATH%")
	message(WARNING "\tNB: On windows there is no RPATH, so generated environment for you "
	                "required for unit tests. Execute the following *before* building "
					"with 'cmake --build . --config ${CMAKE_BUILD_TYPE}'':"
					"\n\t\tCALL build_dll_path_${CMAKE_BUILD_TYPE}.bat")
endif()
