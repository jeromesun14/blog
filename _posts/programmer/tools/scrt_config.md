title: SecureCRT 使用记录
date: 2017-05-02 16:11:24
toc: true
tags: [SecureCRT]
categories: programmer
keywords: [SecureCRT, config, global, log]
description: SecureCRT 配置使用记录
---

## 默认配置文件

Windows 下，SecureCRT 配置文件默认存放于 `C:\Users\xxx\AppData\Roaming\VanDyke\Config`。
可直接备份该目录，复用所有配置。具体详见官方文章[Backing Up and Restoring SecureCRT® for Windows® and SecureFX® Settings](https://www.vandyke.com/support/tips/backupsessions.html)。

## logging 配置

The Log File category of the Session Options dialog allows you to customize the logging options for the selected session A session is a set of options that are assigned to a connection to a remote machine. These settings and options are saved under a session name and allow the user to have different preferences for different hosts..

* Log file name group

This section allows you to specify a default filename for your log file. You can enter a path or filename, or you can use the Browse button to open the Save As dialog and select the path or filename. You can also include any of the variables listed in the Substitution section below. These variables will be expanded when SecureCRT writes to the log file.

* Options group

This section lets you specify log file behaviors.

* Prompt for filename

Check this option to prompt the user for the session log filename when the log is started.

* Start log upon connect

Check this option to start writing to the session log file whenever a connection A data path or circuit between two computers over a phone line, network cable, or other means. is made. By default, this option is not selected.

* Overwrite file

If this option is selected and the session log file already exists, the current log file will be overwritten by the new session log file.

* Append to file

If this option is selected and the session log file already exists, the new session log will be appended to the existing log file. If the log file does not already exist, a new file is created.

* Raw log

Check this option to write every character received by SecureCRT, including terminal A device usually consisting of a keyboard, a display unit such as a cathode ray tube, and a serial port used for entering and sending data to a computer and displaying any data received from the computer. Terminals are usually connected to a computer with a serial line or some other type of connection. There are many terminal types including VT100, VT102, VT220, and others. escape sequences, to the session log file. By default, this option is off.

* Start new log file at midnight

Check this option to have SecureCRT start a new log file each night at midnight. Using this feature can aid in automatic log rotation.

* Custom log data group

This section lets you specify custom messages to be written to the log file. You can include any of the variables listed in the Substitution section below. These variables will be expanded when SecureCRT writes to the log file.

* Upon connect

Entries in this box will be written to the log file when the session connects.

* Upon disconnect

Entries in this box will be written to the log file when the session disconnects.

* On each line

Entries in this box will be written to each line of the log file.

* Log only custom data

If this option is selected, SecureCRT will log only the custom data from the three fields above. All normal log messages will be suppressed.

* Substitutions group

In the dialog, this section lists some of the variables that SecureCRT can expand when writing to the log file. A complete list is shown below.

* %H - hostname
* %S - session name
* %Y - four-digit year
* %y - two-digit year
* %M - two-digit month
* %D - two-digit day of the month
* %P - port
* %h - two-digit hour
* %m - two-digit minute
* %s - two-digit seconds
* %t - three-digit milliseconds
* %% - percent (%)
* %envvar% - environment variable

Note: The environment variable substitution occurs first.
