---
layout: post
title: Serial port details in C#
---

SerialPort class available in `System.IO.Ports` namespace is really simple. When we want to get all port names on the computer there is one method to use: `.GetPortNames()`. It returns an array with all serial ports names in format `COM*` where `*` is a number of the port.

# Problem
I was creating a debug software for my recent project and I needed to simply stream data from serial port to several controls on the screen. Pretty easy, just connect to the communication port, receive some data, parse it and send to controls. But I didn't want to select a right serial port name manually. Also, I wanted to link the connection automatically. How to filter the serial ports then?

# Solution 
Filtering can easily be done by VID and PID from USB device that is emulating the port. There are two ways of obtaining it. First one is a usage of an external library like [USBClassLibrary](http://www.codeproject.com/Articles/60579/A-USB-Library-to-Detect-USB-Devices). Second, the more customizable way is to create an own wrapper around `ManagementObjectSearcher` class.

# Customization

At the beginning, we need to specify a list of parameters required by our application. Full list of fields is available [here](https://msdn.microsoft.com/en-us/library/aa394413(v=vs.85).aspx). After that let's create a struct with specific variables:

{% highlight C# %}
    struct ComPort // custom struct with our desired values
    {
        public string name;
        public string vid;
        public string pid;
        public string description;
    }
{% endhighlight %}

In this example, we want only VID, PID, COM name, and description.

Next, create simple function for extracting all data:

{% highlight C# %}
    private const string vidPattern = @"VID_([0-9A-F]{4})";
    private const string pidPattern = @"PID_([0-9A-F]{4})";
    private List<ComPort> GetSerialPorts()
    {
        using (var searcher = new ManagementObjectSearcher
            ("SELECT * FROM WIN32_SerialPort"))
        {
            var ports = searcher.Get().Cast<ManagementBaseObject>().ToList();
            return ports.Select(p =>
            {
                ComPort c = new ComPort();
                c.name = p.GetPropertyValue("DeviceID").ToString();
                c.vid = p.GetPropertyValue("PNPDeviceID").ToString();
                c.description = p.GetPropertyValue("Caption").ToString();

                Match mVID = Regex.Match(c.vid, vidPattern, RegexOptions.IgnoreCase);
                Match mPID = Regex.Match(c.vid, pidPattern, RegexOptions.IgnoreCase);

                if (mVID.Success)
                    c.vid = mVID.Groups[1].Value;
                if (mPID.Success)
                    c.pid = mPID.Groups[1].Value;

                return c;

            }).ToList();
        }
    }
{% endhighlight %}

In the code above we firstly access `ManagementObjectSearcher` to obtain all serial ports in the system using query. Then we cast results to the desired type and create a list from it. At the end, we map the results by extracting needed information into a previously created struct. Regular expressions are used because VID and PID are stored in one field, which needs to be parsed.

Note, that adding two dependencies listed below is a necessity:

{% highlight C# %}
using System.Management;
using System.Text.RegularExpressions;
{% endhighlight %}

Usage of the function is really simple: `List<ComPort> ports = GetSerialPorts();` will return us a list of serial ports in the system with all the additional fields we needed.

Finally if we want to filter our results to devices with specified VID and PID there is an easy way to achieve that:


{% highlight C# %}
List<ComPort> ports = GetSerialPorts();
//if we want to find one device
ComPort com = ports.FindLast(c => c.vid.Equals("0483") && c.pid.Equals("5740"));
//or if we want to extract all devices with specified values:
List<ComPort> coms = ports.FindAll(c => c.vid.Equals("0483") && c.pid.Equals("5740"));
{% endhighlight %}

Example project using these methods is available on [GitHub](https://github.com/freakone/serial-reader).