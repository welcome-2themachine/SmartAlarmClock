//Smart Alarm Clock
//Derived from the 2014 Tony DiCola Smart Alarm Clock
//Chris Prevost and Rafael Lima

#include <TouchScreen.h>
#include <TFTShield.h>
#include <Process.h>

/*
*  This above libraries allow us to use the touchscreen and interface
*  with the Linux process or on the Arduino Board
*/

#define TEM_ACCOUNT        "arduinotempleu"
#define TEM_APP            "smartalarm"
#define TEM_KEY            "ff8e0dd4b9d949c0aecf5b85e10ce893"
#define CAL_CREDS          "GoogleCalendarNew"
#define CAL_ID             "arduino.test.temple@gmail.com"
#define GM_CREDS           "GoogleMail"
#define GM_KW              "WAKE UP"

#define ALARM_SOUND        "/mnt/sda1/arduino/www/SmartAlarmClock/Annoying_Alarm_Clock.mp3"
#define ALARM_OPTIONS      "--attenuate=-12"
#define ALARM_REFRESH      "60"//minutes between alarm refreshes
#define ALARM_BUFFER       "60"//amount of time before first meeting the alarm sounds

#define TS_MINX 140
#define TS_MAXX 900
#define TS_MINY 120
#define TS_MAXY 940


#define STOCK_COLOR    CYAN
#define TEXT_COLOR     WHITE
#define ALT_COLOR      BLACK
#define R_BUTTON_X     30
#define R_BUTTON_Y     215
#define L_BUTTON_X     30
#define L_BUTTON_Y     30
#define BUTTON_WIDTH   105
#define BUTTON_HEIGHT  50
#define DATE_X         MAX_X - 10
#define DATE_Y         50
#define WEATHER_X      100
#define WEATHER_Y      100
#define HILO_X         130
#define HILO_Y         170
#define TEMP_X         130
#define TEMP_Y         80
#define TIME_X         190
#define TIME_Y         25
#define ALARM_X        70
#define ALARM_Y        30
#define ALARM_TIME_X   50
#define ALARM_TIME_Y   30

unsigned long last_Update = 0;
int time = 0;
int alarm = -1;
int refresh_Time;
char dateString[16];
bool playing;
TouchScreen screen = TouchScreen(A3, A2, A1, A0, 300);
Process linux;

void setup() {
  Bridge.begin();
  Serial.begin(9600);
  
  Tft.init();
  Tft.setDisplayDirect(UP2DOWN);
  Tft.drawString("Clever Startup Message", L_BUTTON_X, L_BUTTON_Y, 3, STOCK_COLOR);
  /*TODO: Set up the startup message with global variables for spacing*/
  time = get_time();
  get_date(dateString);
  alarm = get_alarm();
  draw(time, alarm);
  last_Update = millis();
}
/*
*  Code runs one time at start IOT set alarm conditions
*  draw: draws on the screen
*  millis: gets the current system time
*/


void loop() {
  unsigned long current = millis();
  if(long(current - last_Update) >= 60000L){
    time = time_add(time, 1);
    if(time == 0){
      int temp_time = get_time(); //resets the time at midnight
      
      if((temp_time - time) >= 2){
        if((alarm > time)&&(alarm<temp_time)){
          alarm_start();
        }
      }
      
      time = temp_time;
      /*
      *  This fixes a bug in the open source code that causes time to skip over alarms
      *  upon refreshes
      */
    }
    
    if(time % ALARM_REFRESH == 0){
        refresh();
    }
    /*
    *  Checks the calendar and mail for alarms
    */
    
    if(time == alarm){
      alarm_start();
    }
    /*
    *  starts the alarm if it's time
    */
    
    draw(time, alarm);
    last_Update = millis();
  }
  
  TSPoint touch = screen.getPoint();
  touch.x = map(touch.x, TS_MINX, TS_MAXX, MAX_X, 0);
  touch.y = map(touch.y, TS_MINY, TS_MAXY, MAX_Y, 0);
  
  if(touch.z > screen.pressureThreshhold && playing){
    alarm_stop();
    alarm = -1;
    draw(time, alarm);
  }
  /*
  *  stop the alarm, reset the alarm time to -1 and then redraw the display
  */
  
  if(touch.z > screen.pressureThreshhold && touch.x >= R_BUTTON_X && touch.x <= (R_BUTTON_X + BUTTON_HEIGHT) && touch.y >= R_BUTTON_Y && touch.y <= (R_BUTTON_Y + BUTTON_WIDTH)){
    Tft.fillRectangle(R_BUTTON_X, R_BUTTON_Y, BUTTON_HEIGHT, BUTTON_WIDTH, STOCK_COLOR);
    Tft.drawString("Refresh", R_BUTTON_X + 30, R_BUTTON_Y + 5, 2, ALT_COLOR);
    time = get_time();
    alarm = refresh();
    Tft.paintScreenBlack();
    draw(time, alarm);
  }
  /*
  *  highlights the refresh button and refreshes the alarm times. It then paints
  *  the screen black and redraws the display.
  */
  
  if(alarm != -1 && touch.z > screen.pressureThreshhold && touch.x >= L_BUTTON_X && touch.x <= (L_BUTTON_X + BUTTON_HEIGHT) && touch.y >= L_BUTTON_Y && touch.y <= (L_BUTTON_Y + BUTTON_WIDTH)){
    Tft.fillRectangle(L_BUTTON_X, L_BUTTON_Y, BUTTON_HEIGHT. BUTTON_WIDTH, STOCK_COLOR);
    Tft.drawString("Cancel", L_BUTTON_X + 30, L_BUTTON_Y + 5, 2, ALT_COLOR);
    alarm = -1;
    Tft.paintScreenBlack();
    draw(time, alarm);
  }
  /*
  *  highlights the refresh button and cancels any upcoming alarms. It then paints
  *  the screen black and redraws the display
  */
  
  delay(10);
}

void refresh(){
  get_alarm();
  check_mail_alarm();
  last_Update = millis();
}

void alarm_start(){
  alarm_stop();
  linux.begin("madplay");
  linux.addParameter("-Qr"); // Disable output of status and repeat audio continuously.
  linux.addParameter(ALARM_OPTIONS);
  linux.addParameter(ALARM_SOUND);
  linux.runAsynchronously();
  linux = true;
}
/*
*  ensures that there is not alarm playing then plays the alarm
*/

void alarm_stop(){
  if(linux.running()){
    linux.close();
    linux == false;
  }
}
/*
*  stops the alarm
*/

void time_to_string(int time, char* timeString) {
  int hour12, minute;
  time_to_hour_minute(time, hour12, minute);
  timeString[8] = 0;
  timeString[7] = 0;
  timeString[6] = 0;
  timeString[5] = ' ';
  timeString[4] = '0' + (minute % 10);
  timeString[3] = '0' + (minute / 10);
  timeString[2] = ':';
  timeString[1] = '0' + (hour12 % 10);
  timeString[0] = '0' + (hour12 / 10);
}
/*
*  changes the time input to a string
*/

int time_add(int time, int minutes) {
  int result = time + minutes;
  if (result < 0) {
    return 1440 - ((-1 * result) % 1440);
  }
  else {
    return result % 1440;
  }
}//adds minutes to time

int time_to_hour_minute(int time, int& hour12, int& minute) {
  hour12 = time / 60;
  minute = time % 60;
}//converts the time to hour and minute

int time_to_minutes(int hour24, int minute) {
  return hour24*60+minute; 
}

int get_time() {
  Process date;
  date.begin("date");
  date.addParameter("+%H%M");
  date.run();
  int hour24 = 10 * (date.read() - '0');
  hour24 += date.read() - '0';
  int minute = 10 * (date.read() - '0');
  minute += date.read() - '0';
  return time_to_minutes(hour24, minute);
}

void get_date(char *dateString){
  Process date;
  date.begin("date");
  date.addParameter("+%a:%d:%b:%Y");
  date.run();
  dateString = date.read();
  dateString[4]  = ' ';
  dateString[6]  = ' ';
  dateString[10] = ' ';
}

int get_alarm() {
  Process findAlarm;
  findAlarm.begin("python");
  findAlarm.addParameter("/mnt/sda1/arduino/www/SmartAlarmClock/find_alarm.py");
  findAlarm.addParameter(TEM_ACCOUNT);
  findAlarm.addParameter(TEM_APP);
  findAlarm.addParameter(TEM_KEY);
  findAlarm.addParameter(CAL_CREDS);
  findAlarm.addParameter(CAL_ID);
  findAlarm.run();
  int alarm = -1;
  if (findAlarm.available() >= 2) {
    int alarmHour24 = int(findAlarm.read());
    int alarmMinute = int(findAlarm.read());
    alarm = time_to_minutes(alarmHour24, alarmMinute);
    alarm = time_add(alarm, -ALARM_BUFFER);
  }
  return alarm;
}

unsigned long get_mail_lastseen() {
  Process checkMail;
  checkMail.begin("python");
  checkMail.addParameter("/mnt/sda1/arduino/www/SmartAlarmClock/check_email.py");
  checkMail.addParameter(TEM_ACCOUNT);
  checkMail.addParameter(TEM_APP);
  checkMail.addParameter(TEM_KEY);
  checkMail.addParameter(GM_CREDS);
  checkMail.addParameter(GM_KW);
  checkMail.run();
  unsigned long lastseen = 0;
  if (checkMail.available() >= 4) {
    lastseen |= ((unsigned long)checkMail.read() << 24);
    lastseen |= ((unsigned long)checkMail.read() << 16);
    lastseen |= ((unsigned long)checkMail.read() << 8);
    lastseen |= checkMail.read();
  }
  return lastseen;
}

void check_mail_alarm() {
  unsigned long lastSeen = get_mail_lastseen();
  if (lastSeen > last_Update) {
    alarm_start();
  }
}

void draw(int time, int alarmTime) {
  Tft.paintScreenBlack();
  char alarmString[9];
  char timeString[9];
  time_to_string(time, timeString);
  Tft.drawString("Weather Condition", WEATHER_X, WEATHER_Y, 1, STOCK_COLOR);
  Tft.drawString("Temp", TEMP_X, TEMP_Y, 2, STOCK_COLOR);
  Tft.drawString("Hi|Lo", HILO_X, HILO_Y, 2, STOCK_COLOR);
  Tft.drawString(timeString, TIME_X, TIME_Y, 7, STOCK_COLOR);
  Tft.drawString(dateString, DATE_X, DATE_Y, 2, STOCK_COLOR);
  int hour12, minute;
  time_to_hour_minute(time, hour12, minute);
  if (alarmTime >= 0) {
    time_to_string(alarmTime, timeString);
    Tft.drawString("Alarm:", ALARM_X, ALARM_Y, 2, STOCK_COLOR);
    Tft.drawString(alarmString, ALARM_TIME_X, ALARM_TIME_Y, 2, STOCK_COLOR);
    Tft.drawString("Cancel", 30, 30, 2, TEXT_COLOR);
  }
  Tft.drawString("Refrsh", R_BUTTON_X, R_BUTTON_Y, 2, TEXT_COLOR);
}
