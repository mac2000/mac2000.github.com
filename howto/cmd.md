---
layout: page
title: Windows console
---

Add path
--------

User:

    SETX PATH "%PATH%;C:\MyDir"

System:

    SETX PATH "%PATH%;C:\MyDir" /M

Symlink
-------

    mklink [LINK_NAME] [TARGET_PATH]

And for directories:

    mklink /D [LINK_NAME] [TARGET_PATH]

Robocopy
--------

    robocopy [SOURCE] [TARGET] /MIR

Cd to current directory when run as admin
-----------------------------------------

    cd %~dp0

Get host name by ip
-------------------

    nbtstat -a 192.168.5.42

ssh copy id
-----------

    type  id_rsa.pub | plink.exe -l checker -pw ****** -P 22 example.com "umask 077; test -d .ssh || mkdir .ssh ; cat >> .ssh/authorized_keys"

exit if previous command has non zero exit code
-----------------------------------------------

	REM do something...
	IF %ERRORLEVEL% NEQ 0 EXIT /B %ERRORLEVEL%

find executable and run it
--------------------------

	FOR /F "usebackq delims=" %%I IN (`WHERE /R %~dp0 ReportGenerator.exe`) DO SET REPORTGENERATOR=%%I
	%REPORTGENERATOR% -reports:results.xml -targetdir:coverage -verbosity:Error

find executable and get its directory name
------------------------------------------

	FOR /F "usebackq delims=" %%I IN (`WHERE /R . *.exe`) DO ECHO %%~pI

add all folders with executables to path
----------------------------------------

	FOR /F "usebackq delims=" %%I IN (`WHERE /R packages *.exe`) DO SET PATH=%PATH%;%%~pI
