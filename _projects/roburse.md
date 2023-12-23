---
layout: page
title: Roburse
description: A medical assistant for the sick and elderly.
img: /assets/img/roburse1.jpeg
importance: 2
---

<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/roburse2.jpeg' | relative_url }}" alt="" title="Roburse in action"/>


Most of the elderly people in Europe and North America live alone or in an elderly home, depending on someone to satisfy their needs. Roburse aims to solve that and provide the relatives/closed ones a closer view into lives of their old folks. Roburse provides P2P video chatting, remote navigation, recording daily schedule/diet/medication. We used a Dagu thunder rover with Arduino Uno, a motor driver, a bluetooth module and a phone mounted on top of it. The phone is connected to the Arduino via bluetooth and the caretaker can control the robot via an android application. The robot is also equipped with a camera and a microphone to provide a view of the surroundings to the caretakers. The caretaker can also set alarms for the elderly person and record their daily schedule/diet/medication. The application also provides a videoconferencing interface for the caretaker and the elderly person to communicate with each other. 

**Implementation details**

*Remote navigation*

Navigation commands are sent from the caretaker's phone to a Firebase real-time database. The database then delivers it to the phone mounted on the robot. Since the phone is connected via Bluetooth to the Arduino, the signals are sent via bluetooth to the arduino board's Tx and Rx pins. Depending on the type of signal (left, right, forward, backward), the Arduino board sends an appropriate signal to a PWM motor driver, which helps in spinning the wheels of the robot.

*P2P video calling*

To provide a view of the surroundings to the caretakers, the robot is equipped with video calling features. [Sinch video calling API](https://www.sinch.com/products/apis/calling/video/), a WebRTC based API was used to build the main video calling application. 

**Complete workflow**

The android application starts with a fingerprint authentication activity. Then it opens up a login page for both caretaker and the elderly person. The user can enter his/her phone number in order to make call to the API. We used phone number as the username and client ID for the API for signalling purposes. After both the users log on into the service, an additional Bluetooth configuration page opens for the elderly person after which there’s an option to either set an alarm or place a call for both clients. The application asks for permission to access phone contacts and opens the contact list using which both the users can search for each other’s name and place the call. After the caretaker places a call, the phone rings for 5 seconds and the call is picked automatically on the elder's side. Once the call is established, the caretaker can send the signals via Firebase and control the rover.

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/roburse3.jpeg' | relative_url }}" alt=""/>
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/roburse4.jpeg' | relative_url }}" alt=""/>
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/roburse5.jpeg' | relative_url }}" alt=""/>
    </div>
</div>