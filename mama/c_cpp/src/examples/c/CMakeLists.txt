project (mamac_examples)

get_target_property(MAMA_INCLUDES mama INCLUDE_DIRECTORIES)

include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}
	${MAMA_INCLUDES}
)

include(mama-common)

mamac_binary(mamaftmemberc)
mamac_binary(mamainboxc)
mamac_binary(mamaio)
mamac_binary(mamalistencachedc)
mamac_binary(mamalistenc)
mamac_binary(mamamultisubscriberc)
mamac_binary(mamaproxyc)
mamac_binary(mamapublisherc)
mamac_binary(mamasubscriberc)
mamac_binary(mamasymbollistsubscriberc)

install(DIRECTORY . DESTINATION examples/mama/c
        PATTERN CMakeLists.txt EXCLUDE
		PATTERN *.vcxproj EXCLUDE
		PATTERN SConscript* EXCLUDE)

if (WIN32 AND MSVC)
    install(DIRECTORY . DESTINATION examples/mama/c
		    PATTERN *.vcxproj)
endif()