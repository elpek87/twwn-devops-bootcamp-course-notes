![Logo](assets/devops-bootcamp-logo.png)

## CLOUD AND INFRASTRUCTURE AS A SERVICE BASICS

Add user to run the app:

`# adduser ubuntu`

If you need to allow the user to use sudo then it should be added to sudo group:

`# usermod -aG sudo ubuntu`

Add ssh public key to the new user's authorized_keys file.

Clone the repository:

`$ git clone https://gitlab.com/twn-devops-bootcamp/latest/05-cloud/java-react-example`

In project directory run:

`$ gradle build`

To run Java application artifact on the remote server first copy the artifact to the remote server:

`$ scp build/libs/java-react-example.jar root@ubuntu-vm:/root/`

On the remote server install java - in this case JRE is enough:

`apt install openjdk-17-jre-headless`

Run the application on the server:

`java -jar java-react-example.jar`

Once the app is up it should listen on port 7071 of the remote server - you can check if it does using ss:

`ss -ntulp | grep java`
`tcp   LISTEN 0      100                                     *:7071            *:*    users:(("java",pid=2351,fd=9))`

The web application should then be accessible in web browser under http://server_ip:7071 .

If firewall was used then port 7071 should be open before accessing the app.
