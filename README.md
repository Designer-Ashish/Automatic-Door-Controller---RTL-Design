# Automatic_Door_Controller-RTL_Design
# Specifications
The automatic door controller should:

1. Detect approaching persons using motion sensors

2. Open the door when a person is detected

3. Keep the door open as long as movement continues to be detected

4. Close the door after a timeout period when no more movement is detected

5. Include safety features to prevent closing when obstructed


                    +-------------------+
                    |  Motion Sensor    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Sensor Interface |
                    +---------+---------+
                              |
                    +---------v---------+
                    |   Control Logic   |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Timer/Counter    |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Door Motor Driver|
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Obstruction      |
                    |  Sensor          |
                    +-------------------+
