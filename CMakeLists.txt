# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Tutorial)
include(CTest)

# The version number.
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

#添加环境检查,添加 CheckFunctionExists 宏
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
 # 检查系统是否支持 log,exp 函数
check_function_exists (log HAVE_LOG)
check_function_exists (exp HAVE_EXP)


# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

  
include_directories("${PROJECT_BINARY_DIR}")

# 是否使用自己的 MathFunctions 库
option (USE_MYMATH  
        "Use tutorial provided math implementation" ON) 

# 是否加入 MathFunctions 库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  # 添加子目录
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

		
# 指定生成目标
add_executable(Tutorial main.cpp)

# 添加链接库
target_link_libraries (Tutorial ${EXTRA_LIBS})

# add the install targets
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
         DESTINATION include)
		 

 
# does the application run
add_test (TutorialRuns Tutorial 25)
 
# does it sqrt of 25
add_test (TutorialComp25 Tutorial 25)
set_tests_properties (TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
 
# does it handle negative numbers
add_test (TutorialNegative Tutorial -25)
set_tests_properties (TutorialNegative PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")
 
# does it handle small numbers
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")
 
# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# 生成程序安装包(在构建工程的目录下执行 cpack --config CPackConfig.cmake))
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE  
     "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include (CPack)
