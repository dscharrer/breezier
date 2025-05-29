cmake_minimum_required(VERSION 3.16)
project(breezier)
set(PROJECT_VERSION "6.1.5")
set(PROJECT_VERSION_MAJOR 6)

set(KF5_MIN_VERSION "5.102.0")
set(KF6_MIN_VERSION "6.2.0")
set(KDE_COMPILERSETTINGS_LEVEL "5.82")
set(QT5_MIN_VERSION "5.15.2")
set(QT_MIN_VERSION "6.6.0")

include(GenerateExportHeader)
include(FeatureSummary)

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH}  ${CMAKE_SOURCE_DIR}/cmake)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# We need this because we can't include KDECMakeSettings here, because that
# would need KDEInstallDirs, which we can only include in the qt{5,6} builds
# to get the correct installation dirs for each Qt version.
option(BUILD_TESTING "Build the testing tree." ON)
if(BUILD_TESTING)
    enable_testing()
endif()

include(CMakePackageConfigHelpers)
include(ECMQtDeclareLoggingCategory)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEGitCommitHooks)


option(BUILD_QT6 "Build with Qt6" ON)

set(QT_NO_CREATE_VERSIONLESS_TARGETS ON)
set(QT_NO_CREATE_VERSIONLESS_FUNCTIONS ON)

function(build_Qt6)
    set(QT_MAJOR_VERSION 6)
    include(KDEInstallDirs6)
    include(KDECMakeSettings)

    find_package(Qt6 ${REQUIRED_QT_VERSION} CONFIG REQUIRED DBus)
    set(HAVE_QTDBUS ${Qt6DBus_FOUND})

    find_package(KF6FrameworkIntegration ${KF6_MIN_VERSION} CONFIG )
    set_package_properties(KF${QT_MAJOR_VERSION}FrameworkIntegration PROPERTIES
        DESCRIPTION "KF6 Framework Integration"
        URL "https://projects.kde.org/projects/frameworks/frameworkintegration"
        TYPE OPTIONAL
        PURPOSE "Required to use KStyle convenience functionalities in style")

    find_package(KF6KCMUtils ${KF6_MIN_VERSION})
    set_package_properties(KF6KCMUtils PROPERTIES
        TYPE REQUIRED
        DESCRIPTION "Helps create configuration modules"
        PURPOSE "KCMUtils used for the configuration modules or the decoration and Qt Style"
    )

    find_package(Qt6 ${QT_MIN_VERSION} REQUIRED CONFIG COMPONENTS Widgets)

    find_package(KF6 ${KF6_MIN_VERSION} REQUIRED COMPONENTS
        CoreAddons
        ColorScheme
        Config
        GuiAddons
        I18n
        IconThemes
        WindowSystem)

    find_package(Qt6 ${QT_MIN_VERSION} OPTIONAL_COMPONENTS Quick)
    if(${Qt6Quick_FOUND})
        find_package(KF6KirigamiPlatform ${KF6_MIN_VERSION} REQUIRED)
    endif()

    set(BREEZE_HAVE_KSTYLE ${KF6FrameworkIntegration_FOUND})
    set(BREEZE_HAVE_QTQUICK ${Qt6Quick_FOUND})

    # https://bugreports.qt.io/browse/QTBUG-114706
    add_library(Qt::Core ALIAS Qt6::Core)
    add_library(Qt::Gui ALIAS Qt6::Gui)
    add_library(Qt::OpenGL ALIAS Qt6::OpenGL)
    add_library(Qt::Network ALIAS Qt6::Network)
    if(Qt6_VERSION_MINOR GREATER 6)
        add_library(Qt::PlatformModuleInternal ALIAS Qt6::PlatformModuleInternal)
    endif()

    add_subdirectory(libbreezecommon libbreezecommon6)

    find_package(KDecoration3 REQUIRED)
    add_subdirectory(kdecoration)

    unset(QUERY_EXECUTABLE CACHE)
endfunction()
if(BUILD_QT6)
    build_Qt6()
endif()

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
