To compile the app:

- `brew install scala`
- `brew install sbt`
- `brew install typesafe-activator`
- `activator new app play-scala` (or just clone the repo)
- `cd app`
- `activator dist`
- `cp target/universal the-archive.zip`

First, some setup on the AWS console 

- AWS --> EC2 --> New Instance
- Choose the default Ubuntu 16.04 AMI --> Next
- You can select any instance size, but t2.nano is sufficient for our needs and the cheapest --> Next
- Stick with the default settings until step 6
- To configure the security group, add a new rule: Custom TCP Rules, Port 8080, Source Anywhere --> Next
- Keep clicking next/finish until you get the popup --> generate a new keypair and download it

Now we need to SSH into the instance 

- Get the public DNS name from the list of instances AWS should take you to
- Run `chmod 400 ~/.path/to/keyfile.pem`
- Run `ssh -i ~/path/to/keyfile.pem ubuntu@<public_ip_of_server>`

Some initial instance set up

- `sudo apt-get update`
- `sudo apt-get upgrade`
- `sudo apt-get install default-jdk unzip`
- Make sure you can run `java -version`

Create a new user for the app to run as.

- `sudo adduser --system myapp`
- `sudo passwd myapp` --> create a password for the user

Back on our local machine, in a separate window, we need to transfer our built Play app up to the server.

- Get a copy of the built app archive, or build one yourself 
- Locally, run `scp -i ~/path/to/keyfile.pem the/build/file.zip ubuntu@<public_ip_of_server>:` 
- Don't forget the colon at the end

The code is on the server; we just need to extract it and transfer ownership to the user we created 

- `sudo su` to start a root shell
- `cp the-archive.zip /home/myapp`
- `cd /home/myapp`
- `unzip the-archive.zip`
- `chown -R myapp the-archive`

Now, we can start the app to test it out.

- `sudo -u myapp /home/myapp/the-archive/bin/app -Dplay.crypto.secret="secret" -Dhttp.port="8080"`
- "sudo -u" will switch our user to that user when executing the app 
- the "crypto secret" part isn't that important
- the last argument specifies the port to listen on. 
- Visit http://<public-ip>:8080 and you should see a start page 

But we'd have to keep the terminal open all the time to keep it running like this. 
So lets have it launch in the background using a systemd unit.

- `sudo su`
- `vim /etc/systemd/system/myapp.service` --> Add the following text 

```
[Unit]
Description=my java app

[Service]
ExecStart=/home/myapp/the-archive/bin/app -Dplay.crypto.secret="secret" -Dhttp.port="8080"
User=myapp
```

- `systemctl list-unit-files myapp.service` should now list your service as being available
- `systemctl start myapp.service` to start the app 
- `systemctl list-units myapp.service` should list 1 running unit 
- `journalctl -u myapp.service` should spit out a bunch of logs similar to what we saw earlier
- If you visit the same website as before, you should see your app

Finally, lets clean up after ourselves so we don't get charged too much by AWS.

- On the EC2 instance list, click Actions --> Instance State --> Terminate

