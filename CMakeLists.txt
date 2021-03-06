# GVM
# $Id$
# Description: Top-level cmake control for the Manager.
#
# Authors:
# Matthew Mundell <matthew.mundell@greenbone.net>
# Timo Pollmeier <timo.pollmeier@greenbone.net>
#
# Copyright:
# Copyright (C) 2009-2017 Greenbone Networks GmbH
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.

message ("-- Configuring Greenbone Vulnerability Manager...")

project (gvm C)

cmake_minimum_required (VERSION 2.8)

if (POLICY CMP0005)
  cmake_policy (SET CMP0005 NEW)
endif (POLICY CMP0005)

include (FindPkgConfig)

if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE Debug)
endif (NOT CMAKE_BUILD_TYPE)

if (NOT BACKEND STREQUAL POSTGRESQL)
  set (BACKEND SQLITE3)
endif (NOT BACKEND STREQUAL POSTGRESQL)

## Retrieve svn revision (at configure time)
#  Not using Subversion_WC_INFO, as it would have to connect to the repo
find_program (SVN_EXECUTABLE svn DOC "subversion command line client")

macro (Subversion_GET_REVISION dir variable)
  execute_process (COMMAND ${SVN_EXECUTABLE} info ${CMAKE_SOURCE_DIR}/${dir}
    OUTPUT_VARIABLE ${variable}
    OUTPUT_STRIP_TRAILING_WHITESPACE)
  string (REGEX REPLACE "^(.*\n)?Revision: ([^\n]+).*"
    "\\2" ${variable} "${${variable}}")
endmacro (Subversion_GET_REVISION)

if (NOT CMAKE_BUILD_TYPE MATCHES "Release")
  if (EXISTS "${CMAKE_SOURCE_DIR}/.svn/" OR EXISTS "${CMAKE_SOURCE_DIR}/../.svn/")
    if (SVN_EXECUTABLE)
      Subversion_GET_REVISION(. ProjectRevision)
      set (SVN_REVISION "~svn${ProjectRevision}")
    else (SVN_EXECUTABLE)
      set (SVN_REVISION "~svn")
    endif (SVN_EXECUTABLE)
  endif (EXISTS "${CMAKE_SOURCE_DIR}/.svn/" OR EXISTS "${CMAKE_SOURCE_DIR}/../.svn/")
endif (NOT CMAKE_BUILD_TYPE MATCHES "Release")

## Project version
# The following three variables should be set through the project command once
# we require CMake >= 3.0
set (PROJECT_VERSION_MAJOR 7)
set (PROJECT_VERSION_MINOR 1)
set (PROJECT_VERSION_PATCH 0)

# Set beta version if this is a beta release series,
# unset if this is a stable release series.
set (PROJECT_BETA_RELEASE 1)

if (SVN_REVISION)
  set (PROJECT_VERSION_SVN "${SVN_REVISION}")
endif (SVN_REVISION)

# If PROJECT_BETA_RELEASE is set, the version string will be set to:
#   "major.minor+beta${PROJECT_BETA_RELEASE}"
# If PROJECT_BETA_RELEASE is NOT set, the version string will be set to:
#   "major.minor.patch"
if (PROJECT_BETA_RELEASE)
  set (PROJECT_VERSION_SUFFIX "+beta${PROJECT_BETA_RELEASE}")
else (PROJECT_BETA_RELEASE)
  set (PROJECT_VERSION_SUFFIX ".${PROJECT_VERSION_PATCH}")
endif (PROJECT_BETA_RELEASE)

set (PROJECT_VERSION_STRING "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}${PROJECT_VERSION_SUFFIX}")

## CPack configuration

set (CPACK_CMAKE_GENERATOR "Unix Makefiles")
set (CPACK_GENERATOR "TGZ")
set (CPACK_INSTALL_CMAKE_PROJECTS ".;gvm;ALL;/")
set (CPACK_MODULE_PATH "")
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING")
set (CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")
set (CPACK_RESOURCE_FILE_WELCOME "${CMAKE_SOURCE_DIR}/README.md")
set (CPACK_SOURCE_GENERATOR "TGZ")
set (CPACK_SOURCE_TOPLEVEL_TAG "")
set (CPACK_SYSTEM_NAME "")
set (CPACK_TOPLEVEL_TAG "")

set (CPACK_PACKAGE_VERSION "${PROJECT_VERSION_STRING}${PROJECT_VERSION_SVN}")

set (CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${CPACK_PACKAGE_VERSION}")
set (CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${CPACK_PACKAGE_VERSION}")
set (CPACK_PACKAGE_VENDOR "Greenbone Networks GmbH")
set (CPACK_SOURCE_IGNORE_FILES
"${CMAKE_BINARY_DIR}"
"/.svn/"
"swp$"
)

include (CPack)

## Variables

set (GVMD_DATABASE_VERSION 190)

set (GVMD_SCAP_DATABASE_VERSION 15)

set (GVMD_CERT_DATABASE_VERSION 6)

set (GMP_VERSION "8.0")

if (SYSCONF_INSTALL_DIR)
  set (SYSCONFDIR "${SYSCONF_INSTALL_DIR}")
endif (SYSCONF_INSTALL_DIR)

if (NOT SYSCONFDIR)
  set (SYSCONFDIR "${CMAKE_INSTALL_PREFIX}/etc")
endif (NOT SYSCONFDIR)

if (NOT EXEC_PREFIX)
  set (EXEC_PREFIX "${CMAKE_INSTALL_PREFIX}")
endif (NOT EXEC_PREFIX)

if (NOT BINDIR)
  set (BINDIR "${EXEC_PREFIX}/bin")
endif (NOT BINDIR)

if (NOT SBINDIR)
  set (SBINDIR "${EXEC_PREFIX}/sbin")
endif (NOT SBINDIR)

if (NOT LIBDIR)
  set (LIBDIR "${EXEC_PREFIX}/lib")
endif (NOT LIBDIR)

if (NOT LOCALSTATEDIR)
  set (LOCALSTATEDIR "${CMAKE_INSTALL_PREFIX}/var")
endif (NOT LOCALSTATEDIR)

if (NOT DATADIR)
  set (DATADIR "${CMAKE_INSTALL_PREFIX}/share")
endif (NOT DATADIR)

if (NOT INCLUDEDIR)
  set (INCLUDEDIR "${CMAKE_INSTALL_PREFIX}/include")
endif (NOT INCLUDEDIR)

# The maximum number of seconds the SQLite backend will sleep
# before retrying an SQL operation that is getting a busy response.
#
# 0 means to not sleep at all, and let the db/libsqlite do its own thing.
#
if (NOT DEFINED GVM_SQLITE_SLEEP_MAX)
  set (GVM_SQLITE_SLEEP_MAX 0)
endif (NOT DEFINED GVM_SQLITE_SLEEP_MAX)

set (GVM_SYSCONF_DIR  "${SYSCONFDIR}/gvm")

set (GVM_DATA_DIR     "${DATADIR}/gvm")
if (NOT GVMD_DATA_DIR)
  set (GVMD_DATA_DIR  "${GVM_DATA_DIR}/gvmd")
endif (NOT GVMD_DATA_DIR)

set (GVM_STATE_DIR    "${LOCALSTATEDIR}/lib/gvm")
if (NOT GVMD_STATE_DIR)
  set (GVMD_STATE_DIR  "${GVM_STATE_DIR}/gvmd")
endif (NOT GVMD_STATE_DIR)

set (GVM_LOG_DIR      "${LOCALSTATEDIR}/log/gvm")
set (GVM_SCAP_RES_DIR "${GVM_DATA_DIR}/scap")
set (GVM_CERT_RES_DIR "${GVM_DATA_DIR}/cert")
set (GVM_CA_DIR       "${GVMD_STATE_DIR}/trusted_certs")
set (GVM_LIB_INSTALL_DIR     "${LIBDIR}")

set (GVM_SCANNER_CERTIFICATE "${GVM_STATE_DIR}/CA/servercert.pem")
set (GVM_SCANNER_KEY         "${GVM_STATE_DIR}/private/CA/serverkey.pem")
set (GVM_CLIENT_CERTIFICATE  "${GVM_STATE_DIR}/CA/clientcert.pem")
set (GVM_CLIENT_KEY          "${GVM_STATE_DIR}/private/CA/clientkey.pem")
set (GVM_CA_CERTIFICATE      "${GVM_STATE_DIR}/CA/cacert.pem")

if (NOT GVM_RUN_DIR)
  set (GVM_RUN_DIR      "${LOCALSTATEDIR}/run")
endif (NOT GVM_RUN_DIR)

# TODO: Once we separate the RFP signatures out of "plugins" and
# into var/lib/gvm/rfpsigs/ (via a sync script) then we do not need
# to know about the NVT_DIR anymore.
if (NOT GVM_NVT_DIR)
	set (GVM_NVT_DIR "${LOCALSTATEDIR}/lib/openvas/plugins/")
endif (NOT GVM_NVT_DIR)

if (NOT GVM_ACCESS_KEY_DIR)
  set (GVM_ACCESS_KEY_DIR "${GVM_SYSCONF_DIR}")
endif (NOT GVM_ACCESS_KEY_DIR)

if (NOT GVM_CERT_DATA_DIR)
  set (GVM_CERT_DATA_DIR "${GVM_STATE_DIR}/cert-data")
endif (NOT GVM_CERT_DATA_DIR)

if (NOT GVM_SCAP_DATA_DIR)
  set (GVM_SCAP_DATA_DIR "${GVM_STATE_DIR}/scap-data")
endif (NOT GVM_SCAP_DATA_DIR)

# System username to use when dropping privileges
if (NOT GVM_DEFAULT_DROP_USER)
  set (GVM_DEFAULT_DROP_USER "")
endif (NOT GVM_DEFAULT_DROP_USER)

message ("-- Install prefix: ${CMAKE_INSTALL_PREFIX}")

## Version

set (GVMD_VERSION "${PROJECT_VERSION_STRING}")

# Configure Doxyfile with version number
configure_file (doc/Doxyfile.in doc/Doxyfile)
configure_file (doc/Doxyfile_full.in doc/Doxyfile_full)
configure_file (doc/example-gvm-manage-certs.conf.in doc/example-gvm-manage-certs.conf @ONLY)
configure_file (VERSION.in VERSION)
configure_file (src/gvmd_log_conf.cmake_in src/gvmd_log.conf)
configure_file (src/schema_formats/XML/GMP.xml.in src/schema_formats/XML/GMP.xml @ONLY)
configure_file (tools/greenbone-scapdata-sync.in tools/greenbone-scapdata-sync @ONLY)
configure_file (tools/greenbone-certdata-sync.in tools/greenbone-certdata-sync @ONLY)
configure_file (tools/gvm-portnames-update.in tools/gvm-portnames-update @ONLY)
configure_file (tools/gvm-migrate-to-postgres.in tools/gvm-migrate-to-postgres @ONLY)
configure_file (tools/gvm-manage-certs.in tools/gvm-manage-certs @ONLY)

## Program

set (HARDENING_FLAGS            "-Wformat -Wformat-security -D_FORTIFY_SOURCE=2 -fstack-protector")
set (LINKER_HARDENING_FLAGS     "-Wl,-z,relro -Wl,-z,now")

set (CMAKE_C_FLAGS              "${CMAKE_C_FLAGS} -Wall -D_BSD_SOURCE -D_ISOC99_SOURCE -D_SVID_SOURCE -D_DEFAULT_SOURCE -D_FILE_OFFSET_BITS=64")

set (CMAKE_C_FLAGS_DEBUG        "${CMAKE_C_FLAGS_DEBUG} -Werror")
set (CMAKE_C_FLAGS_RELEASE      "${CMAKE_C_FLAGS_RELEASE} ${HARDENING_FLAGS}")

if (NOT SKIP_SRC)
  add_subdirectory (src)
endif (NOT SKIP_SRC)

## Documentation

add_subdirectory (doc)

## Install

install (CODE "file (MAKE_DIRECTORY \$ENV{DESTDIR}${GVMD_DATA_DIR}/report_formats)")

install (DIRECTORY DESTINATION ${GVMD_STATE_DIR})

install (FILES ${CMAKE_BINARY_DIR}/src/gvmd_log.conf
         DESTINATION ${GVM_SYSCONF_DIR})

install (FILES ${CMAKE_SOURCE_DIR}/src/pwpolicy.conf
         DESTINATION ${GVM_SYSCONF_DIR})

# Report formats.

install (FILES src/report_formats/Anonymous_XML/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5057e5cc-b825-11e4-9d0e-28d24461215b
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Anonymous_XML/Anonymous_XML.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5057e5cc-b825-11e4-9d0e-28d24461215b
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Anonymous_XML/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5057e5cc-b825-11e4-9d0e-28d24461215b
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/ARF/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/910200ca-dc05-11e1-954f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/ARF/ARF.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/910200ca-dc05-11e1-954f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/ARF/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/910200ca-dc05-11e1-954f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/CPE/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5ceff8ba-1f62-11e1-ab9f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CPE/CPE.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5ceff8ba-1f62-11e1-ab9f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CPE/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/5ceff8ba-1f62-11e1-ab9f-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/CSV_Results/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c1645568-627a-11e3-a660-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CSV_Results/CSV_Results.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c1645568-627a-11e3-a660-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CSV_Results/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c1645568-627a-11e3-a660-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/CSV_Hosts/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9087b18c-626c-11e3-8892-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CSV_Hosts/CSV_Hosts.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9087b18c-626c-11e3-8892-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/CSV_Hosts/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9087b18c-626c-11e3-8892-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/HTML/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/6c248850-1f62-11e1-b082-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/HTML/HTML.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/6c248850-1f62-11e1-b082-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/HTML/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/6c248850-1f62-11e1-b082-406186ea4fc5
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/ITG/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/77bd6c4a-1f62-11e1-abf0-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/ITG/ITG.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/77bd6c4a-1f62-11e1-abf0-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/ITG/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/77bd6c4a-1f62-11e1-abf0-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/LaTeX/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a684c02c-b531-11e1-bdc2-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/LaTeX/latex.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a684c02c-b531-11e1-bdc2-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/LaTeX/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a684c02c-b531-11e1-bdc2-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/NBE/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9ca6fe72-1f62-11e1-9e7c-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/NBE/NBE.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9ca6fe72-1f62-11e1-9e7c-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/NBE/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9ca6fe72-1f62-11e1-9e7c-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/LaTeX/latex.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c402cc3e-b531-11e1-9163-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/PDF/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c402cc3e-b531-11e1-9163-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/PDF/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c402cc3e-b531-11e1-9163-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/Topology/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9e5e5deb-879e-4ecc-8be6-a71cd0875cdd/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Topology/hostvisdot-summary.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9e5e5deb-879e-4ecc-8be6-a71cd0875cdd/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Topology/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/9e5e5deb-879e-4ecc-8be6-a71cd0875cdd/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/TXT/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a3810a62-1f62-11e1-9219-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/TXT/TXT.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a3810a62-1f62-11e1-9219-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/TXT/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a3810a62-1f62-11e1-9219-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/XML/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a994b278-1f62-11e1-96ac-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/XML/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/a994b278-1f62-11e1-96ac-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/Verinice_ISM/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c15ad349-bd8d-457a-880a-c7056532ee15/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Verinice_ISM/Verinice_ISM.xsl
               src/report_formats/Verinice_ISM/classification.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c15ad349-bd8d-457a-880a-c7056532ee15/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Verinice_ISM/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/c15ad349-bd8d-457a-880a-c7056532ee15/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/report_formats/Verinice_ITG/report_format.xml
         DESTINATION ${GVMD_DATA_DIR}/report_formats/50c9950a-f326-11e4-800c-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Verinice_ITG/Verinice_ITG.xsl
               src/report_formats/Verinice_ITG/classification-helpers.xsl
         DESTINATION ${GVMD_DATA_DIR}/report_formats/50c9950a-f326-11e4-800c-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/report_formats/Verinice_ITG/generate
         DESTINATION ${GVMD_DATA_DIR}/report_formats/50c9950a-f326-11e4-800c-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

# Schema formats.

install (FILES src/schema_formats/RNC/rnc.xsl
               src/schema_formats/HTML/HTML.xsl
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/02052818-dab6-11df-9be4-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/schema_formats/HTML/generate
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/02052818-dab6-11df-9be4-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/schema_formats/RNC/rnc.xsl
               src/schema_formats/RNC/RNC.xsl
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/787a4a18-dabc-11df-9486-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/schema_formats/RNC/generate
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/787a4a18-dabc-11df-9486-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_BINARY_DIR}/src/schema_formats/XML/GMP.xml
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/18e826fc-dab6-11df-b913-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/schema_formats/XML/generate
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/18e826fc-dab6-11df-b913-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/schema_formats/XML-brief/GMP.xsl
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/d6cf255e-947c-11e1-829a-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES src/schema_formats/XML-brief/generate
         DESTINATION ${GVMD_DATA_DIR}/global_schema_formats/d6cf255e-947c-11e1-829a-406186ea4fc5/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

# Alert methods.

install (FILES src/alert_methods/SCP/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/2db07698-ec49-11e5-bcff-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/Send/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/4a398d42-87c0-11e5-a1c0-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/SMB/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/c427a688-b653-40ab-a9d0-d6ba842a9d63/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/SNMP/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/9d435134-15d3-11e6-bf5c-28d24461215b/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/Sourcefire/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/cd1f5a34-6bdc-11e0-9827-002264764cea/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/TippingPoint/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/5b39c481-9137-4876-b734-263849dd96ce/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/TippingPoint/report-convert.py
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/5b39c481-9137-4876-b734-263849dd96ce/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES src/alert_methods/verinice/alert
         DESTINATION ${GVMD_DATA_DIR}/global_alert_methods/f9d97653-f89b-41af-9ba1-0f6ee00e9c1a/
         PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

 install (CODE "file (MAKE_DIRECTORY \$ENV{DESTDIR}${GVMD_DATA_DIR}/wizards)")

install (FILES src/wizards/quick_first_scan.xml
               src/wizards/get_tasks_deep.xml
               src/wizards/delete_task_deep.xml
               src/wizards/quick_auth_scan.xml
               src/wizards/quick_task.xml
               src/wizards/reset_task.xml
               src/wizards/modify_task.xml
         DESTINATION ${GVMD_DATA_DIR}/wizards/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES tools/portnames_update.xsl
         DESTINATION ${GVMD_DATA_DIR}/
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES tools/cpe_getbyname.xsl tools/cve_getbyname.xsl
         tools/ovaldef_getbyname.xsl
         DESTINATION ${GVM_SCAP_RES_DIR}
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES tools/cert_bund_getbyname.xsl tools/dfn_cert_getbyname.xsl
         DESTINATION ${GVM_CERT_RES_DIR}
         PERMISSIONS OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

install (FILES ${CMAKE_BINARY_DIR}/tools/gvm-portnames-update
         DESTINATION ${SBINDIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_BINARY_DIR}/tools/greenbone-scapdata-sync
         DESTINATION ${SBINDIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_BINARY_DIR}/tools/greenbone-certdata-sync
         DESTINATION ${SBINDIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_SOURCE_DIR}/tools/gvm-lsc-rpm-creator.sh
         DESTINATION ${GVM_DATA_DIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE
                     WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_BINARY_DIR}/tools/gvm-migrate-to-postgres
         DESTINATION ${SBINDIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

if (BACKEND STREQUAL SQLITE3)
  install (FILES ${CMAKE_SOURCE_DIR}/tools/database-statistics-sqlite
           DESTINATION ${SBINDIR}
           PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                       GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
endif (BACKEND STREQUAL SQLITE3)

install (FILES ${CMAKE_BINARY_DIR}/tools/gvm-manage-certs
         DESTINATION ${BINDIR}
         PERMISSIONS OWNER_EXECUTE OWNER_READ OWNER_WRITE
                     GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

install (FILES ${CMAKE_SOURCE_DIR}/doc/gvm-manage-certs.1
          DESTINATION ${DATADIR}/man/man1 )

install (FILES ${CMAKE_BINARY_DIR}/doc/example-gvm-manage-certs.conf
         DESTINATION ${DATADIR}/doc/gvm/ )

## End
