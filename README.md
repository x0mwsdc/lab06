# Отчет по лабораторной работе lab06 + homework

## Выполнение работы

### 1. Основная часть

#### Подготовка окружения и репозитория
был создан новый публичный репозиторий с названием lab06. Локальное рабочее пространство было подготовлено путем копирования файлов lab05. Старый локальный кэш сборки был полностью удален, а репозиторий перепривязан к новому удаленному адресу:

```bash
cd ~/x0mwsdc/workspace/projects
cp -r lab05 lab06
cd lab06
rm -rf build
git remote remove origin
git remote add origin [https://github.com/x0mwsdc/lab06.git](https://github.com/x0mwsdc/lab06.git)
```

#### Настройка инсталляции и базового CPack
Для того чтобы утилита CPack могла упаковать проект, в файл `CMakeLists.txt` были добавлены стандартные инструкции `install`, указывающие пути для установки исполняемых файлов, библиотек и заголовочных файлов в целевой системе. Также была подключена базовая конфигурация CPack для генерации стандартного архива (`TGZ`):

```bash
cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.10)
project(print_project)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" OFF)

include_directories(include)

add_library(print sources/print.cpp)

add_executable(example1 examples/example1.cpp)
target_link_libraries(example1 print)

add_executable(example2 examples/example2.cpp)
target_link_libraries(example2 print)

if(BUILD_TESTS)
  enable_testing()
  set(OLD_FLAGS "\${CMAKE_CXX_FLAGS}")
  set(CMAKE_CXX_FLAGS "\${CMAKE_CXX_FLAGS} -w")
  add_subdirectory(third-party/gtest)
  set(CMAKE_CXX_FLAGS "\${OLD_FLAGS}")
  
  add_executable(check tests/test1.cpp)
  target_link_libraries(check print gtest_main)
  add_test(NAME check COMMAND check)
endif()

# Инструкции инсталляции компонентов
install(TARGETS print DESTINATION lib)
install(TARGETS example1 example2 DESTINATION bin)
install(DIRECTORY include/ DESTINATION include)

# Базовая настройка CPack
set(CPACK_PACKAGE_NAME "print_project")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Simple C++ print library and examples")
set(CPACK_PACKAGE_CONTACT "x0mwsdc <danila@example.com>")
set(CPACK_GENERATOR "TGZ")

include(CPack)
EOF
```

#### Настройка базового сценария CI/CD
Был обновлен файл `.github/workflows/cmake.yml`. В GitHub Actions после этапов компиляции и тестирования была добавлена команда вызова `cpack` для автоматической сборки дистрибутива:

```bash
cat > .github/workflows/cmake.yml <<EOF
name: CMake Build, Test and Package

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
      with:
        submodules: true

    - name: Configure CMake
      run: cmake -B build -S . -DBUILD_TESTS=ON

    - name: Build project
      run: cmake --build build

    - name: Run Tests
      run: cd build && ctest --output-on-failure

    - name: Create Packages via CPack
      run: cd build && cpack
EOF
```
в каталоге `build/` успешно сгенерировался базовый архив проекта.
---

### 2. Выполнение homework

После завершения основной части ладораторной, конфигурация была расширена для выполнения требований домашнего задания по генерации различных типов пакетов (бинарных и исходных) и автоматическому сохранению артефактов сборки на GitHub.

#### Модификация CMakeLists.txt для генерации DEB-пакетов
В секцию CPack были добавлены параметры для сборщика пакетов Debian (`DEB`), а переменная `CPACK_GENERATOR` была изменена так, чтобы утилита генерировала одновременно два принципиально разных формата: tar-архив (`TGZ`) и установочный пакет (`DEB`):

```bash
# Дополнение в CMakeLists.txt (Секция конфигурации CPack для Homework)
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "x0mwsdc")
set(CPACK_GENERATOR "TGZ;DEB")
```

#### Обновление сценария GitHub Actions для сохранения артефактов
Для автоматического сохранения сгенерированных пакетов на сервере в файл `.github/workflows/cmake.yml` был добавлен финальный шаг с использованием экшена `actions/upload-artifact@v4`. Этот шаг находит созданные бинарные пакеты в папке сборки и прикрепляет их к результатам работы GitHub Actions в виде скачиваемого архива:

```bash
# Дополнение в конец файла .github/workflows/cmake.yml для Homework
    - name: Upload Build Packages
      uses: actions/upload-artifact@v4
      with:
        name: generated-packages
        path: |
          build/*.tar.gz
          build/*.deb
```

#### Фиксация изменений и публикация результатов
Все изменения, включающие базовую практическую работу и выполненное домашнее задание, были добавлены в индекс Git, закоммичены и отправлены в удаленный репозиторий `lab06`:

```bash
git add .
git commit -m"implemented cpack configuration with tgz and deb generation"
git push -u origin main
```

Локальная проверка показала успешную параллельную сборку файлов `print_project-1.0.0-Linux.tar.gz` и `print_project-1.0.0-Linux.deb`. GitHub Actions проверено, выполнил все этапы сборки, тестов, упаковки и прикрепил сгенерированные пакеты в раздел *Artifacts*
