In order to create the following guide i used info from various sources like the [documentation page](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
or [this](http://www.learningaboutelectronics.com/Articles/How-to-create-a-gunicorn-service-file-in-linux.php) nice tutorial. 

## Why we need systemd? 
When we create our API we want to be sure that gunicorn (or uvicorn) will run all the time, even when the computer will restart, even after a crash.
### Useful commands:
* `systemctl --type=service`
* `systemctl --type=service --state=active`
* `systemctl --type=service --state=running`

If we want to check often if a service is running, we can add an alias(=shortcut) command (e.g. `myRunningServices`) like this:

`vim ~/.bashrc`

and add the following line and save:

`alias myRunningServices='systemctl --type=service --state=running'`

to immediately activate it (run once):

`source ~/.bashrc`

Now, you whenever you open a new terminal you can just run:

`myRunningServices`

Check in which port the systemd service (that you will create later) is listening:

`ss -ltup | grep my-gunicorn-fastapi`

( `|` takes the ss -ltup results and give them to `grep`. `grep` will filter all the lines and give you the one with the `my-gunicorn-fastapi` keyword)

## Systemd service file

Create your systemd service file in the correct services folder:

`sudo nano /etc/systemd/system/my-gunicorn-fastapi.service`

and paste this:
```
[Unit]
# the description is what you see when you run `systemctl --type=service --state=runnning | grep fastapi`
Description=Start gunicorn uvicorn fastapi at port 80
# with the following line the service will wait for the network initialisation in order to start
After=network.target

[Service]
# we do NOT use docker, but only a python environment. With the following command you enable the environment. Be careful to have your 80 port free or change it to another
ExecStart=/myFastApiMainFolder/myPythonEnvironmentFolder/bin/gunicorn --chdir /myFastApiMainFolder main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:80

# choose what you want to do if the service fails. See the other options from the [documentation]https://www.freedesktop.org/software/systemd/man/systemd.service.html). I've choose on-failure.
Restart=on-failure 

[Install]
# the service will start after the user logs in (before GUI)
WantedBy=multi-user.agent 
```

Save the file (control+O, control+C)


## Enable the service

We enable the service with systemctl (the utility that checks systemd stuff):
`sudo systemctl enable my-gunicorn-fastapi.service`
to run it now:
`sudo systemctl start my-gunicorn-fastapi.service`

if you edit the my-gunicorn-fastapi.service you should run:
`sudo systemctl daemon-reload`
and then:
`sudo systemctl restart my-gunicorn-fastapi.service`

`daemon-reaload` alone will not activate the changes (for example if you change uvicorn port). You should do the `systemctl restart` also.

you can also stop the service, and check if it will be re-enabled automatically after reboot:

`sudo systemctl stop my-gunicorn-fastapi.service`

you can check its status anytime with:

`sudo systemctl status my-gunicorn-fastapi`
