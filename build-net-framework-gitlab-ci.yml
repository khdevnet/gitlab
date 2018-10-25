variables:
  BUILD_OUTPUT: "build\\_PublishedWebsites\\CA.Web"

stages:
  - build
  - test
  - version
  - deploy

build dev:
  stage: build
  except:
    - tags
  tags:
    - windows
  script:
    - nuget restore .\Solution.sln
    - msbuild.exe .\Solution.sln /p:Configuration=Debug /p:OutDir=..\build

build prod:
  stage: build
  only:
    - tags
  tags:
    - windows
  script:
    - nuget restore .\Solution.sln
    - msbuild.exe .\Solution.sln /p:Configuration=Release /p:OutDir=..\build

test:
  variables:
    GIT_STRATEGY: none
    NUNIT_RUNNER: "packages\\NUnit.ConsoleRunner.3.9.0\\tools\\nunit3-console.exe"
    TEST_REPORT: "TestReport.txt"
  stage: test
  artifacts:
    name: "TestReport"
    paths:
      - "%TEST_REPORT%"
      - "TestResult.xml"
    expire_in: 1 week
  tags:
    - windows
  script:
    - call %NUNIT_RUNNER% --output %TEST_REPORT% build\CA.Test.dll

version update:
    variables:
        GIT_STRATEGY: none
    stage: version
    script:
        - cd .\tools\
        - git describe --abbrev=0 --tags > version.txt
        - set /p version=<version.txt
        - echo %version%:%CI_COMMIT_SHA:~0,8%
        - call add-version.bat %version%:%CI_COMMIT_SHA:~0,8% .\..\%BUILD_OUTPUT%\Scripts\
    only:
        - master
        - tags
    tags:
        - windows

deploy dev:
  variables:
    GIT_STRATEGY: none
  stage: deploy
  only:
    - master
  except:
    - tags
  tags:
    - windows
  script:
    - rd /s /q %DEPLOY_FOLDER%\dev.web-destination\bin
    - xcopy .\%BUILD_OUTPUT%\. %DEPLOY_FOLDER%\dev.web-destination\ /s /d /y
    - xcopy %CONFIGS_FOLDER%\dev\ConnectionStrings.config %DEPLOY_FOLDER%\dev.web-destination\ConnectionStrings.config* /s /y

deploy prod:
  variables:
    GIT_STRATEGY: none
  stage: deploy
  only:
    - tags
  tags:
    - windows
  script:
    - rd /s /q %DEPLOY_FOLDER%\web-destination\bin
    - xcopy .\%BUILD_OUTPUT%\. %DEPLOY_FOLDER%\web-destination\ /s /d /y
    - xcopy %CONFIGS_FOLDER%\prod\ConnectionStrings.config %DEPLOY_FOLDER%\web-destination\ConnectionStrings.config* /s /y
