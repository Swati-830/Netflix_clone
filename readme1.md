info 

token    squ_762c77d59a5de3f60902c39eb5887cc79d150ab9







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