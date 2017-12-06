/*
fft_adc_serial.pde
guest openmusiclabs.com 7.7.14
example sketch for testing the fft library.
it takes in data on ADC0 (Analog0) and processes them
with the fft. the data is sent out over the serial
port at 115.2kb.
*/

#define LOG_OUT 1 // use the log output function
#define FFT_N 256 // set to 256 point fft

#include <FFT.h> // include the library

int _7kHz = 3;
int _12kHz = 4;
int _17kHz = 5;
int _660Hz = 6;
int SoundCount;

int twelve = 0; 
int treasure = 0; //0 - no treasure, 1 - 7 khz, 2 - 12 khz, 3 - 17 khz

bool StartDetection(void) {
    cli();  // UDRE interrupt slows this way down on arduino1.0
        for (int i = 0 ; i < 512 ; i += 2) { // save 256 samples
          while(!(ADCSRA & 0x10)); // wait for adc to be ready
          ADCSRA = 0xf7; // restart adc
          byte m = ADCL; // fetch adc data
          byte j = ADCH;
          int k = (j << 8) | m; // form into an int
          k -= 0x0200; // form into a signed int
          k <<= 6; // form into a 16b signed int
          fft_input[i] = k; // put real data into even bins
          fft_input[i+1] = 0; // set odd bins to 0
        }
        fft_window(); // window the data for better frequency response
        fft_reorder(); // reorder the data before doing the fft
        fft_run(); // process the data in the fft
        fft_mag_log(); // take the output of the fft
        sei();
    
    // checks bin 19 and if great than bins to left and right, turns on external LED
    
    if ((fft_log_out[18] > 1.2*fft_log_out[15]) && (fft_log_out[18] > 1.8*fft_log_out[21])) {
      SoundCount++;
      if (SoundCount > 30) return true; //Turn on LED
      //Serial.println("In if");
    }
    else {
      SoundCount = 0;
      return false; //Turn off LED
      //Serial.println("In else");
    }
}

void setup() {
  Serial.begin(115200); // use the serial port
  TIMSK0 = 0; // turn off timer0 for lower jitter
  ADCSRA = 0xe7; // set the adc to free running mode
  ADMUX = 0x45; // use adc5 for microphone
  DIDR0 = 0x01; // turn off the digital input for adc0
  pinMode(_7kHz, OUTPUT); //Setup pin 9 as output
  pinMode(_12kHz, OUTPUT); //Setup pin 9 as output
  pinMode(_17kHz, OUTPUT); //Setup pin 9 as output
  pinMode(_660Hz, INPUT); // Sound Detection

while (1) {
if (StartDetection() == true) break;
if (digitalRead(_660Hz)) break;
}

pinMode(_660Hz, OUTPUT); // Sound Detection
ADCSRA = 0xe5; // set the adc to not free running mode
ADMUX = 0x40; // use adc0 for treasure detection
digitalWrite(_660Hz, HIGH);
}

void loop() {

 // ADMUX = 0x40; // use adc0 for treasure detection
 // ADCSRA = 0xe5; // set the adc to not free running mode
  while(1) { // reduces jitter
    cli();  // UDRE interrupt slows this way down on arduino1.0
    for (int i = 0 ; i < 512 ; i += 2) { // save 256 samples
      while(!(ADCSRA & 0x10)); // wait for adc to be ready
      ADCSRA = 0xf5; // restart adc
      byte m = ADCL; // fetch adc data
      byte j = ADCH;
      int k = (j << 8) | m; // form into an int
      k -= 0x0200; // form into a signed int
      k <<= 6; // form into a 16b signed int
      fft_input[i] = k; // put real data into even bins
      fft_input[i+1] = 0; // set odd bins to 0
    }
    fft_window(); // window the data for better frequency response
    fft_reorder(); // reorder the data before doing the fft
    fft_run(); // process the data in the fft
    fft_mag_log(); // take the output of the fft

    sei();

    //determine treasure detected
    //from lab2 bins corresponding to relevant frequencies are
    // 7 kHz = bin 46  12kHz = bin 78   17kHz = bin 114
    // 7 kHz treasure detected
    int max_bin = 0; //target bin value
    int max_amp = 0; //max fft amplitude
    //extract and sort fft data to find max amplitude and bin 
    for (byte i = 2 ; i < FFT_N/2 ; i++) { 
      if(fft_log_out[i] > max_amp){
        max_amp = fft_log_out[i];
        max_bin = i;
      }
    }
    
    //Serial.println(max_bin); 
    //detect treasure frequency by evaluating bin ranges
    
    //7 khz (max_bin >= 45) && (max_bin <= 47)
    if((max_bin >= 45) && (max_bin <= 57) && (max_amp > 110)){
      Serial.print("7 kHz Treasure detected  ");
      Serial.println(max_bin);
      Serial.print("  ");
      Serial.print(max_amp);
      treasure = 1; 
      digitalWrite(_7kHz, HIGH);
      delay(500);
    }

    //12 khz (max_bin >= 79) && (max_bin <= 81)
    //prev 100
    else if((max_bin >= 65) && (max_bin <= 85) && (max_amp > 50)){
      Serial.print("12 kHz Treasure detected ");
      Serial.println(max_bin);
      //twelve++;
      treasure = 2; 
      digitalWrite(_12kHz, HIGH);
      delay(500);
    }
    
    //17 khz max_bin >= 113) && (max_bin <= 115
    //prev 100
    else if((max_bin >= 110) && (max_bin <= 120) && (max_amp > 80)){
      Serial.print("17 kHz Treasure detected ");
      Serial.println(max_bin);
      treasure = 3; 
      digitalWrite(_17kHz, HIGH);
      delay(500);
    }
    
    //no treasure
    else{
     Serial.println("No Treasure detected");
     treasure = 0; 
     twelve = 0;
     digitalWrite(_7kHz, LOW);
     digitalWrite(_12kHz, LOW);
     digitalWrite(_17kHz, LOW);
    }
  }
}
