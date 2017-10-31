# parking
*
 * Mini-Smart-Vehicles.
 *
 * This software is open source. Please see COPYING and AUTHORS for further information.
 */

#include <stdio.h>
#include <math.h>

#include "core/io/ContainerConference.h"
#include "core/data/Container.h"
#include "core/data/Constants.h"
#include "core/data/control/VehicleControl.h"
#include "core/data/environment/VehicleData.h"

// Data structures from msv-data library:
#include "SteeringData.h"
#include "SensorBoardData.h"
#include "UserButtonData.h"

#include "Driver.h"

namespace msv {

        using namespace std;
        using namespace core::base;
        using namespace core::data;
        using namespace core::data::control;
        using namespace core::data::environment;

        Driver::Driver(const int32_t &argc, char **argv) :
	        ConferenceClientModule(argc, argv, "Driver") {
        }

        Driver::~Driver() {}

        void Driver::setUp() {
	        // This method will be call automatically before running body().
        }

        void Driver::tearDown() {
	        // This method will be call automatically after return from body().
        }

        // This method will do the main data processing job.
        ModuleState::MODULE_EXITCODE Driver::body() {

        int minRightSideDist1 = 5;
       
	//  int leftSteerAngle = -20;
	//  int rightSteerAngle = 19;
      int rightSteerAngle =90;
	 int loopCounter = 0;
	 int loopCounter2 = 0;
     int maxLoopCount = 33.5;
     int maxLoopCount2 = 8.3;
     int maxLoopCount3 = 14;
     int maxLoopCount4 = 35;
     int maxLoopCount5 = 20;
   //  int Totaltime = 0;
     bool turnright = false;   
     bool turnleft = false;
     bool goingstraightinsidelot = false;
     bool goingstraight = false;
     bool gap_scanner = false;
     bool parkingstate_right = false;
     bool parkingstate_left = false;
     



	        while (getModuleState() == ModuleState::RUNNING) {
                // In the following, you find example for the various data sources that are available:

		        // 1. Get most recent vehicle data:
		        Container containerVehicleData = getKeyValueDataStore().get(Container::VEHICLEDATA);
		        VehicleData vd = containerVehicleData.getData<VehicleData> ();
		        cerr << "Most recent vehicle data: '" << vd.toString() << "'" << endl;

		        // 2. Get most recent sensor board data:
		        Container containerSensorBoardData = getKeyValueDataStore().get(Container::USER_DATA_0);
		        SensorBoardData sbd = containerSensorBoardData.getData<SensorBoardData> ();
		        cerr << "Most recent sensor board data: '" << sbd.toString() << "'" << endl;

		        // 3. Get most recent user button data:
		        Container containerUserButtonData = getKeyValueDataStore().get(Container::USER_BUTTON);
		        UserButtonData ubd = containerUserButtonData.getData<UserButtonData> ();
		        cerr << "Most recent user button data: '" << ubd.toString() << "'" << endl;

		        // 4. Get most recent steering data as fill from lanedetector for example:
		        Container containerSteeringData = getKeyValueDataStore().get(Container::USER_DATA_1);
		        SteeringData sd = containerSteeringData.getData<SteeringData> ();
		        cerr << "Most recent steering data: '" << sd.toString() << "'" << endl;

// Create vehicle control data.
		        VehicleControl vc;


                // Design your control algorithm here depending on the input data from above.

        if (turnright == false
          && turnleft == false
          && goingstraightinsidelot == false
          && gap_scanner == false
          && parkingstate_right == false
          && parkingstate_left == false){
             goingstraight = true;
}
   // initial starting of the car 
             if (goingstraight == true) {
            
             vc.setSteeringWheelAngle(0);
             vc.setSpeed(1);
             loopCounter++;
             cerr << "LoopCounter: " << loopCounter << endl;
             if (loopCounter > maxLoopCount) {
             goingstraight = false;
             turnright = true;
	         loopCounter = 0;
			    
			  }
         }
 // car going inside parking boundary but not parking lot
                if (turnright == true) {
          
             vc.setSteeringWheelAngle(rightSteerAngle * Constants::DEG2RAD);  
             loopCounter++;
             cerr << "LoopCounter: " << loopCounter << endl;
            if (loopCounter > maxLoopCount2) {
                turnright = false;
			    goingstraightinsidelot = true;
                loopCounter = 0;
			    vc.setSteeringWheelAngle(0);
			  }
         }

// if statement which makes the car travel a bit further inside the parking lot for ignoring the first two gaps of the parking lot
        
               if (goingstraightinsidelot == true) {
            
             vc.setSteeringWheelAngle(0);
             vc.setSpeed(0.5);
             loopCounter++;
             cerr << "LoopCounter: " << loopCounter << endl;
             if (loopCounter > maxLoopCount3) {
             goingstraightinsidelot = false;
       	    gap_scanner = true;
             loopCounter = 0;
			    
			  }
         }
  
// car starts to scan for a gap to park after ignoring the first two gaps on the left and one on the right

             if (gap_scanner == true){             
             vc.setSteeringWheelAngle(0);
             vc.setSpeed(0.3);
             loopCounter++;
           
// right front sensor check for no gap
             if ((int) sbd.getDistance(4) < minRightSideDist1){
             loopCounter2++;
             cerr << "LoopCounter: " << loopCounter << endl;
             cerr << "LoopCounter2: " << loopCounter2 << endl;
            
// left front sensor check for no gap             
             if ((int) sbd.getDistance(5) < minLeftSideDist1  {             
             loopCounter3++;
             cerr << "LoopCounter3: " << loopCounter3 << endl;
             
// right front sensor check to initialize parking state if gap is found on right side

             if ((int) sbd.getDistance(4) > minRightSideDist1  {             
             gap_scanner = false;
             parkingstate_right = true; 

// right front sensor check to initialize parking state if gap is found on left side

   
            if ((int) sbd.getDistance(5) > minLeftSideDist1  {             
            gap_scanner = false;
            parkingstate_left = true; 

            if (loopCounter > maxLoopCount4) {
            gap_scanner = false;
            loopCounter = 0;
            if (loopCounter2 > maxLoopCount5){
            loopCounter2 =0;
}
} 
      }
}
}
             if (parkstate_right == true){
             if (parkstate_left == true) { 
//                        need    to write the car movements inside these loops for the car to park

//  PLEASE READ THIS
// if the parking state is true the car will go few loops forward to the left or the right and 
  // then start to go back to the parking lot with a negative velocity. the infrared will check whether the car will 
// hit the side of the parking or the back boundary on the back side of the parking








  
            
  
                // With setSpeed you can set a desired speed for the vehicle in the range of -2.0 (backwards) .. 0 (stop) .. +2.0 (forwards)
		        vc.setSpeed(1);

                // With setSteeringWheelAngle, you can steer in the range of -26 (left) .. 0 (straight) .. +25 (right)
         //       double desiredSteeringWheelAngle = 0; // 4 degree but SteeringWheelAngle expects the angle in radians!
		   //     vc.setSteeringWheelAngle(desiredSteeringWheelAngle * Constants::DEG2RAD);

                // You can also turn on or off various lights:
                vc.setBrakeLights(false);
                vc.setLeftFlashingLights(false);
                vc.setRightFlashingLights(true);
