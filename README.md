[![Build Status](https://travis-ci.com/kbrotskiy/hw06.svg?branch=master)](https://travis-ci.com/github/kbrotskiy/hw06)

## Homework VI

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для изменений, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)


Настройка **CPack** через определение CPack-переменных внутри файла `CMakeLists.txt`
Добавление истории версий для проекта и установка версии 1.0.0
```sh
% sed -i "" '/project(solver)/a\
set(SOLVER_VERSION_STRING "v\${SOLVER_VERSION}")
' CMakeLists.txt
% sed -i "" '/project(solver)/a\
set(SOLVER_VERSION\
  \${SOLVER_VERSION_MAJOR}.\${SOLVER_VERSION_MINOR}.\${SOLVER_VERSION_PATCH}.\${SOLVER_VERSION_TWEAK})
' CMakeLists.txt
% sed -i "" '/project(solver)/a\
set(SOLVER_VERSION_PATCH 0)
' CMakeLists.txt
% sed -i "" '/project(solver)/a\
set(SOLVER_VERSION_MINOR 0)
' CMakeLists.txt
% sed -i "" '/project(solver)/a\
set(SOLVER_VERSION_MAJOR 1)
' CMakeLists.txt
```
Создание файлов `DESCRIPTION`(описание пакета) и `ChangeLog.md` (описание изменений в пакете)
```sh
% touch DESCRIPTION && atom DESCRIPTION
% touch ChangeLog.md
% export DATE="`LANG=en_US date +'%a %b %d %Y'`"
% cat > ChangeLog.md <<EOF
* ${DATE} kbrotskiy <brotskia_26@mail.ru> 1.0.0
- Initial RPM release
EOF
```
Создание и заполнение файла `CPackConfig.cmake`
```sh
# Подключение необходимых системных библиотек
% cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```
Установка значений переменных в пакете
```sh
% cat >> CPackConfig.cmake <<EOF

# Установка контакта
set(CPACK_PACKAGE_CONTACT brotskia_26@mail.ru)
# Установка версии пакета
set(CPACK_PACKAGE_VERSION_MAJOR \${SOLVER_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${SOLVER_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${SOLVER_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION \${SOLVER_VERSION})
#  Установка с файлом описания проекта
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
# Краткое описание проекта
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Application Solver for square equation")
EOF
```
Добавление файлов в пакет
```sh
% cat >> CPackConfig.cmake <<EOF

# Установка лицензии
set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
# Добавление README.md
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```
Настройки для RPM-пакета
```sh
$ cat >> CPackConfig.cmake <<EOF

# Имя пакета RPM
set(CPACK_RPM_PACKAGE_NAME "solver-devel")
# Лицензионная политика пакета RPM.
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
# Группа пакетов RPM
set(CPACK_RPM_PACKAGE_GROUP "solver")
# Добавление файла с описанием изменений пакета
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
# Выпуск пакета RPM
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```
Настройки для Debian-пакета CPACK_OSX_PACKAGE_VERSION
```sh
% cat >> CPackConfig.cmake <<EOF

# Имя пакета DEBIAN
set(CPACK_DEBIAN_PACKAGE_NAME "AppSolver")
# Условия для работы пакета
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
# Выпуск пакета DEBIAN
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```
Настройки для DragNDrop-пакета
```sh
% cat >> CPackConfig.cmake <<EOF

# Имя созданного пакета
set(CPACK_DMG_VOLUME_NAME "solverOS")
# Минимальная версия OSX
set(CPACK_OSX_PACKAGE_VERSION 10.5)
set(CPACK_WIX_LICENSE_RTF ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt)
EOF
```
Настройки для NSIS-пакета CPACK_NSIS_CONTACT
```sh
% cat >> CPackConfig.cmake <<EOF

# Помощь в установке пакета
set(CPACK_NSIS_HELP_LINK https://github.com/kbrotskiy/hw06)
# Помощь в использовании
set(CPACK_NSIS_URL_INFO_ABOUT https://github.com/kbrotskiy/hw06)
# Контактная информация
set(CPACK_NSIS_CONTACT brotskia_26@mail.ru)
EOF
```
Подключение модуля CPack
```sh
% cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
```
Добавление `CPackConfig.cmake` в основной `CMakeLists.txt`
```sh
% cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```
Перепишем `.travis.yml`
```sh
% cat >> .travis.yml <<EOF
language: cpp
jobs:
  include:
  - os: osx
    script:
    - cmake -H. -B_build
    - cmake --build _build
    - cd _build
    - cpack -G DragNDrop
    - cd ..
  - os: linux
    script:
    - cmake -H. -B_build
    - cmake --build _build
    - cd _build
    - cpack -G DEB
    - cpack -G RPM
    - cd ..
addons:
  apt:
    packages:
      - cmake
      - cmake-data
      - rpm
EOF
```
Авторизация в `travis-ci` и добавление зашифрованного API-ключа
```sh
% travis login --auto --ppro
Successfully logged in as kbrotskiy!
% travis enable --pro
kbrotskiy/lab06: enabled :)
% travis setup releases --pro
Username: kbrotskiy
Password for kbrotskiy:
Deploy only from kbrotskiy/hw06? |yes| no
Encrypt API key? |yes| yes
% cat >> CMakeLists.txt <<EOF
file:
  - ./_build/solver-1.0.0.-Darwin.dmg
  - ./_build/solver-1.0.0.-Linux.deb
  - ./_build/solver-1.0.0.-Linux.rpm
skip_cleanup: true
on:
  tags: true
EOF
```
```sh
% git add .
% git commit -m "First release"
% git tag v1.0.0
% git push origin master --tags
```
Создание `appveyor.yml`
```sh
% cat >> appveyor.yml <<EOF
image: Visual Studio 2019
platform:
  - x86
  - x64
configuration: Release

build_script:
  - cmd: cmake -H. -B_build
  - cmd: cmake --build _build --config Release
  - cmd: cd _build
  - cmd: ls
  - cmd: cpack -G NSIS
  - cmd: cd ..

artifacts:
  - path: ./_build/*.msi
    name: solver
deploy:
  release: $(APPVEYOR_REPO_TAG_NAME)
  description: 'Release description'
  provider: GitHub
  auth_token:
    secure: tZdq4qXRLud/z27+KjHqLP0jpdTdUkrQ1XmSJMbSAnd2KTvjSqLjRlueHzHPhzLe
  artifact: print
  on:
    APPVEYOR_REPO_TAG: true

EOF
```
В качестве подсказки:
```sh
$ cat .travis.yml
os: osx
script:
...
- cpack -G DragNDrop # dmg

$ cat .travis.yml
os: linux
script:
...
- cpack -G DEB # deb

$ cat .travis.yml
os: linux
addons:
  apt:
    packages:
    - rpm
script:
...
- cpack -G RPM # rpm

$ cat appveyor.yml
platform:
- x86
- x64
build_script:
...
- cpack -G NSIS # msi
```

Для этого нужно добавить ветвление в конфигурационные файлы для **CI** со следующей логикой:</br>
если **commit** помечен тэгом, то необходимо собрать пакеты (`DEB, RPM, WIX, DragNDrop, ...`) </br>
и разместить их на сервисе **GitHub**. (см. пример для [Travi CI](https://docs.travis-ci.com/user/deployment/releases))</br>

## Links
- [DMG](https://cmake.org/cmake/help/latest/module/CPackDMG.html)
- [DEB](https://cmake.org/cmake/help/latest/module/CPackDeb.html)
- [RPM](https://cmake.org/cmake/help/latest/module/CPackRPM.html)
- [NSIS](https://cmake.org/cmake/help/latest/module/CPackNSIS.html)
- [Travis Client](https://github.com/travis-ci/travis.rb)
- [AppVeyour](https://www.appveyor.com/)
- [GitLab CI](https://about.gitlab.com/gitlab-ci/)

```
Copyright (c) 2015-2020 The ISC Authors
```
