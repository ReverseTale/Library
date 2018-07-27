#######################
# FMT 

    set(FMT_CONFIGURE
        "-DFMT_DOC=OFF"
        "-DFMT_TEST=OFF"
    )

    RequireExternal(
        TARGET ReverseTale-Library
        MODULE "fmtlib/fmt:b64d13a357d3556e54ed92a959baf3a14e2f6078"
        CONFIGURE_ARGUMENTS ${FMT_CONFIGURE}
        INSTALL_NAME fmt
    )

#######################
# BOOST

if (USE_SYSTEM_BOOST)
    find_package(Boost 1.62 REQUIRED)
    set(BOOST_DEPENDENCIES "")

    AddDependency(
        TARGET ReverseTale-Library
        DEPENDENCY Boost
    )
else()
    set(BOOST_DEPENDENCIES
        # Lockfree dependencies
        lockfree core config assert static_assert type_traits mpl preprocessor
        array throw_exception functional align parameter atomic detail
        utility predef winapi
    )

    foreach (dep ${BOOST_DEPENDENCIES})
        RequireExternal(
            TARGET ReverseTale-Library
            EXCLUDE
            SKIP_CONFIGURE
            SKIP_BUILD
            SKIP_LINK
            MODULE "boostorg/${dep}:develop"
            INSTALL_INCLUDE
        )
    endforeach()
endif()
