
faster-file-rotator
===================

NodeJS file stream rotator

> Fork from [file-stream-rotator](https://github.com/rogerc/file-stream-rotator),but much faster

## Why rewrite？

Looks like file-stream-rotator no one maintained.

> File-stream-rotator generate audit.json file to rotate file，but audit.json will become pretty large because of adding one log stamp in every new stream. As audit.json is actually an array used in filter, and this array's length may be 800 thousand, it will seriously affect server performance.

## Purpose 

To provide an automated rotation of Express/Connect logs or anything else that writes to a log on a regular basis that needs to be rotated based on date. It can also be rotated based on a size limit and remove old log files based on count or elapsed days. 

## Install

```
npm install faster-file-rotator
```

## Options

 - *filename*:       Filename including full path used by the stream
 - *frequency*:      How often to rotate. Options are 'daily', 'custom' and 'test'. 'test' rotates every minute.
                     If frequency is set to none of the above, a YYYYMMDD string will be added to the end of the filename.
 - *verbose*:        If set, it will log to STDOUT when it rotates files and name of log file. Default is TRUE.
 - *date_format*:    Format as used in moment.js http://momentjs.com/docs/#/displaying/format/. The result is used to replace
                     the '%DATE%' placeholder in the filename.
                     If using 'custom' frequency, it is used to trigger file rotation when the string representation changes.
 - *size*:           Max size of the file after which it will rotate. It can be combined with frequency or date format.
                     The size units are 'k', 'm' and 'g'. Units need to directly follow a number e.g. 1g, 100m, 20k.
 - *max_logs*        Max number of logs to keep. If not set, it won't remove past logs. It uses its own log audit file
                     to keep track of the log files in a json format. It won't delete any file not contained in it.
                     It can be a number of files or number of days. If using days, add 'd' as the suffix.

 - *audit_file*      Location to store the log audit file. If not set, it will be stored in the root of the application.

## Example Usage
```javascript
    // Default date added at the end of the file
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"daily", verbose: false});
 
    // Default date added using file pattern
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test-%DATE%.log", frequency:"daily", verbose: false});
 
    // Custom date added using file pattern using moment.js formats
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test-%DATE%.log", frequency:"daily", verbose: false, date_format: "YYYY-MM-DD"});
 
    // Rotate when the date format as calculated by momentjs is different (e.g monthly)
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test-%DATE%.log", frequency:"custom", verbose: false, date_format: "YYYY-MM"});
 
    // Rotate when the date format as calculated by momentjs is different (e.g weekly)
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test-%DATE%.log", frequency:"custom", verbose: false, date_format: "YYYY-ww"});
 
    // Rotate when the date format as calculated by momentjs is different (e.g AM/PM)
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test-%DATE%.log", frequency:"custom", verbose: false, date_format: "YYYY-MM-DD-A"});
 
    // Rotate on given minutes using the 'm' option i.e. 5m or 30m
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"5m", verbose: false});
     
    // Rotate on the hour or any specified number of hours
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"1h", verbose: false});

    // Rotate on the hour or any specified number of hours and keep 10 files
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"1h", verbose: false, max_logs: 10});

    // Rotate on the hour or any specified number of hours and keep 10 days
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"1h", verbose: false, max_logs: "10d"});

    // Rotate on the hour or any specified number of hours and keep 10 days and store the audit file in /tmp/log-audit.json
    var rotatingLogStream = require('file-stream-rotator').getStream({filename:"/tmp/test.log", frequency:"1h", verbose: false, max_logs: "10d", audit_file: "/tmp/log-audit.json"});

    //.....    
    
    // Use new stream in express
    app.use(express.logger({stream: rotatingLogStream, format: "default"}));
    
    //.....

```
    
You can listen to the *open*, *close*, *error* and *finish* events generated by the open stream. You can also listen for custom events:

  * *rotate*: that will pass two parameters to the callback: (*oldFilename*, *newFilename*)
  * *new*: that will pass one parameter to the callback: *newFilename*
  
You can also limit the size of each file by adding the size option using "k", "m" and "g" to specify the size of the file in kiloybytes, megabytes or gigabytes. When it rotates a file based on size, it will add a number to the end and increment it for every time the file rotates in the given period as shown below.
  
```
  3078  7 Mar 13:09:58 2017 testlog-2017-03-07.13.09.log.20
  2052  7 Mar 13:10:00 2017 testlog-2017-03-07.13.09.log.21
  3078  7 Mar 13:10:05 2017 testlog-2017-03-07.13.10.log.1
  3078  7 Mar 13:10:08 2017 testlog-2017-03-07.13.10.log.2
  3078  7 Mar 13:10:11 2017 testlog-2017-03-07.13.10.log.3
  3078  7 Mar 13:10:14 2017 testlog-2017-03-07.13.10.log.4
```  

  The example below will rotate files daily but each file will be limited to 5MB.
  
```javascript
    // Rotate every day or every 5 megabytes, whatever comes first.
    var rotatingLogStream = require('file-stream-rotator').getStream(
        {
            filename:"/tmp/test-%DATE%.log", 
            frequency:"custom", 
            verbose: false, 
            date_format: "YYYY-MM-DD",
            size: "5M" // its letter denominating the size is case insensitive
        }
    );
    rotatingLogStream.on('rotate',function(oldFile,newFile){
        // do something with old file like compression or delete older than X days.
    })
```
