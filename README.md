# genmon
# Generator Monitoring Application

    This project will monitor a backup generator that utilized the Generac Evolution
    Controller. The project is based on a python application and has been tested with a
    Raspberry Pi3. Ideally you would need to create a physical enclosure for your
    raspberry pi and possibly make a cable to connect the raspberry pi to the Evolution
    controller. If you are comfortable doing these things and you have a backup generator
    that has an Evolution controller then this project may be of interest to you.

## Functionality
   The software supports the following features:

    Monitoring of the generator to to detect and report the following:
        - Maintenance, Start / Stop and Alarm Logs
        - Generator warnings and faults (Wiring Error, High Temp on air cooled models, Low Oil Pressure,
                low coolant on liquid cooled models)
        - Generator Status:
            - Engine State
                - Generator Switch State (Auto, On, Off)
                - Generator Engine State (Stopped, Starting, Exercising, Running Manual,
                    Running Utility Loss, Stopped due to Alarm, Cooling Down)
                - Battery Voltage and Charging Status
                - Relay Output State: (Starter, Fuel Relay, Battery Charger, others for liquid cooled models)
                - Engine RPM, Hz and Voltage Output
                - Generator Controller Time
            - Line State
                - Utility Voltage Level
                - Transfer Switch State
        - Outage Information
            - Time since last outage
            - Current Utility Voltage
            - Min and Max Utility Voltage since program started
        - Maintenance Information
            - Weekly Exercise time, day and duration
            - Hours till next scheduled service
            - Total Run Hours
            - Firmware and Hardware versions
        - Various statics from the generator monitor including time since program launched,
              MODBUS / serial communications health and program health.
    Email notification of :
        - Engine state change
        - Switch state change
        - Critical or Warning messages from the generator
    Web based application for viewing status of the generator

## Testing
    This software was written by one person with access to one generator. The model used
    for testing and development is a liquid cooled model. The software was written with
    every intention of working on liquid and air-cooled models with the Evolution
    controller however the author has not tested all scenarios.

## Known Issues:
    Some code exist for remotely starting the generator and remotely activating the
    transfer switch, however this is highly experimental. Additional work is needed before
    this functionality is fully enabled.

    The ability to determine the hours the generator has exercised vs backup vs total run
    hours is enabled however the total run hours is the value with the most confidence.
    More time and data points are needed to increase the confidence in this metric.

    The Controller contains a register that should hold details about the model of the
    generator. My controller was replaced and as a result, the register on my system has
    not been properly initialized. I would need feedback from other people to determine
    the format of this register.

## Connectivity
    This application was written to be agnostic of the underlying network media (i.e. WiFi,
    Ethernet, etc). Testing and development was performed with WiFi. WiFi access points
    were connected to an uninterruptible power supply (UPS) so connectivity is not lost
    power is transferred from the utility to the generator.

## Software

    genmon.py (required)

        genmon.py is a python program to communicate with the Generac Evolution Controller
        used in some liquid and air cooled stand by generators. The program is writen
        using python 2.7 but earlier versions should work. Python 3.x has not been
        tested.
        This program will communicate with the Evolution Controller over the serial
        port using the MODBUS protocol. The application will allow the display and
        monitoring of the generator much like the Generac Mobile Link product.

        Once setup the genmon.py program will send an email when your generator does
        interesting things (power goes out, alarm on the generator, weekly exercise, etc).
        The program will also allow emails to be sent to an IMAP email folder. The program
        monitors the IMAP folder and takes commands from the email subject. The format of
        the subject of the email should have "generator:"  followed by one or more of the
        following commands:

        Commands:
           status      - display engine and line information
           maint       - display maintenance and service information
           outage      - display current and last outage (since program launched) info,
                            also shows utility min and max values
           monitor     - display communication statistics and monitor (genmon.py) health
           logs        - display all alarm, on/off, and maintenance logs
           registers   - display contents of registers being monitored
           settime     - set generator time to system time
           setexercise - set the exercise time of the generator. i.e. setexercise=Monday,13:30
           setquiet    - enable or disable exercise quiet mode, i.e.  setquiet=on or setquiet=off

         Example subject:
            generator: status maint

            generator: setexercise=Tuesday,23:45 setquiet=on

        The program uses the file genmon.conf to for configuration data. Edit this file
        place it in the /etc directory before running the program.
        genmon.py uses the following modules so they are external dependancies
        of the program and they will need to be install before running the program:

        crcmod - https://pypi.python.org/pypi/crcmod, used for MODBUS CRC calculation
        pyserial - https://pypi.python.org/pypi/pyserial/2.7, used for serial. Download
            the source and follow the instructions at https://github.com/gsutil-mirrors/crcmod
            communication. This can be installed via the following command:
                sudo apt-get install python-serial

        In addition the the external dependancies there are additional python modules
        included in this project that are used:

        mymail.py - This module provides support for sending email and monitoring an IMAP
        email folder for incoming mail. The mail module can be configured by modifying the
        mymail.conf file and placing it in the /etc directory.
        mylog.py - This modules provides logging support to the other modules.
        ALARMS.txt - This text file contains generator error codes with descriptions and
            potential solutions. It is used by the program to supply additional details when
            error conditions are detected by the generator monitor.

        Log files for errors are generated by genmon.py, mymail.py and for serial
        communications. These files are written to /var/log and named genmon.log,
        mymail.log and myserial.log.

        A note about email and security: The genmon.py app monitors folder on an IMAP
        email account for emails with a subject containing "generator:". If this is found,
        the remaining characters of the subject filed are parsed for commands for the
        monitor program. Since commands are sent to genmon.py via the subject  line of an
        email, is is suggested that some level of email filtering occur on the receiving
        email account. For example, gmail supports email filters that if emails arrive
        from specific people with specific words in the subject, move them ot specific
        folders. You could then have genmon.py monitor the specific folder used in your
        filter to only allow specific emails to send commands to genmon.py. If this approach 
        is used then genmon.py can be configured (via mymail.com) to monitor a specific 
        IMAP folder.

        Gmail IMAP has been tested with this application. It should be easy to test this
        other email providers. All mail processing is contained in mymail.py and
        mymail.conf.

        Due to the location of log files and conf files, root privileges are needed to
        execute the program. Other than these files, there are no requirements of root
        privileges.

        Installation of genmon.py is simple. All python source files (*.py) can be put in
        a single directory, all config files (*.conf) go in /etc. Edit the *.conf files so
        they are specific to your installation (email user name and password, etc). You
        can launch the program as a background application if needed,
        ie. "python genmon.py &". Also I use crontab to load the application as a
        background process on boot by launching a bash script via crontab. The script loads
        the app via the "python genmon.py &" command.

        The program ClientInterface.py is a test application for communicating with
        genmon.py via sockets. The ClientInterface.py app takes one command line argument,
        the IP address of the computer running genmon.py. To issues commands to an instance
        genmon.py on a system at IP address of 192.168.11.100 you would do the following:

            python ClientInterface.py 192.168.11.100

        Once the app is executed you should be faced with a prompt ">". From this prompt
        you can send commands to genmon.py. Commands are prefaced with "generator:". For
        example to issue the command "status" you would enter the "generator: status" at
        the ">" prompt when running ClientInterface.py.

        The genmon.py appplication supports a a socket interface for communication with 
        exeternal applications. In addition the the above mentioned ClientInterface.py 
        application, genmon.py supports communicating via the socket interface so the 
        application and generator can be monitored by network monitoring tools like Nagios.
        The program check_monitor_system.py can be used with as a Nagios Plugin to monitor
        genmon.py. See https://www.nagios.org/ for Nagios details. check_monitor_system.py 
        is the name of the supplied nagios plug-in.
    
    server/genserv.py (optional)
        genserv.py is a python application that uses the Flask library/framework.
        (http://flask.pocoo.org/). This approach allows a quick and simple python socket
        interface to be translated to a javascript based web interface. The genserv.py app,
        when executed, will serve up a simple web page that will display the status of the
        generator. Both the genserv.py app and the genmon.py app can be hosted on a single
        Raspberry Pi although you should be able to move the genserv.py program to another
        system with little modification. The web application provides all most of the
        information supplied by genmon.py. The "registers" and "settime" commands are not
        supported by the web interface (decided to keep it simple).
        The setup for flask is detailed at http://flask.pocoo.org/. I did not used a
        virtual environment since this is a single purpose and low traffic web app (i.e.
        I do not expose the web app to the internet, only my local network). If you want
        expose the web app to the internet I would recommend adding authentication, using
        virtual environment and possibly a full web server to actually serve up the web
        pages since security concerns would be heightened on a public web server.

        The genserv.py program also uses the mylog.py module. Genserv.py also uses the
        same configuration file /etc/genmon.py. The file /var/log/genserv.log is used for
        logging errors. The flask library serve up static HTML, CSS and javascript files
        which are stored in a directory below the genserv.py app named static. Below are
        files and locations for genserv.py

        ./genserv.py                        - main app
        ./template/command_template.html    - used for error processing in flask
        ./static/index.html                 - main site page
        ./static/genmon.css                 - main site style sheet
        ./static/genmon.js                  - main site javascript
        ./static/favicon.ico                - icon for html file

        The default settings provide for hosting the web app on port 8000:

          Example: http://YourIPAddressGoesHere:8000

        Internally, the javascript, calls to the genserv.py app, which communicates with
        genmon.py via private socket calls.

## Hardware

    This project has been developed and tested with a Raspberry Pi 3 as the base platform.
    Since the serial port and network are the only external ports used, the program could
    be used on other platforms with minor modifications and testing.

    In development and testing I used the Raspberry Pi3 with built in WiFi. Depending on your
    WiFi signal and your generator proximity to the access point your results may vary.

    Below is a list of hardware that I used. Since your generator may be different and
    your network will be different you will need to validate these for your setup.

    - Raspberry PI 3 and SD Card

    Power supply for the Raspberry Pi is attached to battery on the generator. Note, this is 
    needed to ensure the raspberry Pi and the generator controller share a common ground. 
    If you use a two prong wall adapter to power your pi you will likely see CRC errors since 
    the controller cable does not include a ground wire for the serial device.

    This is the power supply I used, which allows the Pi to be powered from the generator battery:
    https://smile.amazon.com/gp/product/B01DYE54LI/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1

    The enclosure I used. This may be to large for some smaller air-cooled generators:
    https://smile.amazon.com/gp/product/B005UPAN0W/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1

    Internal Panel for Enclosure. This may be to large for some smaller air-cooled generators.
    https://smile.amazon.com/gp/product/B005UPE614/ref=od_aui_detailpages01?ie=UTF8&psc=1

    Adhesive Magnets from Hobby Lobby (used to attach enclosure to generator). I attached the
    enclosure to the inside of my generator housing but your generator may be different.
    http://www.hobbylobby.com/Crafts-Hobbies/Basic-Crafts/Magnets/1%22---8-Pieces-Square-Magnets-with-Foam-Adhesive/p/25089

    Wire (for cable)
    https://smile.amazon.com/gp/product/B00INVEWJ8/ref=oh_aui_detailpage_o04_s00?ie=UTF8&psc=1

    Tubing for cable (not recommended, possibly use a smaller diameter tube)
    https://smile.amazon.com/gp/product/B00542XWXG/ref=oh_aui_detailpage_o04_s00?ie=UTF8&psc=1

    The cable I used connects the Molex connector on the Evolution Controller to a DB-9 break-out
    connector. The DB-9 break-out connector is then attached to the RS-232 to TTL converter for the
    Raspberry Pi. Below are the links for the items used in cabeling:

    Crimp tool for cable
    https://smile.amazon.com/gp/product/B00OMM4YUY/ref=oh_aui_detailpage_o06_s00?ie=UTF8&psc=1

    3.3v TTL to RS232 for RPi
    https://smile.amazon.com/gp/product/B00EJ9NAKA/ref=oh_aui_detailpage_o03_s00?ie=UTF8&psc=1

    Break out db-9 (male or female, depending on the above level converter you use)
    https://smile.amazon.com/gp/product/B014FM8MNK/ref=oh_aui_detailpage_o05_s00?ie=UTF8&psc=1
    https://smile.amazon.com/gp/product/B014FKU5W8/ref=oh_aui_detailpage_o06_s00?ie=UTF8&psc=1

    I used the following Digi-key part numbers for the molex connectors for the cable. The
    evolution controller uses a molex type receptacle:

    Plug = WM3603-ND
    Male pin = WM2500CT-ND

    Receptacle = WM3703-DN
    Female pin = WM3279CT-DN


    Evolution Controller has Receptacle so cable should look like this:

    To make inline cable extension from Controller to Raspberry Pi:
    Controller + Plug + Cable length + Receptacle + ML

    I used a molex connector on my enclosure and routed the two wires to a break-out box that
    a DB-9 (see link above)


