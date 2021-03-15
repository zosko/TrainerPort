#define INPUT_FREQUENCY 50
#define CHANNEL_NUMBER 4  //set the number of chanels
#define FRAME_LENGTH 22500  //set the PPM frame length in microseconds (1ms = 1000µs)
#define PULSE_LENGTH 300  //set the pulse length
#define onState 1  //set polarity of the pulses: 1 is positive, 0 is negative
#define PPM_SIGNAL 3  //set PPM signal output pin on the arduino

int ppm[CHANNEL_NUMBER];

void setup(){
  delay(300);
  Serial.begin(57600);

  for(int i = 0; i < CHANNEL_NUMBER; i++){
      ppm[i]= 1500;
  }

  pinMode(PPM_SIGNAL, OUTPUT);
  digitalWrite(PPM_SIGNAL, !onState);  //set the PPM signal pin to the default state (off)
  
  cli();
  TCCR1A = 0; // set entire TCCR1 register to 0
  TCCR1B = 0;
  
  OCR1A = 100;  // compare match register, change this
  TCCR1B |= (1 << WGM12);  // turn on CTC mode
  TCCR1B |= (1 << CS11);  // 8 prescaler: 0,5 microseconds at 16mhz
  TIMSK1 |= (1 << OCIE1A); // enable timer compare interrupt
  sei();
}

void loop(){  
//  ppm[0] = 1500; //rudder
  ppm[1] = map(analogRead(0), 0, 1024, 2000, 1000); // ELEVATOR
//  ppm[2] = map(analogRead(2), 0, 1024, 1000, 2000); // THROTTLE
  ppm[3] = map(analogRead(1), 0, 1024, 1000, 2000); // ALERONS

  delay(1000 / INPUT_FREQUENCY);
}

ISR(TIMER1_COMPA_vect){  //leave this alone
  static boolean state = true;
  
  TCNT1 = 0;
  
  if (state) {  //start pulse
    digitalWrite(PPM_SIGNAL, onState);
    OCR1A = PULSE_LENGTH * 2;
    state = false;
  } 
  else {  //end pulse and calculate when to start the next pulse
    static byte cur_chan_numb;
    static unsigned int calc_rest;
  
    digitalWrite(PPM_SIGNAL, !onState);
    state = true;

    if(cur_chan_numb >= CHANNEL_NUMBER){
      cur_chan_numb = 0;
      calc_rest = calc_rest + PULSE_LENGTH;
      OCR1A = (FRAME_LENGTH - calc_rest) * 2;
      calc_rest = 0;
    }
    else{
      OCR1A = (ppm[cur_chan_numb] - PULSE_LENGTH) * 2;
      calc_rest = calc_rest + ppm[cur_chan_numb];
      cur_chan_numb++;
    }     
  }
}
