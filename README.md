# EventLog

### Design Decision #1: User shall be able to log any LabVIEW datatype.  (not just strings)

![image](https://user-images.githubusercontent.com/7429922/209487674-4cae87da-0c9b-424f-9af4-e94b6912f716.png)


Note that the Message input is a variant.  The user can pass in any LabVIEW data structure to be logged.

There are a few reasons for wanting this:

* Most event loggers are string-based.  It's often necessary for programmers to have to write small pieces of code to generate an appropriate string message from relevant data.  This code is annoying to write and time consuming.
* It's advantageous to be able to pass a LabVIEW structure into the logger.  If this structure changes in the future (for example, maybe someone adds a new data element), then this is automatically captured.  Conversely, in this example, it's unlikely that the programmer remembered to update the string log message maker, so the change isn't captured.  Basically, this solution here is more future-proof.
* Making event logging less painful will make me want to do it more often.  I am lazy like that.

A few notes about this:

* The UI runs in real-time with event logs, and displays new messages as they arrive
* The UI is able to consume log files.  This means that we are able to analyze logs both in real-time and after-the-fact using the same user interface.
* Lots of helpful icons
* Tables headers can be clicked to adjust sort order
* Lost of polish, feels good to use.

---

### Design Decision #2: The Elog UI shall be able to adapt to log datatype
As previously, mentioned, the log datatype is able to be any LabVIEW data structure.  The Elog UI shall be able to adapt to this datatype to display the data in an optimized manner depending on each log type.

Here are currently supported examples (not the full list):

| Datatype        | How it's viewed           | screenshot  |
| ------------- |:-------------:| -----:|
| String | String indicator | ![image](https://user-images.githubusercontent.com/7429922/209487905-dad7d967-29bb-4dac-b0b7-138243dd3547.png) |
| Error cluster | Error Cluster indicator | ![image](https://user-images.githubusercontent.com/7429922/209487920-86bf32ea-8f01-48d6-be3c-868d38401d53.png) |
| 1D or 2D primitive | Table indicator | ![image](https://user-images.githubusercontent.com/7429922/209488037-1dbb3340-b382-467b-adc4-243f3c385e9d.png) |
| NI Tree | Tree indicator | ![image](https://user-images.githubusercontent.com/7429922/209488105-7b81e49e-76d2-47bf-8385-5fdac4d81d1b.png) |
| BlueTreeMap      | BlueTreeMap Viewer | ![image](https://user-images.githubusercontent.com/7429922/209488112-288730af-b971-4df1-be8a-dfa5d7c0b1bd.png) |
| Anything else | Blue Variant Viewer | ![image](https://user-images.githubusercontent.com/7429922/209487931-1a38a1b3-f3c0-44cd-a4b9-8835f0e36a42.png) |

---

### Design Decision #3: There shall exist a top-level "logspace" tree to keep logging spaces silo'd

![image](https://user-images.githubusercontent.com/7429922/209488281-e00e7876-8afb-493a-b5c4-28e0a6d70814.png)

Each log space represents a different organizational unit for logs.

All event logs should go to the same Elogger, but simply used a different logspace value to keep things nicely categorized and easy to filter at the UI.

---

### Design Decision #4: There shall exist a "unique ID" input for each log message

![image](https://user-images.githubusercontent.com/7429922/209488309-e2c86b46-920d-4df9-9b16-6615696587b3.png)

![image](https://user-images.githubusercontent.com/7429922/209488320-ff418dd5-d951-4c09-8d6f-e40d94e46feb.png)

Each log message shall be coupled with a Unique string ID.  Each unique ID has the following characteristics:

* Each unique ID holds onto a "count" value, and appears as a separate row on the Lod UI table (shown above)
* Each unique ID holds into a circular buffer for log messages

---

### Design Decision #5: There shall exist a circular buffer for each logspace + Log UID

![image](https://user-images.githubusercontent.com/7429922/209488369-47afda6a-e035-417e-82b4-98953af53d1b.png)

The Max entry count defines the circular buffer size.

The circular buffer is intended as a mechanism to prevent memory leak for runtime applications.

The circular buffer is non-preallocated.  This means that the array size grows until it reached the max entry count, and then it is capped.  The capped array does NOT rotate.  Instead, we keep track of the current index value:

---

### Design Decision #6: The log-to-disk method shall be a class, which programmers can override if they want

![image](https://user-images.githubusercontent.com/7429922/209488406-3199b151-edec-4846-a52e-93de8317a5f8.png)

There exists a Logger.lvclass, which implements the actual datalogger.  In the above snippet, you can see that we currently have 2 loggers:

| Logger        | Extensions           | Screenshot  |
| ------------- |:-------------:| -----:|
| LabVIEW Event Log      | \*.lvelog | ![image](https://user-images.githubusercontent.com/7429922/209488479-555060fc-cc6f-40f5-8a6f-9483ffad6d08.png) |
| CSV      | \*.csv      |  ![image](https://user-images.githubusercontent.com/7429922/209488484-219be5d3-35c3-4d65-898e-1bb9ed1f2a39.png) |

This allows for users to implement whatever type of datalogger file/mechanism that they want.  This can be done without modifying the core of Elogger.

Log files can be streamed to as the data comes in, or log files can simply be written to with all cached logs in memory.

---

### Design Decision #7: Elogger shall be able to load *.lvelog files

![image](https://user-images.githubusercontent.com/7429922/209488509-91f2bdc8-fd97-4b99-b457-acb7ace9ffc5.png)

I've already said this... I hate opening 2GB log files in notepad++.

Instead, the event log files shall open into the Elog UI.

Only the lossless \*.lvelog file supports loading.

---

### Design Decision #8: Elogger shall support polling of log files when loading
Basically, this means that it's possible to open and load a \*.lvelog into the Elog UI even while the file is actively being written to by an Elogger somewhere.

The Elog viewer will then poll the file for updates at 1hz.  If new log messages are found, then only the new log messages are read from file, and then added into the UI.  This works across applications thanks to the magic of TDMS files, which we are using under the hood here.

---

### Design Decision #9: Elogger shall be performant
* Shall be designed to support large (10GB?) log files
* Shall only update GUI components as necessary.  (Trees and listboxes are computationally expensive.)
* Expectation of handling cyclic log messages... maybe something like a few hundred log messages per second
* 64 bit LabVIEW

---

### Design Decision #10: There shall exist an installer for an Elogger viewing application
* LabVIEW 2020 SP1 64 bit
* Installer
* Shall apply custom icon to \*.lvelog files
* When \*.lvelog files are double clicked, this shall auto-launch the viewer
* shall support multiple instances of the application
