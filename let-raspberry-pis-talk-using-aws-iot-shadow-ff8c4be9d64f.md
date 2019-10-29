
# Let Raspberry Pi’s talk using AWS IoT Shadow



During Christmas break I was planning to implement a smart door bell. A Raspberry Pi in the porch with a camera attached to it. Pi will capture the image of the person, recognize the face and if face matches, it will signal another Pi inside the house which will unlock the door. Sounds very cool right? But immediately I realized I need a way to communicate between two Raspberry Pis which are not physically connected.

I did some hobby projects with AWS IoT and thought that would be a very good candidate to act as a message hub (basically MQTT hub). The Pi at porch will send a signal by updating to AWS IoT Thing shadow. The other Pi inside house will listen for any delta changes in that Thing shadow. When there is a change and the command is to open the door, it will turn the door lock or the latch.

At the time of writing this blog, I have not finished that project but I established the communication between two Raspberry Pis. I attached a switch to one and a LED to another. Pressing the switch on A will turn the LED on on another Pi.

I thought this might be very helpful if want to do something similar and I created a video tutorial on this.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/MCXXoyV_j4w" frameborder="0" allowfullscreen></iframe></center>

For the code and other instructions please visit my blog here [http://crazykoder.com/2019/01/13/let-raspberry-pi…g-aws-iot-shadow/](http://crazykoder.com/2019/01/13/let-raspberry-pis-talk-using-aws-iot-shadow/)
