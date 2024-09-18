cmake_minimum_required(VERSION 3.10)

# Версия проекта
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

# Поиск библиотеки Boost
find_package(Boost COMPONENTS unit_test_framework REQUIRED)

# Создание заголовочного файла с версией
configure_file(version.h.in version.h)

# Основные цели
add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

# Настройки стандартов C++
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Включение директории для заголовочных файлов
target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

# Настройки для Boost и тестов
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
)
target_include_directories(test_version PRIVATE ${Boost_INCLUDE_DIRS})

# Линковка
target_link_libraries(helloworld_cli PRIVATE helloworld)
target_link_libraries(test_version PRIVATE ${Boost_LIBRARIES} helloworld)

# Настройки компилятора
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else()
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

# Установка исполняемых файлов
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Настройки CPack для упаковки
set(CPACK_GENERATOR "DEB;TGZ") # TGZ добавлен для поддержки macOS

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT "example@example.com")

include(CPack)

# Включение тестирования
enable_testing()

# Добавление тестов
add_test(test_version test_version)
