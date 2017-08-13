# Robot_project_public
Robot with line follower
/*#include "mbed.h"
 
//AnalogIn analog_value(A0);
// 
//DigitalOut led(LED1);

int main() {
    
    int x=2;
    
    while(x==1|x==2) {
        printf("\nAnalogIn example\n");
    }
}*/

// Ahmed Babader        ID/ 13001782


#include "mbed.h"


//BusOut stepPins(PC_0, PC_1, PC_2, PC_3);
DigitalIn powerbutton(PC_13), usEcho(PC_5), usEcho2(PA_6);
DigitalOut in_1(PB_10), in_2(PA_8), in_3(PA_9), in_4(PC_7), green(PA_5), red(PB_9);
PwmOut pwm1_2(PB_4), pwm3_4(PB_6), usTrig(PC_6), usTrig2(PA_7);

DigitalIn outR(D2), outM(D3), outL(D4);             // looking from the back of the car
//DigitalOut in_1(PB_10), in_2(PA_8), in_3(PA_9), in_4(PC_7), g(PA_5);
//PwmOut pwm1_2(PB_4), pwm3_4(PB_6);



Timer A;

int echo(void);
int TurnRight;
int small_range;

class Motors {
public:
    Motors(void): poweroff(1), clockwise(1) {}        // delay(10) {}       // dont need the delay varible it was use to change
                                                                // the speed of the stepper but we will use PMW for DC 
    void tick(void);
    
    void forward(void);
    void backward(void);
    void left(void);
    void right(void);
    
    void stopLF(void);
    void forwardLF(void);
    void backwardLF(void);
    void rightLF(void);
    void leftLF(void);
    
//    void set_delay(int d) {delay = d;}
    void power(void) {
        if(poweroff == 0) {             // use user push button
            poweroff = 1;
        }else {
            poweroff = 0;
        }
    }
    void direction(void) {
        if (clockwise == 1) {
            clockwise = 0;
        } else {
            clockwise = 1;
        }
    }
    

    
private:
    int poweroff;
    int clockwise;
//    int right;    
};

int main()
{
    //pwm1_2.period_us(20000);        // RIGHT WHEELS LOOKING FROM THE BACK
    //pwm3_4.period_us(20000);
    usTrig.period_us(65535);
    usTrig2.period_us(65535);
    
    usTrig.pulsewidth_us(10);
    usTrig2.pulsewidth_us(10);
    
    
//    Motors sm;
    
    Motors All_motors;                      // later when we get another motor drive control each motor seprately
    
    int obstical_range;
      
    
    Ticker t;
    t.attach_us(&All_motors, &Motors::tick, 100000);
    
    
    while (1) {
//        char s[80];
//        int n;
//        gets(s);
//        puts(s);

        obstical_range = echo();
        
        if (!powerbutton.read()) {  // user button to toggle power insead
            All_motors.power(); 
            //wait(.2);    
        }
        
        if(obstical_range <= 10) {
            small_range=1;
            red.write(1);
            All_motors.direction();
        } else {
            red.write(0);
            small_range=0;
        }
        
        
          
          
          
/*        } else if (s[0] == 'D') {     // use it to change PWM i.e. speed
            sscanf(s+1, "%d", &n);
            if ((n > 3) && (n < 21)) {
                All_motors.set_delay(n);
            }*/
        
    }
}


int echo(void){         // the while in this function may cause the robot to stop if the echo is not reading
    int distance;
    int distance2;
    int time1;
    int time2;
    while (! usEcho.read()); //wait for us_echo to go high // osama added 21/7
    A.start();
    while (usEcho.read()); //wait for echo to go low   // osama added 21/7
    A.stop();
    time1=A.read_us();
    
    A.reset();
    
    while (!usEcho2.read());
    A.start();
    while (usEcho2.read());
    A.stop();
    time2=A.read_us();
    
    if (time1 > time2){
        distance2 = (time2/58);
        printf("D2 : %d\n", distance2);
        wait(0.2);
        TurnRight=0;
        //clockwise = 1;
        A.reset();
        return distance2;
    }else{
        distance = (time1/58);
        printf("D1 : %d\n",distance );
        wait(0.2);      
        TurnRight=1;
        //clockwise = 1;
        A.reset();
        return distance;
    }
//    printf("%d\n", A.read_us());
//    wait_us(100000);       // add if does not work
    //distance = A.read_us() / 58;
    
     
    //A.reset();
    //return distance;         // if does not return correctly put t.read_us() into integer varible  
}






void Motors::tick(void) {
//    static int counter = 0, shift = 0;              // dont need shift and counter
    if (poweroff){            //&& ((counter % delay) == 0)) {     // keep the poweroff varible
        if (small_range){
            if (!outR.read() && !outM.read() && !outL.read() ){       // out of line 
                if (clockwise) {        //if clockwise && distanse >10
                    forward();           
                    printf("\ninside ticker F" );
                }else{
                    if (!TurnRight) {
                        left();
                        printf("\ninside ticker L" );
                    }else if(TurnRight){
                        right();
                        printf("\ninside ticker R" );
                    } 
                }           
            }else if (outR.read() && (!outM.read()||outM.read()) && (!outL.read()||outL.read())){   //100,101,110,111
                stopLF();
                //printf("Turn Right\n");
            }else if (!outR.read() && (!outM.read()|| outM.read()) && outL.read()){   //001, 011
                stopLF();
                //printf("Turn Left\n");    
            }else if (!outR.read() && outM.read() && !outL.read()){   // 010
                stopLF();
                //printf("Turn Left\n");    
            }
        }else{
            if (!outR.read() && outM.read() && !outL.read()){       // when the R and L don't read black
                forwardLF();
                printf("GO FORWARD\n");  
            }else if (!outR.read() && !outM.read()|outM.read() && outL.read()){
                leftLF();
                printf("Turn Right\n");
        
            }else if (outR.read() && !outM.read()|outM.read() && !outL.read()){
                rightLF();
                printf("Turn Left\n");
            
            }else if (outR.read() && outM.read() && outL.read()){
                stopLF();      //right chosen randomly
                printf("STOP\n");
            }else if (outR.read() && !outM.read() && outL.read()){
                rightLF();         //right chosen randomly
                //printf("STOP\n");
            }else {
                printf("Nothing");
            }
        }
            
//        shift++;
    } else if (poweroff == 0) {     // keep the poweroff varible
        //stepPins.write(0);
        in_1.write(0); 
        in_2.write(0);
        in_3.write(0);
        in_4.write(0);
    }
//    counter++;
} 


void Motors::forward(void) {
    pwm1_2.pulsewidth_us(8000);
    pwm3_4.pulsewidth_us(8000);
    in_1.write(1); 
    in_2.write(0);
    in_3.write(0);
    in_4.write(1);
     
}

void Motors::backward(void) {
    left();
    in_1.write(0); 
    in_2.write(1);
    in_3.write(1);
    in_4.write(0);
    wait(2);
    clockwise = 1;
    
}

void Motors::left(void) {
    pwm1_2.pulsewidth_us(15000);
    pwm3_4.pulsewidth_us(15000);
    //green.write(1);
    in_1.write(1); 
    in_2.write(0);
    in_3.write(1);
    in_4.write(0); 
    wait(0.2);
    //green.write(0);
    clockwise = 1;
}

void Motors::right(void) {
    pwm1_2.pulsewidth_us(15000);
    pwm3_4.pulsewidth_us(15000);
    in_1.write(0); 
    in_2.write(1);
    in_3.write(0);
    in_4.write(1); 
    wait(0.2);
    clockwise = 1;
}
/////////// Linefollower/////////////////

void Motors::stopLF(void) {
    in_1.write(0); 
    in_2.write(0);
    in_3.write(0);
    in_4.write(0); 
}

void Motors::forwardLF(void) {
    pwm1_2.pulsewidth_us(20000); //8000
    pwm3_4.pulsewidth_us(5000); //8000
    in_1.write(1); 
    in_2.write(0);
    in_3.write(0);
    in_4.write(1); 
}

void Motors::backwardLF(void) {
    left();
    in_1.write(0); 
    in_2.write(1);
    in_3.write(1);
    in_4.write(0);
    //clockwise=1;  
}

void Motors::leftLF(void) {
    pwm1_2.pulsewidth_us(40000); //15000
    pwm3_4.pulsewidth_us(10000); //15000
    in_1.write(0); //0
    in_2.write(1); //1
    in_3.write(0); //0
    in_4.write(1); //1
    wait(0.05);
    //clockwise=1;
}

void Motors::rightLF(void) {
    pwm1_2.pulsewidth_us(40000); //15000
    pwm3_4.pulsewidth_us(10000); //15000
    in_1.write(1); //1
    in_2.write(0); //0
    in_3.write(1); //1
    in_4.write(0); //0
    wait(0.05);
    //clockwise=1;
}
