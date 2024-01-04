# PSA Electric Vehicle Communicator

PSA Electric Vehicle Communicator is a project which is focused on solving the main problems of the mobile app for the Stellantis electric vehicles. This code of this project is contained within the following two GitHub repositories [kyovchev/psa-ev-comm-web](https://github.com/kyovchev/psa-ev-comm-web) and [kyovchev/psa-ev-comm-server](https://github.com/kyovchev/psa-ev-comm-server). The first one is for the front end developed as Next.js React single page application. The second one is a Python program which is responsible for processing the gathered car data.

# Introduction

The Android [MY PEUGEOT App](https://play.google.com/store/apps/details?id=com.psa.mym.mypeugeot) has a very long refresh time and you will have to wait a lot in order to get information about the battery and the charging state. You cannot limit the charging level in the app. If you want to stop the charging at a specific percentage, you will have to manually refresh the info in the app and to manually select delayed charging.

There is an open source project: [Remote Control of PSA car](https://github.com/flobz/psa_car_controller). This project is a Python program to control and get information from a PSA car. The project is using the original MY PEUGEOT App to communicate with the car through the [Stellantis Connected Vehicle API](https://developer.groupe-psa.io/). The Python program can be used both for getting the car info and to limit the battery charging percentage. To be accessible from everywhere it requires to be hosted on a dedicated server with an external IP address with the appropriate network settings.

The current PSA EV Communicator project will rely on the Remote Control of PSA car for communication with the car but will be developed in such a way that it will not require external IP or a specific network configuration. The server hardware requirements will be the lowest possible with the idea both the communicator server and the PSA EV communicator to run on a tiny [Raspberry Pi Zero W](https://www.raspberrypi.com/products/raspberry-pi-zero-w/) server (1GHz, single-core CPU with a 512MB RAM). The project will provide a modern web based user interface which can be deployed and hosted on an external service, e.g. [Google Firebase Hosting](https://firebase.google.com/docs/hosting). The user interface will show the latest gathered information about the status of the car without any delay or waiting for the loading. This can be achieved by reading the latest information from a cloud based database service, e.g. [Google Cloud Firestore](https://firebase.google.com/docs/firestore). The communicator server will send browser notifications to the users of the supported browsers (e.g. Google Chrome on the Android platform). The complete architecture of the PSA Electric Vehicle Communicator is given below.

# Software Architecture

If we consider the system from end user point of view the architecture can be divided into the following four layers:

1. **Car embedded communicator** module which is managing communication between the car and the Stellantis backend.
2. **Car data and control monitor** which is communicating with the Stellantis API to gather car data and send control commands.
3. **User inteface backend** which is processing the car data and storing the car data within a specific database.
4. **Web based user interface** used for showing the car data to the end user. Required to show the car data as fast as possible with minimum delay.

The first layer is using an embedded into the car closed source module over which we can have no control. It communicates directly with the Stellantis backend. The Stellantis backend can be accessed through their API but this requires registration and authorization or your app which still is not open to the public. This can be solved by using their Android based app. You need to first download and register within the app and then the data of the car can be fetched by the app.

The second layer is responsible for automatization of the app usage. This can be done with the Python program [Remote Control of PSA car](https://github.com/flobz/psa_car_controller). It is open source and even distributed as a Python module. This program acts like the original Android app in order to communicate with the car through the Stellantis API. As already stated, this Python program can be hosted on your server, but this will require that you have external IP address and sufficient server resources in order to provide seamless user experience. In our approach we are introducing two more layers which are using cloud-based services to eliminate the specific server requirements and to easily customize the data shown to the end user.

The third layer is communicating with the second layer. It processes the car data and stores it in a local database. The most actual data is stored in a Google Cloud Firestore which can then be consumed by the web-based user interface. This layer is developed as a Python program and managed in the following GitHub repository: [kyovchev/psa-ev-comm-server](https://github.com/kyovchev/psa-ev-comm-server). It is supposed to be running on the same tiny server as the second layer. This allows it to access the data with a simple localhost HTTP request and in this way eliminating the need for a specific network configuration and security of the inbound connections. It just needs outbound internet access. The program is developed in such a way that it is monitoring the status of the connection with the car. In case of a failure restarts the second layer service and sends a notification to the browser of the end user. It also sends a notification for a specified battery levels. The notifications are also handled by the Firebase Cloud Messaging service. The third layer updates the car info at a preset time interval. This allows the user to open its browser and see the latest car status without the need to wait or to have to initiate car status update command from the original app.

The end user is using the web interface from the fourth layer to get the latest car data. The web interface is implemented as a single page app by using the Next.js React Framework. The code can be found at this GitHub repository: [kyovchev/psa-ev-comm-server](https://github.com/kyovchev/psa-ev-comm-server). It can easily be deployed to a cloud hosting service. The web user interface is fetching the data from the Cloud Firestore. It also implements Firebase authentication, so that only authorized users will be able to retrieve the sensitive car data, such as the current GPS location. Each user can be associated with a Firebase Cloud Messaging ID and then the user will be able to receive notifications from the third layer service.

# Web UI Implemented Functionality

The following functionality is working right now:

1. User authentication
2. Monitoring of the current battery status
3. Monitoring of the current charging status
4. Monitoring of the car state (incl. GPS position and odometer)
5. Subscription and reception of browser notifications

# Future Features

The following features are planned for next releases:

1. Monitoring and setup of the charging limit
2. Monitoring and setup of the car preconditioning
3. Implementing a daily mileage tracker and statistics

