# raspberry pi based security system
(end goal, face and number plate recog, sends image to user)

starting from:
https://github.com/HackerHouseYT/Smart-Security-Camera

using pyimagesearch 20170904 guide to get openCV setup on a pi 2b running stretch, using python 3.
https://www.pyimagesearch.com/2017/09/04/raspbian-stretch-install-opencv-3-python-on-your-raspberry-pi/

# opencv dependencies
check all filesystem space has been used (is by default, but double check)
> df -h
if /dev/root isn't the size of your card, then run > sudo raspi-config and expand the file system.

### to save a gig of space
$ sudo apt-get purge wolfram-engine
$ sudo apt-get purge libreoffice*
$ sudo apt-get clean
$ sudo apt-get autoremove

$ sudo apt-get update && sudo apt-get upgrade

$ sudo apt-get install build-essential cmake pkg-config

$ sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev

$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
$ sudo apt-get install libxvidcore-dev libx264-dev

$ sudo apt-get install libgtk2.0-dev libgtk-3-dev

$ sudo apt-get install libatlas-base-dev gfortran

$ sudo apt-get install python2.7-dev python3-dev

# download opencv
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git

# setup Python
$ wget https://bootstrap.pypa.io/get-pip.py
$ sudo python get-pip.py
$ sudo python3 get-pip.py
## with virtual environments
$ sudo pip install virtualenv virtualenvwrapper
$ sudo rm -rf ~/.cache/pip
## update ~/.profile
$ echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.profile
$ echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.profile
$ echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile
## load the update profile
$ source ~/.profile

$ mkvirtualenv cv -p python3
## to get into the cv virtual environment
$ source ~/.profile
$ workon cv

(next command provides no progress indication and can take a minute...)
$ pip install numpy

# compile and install opencv
$ workon cv

$ cd ~/opencv-3.3.0/
$ mkdir build
$ cd build
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.3.0/modules \
    -D BUILD_EXAMPLES=ON ..

"this didn't work first time, but turned the pi off and when I got back to this a week later it did work, so maybe a reboot, or terminal restart did the trick" - redtahr

to verify CMake output, check: https://www.pyimagesearch.com/wp-content/uploads/2017/08/python3_opencv-768x511.png
key points, the Python 2 or 3 (depending on your choices earlier) sections, Interpreter and numpy should point to the .../virtualenvs/cv/...

## configure swap space before compiling
/etc/dphys-swapfile  and then edit the CONF_SWAPSIZE=100 variable:
CONF_SWAPSIZE=1024
this gets all four cores on the pi running (and may thrash your sd card, so change it back when you're done)
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo /etc/init.d/dphys-swapfile start

# build time
make -j4
...sometime later...on a pi 3 ~90 minutes, on a pi 2b ~140 minutes, on a pi zero 8 hours

# install time
sudo make install
sudo ldconfig

# finish install openCV on your pi
(I've pasted the entire section from pyimagesearch as there's something buggy about building of python 3)
## For Python 3:
After running make install , your OpenCV + Python bindings should be installed in /usr/local/lib/python3.5/site-packages . Again, you can verify this with the ls command:

>$ ls -l /usr/local/lib/python3.5/site-packages/
>total 1852
>-rw-r--r-- 1 root staff 1895932 Mar 20 21:51 cv2.cpython-34m.so

I honestly don’t know why, perhaps it’s a bug in the CMake script, but when compiling OpenCV 3 bindings for Python 3+, the output .so file is named cv2.cpython-35m-arm-linux-gnueabihf.so (or some variant of) rather than simply cv2.so (like in the Python 2.7 bindings).

Again, I’m not sure exactly why this happens, but it’s an easy fix. All we need to do is rename the file:

>$ cd /usr/local/lib/python3.5/site-packages/
>$ sudo mv cv2.cpython-35m-arm-linux-gnueabihf.so cv2.so

After renaming to cv2.so , we can sym-link our OpenCV bindings into the cv virtual environment for Python 3.5:

>$ cd ~/.virtualenvs/cv/lib/python3.5/site-packages/
>$ ln -s /usr/local/lib/python3.5/site-packages/cv2.so cv2.so

#7: Testing your OpenCV 3 install
Open up a new terminal, execute the source and workon commands, and then finally attempt to import the Python + OpenCV bindings:
$ source ~/.profile 
$ workon cv
$ python
>>> import cv2
>>> cv2.__version__
'3.3.0'
>>>

# tidying up
### remove the git clones of the code if you wish.
rm -rf opencv
rm -rf opencv_contrib

### change the swapfile size back
/etc/dphys-swapfile CONF_SWAPFILE=100
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo /etc/init.d/dphys-swapfile start


# onward to hackerhouse smart-security-camera
https://github.com/HackerHouseYT/Smart-Security-Camera

>source ~/.profile
>workon cv

Next, navigate to the repository directory

>cd Smart-Security-Camera
and install the dependencies for the project
>pip install -r requirements.txt
Note: If you're running python3, you'll have to change the import statements at the top of the mail.py file
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
and change your print statements from quotes to parenthesis
print "" => print()

##Customization
To get emails when objects are detected, you'll need to make a couple modifications to the mail.py file.
### Email you want to send the update from (only works with gmail)
fromEmail = 'myemail@gmail.com'
fromEmailPassword = 'password1234'
### Email you want to send the update to
toEmail = 'anotheremail@gmail.com'
and replace with your own email/credentials. The mail.py file logs into a gmail SMTP server and sends an email with an image of the object detected by the security camera.

### You can also modify the main.py file to change some other properties.
email_update_interval = 600 # sends an email only once in this time interval
video_camera = VideoCamera(flip=True) # creates a camera object, flip vertically
object_classifier = cv2.CascadeClassifier("models/fullbody_recognition_model.xml") # an opencv classifier
Notably, you can use a different object detector by changing the path "models/fullbody_recognition_model.xml" in object_classifier = cv2.CascadeClassifier("models/fullbody_recognition_model.xml").

to a new model in the models directory.

facial_recognition_model.xml
fullbody_recognition_model.xml
upperbody_recognition_model.xml

## Run the program

python main.py

You can view a live stream by visiting the ip address of your pi in a browser on the same network. You can find the ip address of your Raspberry Pi by typing ifconfig in the terminal and looking for the inet address.

Visit <raspberrypi_ip>:5000 in your browser to view the stream.

Note: To view the live stream on a different network than your Raspberry Pi, you can use ngrok to expose a local tunnel. Once downloaded, run ngrok with ./ngrok http 5000 and visit one of the generated links in your browser.

Note: The video stream will not start automatically on startup. To start the video stream automatically, you will need to run the program from your /etc/rc.local file see this video for more information about how to configure that.