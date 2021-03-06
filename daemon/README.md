# Installing as a Service #
source: https://github.com/torfsen/python-systemd-tutorial

## Pre-reqs ##
Need python-systemd package
* `sudo apt-get install python3-systemd`
* or `pip install systemd-python`

Need to create a config file `/etc/mpp-solar/mpp-solar.conf`
* Use the example `/etc/mpp-solar/mpp-solar.conf.example` as a start

[see here for examples](../docs/configfile.md#Config-file-examples)

## Add mpp-solar service ##

* Check the service exists
`systemctl --user list-unit-files|grep mpp-solar`
```
mpp-solar.service              disabled
```
* Start the service
`systemctl --user start mpp-solar`

* Check service status
`systemctl --user status mpp-solar`
```
mpp-solar.service - MPP Solar Service
   Loaded: loaded (/etc/systemd/user/mpp-solar.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-04-08 16:19:46 NZST; 10s ago
 Main PID: 21724 (python)
   CGroup: /user.slice/user-1000.slice/user@1000.service/mpp-solar.service
           └─21724 /usr/bin/python /usr/local/bin/mpp-solar-service -c /etc/mpp-solar/mpp-solar.conf

Apr 08 16:19:46 batteryshed python[21724]: MPP-Solar-Service: Config file: /etc/mpp-solar/mpp-solar.conf
Apr 08 16:19:46 batteryshed python[21724]: MPP-Solar-Service: Config setting - pause: 1
Apr 08 16:19:46 batteryshed python[21724]: MPP-Solar-Service: Config setting - mqtt_broker: mqtthost
Apr 08 16:19:46 batteryshed python[21724]: MPP-Solar-Service: Config setting - command sections found: 2
...
```

* Stopping the service
`systemctl --user stop mpp-solar`

* Restart the service - for example after a config file change
`systemctl --user restart mpp-solar`

Logs and service output
* The output should show up in systemd's logs, which by default are redirected to syslog:
`grep 'MPP-Solar-Service' /var/log/syslog`
```
Apr  8 16:23:27 batteryshed python[21724]: MPP-Solar-Service: item {'tag': u'QPGS0', 'command': u'QPGS0', 'mp': <mppsolar.mpputils.mppUtils instance at 0x75d61c88>, 'format': u'influx2'}
Apr  8 16:23:30 batteryshed python[21724]: MPP-Solar-Service: item {'tag': u'QPGS1', 'command': u'QPGS1', 'mp': <mppsolar.mpputils.mppUtils instance at 0x75d61df0>, 'format': u'influx2'}
Apr  8 16:23:32 batteryshed python[21724]: MPP-Solar-Service: sleeping for 1sec
```

* Another way to display the service's output is via
`journalctl --user-unit mpp-solar`

## Automatically Starting the Service during Boot ##
```
systemctl --user enable mpp-solar
sudo loginctl enable-linger $USER
```

* To disable autostart, simply disable your service:
`systemctl --user disable mpp-solar`

Note that simply enabling a service does not start it, but only activates autostart during boot-up. Similarly, disabling a service doesn't stop it, but only deactivates autostart during boot-up. If you want to start/stop the service immediately then you still need to do that manually

To check whether your service is enabled, use

`systemctl --user list-unit-files | grep mpp-solar`
```
mpp-solar.service              enabled
```

# Creating a System Service #
Once you have a working user service you can turn it into a system service. Remember, however, that system services run in the system's central systemd instance and have a greater potential for disturbing your system's stability or security when not implemented correctly. In many cases, this step isn't really necessary and a user service will do just fine.

## Stopping and Disabling the User Service ##
Before turning our service into a system service let's make sure that its stopped and disabled. Otherwise we might end up with both a user service and a system service.
```
$ systemctl --user stop mpp-solar
$ systemctl --user disable mpp-solar
```
## Moving the Unit File ##
/etc/systemd/user/mpp-solar.service
Previously, we stored our unit file in a directory appropriate for user services (`/etc/systemd/user/mpp-solar.service`). As with user unit files, systemd looks into more than one directory for system unit files. We'll be using `/etc/systemd/user/mpp-solar.service`, so move your unit file there and make sure that it has the right permissions
```
$ sudo mv /etc/systemd/user/mpp-solar.service /etc/systemd/system/
$ sudo chown root:root /etc/systemd/system/mpp-solar.service
$ sudo chmod 644 /etc/systemd/system/mpp-solar.service
```
Our service is now a system service! This also means that instead of using `systemctl --user ...` we will now use `systemctl ...` (without the `--user` option) instead (or `sudo systemctl ...` if we're modifying something). For example:
```
$ systemctl list-unit-files | grep mpp-solar
python_demo_service.service                disabled
```
Similarly, use `journalctl --unit mpp-solar` to display the system service's logs.


## troubleshooting ##
```
Traceback (most recent call last):
  File "/home/pi/venv/mppsolar/bin/mpp-solar", line 11, in <module>
    load_entry_point('mpp-solar', 'console_scripts', 'mpp-solar')()
  File "/home/pi/venv/mppsolar/src/mpp-solar/mppsolar/__init__.py", line 211, in main
    systemd.daemon.notify("READY=1")
  File "/home/pi/venv/mppsolar/lib/python3.7/site-packages/systemd/daemon.py", line 39, in notify
    raise TypeError("state must be an instance of Notification")
TypeError: state must be an instance of Notification
```
have systemd instead of systemd-python
$ pip uninstall systemd
$ pip install systemd-python
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Collecting systemd-python
  Downloading https://www.piwheels.org/simple/systemd-python/systemd_python-234-cp37-cp37m-linux_armv7l.whl (179kB)
    100% |████████████████████████████████| 184kB 51kB/s
Installing collected packages: systemd-python
Successfully installed systemd-python-234
