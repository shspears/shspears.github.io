---
layout: "post"
title: "code_samples"
date: "2020-05-06 21:34"
---

### Code Samples
> A place to store useful routines, functions, etc, in C++ - mainly for the *nix OS
>
> Pretty small right now..

- Convert all characters in a string, to [lower case](#lover-case)
- [Split a string](#split_string) into a vector of characters (strings).
- Search a [directory](#getdir) and put all file names into a vector of strings.
- Monitor a directory for newly written and closed files using [inotify](#inotify)
- Calculate the [Great Circle Distance](#gccalc) between two waypoints.
- Calculate the [Bearing](#gcbearing) between two waypoints.
- Generate a [new waypoint](#generate-waypoint) from an existing waypoint, distance, and heading
- A example of using [gnuplot](#gnuplot) which requires [gnuplot-iostream.h](https://github.com/dstahlke/gnuplot-iostream)


#### General routines


##### Date and Time
```cpp
        #include <chrono>  // chrono::system_clock
        #include <ctime>   // localtime
        #include <sstream> // stringstream
        #include <iomanip> // put_time
        #include <string>  // string

        std::string return_current_time_and_date()
        {
            auto now = std::chrono::system_clock::now();
            auto in_time_t = std::chrono::system_clock::to_time_t(now);

            std::stringstream ss;
            ss << std::put_time(std::localtime(&in_time_t), "%Y-%m-%d %X");
            return ss.str();
        }
```


##### Lower Case
- convert a string to lower case

```cpp

        #include <algorithm>
        #include <cctype>
        #include <string>

        std::string data = "Abc";
        std::transform(data.begin(), data.end(), data.begin(),
            [](unsigned char c){ return std::tolower(c); });
```

##### Split_string
- a string into vector of strings (characters). GPS parsing, etc.

```cpp
    std::string nmeaStr = serial_.readline();
    //std::string nmeaStr{"This is a string I want to break into characters"};

    std::stringstream ss(nmeaStr);

    std::vector<std::string> nmeaCharacters;


    while( ss.good() )
    {
      std::string substr;
      std::getline( ss, substr, ',');
      nmeaCharacters.push_back( substr );
    }

```

##### getdir
- get list of files from a directory
```cpp
    #include <dirent.h>
    ...
    int getdir (string dir, vector<string> &files)
    {
        DIR *dp;
        struct dirent *dirp;
        if((dp  = opendir(dir.c_str())) == NULL) {
            cout << "Error(" << errno << ") opening " << dir << endl;
            return errno;
        }

        while ((dirp = readdir(dp)) != NULL) {
            files.push_back(string(dirp->d_name));
        }
        closedir(dp);
        return 0;
    }
```

##### inotify
- Monitor a directory and create an event when a file has been written and closed
- inotify is a linux event handler that can be configured to handle any file driven event within a directory (and/or subdirectories).
- It does not work over NFS - only on a local machine
- for more information see: [inotify man page](http://man7.org/linux/man-pages/man7/inotify.7.html)


> The following example monitors a specific directory (iNotifyDir) for new files to be written to and closed cout
> These file names are added to the vector of strings fileNames.
> If a new directory is created in the iNotifyDir, then that directory is also added to the monitor list, to check for new files that were written to and closed
```cpp

    #include <errno.h>
    #include <poll.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/inotify.h>
    #include <unistd.h>
    ...

    //variables to be used with inotify event
    int inotifyFd, wd, wDir;
    std::string iNotifyDir;
    char buf[BUF_LEN] __attribute__ ((aligned(8)));
    ssize_t numRead;
    char *p;
    struct inotify_event *event;

    std::vector<std::sting> fileNames; //vector to keep file names in for later processing
    ...

    //In main()
    inotifyFd = inotify_init();                 // Create inotify instance
    if (inotifyFd == -1)
        printf("inotify_init failed");

    //wd = inotify_add_watch(inotifyFd, sdfDir.c_str(), IN_CLOSE_WRITE | IN_CREATE); //Notify when a new file has been written to and closed
    wd = inotify_add_watch(inotifyFd, iNotifyDir.c_str(), IN_CREATE);
    if (inotifyFd == -1)
        printf("inotify_add_watch failed");

    ...

    //in a while loop to continously monitor the iNotifyDir
    numRead = read(inotifyFd, buf, BUF_LEN);
    for (p = buf; p < buf + numRead; )
    {
        event = (struct inotify_event *) p;
        if(event->mask & IN_CLOSE_WRITE)
        {

            cout << "File written and closed: " << event->name << endl;
            std::string newFile = string(event->name);
            std::string newFileExtension = newFile.substr(newFile.find_last_of(".") + 1);

            if(newFileExtension == "ext" || newFileExtension == "EXT")
            {
                fileNames.push_back(sdf);
            }
        }
        else if(event->mask & IN_CREATE)
        {
            std::string dirCreated(event->name);
            string fullDir = iNotifyDir + dirCreated;
            cout << "Directory Created " << dirCreated << endl;

            cout << "Full Path of new directory " << fullDir << endl;

            //Now we add a inotify event to the new directory incase we get new files in there!
           wDir = inotify_add_watch(inotifyFd, fullDir.c_str(), IN_CLOSE_WRITE);
           if (inotifyFd == -1)
                printf("inotify_add_watch failed");
        }

        //go through the entire struct for information we're interested in
        p += sizeof(struct inotify_event) + event->len;
    }

```
##### GCCalc
- Calculate the Great Circle Distance between to points on the earth.
- latitude and longitude are expected to be in degrees.degrees


```cpp

    #define R_EARTH 6378.1                        //Radius of the earth in kms
    #define PI 3.1415926535897932384626433832795
    #define DEG2RAD 0.01745329252               // pi/180
    #define RAD2DEG 57.29577951308              // 180/pi
...

    double GCCalc(double lat1, double lon1, double lat2, double lon2)

    {
        /*
        This function takes to GPS coordinates and calculates the Great Circle Distance between these two corodinates. It returns a double value that is in kms
        */

        /*
        Advisable to do some error checking (make sure lats and lons are <90 && >(-90))

        */


        double dlat1 = lat1*DEG2RAD;
        double dlat2 = lat2*DEG2RAD;
        double dlon1 = lon1*DEG2RAD;
        double dlon2 = lon2*DEG2RAD;

        double deltaLon = dlon1 - dlon2;
        double deltaLat = dlat1 - dlat2;

        double a = pow(sin(deltaLat / 2.0), 2.0) + cos(dlat1)*cos(dlat2)*pow(sin(deltaLon / 2), 2);
        double c = 2 * atan2(sqrt(a), sqrt(1.0 - a));

        double distance = R_EARTH * c;
        return distance;
    }

```
##### GCBearing
- Calculate the bearing between two waypoints
- Waypoint is a Latitude / Longitude pair, in degrees.degrees

```cpp
    #define R_EARTH 6378.1                        //Radius of the earth in kms
    #define PI 3.1415926535897932384626433832795
    #define DEG2RAD 0.01745329252               // pi/180
    #define RAD2DEG 57.29577951308              // 180/pi

    ...

    double GCBearing(double lat1, double lon1, double lat2, double lon2)
    {

        double x, y, bearing;

        x = cos(lon2) * sin(lon2 - lon1);
        y = (cos(lat1) * sin(lat2) - (sin(lat1) * cos(lat2) * cos(lon2 - lon1)));

        bearing = (atan2(x,y)*RAD2DEG);

        if(bearing < 0)
            return 360 + bearing;
        else
            return bearing;

    }
```
##### Generate waypoint
- Generate a waypoint (a lat lon pair, in degrees.degrees) from a single point, a distance from that point, and a heading

```cpp

    #define R_EARTH 6378.1                        //Radius of the earth in kms
    #define PI 3.1415926535897932384626433832795
    #define DEG2RAD 0.01745329252               // pi/180
    #define RAD2DEG 57.29577951308              // 180/pi

    ...

    typedef struct
    {
        double latitude;
        double longitude;
    }waypoint;

    ...

    waypoint genWP(waypoint wp, double distance, double heading )
    {

        //A waypoint contains a lat and a lon
        //contains

        double d = distance / 1000.0; //put this into kms

        //std::cout << "In the genWP function " << distance << std::endl;
        double dlat1 = wp.latitude*DEG2RAD;
        double dlon1 = wp.longitude*DEG2RAD;

        waypoint newWP;

        double lat2 = asin(sin(dlat1)*cos(d/R_EARTH) + cos(dlat1)*sin(d/R_EARTH)*cos(heading*DEG2RAD));

        newWP.latitude = lat2*RAD2DEG;

        double lon2 = dlon1 + atan2((sin(heading*DEG2RAD)*sin(d/R_EARTH)*cos(dlat1)), (cos(d/R_EARTH) - sin(dlat1)*sin(lat2)));

        newWP.longitude = lon2*RAD2DEG;

        return newWP;

    }


```
##### Gnuplot
- Just some messing arround with [gnuplot-iostream](https://github.com/dstahlke/gnuplot-iostream)
- more examples can be found on the link above
- Don't forget to have gnuplot installed

```
    sudo apt update
    sudo apt install gnuplot
```
```cpp

    //declare your plots
    Gnuplot gpPitch, gpRoll, gpLatLon;
    //plots need to be populated using a std::vector<std::pair<type, type>>
    std::vector<std::pair<double, int>> pitchData, rollData;
    std::vector<std::pair<double, double>> latlonData;

    //Using x from 0 to size -1 , but could use time
    for(int x = 0; x < iverLog.size() - 1; x++)
    {
        double y = iverLog[x].pitchAngle;
        pitchData.push_back(std::make_pair(y,x));
        double z = iverLog[x].rollAngle;
        rollData.push_back(std::make_pair(z,x));
        double lat = iverLog[x].latitude;
        double lon = iverLog[x].longitude;
        latlonData.push_back(std::make_pair(lon, lat));
    }

    //Plot Pitch, plot roll, plot lat/lon, and plot depth and altitude

    //gp << "reset\nset size 2,2\nset multiplot\nset size 1,2\n set origin 0,0\n";
    //plot xy graph
    gpPitch << "plot '-' with lines lc rgb 'black' title 'Pitch Data'\n";
    gpPitch.send1d(pitchData);
    //gp << "set origin 1,0\n";
    gpRoll << "plot '-' with lines lc rgb 'blue' title 'Roll Data'\n";
    gpRoll.send1d(rollData);
    gpLatLon << "plot '-' with lines lc rgb 'red' title 'AUV Track'\n";
    gpLatLon.send1d(latlonData);

    //getchar() is used to keep the program from terminating before seeing the plots
    getchar();

```



#### File IO
#### OpenCV

#### ROS2
