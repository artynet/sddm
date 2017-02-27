# Copy maya to have it available as an external (non-embedded) theme
file(COPY "../../src/greeter/theme/" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/maya")

# Copy Main.qml and image resources from upstream maya to debian-maya
file(COPY "../../src/greeter/theme/Main.qml" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-maya")
file(GLOB maya_PNGS "../../src/greeter/theme/*.png")
foreach(maya_PNG ${maya_PNGS})
    file(COPY "${maya_PNG}" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-maya")
endforeach(maya_PNG)
# Replace default background source by config key config.background
set(SED_EXPR "s=^\\([[:space:]]*source[[:space:]]*:[[:space:]]\\)\\\"qrc:/theme/background.png\\\"[[:space:]]*$=\\1config.background=")
execute_process(COMMAND sed -i -e "${SED_EXPR}" "${CMAKE_CURRENT_SOURCE_DIR}/debian-maya/Main.qml" RESULT_VARIABLE RES OUTPUT_VARIABLE OUT)
message("Result of sed replacement for maya's background source: (Res: ${RES}, Out: ${OUT})")

# Copy selected files from upstream Elarun to debian-elarun
file(COPY "elarun/Main.qml" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(COPY "elarun/angle-down.png" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(COPY "elarun/elarun.jpg" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(GLOB ELARUN_IMGS "elarun/images/*")
foreach(ELARUN_IMG ${ELARUN_IMGS})
    file(COPY "${ELARUN_IMG}" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun/images/")
endforeach(ELARUN_IMG)

set(THEMES elarun maldives maya debian-maya debian-elarun)

foreach(THEME ${THEMES})
    # We assume that theme-specific translations are stored
    # in ${THEME}/translations directory
    set(TRANSLATIONS_DIR "${THEME}/translations")
    file(GLOB TRANSLATION_SOURCES "${TRANSLATIONS_DIR}/*.ts")

    configure_file("${THEME}/Main.qml"                  "${THEME}/Main.qml")
    configure_file("${THEME}/metadata.desktop"          "${THEME}/metadata.desktop")

    set_source_files_properties(${TRANSLATION_SOURCES} PROPERTIES OUTPUT_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/${TRANSLATIONS_DIR}")

    qt5_add_translation(QM_FILES "${TRANSLATION_SOURCES}")

    install(DIRECTORY "${THEME}" DESTINATION "${DATA_INSTALL_DIR}/themes" PATTERN "${THEME}/*.ts"
            EXCLUDE PATTERN "${THEME}/Main.qml"
            EXCLUDE PATTERN "${THEME}/metadata.desktop"
            EXCLUDE PATTERN "${THEME}/.gitattributes"
            EXCLUDE)
    install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/${THEME}" DESTINATION "${DATA_INSTALL_DIR}/themes")

    list(APPEND THEMES_QM_FILES ${QM_FILES})
endforeach(THEME)

add_custom_target(themes-translation DEPENDS ${THEMES_QM_FILES})
