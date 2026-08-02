cmake_minimum_required(VERSION 3.20)
project(AccountsQt6 VERSION 1.17 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

find_package(Qt6 6.5 REQUIRED COMPONENTS Core Xml)
find_package(PkgConfig REQUIRED)
pkg_check_modules(ACCOUNTS_GLIB REQUIRED IMPORTED_TARGET libaccounts-glib glib-2.0 gobject-2.0)

add_library(accounts-qt6 SHARED
    Accounts/manager.cpp
    Accounts/account.cpp
    Accounts/account-service.cpp
    Accounts/application.cpp
    Accounts/auth-data.cpp
    Accounts/error.cpp
    Accounts/provider.cpp
    Accounts/service.cpp
    Accounts/service-type.cpp
    Accounts/utils.cpp
)
add_library(AccountsQt6::AccountsQt6 ALIAS accounts-qt6)
set_target_properties(accounts-qt6 PROPERTIES
    VERSION 1.5
    SOVERSION 1
    OUTPUT_NAME accounts-qt6
    EXPORT_NAME AccountsQt6
)
target_compile_definitions(accounts-qt6 PRIVATE
    BUILDING_ACCOUNTS_QT
    QT_NO_CAST_TO_ASCII
    QT_NO_CAST_FROM_ASCII
    QT_NO_KEYWORDS
)
target_include_directories(accounts-qt6
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/accounts-qt6>
)
target_link_libraries(accounts-qt6 PUBLIC Qt6::Core Qt6::Xml PkgConfig::ACCOUNTS_GLIB)

set(public_headers
    Accounts/accountscommon.h
    Accounts/manager.h
    Accounts/account.h
    Accounts/account-service.h
    Accounts/application.h
    Accounts/auth-data.h
    Accounts/error.h
    Accounts/provider.h
    Accounts/service.h
    Accounts/service-type.h
)
install(TARGETS accounts-qt6 EXPORT AccountsQt6Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES ${public_headers} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/accounts-qt6/Accounts)
install(FILES
    Accounts/Manager
    Accounts/Account
    Accounts/AccountService
    Accounts/Application
    Accounts/AuthData
    Accounts/Error
    Accounts/Provider
    Accounts/Service
    Accounts/ServiceType
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/accounts-qt6/Accounts)

configure_package_config_file(cmake/AccountsQt6Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/AccountsQt6Config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AccountsQt6)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/AccountsQt6ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion)
install(EXPORT AccountsQt6Targets
    NAMESPACE AccountsQt6::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AccountsQt6)
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/AccountsQt6Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/AccountsQt6ConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AccountsQt6)

set(PC_PREFIX "${CMAKE_INSTALL_PREFIX}")
set(PC_LIBDIR "${CMAKE_INSTALL_FULL_LIBDIR}")
configure_file(cmake/accounts-qt6.pc.in accounts-qt6.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/accounts-qt6.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
