#Disable some warnings which are all over Qt headers and make finding error messages hell
#if(${CMAKE_CXX_FLAGS} MATCHES "-Wconversion") 
#  string(REGEX REPLACE "-Wconversion(.+)" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
#endif()

set(SUBSYS_NAME cloud_composer)
set(SUBSYS_DESC "Cloud Composer - Application for Manipulating Point Clouds")
set(SUBSYS_DEPS common io visualization filters)

# Find VTK 
if(NOT VTK_FOUND)
  set(DEFAULT FALSE)
  set(REASON "VTK was not found.")
else(NOT VTK_FOUND)
  set(DEFAULT TRUE)
  set(REASON)
  include (${VTK_USE_FILE})
endif(NOT VTK_FOUND)

# QT4 Found?
if(NOT QT4_FOUND)
  set(DEFAULT FALSE)
  set(REASON "Qt4 was not found.")
else(NOT QT4_FOUND)
  set(DEFAULT TRUE)
  set(REASON)
endif(NOT QT4_FOUND)

# QVTK?
if(NOT VTK_USE_QVTK)
  set(DEFAULT FALSE)
  set(REASON "Cloud composer requires QVTK")
else(NOT VTK_USE_QVTK)
  set(DEFAULT TRUE)
  set(REASON)
endif(NOT VTK_USE_QVTK)

#Default to not building for now
set(DEFAULT FALSE)

PCL_SUBSYS_OPTION(build ${SUBSYS_NAME} ${SUBSYS_DESC} ${DEFAULT} ${REASON})
PCL_SUBSYS_DEPEND(build ${SUBSYS_NAME} ${SUBSYS_DEPS})

PCL_ADD_DOC(${SUBSYS_NAME})

if(build)

  include_directories (${CMAKE_CURRENT_BINARY_DIR})
  include_directories (${CMAKE_CURRENT_SOURCE_DIR}/include)
  
  if (VTK_FOUND AND QT4_FOUND AND VTK_USE_QVTK)
    #Sources & Headers for main application
    set(incs include/pcl/apps/${SUBSYS_NAME}/qt.h
             include/pcl/apps/${SUBSYS_NAME}/cloud_composer.h
             include/pcl/apps/${SUBSYS_NAME}/project_model.h
             include/pcl/apps/${SUBSYS_NAME}/cloud_viewer.h
             include/pcl/apps/${SUBSYS_NAME}/cloud_view.h
             include/pcl/apps/${SUBSYS_NAME}/cloud_browser.h
             include/pcl/apps/${SUBSYS_NAME}/item_inspector.h
             include/pcl/apps/${SUBSYS_NAME}/tool_interface/abstract_tool.h
             include/pcl/apps/${SUBSYS_NAME}/tool_interface/tool_factory.h
             include/pcl/apps/${SUBSYS_NAME}/commands.h
             include/pcl/apps/${SUBSYS_NAME}/work_queue.h
             include/pcl/apps/${SUBSYS_NAME}/toolbox_model.h
             include/pcl/apps/${SUBSYS_NAME}/properties_model.h
             include/pcl/apps/${SUBSYS_NAME}/signal_multiplexer.h
             include/pcl/apps/${SUBSYS_NAME}/merge_selection.h
             include/pcl/apps/${SUBSYS_NAME}/transform_clouds.h)
          
    set(srcs src/main.cpp 
             src/cloud_composer.cpp
             src/project_model.cpp
             src/cloud_viewer.cpp
             src/cloud_view.cpp
             src/item_inspector.cpp
             src/cloud_browser.cpp
             src/commands.cpp
             src/work_queue.cpp
             src/toolbox_model.cpp
             src/properties_model.cpp
             src/signal_multiplexer.cpp
             src/merge_selection.cpp
             src/tool_interface/abstract_tool.cpp
             src/transform_clouds.cpp)
          
    set(impl_incs include/pcl/apps/${SUBSYS_NAME}/impl/cloud_item.hpp
                  include/pcl/apps/${SUBSYS_NAME}/impl/merge_selection.hpp
                  include/pcl/apps/${SUBSYS_NAME}/impl/transform_clouds.hpp)
              
    #Sources and headers for item types      
    set(item_incs include/pcl/apps/${SUBSYS_NAME}/items/cloud_composer_item.h
                  include/pcl/apps/${SUBSYS_NAME}/items/cloud_item.h
                  include/pcl/apps/${SUBSYS_NAME}/items/normals_item.h
                  include/pcl/apps/${SUBSYS_NAME}/items/fpfh_item.h)
           
    set(item_srcs src/items/cloud_composer_item.cpp
                  src/items/cloud_item.cpp
                  src/items/normals_item.cpp
                  src/items/fpfh_item.cpp)
                  
    #Sources and headers for point selectors
    set (selector_incs include/pcl/apps/${SUBSYS_NAME}/point_selectors/interactor_style_switch.h
                       include/pcl/apps/${SUBSYS_NAME}/point_selectors/rectangular_frustum_selector.h
                       include/pcl/apps/${SUBSYS_NAME}/point_selectors/selected_trackball_interactor_style.h
                       include/pcl/apps/${SUBSYS_NAME}/point_selectors/click_trackball_interactor_style.h)
                       
    set (selector_srcs src/point_selectors/interactor_style_switch.cpp
                       src/point_selectors/selection_event.cpp
                       src/point_selectors/rectangular_frustum_selector.cpp
                       src/point_selectors/selected_trackball_interactor_style.cpp
                       src/point_selectors/click_trackball_interactor_style.cpp
                       src/point_selectors/manipulation_event.cpp)
                       
    set(uis src/cloud_composer_main_window.ui)
    set(resources resources/resources.qrc)

    QT4_WRAP_UI(cloud_composer_ui ${uis})
    QT4_WRAP_CPP(cloud_composer_moc ${incs} OPTIONS -DBOOST_TT_HAS_OPERATOR_HPP_INCLUDED)
    QT4_ADD_RESOURCES(resource_srcs ${resources})
    
    set(EXE_NAME pcl_${SUBSYS_NAME})
    PCL_ADD_EXECUTABLE(${EXE_NAME} ${SUBSYS_NAME} ${cloud_composer_ui} ${cloud_composer_moc} ${srcs} ${resource_srcs} ${item_srcs} ${selector_srcs} ${impl_incs})
    target_link_libraries(${EXE_NAME} pcl_common pcl_io pcl_visualization pcl_filters QVTK ${QT_LIBRARIES})

    # Install include files
    PCL_ADD_INCLUDES(${SUBSYS_NAME} ${SUBSYS_NAME} ${incs} ${item_incs} ${selector_incs})
    PCL_ADD_INCLUDES(${SUBSYS_NAME} ${SUBSYS_NAME}/impl ${impl_incs})
    
    PCL_MAKE_PKGCONFIG(${EXE_NAME} ${SUBSYS_NAME} "${SUBSYS_DESC}" "" "" "" "" "")
    

    #TOOL BUILDING SCRIPTS
    #Create subdirectory for plugin libs
    set (CLOUD_COMPOSER_PLUGIN_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/cloud_composer_plugins)
    make_directory (${CLOUD_COMPOSER_PLUGIN_DIR})

    set(INTERFACE_HEADERS include/pcl/apps/${SUBSYS_NAME}/tool_interface/abstract_tool.h)
    set(INTERFACE_SOURCES src/tool_interface/abstract_tool.cpp)
    
    QT4_WRAP_CPP(INTERFACE_HEADERS_MOC ${INTERFACE_HEADERS} OPTIONS -DBOOST_TT_HAS_OPERATOR_HPP_INCLUDED)
    PCL_ADD_LIBRARY(pcl_cc_tool_interface ${SUBSYS_NAME} ${INTERFACE_SOURCES} ${INTERFACE_HEADERS_MOC})
    target_link_libraries(pcl_cc_tool_interface pcl_common ${QT_LIBRARIES})
    
    include(ComposerTool.cmake REQUIRED)
    
    #FPFH Tool
    set (FPFH_DEPS pcl_features pcl_kdtree pcl_filters)
    set (FPFH_SOURCES tools/fpfh_estimation.cpp)
    set (FPFH_HEADERS include/pcl/apps/${SUBSYS_NAME}/tools/fpfh_estimation.h)
    define_composer_tool (fpfh_estimation "${FPFH_SOURCES}" "${FPFH_HEADERS}" "${FPFH_DEPS}")
    
    #Normals Tool
    set (NORMALS_DEPS pcl_features pcl_kdtree)
    set (NORMALS_SOURCES tools/normal_estimation.cpp)
    set (NORMALS_HEADERS include/pcl/apps/${SUBSYS_NAME}/tools/normal_estimation.h)
    define_composer_tool (normal_estimation "${NORMALS_SOURCES}" "${NORMALS_HEADERS}" "${NORMALS_DEPS}")

    #Euclidean Clustering Tool
    set (EC_DEPS pcl_segmentation pcl_kdtree)
    set (EC_SOURCES tools/euclidean_clustering.cpp)
    set (EC_HEADERS include/pcl/apps/${SUBSYS_NAME}/tools/euclidean_clustering.h)
    define_composer_tool (euclidean_clustering "${EC_SOURCES}" "${EC_HEADERS}" "${EC_DEPS}")
    
    #Statistical Outlier Removal Tool
    set (SOR_DEPS pcl_filters)
    set (SOR_SOURCES tools/statistical_outlier_removal.cpp)
    set (SOR_HEADERS include/pcl/apps/${SUBSYS_NAME}/tools/statistical_outlier_removal.h)
    define_composer_tool (statistical_outlier_removal "${SOR_SOURCES}" "${SOR_HEADERS}" "${SOR_DEPS}")
    
    #Vox Grid Downsample Tool
    set (VOXDS_DEPS pcl_filters)
    set (VOXDS_SOURCES tools/voxel_grid_downsample.cpp)
    set (VOXDS_HEADERS include/pcl/apps/${SUBSYS_NAME}/tools/voxel_grid_downsample.h)
    define_composer_tool (voxel_grid_downsample "${VOXDS_SOURCES}" "${VOXDS_HEADERS}" "${VOXDS_DEPS}")
    
    
  endif ()  #VTK_FOUND AND QT4_FOUND AND VTK_USE_QVTK


endif(build)
