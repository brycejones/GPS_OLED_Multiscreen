Note: updated Adafruit's GPS library to include "GPS.antennastatus" method.
Date: brycej 20140225

<original text>
This is the Adafruit GPS library - the ultimate GPS library
for the ultimate GPS module!

Tested and works great with the Adafruit Ultimate GPS module
using MTK33x9 chipset
    ------> http://www.adafruit.com/products/746

These modules use TTL serial to communicate, 2 pins are required to  
interface

Adafruit invests time and resources providing this open source code, 
please support Adafruit and open-source hardware by purchasing 
products from Adafruit!

Written by Limor Fried/Ladyada  for Adafruit Industries.  
BSD license, check license.txt for more information
All text above must be included in any redistribution

To download. click the DOWNLOADS button in the top right corner, rename the uncompressed folder Adafruit_GPS. Check that the Adafruit_GPS folder contains Adafruit_GPS.cpp and Adafruit_GPS.h

Place the Adafruit_GPS library folder your <arduinosketchfolder>/libraries/ folder. You may need to create the libraries subfolder if its your first library. Restart the IDE.
<end original text>

***Quickread: I have found Adafruit_GPS library for Adafruit's Ultimate GPS may need 
modifications to correctly read and report antenna status reporting. I was required
to add $PGTOP as defined in Global Top's datasheet to allow the GPS to report 
antenna status. 


**Detail: Highlighted below are modifications I completed to get the active antenna to 
correctly report status. I implemented this after reading Global Top's datasheet
titled GPS_GlobalTop-FGPMMOPA6H-Datasheet-V0A.pdf. To use the new GPS.antennastaus
method you can use the included modified Adafruit GPS libray Adafruit_GPSmod by including
in your libraries directory.

-- 
Cheers, Bryce

|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
File= libraries/Adafruit_GPSmod.h  -additions included below within existing code:

// ask for the release and version
#define PMTK_Q_RELEASE "$PMTK605*31"

// *************************************************************
// Believe this implementation must be modified after reading specs.
// comment: brycej 02/18/2014 Correct is $PGTOP
// request for updates on antenna status.
#define PGCMD_ANTENNA "$PGCMD,33,1*6C" 
#define PGCMD_NOANTENNA "$PGCMD,33,0*6D"

// Modified by brycej correcting to read antenna status per datasheet:
#define PGTOP_ACTIVEANTENNA "$PGTOP,11,3,*6F"    // Using Active Antenna
#define PGTOP_ANTENNA "$PGTOP,11,2*6E"                 // Using internal Antenna
#define PGTOP_SHORTANTENNA "$PGTOP,11,2*6D"      // Active Antenna Shorted!! 
// *************************************************************

uint8_t antennastatus;            //added for parsing antenna status -brycej



|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
File= libraries/Adafruit_GPSmod.cpp -additions shown below within existing code:

  ///////////////////////////////////////////////////// 
  // 
  // look for a few common sentences
  //
  ///////////////////////////////////////////////////// 

  // Is the sentence found $PGTOP?, If yes, then parse for antenna status:
  // Format= $PGTOP,11,3*6F note: added by brycej 02/25/2014

  if (strstr(nmea, "$PGTOP")) {
    // found PGTOP
    char *p = nmea;
    // get antenna status
    p = strchr(p, ',')+1;
    p = strchr(p, ',')+1;   //antenna status field
    antennastatus = atoi(p);



|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
Example Arduino sketch additions using newly supported Antenna Supported Protocol method: 

      Serial.print("Angle: "); Serial.println(GPS.angle);
      Serial.print("Altitude: "); Serial.println(GPS.altitude);
      Serial.print("Satellites: "); Serial.println((int)GPS.satellites);
      Serial.print("Antenna Status: "); Serial.print(GPS.antennastatus, DEC);  //print antenna status
  
      switch ( GPS.antennastatus ) {
           case 1:
              Serial.print(" -Antenna Shorted!!\n");        // Fault with Antenna
           break;
           case 2:
              Serial.print(" -Using Internal Antenna\n");   // GPS using internal ceramic antenna
           break;
           case 3:
              Serial.print(" -Using Active Antenna\n");     // GPS using active antenna
           break;
           
           default:
           // Get out of here
           break;
           }



|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
EXAMPLE CONSOLE OUTPUT:

Time: 18:55:50.0
Date: 2/26/2014
Fix: 1 quality: 2
Location: 3854.9091N, 9439.3886W
Speed (mph): 0.01
Angle: 304.97
Altitude: 314.00
Satellites: 9
Antenna Status: 3 -Using Active Antenna

$PGTOP,11,3*6F
$GPGGA,185551.000,38__.9093,N,094__.3888,W,2,9,1.01,314.0,M,-30.2,M,0000,0000*66
$GPRMC,185551.000,A,38__.9093,N,094__.3888,W,0.02,304.97,260214,,,D*79



<fini>
