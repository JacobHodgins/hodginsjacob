---
layout: post
title: "Digital Metronome with Arduino"
date: 2023-12-27
---
<div class="post">
  <div class="post-intro">
    <p>
    Welcome! For my first ever project uploaded to this website, I decided to combine my knowledge of circuits and software (prior to second year of university) to create a functional and practical design. In this case, I created a digital metronome using an Arduino UNO R3 microcontroller, various electrical components,  and a solderless breadboard.
    </p>
  </div>
  <div class="post-header">What is a metronome?</div>
  <div class="post-content">
    <p>
      A metronome is used by musicians to aid in learning a particular song, develop rhythmic skills for an instrument, and compose music. It has two important functions that are common amongst devices sold on the market: a ticking noise that repeats at a consistent tempo and the ability to change tempo for different songs. The tempo is measured in beats per minute (bpm), where standard values used in songs range from grave (usually 20 to 40 bpm) to prestissimo (over 200 bpm).
    </p>
  </div>
  <div class="post-header">Virtual Design and Verification</div>
  <div class="post-content">
    <p>
     Before making a physical prototype, I used Tinkercad’s circuit design capabilities to design the metronome and develop a program for the metronome’s function. The idea is to plan a structure for the device and troubleshoot all of the included functions to limit the number of issues or mistakes that can be encountered when putting the prototype together.
    </p>
  </div>
  <div class="post-content">
    <p>
     The metronome consists of an Arduino UNO R3 microcontroller, a solderless breadboard, and various electrical components (a piezo speaker, two pushbuttons, one slide switch, a 16x2 LCD screen, a potentiometer, and male to male wires). The piezo speaker produces the ticking noise previously mentioned, although since it’s a cheap component, it produces more of an annoying buzz. The two pushbuttons are used to adjust the tempo’s bpm value up and down, one button to raise the value and one to lower. The 16x2 LCD screen is used as a console to print to, in this case it displays the status of the metronome (ON/OFF) and the tempo value in bpms. The potentiometer accompanies the LCD screen and varies the brightness of the screen by varying the resistance value.
    </p>
  </div>
  <div class="post-content">
    <p>
     The program in Tinkercad has three main sections: definitions, program setup, and an execution loop. The most important section is the execution loop, where most of the program is interacted with by users. The following diagram demonstrates the basic function of the execution loop. The entire program is available at the end of the post.
    </p>
  </div>
  <img src="{{ site.baseurl }}images/metronome post picture 2.png" alt="Execution Loop Basic Functions Diagram" width="850" length="597">
  <div class="figure-name"><b>Figure 1</b> - Diagram of basic functions of execution loop</div>
  <div class="post-content">
    <p>
     Another interesting thing to note in the execution is the calculations done for what I’ve defined as the period and delay time. The metronome ticking sequence occurs for a whole bar or 4 counts since the metronome developed is in 4/4 count. In order for an individual buzzing sound from the piezo speaker to be recognizable by ear, I added a delay time to the length of the sound. It’s a helpful feature, but it has some issues. For a given tempo, there is a constant period of time between beats that I have defined as the period. When this value is large, ticks occur less often in a minute, and therefore a slower tempo is produced. If the value is small, a quicker tempo is produced. Since the delay time has to use up a certain amount of real time, this must be accounted for in the time spent during a period, which is relatively straight forward. Just subtract the delay time from the period, to get the time between sounds. The delay time is also a constant value defined at the beginning of the program and remains unchanged. That means that if the period becomes equal to or less than the delay time, the metronome will either sound strange or just sound like one long beep. This occurs when the bpm is about 400. Most songs do not reach this tempo and usually stay within the 100 to 140 bpm range. So for general use it is not a problem but it does create a limitation on the capabilities of the metronome.
    </p>
  </div>
  <img src="{{ site.baseurl }}images/metronome period delay time.png" alt="Period and Delay Time Diagram" width="777" length="308">
  <div class="figure-name"><b>Figure 2</b> - Diagram of period and delay time</div>
  <div class="post-header">Physical Realization</div>
  <img class="post-image" src="{{ site.baseurl }}images/IMG_7962.JPG" alt="Physical Digital Metronome" width="600" length="600">
  <div class="post-content">
    <p>
     Figure # shows the physical metronome made using an arduino and solderless breadboard. This device is a one-to-one copy of the virtual design made in Tinkercad. Typically with breadboard implementations, soldering is used to make more seccure connections for the wires and electrical components. However, I decided not to pursue soldering as it would leave permanenent residue on the through-hole pins, removing the reusability of the components for experimenting with other circuit types.
    </p>
  <div class="figure-name"><b>Figure 3</b> - Physical Implementation of Digital Metronome</div>
  <div class="post-header">Testing and Challenges</div>
  <div class="post-content">
    <p>
     Once the metronome was created, it was necessary to test how accurate it performed. During testing, I noticed a constant delay time that built up over time between buzzes, making the metronome go out of sync the longer it plays. I believe the root of the delay issue stems from how the metronome sequence is integrated into the execution loop. The sequence is an if conditional that activates after the computer checks the status of every single button and conducts the necessary calculations required for the sequence. That translates to a constant time spent by the computer before each bar checking unnecessary changes in the status of every button. To fix this I made a minor change to how the metronome ticking sequence occurs. Instead of an if conditional that activates at the end of the entire execution loop, I made the sequence a repeating while loop that only checks for the status of the play switch at the end of each bar. With this change, the metronome became indistinguishable with other metronomes found online and I could not interpret any stacking delays.
    </p>
  </div>
  <div class="post-header">Final Arduino Program</div>
  <div>
    <pre>
// libraries
#include <LiquidCrystal.h>

// lcd screen pins
LiquidCrystal lcd(2, 3, 4, 5, 6, 7);

// device pins
const int piezo = 10;
const int onSwitch = 11;
const int upButton = 12;
const int downButton = 13;

// variables
int switchState;
int upButtonState;
int downButtonState;
int flag = 0;
float bpm = 90;
float period;
float delayTime;

void setup()
{
  // display serial monitor
  Serial.begin(9600);
  
  // buttons
  pinMode(onSwitch, INPUT);
  pinMode(upButton, INPUT);
  pinMode(downButton, INPUT);
  
  // outputs
  pinMode(piezo, OUTPUT);
  
  // lcd setuo
  lcd.begin(16, 2);
}

void loop()
{
  // independent variables
  switchState = digitalRead(onSwitch);
  upButtonState = digitalRead(upButton);
  downButtonState = digitalRead(downButton);
  delayTime = 150;

  // display OFF to lcd screen on start
  if(switchState == LOW)
  {
    lcd.setCursor(0,0);
    lcd.print("OFF                    ");
  }

  // start count sequence
  if(switchState == HIGH)
  {
    flag = HIGH;
  }
  
  // bpm changing buttons
  if(upButtonState == HIGH)
  {
    bpm = bpm + 1;
    delay(200);
  }
  else if(downButtonState == HIGH)
  {
    bpm = bpm - 1;
    delay(200);
  }
  
  // bpm restrictions
  if(bpm < 24)
  {
   	bpm = 24;
  }
  if(bpm > 220)
  {
    bpm = 220;
  }
  
  // calculate period between metronome beeps
  period = 60/bpm*1000;
  
  // display BPM on start
  lcd.setCursor(0,-1);
  lcd.print("BPM = ");
  lcd.print(bpm);
  lcd.print("                        ");
  
  while(flag == HIGH)
  {
    //redeclare switch state within while loop
    switchState = digitalRead(onSwitch);

    // display on signal
    lcd.setCursor(0,0);
    lcd.print("ON                      ");
    
    // metronome sound
    tone(piezo, 50);
    delay(delayTime);
    noTone(piezo);
    delay(abs(period - delayTime));
    
	  tone(piezo, 40);
    delay(delayTime);
    noTone(piezo);
    delay(abs(period - delayTime));
    
    tone(piezo, 40);
    delay(delayTime);
    noTone(piezo);
    delay(abs(period - delayTime));
    
    tone(piezo, 40);
    delay(delayTime);
    noTone(piezo);
    delay(abs(period - delayTime));
    
    // metronome sound
    if(switchState == LOW)
    {
      flag = LOW;
    }
  } 
}
    </pre>
  </div>
</div>



