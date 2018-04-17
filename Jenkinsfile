#!/usr/bin/env groovy

// Currently unused, but the goal would be to generate the stages from this.
def clang_builds = ["Linux && clang && LeakSanitizer",
                    "macOS && clang"]
def gcc_builds = ["Linux && gcc4.8",
                  "Linux && gcc4.9",
                  "Linux && gcc5.1",
                  "Linux && gcc6.3",
                  "Linux && gcc7.2"]
def msvc_builds = ["msbuild"]

// The options we use for the builds.
def gcc_cmake_opts = "-DCAF_NO_PROTOBUF_EXAMPLES:BOOL=yes " +
                     "-DCAF_NO_QT_EXAMPLES:BOOL=yes " +
                     "-DCAF_MORE_WARNINGS:BOOL=yes" +
                     "-DCAF_ENABLE_ADDRESS_SANITIZER:BOOL=yes" +
                     "-DCAF_ENABLE_RUNTIME_CHECKS:BOOL=yes" +
                     "-DCAF_USE_ASIO:BOOL=yes" +
                     "-DCAF_NO_BENCHMARKS:BOOL=yes" +
                     "-DOPENSSL_ROOT_DIR=/usr/local/opt/openssl" +
                     "-DOPENSSL_INCLUDE_DIR=/usr/local/opt/openssl"

def clang_cmake_opts = "-DCAF_NO_PROTOBUF_EXAMPLES:BOOL=yes " +
                       "-DCAF_NO_QT_EXAMPLES:BOOL=yes " +
                       "-DCAF_MORE_WARNINGS:BOOL=yes " +
                       "-DCAF_ENABLE_ADDRESS_SANITIZER:BOOL=yes " +
                       "-DCAF_ENABLE_RUNTIME_CHECKS:BOOL=yes " +
                       "-DCAF_USE_ASIO:BOOL=yes " +
                       "-DCAF_NO_BENCHMARKS:BOOL=yes " +
                       "-DCAF_NO_OPENCL:BOOL=yes " +
                       "-DOPENSSL_ROOT_DIR=/usr/local/opt/openssl " +
                       "-DOPENSSL_INCLUDE_DIR=/usr/local/opt/openssl"

def msbuild_opts = "-DCAF_BUILD_STATIC_ONLY:BOOL=yes " +
                   "-DCAF_NO_BENCHMARKS:BOOL=yes " +
                   "-DCAF_NO_EXAMPLES:BOOL=yes " +
                   "-DCAF_NO_MEM_MANAGEMENT:BOOL=yes " +
                   "-DCAF_NO_OPENCL:BOOL=yes " +
                   "-DCAF_LOG_LEVEL:INT=0 " +
                   "-DCMAKE_CXX_FLAGS=\"/MP\""

pipeline {
  agent none

  stages {
    stage ('Preparation') {
      steps {
        node ('master') {
          echo "Starting Jenkins stuff"
          // TODO: pull github branch into mirror
          // TODO: set URL, refs, prNum?
          // TODO: maybe static tests?
        }
      }
    }
    stage ('Build & Test') {
      parallel {
        // gcc builds
        stage ("Linux && gcc4.8") {
          agent { label "Linux && gcc4.8" }
          steps { do_unix_stuff("Linux && gcc4.8", gcc_cmake_opts) }
        }
        /*
        stage ("Linux && gcc4.9") {
          agent { label "Linux && gcc4.9" }
          steps { do_unix_stuff("Linux && gcc4.9", gcc_cmake_opts) }
        }
        stage ("Linux && gcc5.1") {
          agent { label "Linux && gcc5.1" }
          steps { do_unix_stuff("Linux && gcc5.1", gcc_cmake_opts) }
        }
        stage ("Linux && gcc6.3") {
          agent { label "Linux && gcc6.3" }
          steps { do_unix_stuff("Linux && gcc6.3", gcc_cmake_opts) }
        }
        stage ("Linux && gcc7.2") {
          agent { label "Linux && gcc7.2" }
          steps { do_unix_stuff("Linux && gcc7.2", gcc_cmake_opts) }
        }
        // clang builds
        stage ("macOS && clang") {
          agent { label "macOS && clang" }
          steps { do_unix_stuff("macOS && clang", clang_cmake_opts) }
        }
        stage('Linux && clang && LeakSanitizer') {
          agent { label "Linux && clang && LeakSanitizer" }
          steps { do_unix_stuff("Linux && clang && LeakSanitizer", clang_cmake_opts) }
        }
        // windows builds
        stage('msbuild') {
          agent { label "msbuild" }
          steps { do_ms_stuff("msbuild", msbuild_opts) }
        }
        */
      }
    }
    /*
    stage ('Test') {
      steps {
        // execute unit tests?
        echo "Testing all the things"
      }
    }
    */
  }
  post {
    success {
      echo "Yeah!"
      // TODO: Gitter?
    }
    failure {
      echo "God damn it! But there don't seem to be mails at all ..."
      // TODO: Gitter?
      // TODO: Email
      emailext(
        subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']]
            // recipientProviders: [[$class: 'DevelopersRecipientProvider']]
      )
      // This does not seem to work outside a node environment:
      // step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']])])
    }
  }
}

def do_unix_stuff(tags,
                  cmake_opts = "",
                  build_type = "Debug",
                  generator = "Unix Makefiles",
                  build_opts = "",
                  clean_build = true) {
  currentBuild.result = 'FAILURE'
  return
  deleteDir()
  // TODO: pull from mirror, not from GitHub, (RIOT fetch func?)
  // checkout scm
  // Configure and build.
  cmakeBuild buildDir: 'build', buildType: "$build_type", cleanBuild: clean_build, cmakeArgs: "$cmake_opts", generator: "$generator", installation: 'cmake in search path', preloadScript: '../cmake/jenkins.cmake', sourceDir: '.', steps: [[args: 'all']]
  // Some setup also done in previous setups.
  ret = sh(returnStatus: true,
           script: """#!/bin/bash +ex
                      cd build || exit 1
                      if [ `uname` = "Darwin" ] ; then
                        export DYLD_LIBRARY_PATH="$PWD/build/lib"
                      elif [ `uname` = "FreeBSD" ] ; then
                        export LD_LIBRARY_PATH="$PWD/build/lib"
                      else
                        export LD_LIBRARY_PATH="$PWD/build/lib"
                        export ASAN_OPTIONS=detect_leaks=1
                      fi
                      exit 0""")
  if (ret) {
    echo "[!!!] Setting up variables failed!"
    currentBuild.result = 'FAILURE'
    return
  }
  // Test.
  ctest arguments: '--output-on-failure', installation: 'cmake auto install', workingDir: 'build'
}

def do_ms_stuff(tags,
                cmake_opts = "",
                build_type = "Debug",
                generator = "Visual Studio 15 2017",
                build_opts = "",
                clean_build = true) {
  withEnv(['PATH=C:\\Windows\\System32;C:\\Program Files\\CMake\\bin;C:\\Program Files\\Git\\cmd']) {
    deleteDir()
    // TODO: pull from mirror, not from GitHub, (RIOT fetch func?)
    checkout scm
    // Configure and build.
    cmakeBuild buildDir: 'build', buildType: "$build_type", cleanBuild: clean_build, cmakeArgs: "$cmake_opts", generator: "$generator", installation: 'cmake in search path', preloadScript: '../cmake/jenkins.cmake', sourceDir: '.', steps: [[args: 'all']]
    // Test.
    ctest arguments: '--output-on-failure', installation: 'cmake auto install', workingDir: 'build'
  }
}