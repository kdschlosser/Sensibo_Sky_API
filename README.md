
***pySensibo_Sky***
===================

Requires the requests library to be installed.

If you receive an SSL error this is because you do not have a suitable
certificate to connect with the server. You need to place a pem
container with valid certificates into the requests library root
directory. The file must be named cacerts.pem


*Connection:*
_____________

    import pySensibo_Sky

    API_KEY = "Your API key here"
    client = pySensiby_Sky.Client(API_KEY)


*Device Selection:*
___________________

 Devices are singleton class objects and there will only be a single
 instance produced for each api key that is used.

    DEVICE_NAME = "Your device name here" # Pod name

    device = client.get_device(DEVICE_NAME)

    if device is None:
        raise AttributeError(
            'No device by the name {0} found.'.format(DEVICE_NAME)
        )

*Listing Devices:*
__________________

    for name in client.device_names:
        print name

*Operating Modes:*
__________________

 There are several operating modes that a unit can be set to. To obtain
 the modes that your model supports is shown in the following code.

    modes = dict()
    for mode in device.supported_modes:
        print mode.name
        modes[mode.name] = mode


 The modes are singleton classes instances just like the devices are.
 So we are able to store them for future use.

 To set the mode we are able to do this one of 2 ways.

 Using the *`Mode`* instance.

     mode = modes['heat']
     mode.activate()

 Setting the *`Mode`* instance.

     device.mode = mode

 By the name of the mode.

     device.mode = 'heat'

 The only issue with using the name of the mode is unless you have
 verified that your unit supports the mode it can raise a `ValueError`.
 That is why it is best to use the supported modes to gather the modes
 your unit supports.

 To obtain which mode is active.

     mode = device.mode

*Room Temperature:*
___________________

    print '{0:.2f}'.format(device.temp)

*Room Humidity:*
________________

    print '{0:.2f}'.format(device.humidity)

*Device Power:*
_______________

Getting the device power.

    power = 'ON' if device.power else 'OFF'
    print power

Turning on the device.

    device.power = True

Turning off the device.

    device.power = False

*Device Firmware Version:*
__________________________

    print device.firmware_version

*Device Model Number:*
__________________________

    print device.model

*Device Battery Voltage:*
__________________________

If your device supports the use of a battery you are able to obtain the
voltage of the battery. Knowing the battery voltage you can run some
calculations that will ley you know how long the battery will last. I do
not know the voltage in which the unit will shutdown and if someone
would provide me this information i would be able to set up an estimated
runtime remaining.

If your unit does not support the battery an `AttributeError` will be
raised.

    print device.battery_voltage



*Temperature Set Point:*
________________________

To get the set point or to set it is done through the *`Mode`* instance.

    mode = device.mode

    print mode.temp

When setting the setpoint the value supplied has to be an *`int`* and
it also needs to be within the range of supported temperatures. The
library will automatically check to verify if it is and raise a
`ValueError` if it is not.

    mode.temp = 74

You are able to retrieve a *`list`* of supported temperatures for a
graphical control, simple printout, or for any other use by using the
following code. The returned list of supported temperatures refelects
the unit of measure the mode is set to.

    for temp in mode.supported_temps:
        print temp

To obtain the measurement scale that is used.

    print mode.temp_unit

If you want to change the scale.

    mode.temp_unit = 'F'

To list the supported scales.

    for scale in mode.supported_temp_units:
        print scale

*Air Handler Swing:*

On some units there is a motorized air direction control. In the units
that have these you are able to change the output air direction.

    print mode.swing

To get a list of supported directions.

    for swing in mode.supported_swing_modes:
        print swing

To set the swing mode

    mode.swing = 'stopped'

*Fan Speed:*

Get the fan speed.

    print mode.fan_level

List supported fan speeds.

    for speed in mode.supported_fan_levels:
        print speed

Set the fan speed.

    mode.fan_level = 'auto'


*Device Change Events:*
_______________________

Now we get to the really cool stuff. I have built in a callback and
polling system that will notify you if anything has changed on the
device. Whether it be the room temperature, or someone manually changes
the swing control from the unit it's self.

The way the event system has been built is you can register for events
in multiple places. If you register for an event using a *`Pod`* instance
you will only receive events for that specific device. If you register
for events using a *`Mode`* instance you will only gets events for that
mode for the device the mode belongs to. You also have the ability of
registering for events from the client instance and the events that you
register for here will be global and for all devices and modes.

You are only able to register for specific events depending on where
you register.

If you register with a mode you can register for the following events.
  * `'swing'`
  * `'temp'`
  * `'fan_level'`
  * `'temp_unit'`

If you register with the device.
  * `'mode'`
  * `'power'`
  * `'temp'`
  * `'humidity'`
  * `'battery_voltage'`
  * `'swing'`
  * `'temp'`
  * `'fan_level'`
  * `'temp_unit'`

If you register with the client.
  * `'swing'`
  * `'temp'`
  * `'fan_level'`
  * `'temp_unit'`
  * `'mode'`
  * `'power'`
  * `'temp'`
  * `'humidity'`
  * `'battery_voltage'`

Just remember that if you register with the client it registers with all
devices. and if you register with the device it registers with all of
the modes.

I also added a quick method to register for all events that come from
all devices, all events for a specific device, and all events for a
specific mode on a device. You can pass a `'*'` instead of an event name.

In order to register for an event you need to create a callback
function/method. when an event occurs and a callback is made there are
3 bits of information that are passed along. Those are the event name,
the new value and the object that holds the new value (device or mode).

    def callback(event, value, event_object):
        print event, '=', value
        print event_object.name

    # room temp for a specific device
    room_temp_event = device.bind('room_temp', callback)

    # temp set point for all modes
    device_temp_event = device.bind('temp', callback)

    # temp setpoint for the heat mode.
    heat_temp_event = device.heat.bind('temp', callback)

    # temp for all devices on all modes.
    client_temp_event = client.bind('temp', callback)


Now simply because we have bound to receive events does not mean we are
going to get them. You need to poll for changes. I places the ability to
start polling with the devices. I did this because there is no need to
poll a device if you do not want to monitor that device. Plus each time
you start the poling for a different device it creates a new thread.
This will reduce latency by having multiple blocks of code run for each
device at the ame time. You are also able to supply different interval
values (how fast to check to see if anything has changed) for each device.

The interval is passed as a float and a value of 1.0 is one second.

    # check for changes 10 times a second.
    device.start_poll(0.1)

If you wish to unbind from an event you are only able to unbind using
the event identifier that was returned when you called bind. unbind is
only located in the client instance, all events have to be unbound from
there.

You can also stop a polling loop by calling stop_poll in the instance
of a device.
























