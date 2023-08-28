---
layout: page
title: Roburse
description: A medical assistant for the sick and elderly.
img: /assets/img/roburse1.jpeg
importance: 2
category: School
---

<img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/roburse2.jpeg' | relative_url }}" alt="" title="Roburse in action"/>


Most of the elderly people in Europe and North America live alone or in an elderly home, thus depending on someone to satisfy their needs. Roburse aims to solve that and provide the relatives/closed ones a closer view into lives of their old folks. Roburse provides P2P video chatting, remote navigation, recording daily schedule/diet/medication. We use a dago thunder rover with Arduino and android phone mounted on it
which is used in the elderly person's side and an android app on the caretaker side. 

**Implementation details**

*Remote navigation*

Navigation commands are sent from the caretaker's phone to firebase's real-time database. The database then delivers it to the phone mounted on the robot. Since the phone is connected via bluetooth to the Arduino, the signals are sent via bluetooth to the arduino board's tx and rx pins. Depending on the type of signal (left, right, forward, backward), the arduino board sends  an appropriate signal to a PWM motor driver, which helps in spinning the wheels of the robot.

*P2P video calling*

To provide a view of the surroundings to the caretakers, the robot is equipped with video calling features. [Sinch video calling API](https://www.sinch.com/products/apis/calling/video/), a WebRTC based API was used to build the main video calling application. 

**Complete workflow**

The android application starts with a fingerprint authentication activity. Then it opens up a login page for both caretaker and the elderly person. The user can enter his/her phone number in order to make call to the API. Since the API requires both the clients to know each other’s login information and the information has to be unique as well, so we used phone number as the username and client ID for the API. So after both the users log on into the service, an additional bluetooth configuration page opens for the parent after which there’s an option to either set an alarm or place a call for both clients. The application asks for permission to access phone contacts and opens the contact list using which both the users can search for each other’s name and place the call. After the caretaker places a call, the phone rings for 5 seconds and the call is picked automatically on the elder's side. Once the call is established, the caretaker can send the signals via firebase and control the rover.

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