info 




3.99.180.221

jenkins  8080
a0d4c6ec56184e9b9482cf92b76449bd
Swati
Swati@29

sonaraube  9000
tush


prometheus  9090



Grafan pass  3000
admin
tush


app password  yghd wsty gdam ekuy

 token     squ_51bbf2421946d52beee2b31ff1c1a93e33358aa5








 Step 1: Update Package Index
First, ensure your package index is up-to-date by running the following command:

sh
sudo apt update
Step 2: Install JDK
Install the JDK package using the apt package manager. For JDK 17, you can run:

sh
sudo apt install openjdk-17-jdk
Step 3: Verify Installation
Verify that the installation was successful by checking the JDK version:

sh
java -version
You should see output indicating that JDK 17 is installed.

Step 4: Set JAVA_HOME Environment Variable
To set the JAVA_HOME environment variable, follow these steps:

Open the /etc/environment file in a text editor (e.g., nano):

sh
sudo nano /etc/environment
Add the following line to the file, replacing /usr/lib/jvm/java-17-openjdk-amd64 with the path to your JDK installation if it's different:

sh
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
Save the file and exit the editor (in nano, press Ctrl+X, then Y, and Enter).

Load the new environment variable:

sh
source /etc/environment
Step 5: Verify JAVA_HOME
Check if the JAVA_HOME environment variable is set correctly:

sh
echo $JAVA_HOME