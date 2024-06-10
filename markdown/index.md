---
title: 'EEC 172 Final Lab: AWS Integrated Crash Detection'
author: 'Ernest Wang and Wahad Latif'
date: '*EEC172 SQ24*'

subtitle: '<blockquote><b>EEC172 Final Project</b><br/>
The website source is hosted 
<a href="https://github.com/Wahad10/EEC172FinalProjectWebsite">on github</a>.
</blockquote>'

toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'We will be implementing a simple crash detection/alert device using in-built sensors and other interfacing technologies. We hope to primarily use the accelerometer onboard the LaunchPad microcontroller to gather the raw acceleration data for processing. We plan to use OLEDs to display crash reports, IR Receiver and TV Remote to get user input to confirm/deny email notification, AWS transactions to send emails of crash reports, and etc.. Our end goal is to interface local crash detection to a wider ecosystem of notifications.
<br/><br/>
Our source code can be found 
<!-- replace this link -->
<a href="https://github.com/Wahad10/EEC172FinalProject">
  here</a>.

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe style="left:0;top:0;width:100%;height:100%;position:absolute;" width="560" height="315" src="https://www.youtube.com/embed/9T2GwJXAk10" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
'
---

<!-- EDIT METADATA ABOVE FOR CONTENTS TO APPEAR ABOVE THE TABLE OF CONTENTS -->
<!-- ALL CONTENT THAT FOLLWOWS WILL APPEAR IN AND AFTER THE TABLE OF CONTENTS -->


# Design

## Overview

Two CC3200 boards will be used to implement an user-observer system. The user system will poll for a crash and send the information via AWS to the observer. The observer will process the data and send a response back. 

## System Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style='display: inline-block; vertical-align: top;'>
    <img src="./media/architecture_board1.png" style="width:auto;height:200"/>
    <span class="caption">Figure 1: System Architecture Board 1 - User</span>
  </div>
  <div style='display: inline-block; vertical-align: top;'>
    <img src="./media/arch_board2.png" style="width:auto;height:200" />
    <span class="caption">Figure 2: System Architecture Board 2 - Observer</span>
  </div>
</div>

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style='display: inline-block; vertical-align: top;'>
    <img src="./media/functional_spec_board_1.png" style="width:auto;height:200"/>
    <span class="caption">Figure 3: Functional Specification Board 1 - User</span>
  </div>
  <div style='display: inline-block; vertical-align: top;'>
    <img src="./media/functional_spec_board_2.png" style="width:auto;height:200" />
    <span class="caption">Figure 4: Functional Specification Board 2 - Observer</span>
  </div>
</div>

## Description

<ol>
  <li>Board 1 starts in the crash detection state where it polls the accelerometer data and determines if there is a crash or not. Once the data from the accelerometer surpasses a user set threshold, the user board sends a post request to AWS and awaits for a decision from the observer. After receiving the decision, the board can acknowledge the decision and reset the system.

  Board 1 will be connected to the accelerometer sensor on the board via an internal I2C line. This will allow the board to have access to acceleration data. The board will also be able to poll SW 3 using GPIO for user input for the acknowledge. In addition to hardware connections, the board will be connected to the internet using Ti's SimpleLink capabilities to enable communication with the AWS servers. </li>

  <li>Board 2 starts in the crash detection state where it polls the AWS server for the message that a crash was detected. Once the crashed flag is found in the AWS IOT server, the program displays a corresponding on the OLED and asks for the user for input. The user will input a decision through IR communication between the board and a remote. Once a decision is made, the board posts the decision to AWS and returns to poll for a crash message.

  Board 2 will primarily be connected to AWS via the SimpleLink protocol to read the state of the IOT server. Once the crash state is read, the Board uses timer interrupts on a GPIO pin to record the IR signal from the remote; the board also will update the OLED screen to ask the observer for a decision. The board then interprets the IR signal and sends an update to the server while changing the OLED screen to reflect the decision. </li>
</ol>


# Implementation

## Board 1

<ol>
  <li><strong>Crash Detection</strong>
  
  The board communicates with the on-board accelerometer via an internal I2C line on Pin 1 and 2. The board reads 3 values from 3 addresses on the I2C corresponding to the x,y,z acceleration data. The board then calculates the sum of the directional-magnitudes squared to get a representative value of the overall magnitude. This value is then compared against a preset value to determine whether or not a crash happened. </li>

  <li><strong>Send Post</strong> 
  
  The board posts the crashed message to AWS IOT using SimpleLink protocols. The function was written in lab 3 and was modified to be able to post any arbitrary string. The modifications use string concatenation to generate any message that the user wants.  </li>

  <li><strong>Wait for Decision</strong> 
  
  The board gets the AWS IOT shadow every 800 ms using the get function written in lab 3. The input is then searched through to find whether or not the crash message was acknowledged or ignored. The program then automatically advances to the user acknowledge state. </li>

  <li><strong>User Acknowledge</strong>
  
  The board in the user acknowledge state polls the input on SW3. When the switch is pressed, the board acknowledges the result and resets the AWS shadow to the default state and returns to polling for a crash. </li>
</ol>

## Board 2

<ol>
  <li><strong>Crash Detection State</strong>
  
  The board gets the AWS IOT shadow every 800 ms using the get function written in lab 3. The input is then searched through to find whether or not the crash message was received. Once the message is found, the program then automatically advances to the user decision select state.  </li>

  <li><strong>Decision Select State</strong>
  
  The board writes a crash image to the OLED and ask the user to either acknowledge the crash or ignore the crash. The board then begins to poll the IR sensor for IR signals from the remote. This code was written in lab 3 and remains largely the same. Our implementation requires only 3 buttons: 1 for acknowledge, 2 for ignore, 3 for send. Thus, the other cases are ignored.  </li>

  <li><strong>Post Decision</strong>
  
  The board writes the decision to the AWS IOT shadow, using the same logic from the Sent Post State on board one, except the posted message contains the decision. The program is then returned to polling for another crashed message. </li>
</ol>

## IR Sensor Connection

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <img src="./media/circuit.png" style="width:auto;height:200"/>
  <span class="caption">Figure 5: Circuit Provided in Lab 3</span>
</div>


# Challenges

The primary challenge throughout the implementation of the crash detection protocol was ensuring the state transitions and state logic was robust and sound. There was a need to test each transition between states and ensure if there are unexpected outputs, the overall program would not derail. For board 1, it was ensuring that after the first crash detected, the board would stop polling for input as the decision is being processed or that inputs on the switch would not effect the state when the board is polling for the observer's response. For board 2, it was ensured that the crash detection state was not triggered by remote buttons, and that the remote buttons that were not used did not introduce bugs within the program. Overall, to ensure desired behavior, we chose to restrict some polling to the respective states that need said polling. This prevented unwanted input bugs and streamlined the logical flow of the code. Functions written in previous labs were used to implement the whole project, thus we were confident in the functionality of said functions. One major hurdle that needed to be overcome was the ability to write arbitrary messages to the AWS shadow. The message sent has to match that of the JSON format stored in the AWS cloud, thus we need a tailor-able system that allowed us to send the messages that we want. To send the messages, we leveraged string concatenation allowing us to keep the general format of a header block and tail block while inserting the message of interest in between. This allowed for the posting of arbitrary string messages. Another major hurdle is finding the message that we wanted within the get request. After some research, a simple function that found a sub string with in a string (strstr) was found and used in our implementation. Initially, it was planned to provide feedback to the user on the user's board via the LED's, but the internal I2C lines use pin 1 and 2, thus preventing the addressing of the LEDs. We weren't able to implement the feedback on the board, but there is a potential implementation where the LEDs are off the board and controlled through addressable GPIO pins. Overall, the challenges faced with implementing the crash detection protocol were mostly solved, except for some improvements that can be made in the future.


# Future Work

The overall goal of this project is to lay frame work that mediates control between a user and an observer for crash detection. The logical next steps are the inclusion of other interactions that the board can support. As mentioned in the challenges section, there is no user feedback on the board itself. This can easily be remedied using other GPIO pins to toggle LEDs connected to the board via bread board(for prototype) or PCB for a "final" product. User usage can be further improved with the implementation of the OLED and messages for the user when they fall. The core algorithm can also be modified to change such that if there is a detected crash the user can self report if the crash was a false alarm or not, by passing the need for the observer to decide whether or not the crash should be acknowledged or ignored. More interconnected routines can be added such as manual crash notifications on the user side through the use of another button, allowing the user to call for help without the need of a crash. Local alarms could also be implemented with another CC3200 board that continuously polls the AWS shadow for the crashed image and trigger a local speaker to alert people with the device of a crash. Speakers can be added to the observer board to further draw attention when a crash is detected.  In generality, future additions to this project will center around the interfacing of the core crash detection to other services/routines. 


# Finalized Bill of Materials (BOM)

Note that all of these materials were provided in the lab.

<!-- you can convert google sheet cells to html for free using a converter
  like https://tabletomarkdown.com/convert-spreadsheet-to-html/ -->

<table style="border-collapse:collapse;">
<thead>
  <tr>
    <th><p>NUMBER</p></th>
    <th><p>QUANTITY</p></th>
    <th><p>MATERIAL</p></th>
    <th><p>DESCRIPTION</p></th>
    <th><p>COST</p></th>
    <th><p>WHERE TO OBTAIN</p></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><p>1</p></td>
    <td><p>1</p></td>
    <td><p>SimpleLink CC3200 LaunchPad (CC3200-LAUNCHXL)</p></td>
    <td><p>LaunchPad Microcontoller, comes with an onboard Bosch BMA222 acceleration sensor connected via I2C interface</p></td>
    <td><p>$66.00</p></td>
    <td><p>ti.com</p></td>
  </tr>
  <tr>
    <td><p>2</p></td>
    <td><p>1</p></td>
    <td><p>Code Composer Studio (CSS) IDE</p></td>
    <td><p>IDE used to code, debug, and build the programs for the micro-controller</p></td>
    <td><p>Free</p></td>
    <td><p>ti.com</p></td>
  </tr>
  <tr>
    <td><p>3</p></td>
    <td><p>1</p></td>
    <td><p>CC3200 Software Development Kit and SDK-ServicePack</p></td>
    <td><p>Helper folders and files for the CSS IDE containing demo programs</p></td>
    <td><p>Free</p></td>
    <td><p>ti.com</p></td>
  </tr>
  <tr>
    <td><p>4</p></td>
    <td><p>1</p></td>
    <td><p>TI SysConfig Tool</p></td>
    <td><p>TI Pin Mux tool that helps configure GPIO and other pin configurations. The software outputs C files which can be added to CC3200 projects in the CSS IDE.</p></td>
    <td><p>Free</p></td>
    <td><p>ti.com</p></td>
  </tr>
  <tr>
    <td><p>5</p></td>
    <td><p>1</p></td>
    <td><p>CCS UniFlash</p></td>
    <td><p>Flashing software that helps program the CC3200 projects onto the serial Flash chip onboard the LaunchPad to allow the micro-controller to run programs without downloading them from a running CSS IDE</p></td>
    <td><p>Free</p></td>
    <td><p>ti.com</p></td>
  </tr>
  <tr>
    <td><p>6</p></td>
    <td><p>1</p></td>
    <td><p>PuTTY</p></td>
    <td><p>Windows terminal emulator which the LaunchPad connects to via COM4 port</p></td>
    <td><p>Free</p></td>
    <td><p>putty.org</p></td>
  </tr>
  <tr>
    <td><p>7</p></td>
    <td><p>1</p></td>
    <td><p>Breadboard Kit with Connector Wires, include at least 1 Resistor and Capacitor</p></td>
    <td><p>Simple breadboard with hardware components to help build a circuit to connect Launchpad to other sensors/devices</p></td>
    <td><p>$20.00</p></td>
    <td><p>amazon.com</p></td>
  </tr>
  <tr>
    <td><p>8</p></td>
    <td><p>1</p></td>
    <td><p>OLED (Organic Light Emitting Diode) Display</p></td>
    <td><p>This is the Adafruit OLED Breakout Board Display - 16-bit Color 1.5" which is connected to the LaunchPad using a small breadboard via the SPI interface. It will show a message indicating crash report.</p></td>
    <td><p>$40.00</p></td>
    <td><p>amazon.com</p></td>
  </tr>
  <tr>
    <td><p>9</p></td>
    <td><p>1</p></td>
    <td><p>AT&T S10-S3 Remote</p></td>
    <td><p>This is the IR transmitter which is just a simple TV remote configured with a specific code to send specific waveforms to the receiver module. A datasheet is provided to help identify waveforms and translate to binary.</p></td>
    <td><p>$40.00</p></td>
    <td><p>amazon.com</p></td>
  </tr>
  <tr>
    <td><p>10</p></td>
    <td><p>1</p></td>
    <td><p>IR Receiver (Vishay TSOP311xx / 313xx / 315xx)</p></td>
    <td><p>This is the IR receiver module which is wired up in a simple application circuit and conencted to a GPIO pin on the Launchpad. It receives signals from the remote and sends them to the Launchpad for decoding.</p></td>
    <td><p>$7.00</p></td>
    <td><p>amazon.com</p></td>
  </tr>
  <tr>
    <td><p>11</p></td>
    <td><p>1</p></td>
    <td><p>AWS IoT Ecosystem</p></td>
    <td><p>Email notification service connected to the Launchpad and a user email via AWS.</p></td>
    <td><p>Free</p></td>
    <td><p>aws.com</p></td>
  </tr>
  <tr>
    <td><p>TOTAL PARTS</p></td>
    <td><p>11</p></td>
    <td colspan="2"><p>TOTAL</p></td>
    <td colspan="2"><p>$167.00</p></td>
  </tr>
</tbody>
</table>

# References
<a href="https://ucd-eec172.github.io/labs/project.html">https://ucd-eec172.github.io/labs/project.html</a>
