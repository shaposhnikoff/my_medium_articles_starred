Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m152[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m183[39m, end: [33m192[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m282[39m, end: [33m288[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m396[39m, end: [33m406[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m161[39m, end: [33m171[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m152[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m182[39m, end: [33m191[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m133[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m29[39m }

# Efficient IoT with the ESP8266, Protocol Buffers, Grafana, Go, and Kubernetes



At [KubeCon EU 2018](https://events.linuxfoundation.org/events/kubecon-cloudnativecon-europe-2018/), I had the opportunity to do a presentation on the use of [protocol buffers and gRPC for efficient IoT](https://www.youtube.com/watch?v=c9z_o5lu0dI). This post is based on one of the projects I discussed which uses an ESP8266, protocol buffers, a server program (written in Go), InfluxDB, and Grafana, to collect and visualize temperature data.

## About the project

This is a relatively simple project which uses the components as depicted in the following figure.

![Collect temperature data with the ESP8266, DHT11, protocol buffers, and Go](https://cdn-images-1.medium.com/max/2000/1*NoHhmXREYAwroc06ZMkSxA.png)*Collect temperature data with the ESP8266, DHT11, protocol buffers, and Go*

1. The ESP8266, an inexpensive WIFI-enabled project board, is used to collect data from a temperature/humidity sensor (the DHT11).

1. The ESP8266 is programmed using Arduino C to encode the collected data using as Protocol Buffers.

1. The data is serialized and sent to a Go backend server using a TCP socket over WIFI.

1. The server program saves the temperature entries as time series data to an InfluxDB table.

1. Grafana is used to visualize the temperature data, stored in InfluxDB.

### What you will need

For this project you will need the followings

* [ESP8266](http://esp8266.net/) (I used the [NodeMCU](https://www.amazon.com/pack-ESP8266-microcontroller-NodeMCU-CP2102/dp/B071NSSQTL))

* [Arduino core libraries](https://github.com/esp8266/Arduino#installing-with-boards-manager) for the ESP8266

* DHT11 (temp/humidity sensor or [similar](https://www.amazon.com/RobotDyn-Temperature-Humidity-projects-Raspberry/dp/B073S3SJXP))

* [Adafruit Arduino sensor libraries](https://github.com/adafruit/Adafruit_Sensor) (and for the [DHT11](https://github.com/adafruit/Adafruit_DHT_Unified))

* [Arduino IDE](https://www.arduino.cc/en/Main/Software)

* [Protocol buffers](https://developers.google.com/protocol-buffers/) compiler (protoc)

* [Nanopb](https://github.com/nanopb/nanopb) (C protoc plugin)

* [InfluxDB](https://www.influxdata.com/)

* [Grafana](https://grafana.com/)

* The [Go](https://golang.org/) programming language

* [Golang-protobuf](https://github.com/golang/protobuf) (Go protoc plugin)

* [Kubernetes](https://kubernetes.io/) (optional)
> This Medium writeup is a high-level summary of the steps required to create this project. You can find a more detail and complete set of instructions (including source code) at this [GitHub repository](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp).

## Protocol buffers for IoT

When creating IoT applications that runs on constrained devices such as the ESP8266, efficiently storing and serialization of data can impactful on battery and performance. It turns out that Protocol buffers is a great option for use as IoT data serialization.

### What are protocol buffers?

Protocol buffers (protobuf) is an efficient language and platform neutral mechanism for serializing structured data into binary format. Protobuf supports many computer languages including C/C++, Java, Python, and Go. Protocol buffers can be deployed on an array of computing platforms from small devices like the ESP8266, mobile devices, to datacenter-class computing devices.

### Using Protocol buffers

In general, using protocol buffers involve the three steps illustrated below.

![Steps to using Protocol Buffers](https://cdn-images-1.medium.com/max/5388/1*jsA7FFfpaGgpvUieBLQYjw.png)*Steps to using Protocol Buffers*

1. Define IDL — use protobuf’s Interface Definition Language (IDL) to define the type and structure of the data that will be serialized.

1. Compile — next, use the protobuf compiler (protoc) to compile the IDL into code that, not only represents the defined data but, can serialize and deserialize the data using a targeted language. The protocol buffers compiler uses a plugin architecture allowing it to generate code for many languages such as C/C++, Java, Python, Go.

1. Integrate — the generated code can then be integrated into your own code and used to serialize data for storage or communication between computing processes or nodes.

## Defining the IDL for temperature data

Before we jump into the code for this project, let us look at the IDL that defines the structure for the temperature data that will be sent from the ESP8266 device to the Go server program.

    **syntax** = "proto2";
    **package** pb;
    
    **message** TempEvent {
        **required int32** deviceId = 1;
        **required int32** eventId = 2;
    
        **required float** humidity = 3;
        **required float** tempCel = 4;
        **required float** heatIdxCel = 5;
    }
> See the protocol buffers IDL file [temp.proto](https://github.com/vladimirvivien/iot-dev/blob/master/esp8266/esp8266-dht11-temp/protobuf/temp.proto).

The message block declares a data structure container. Inside, it encloses several fields that define the data types that make up the message to be serialized. Once we have a defined IDL, we can compile it to generate code, in a chosen language, that can be used to encode/decode protobuf-encoded data.

## Programming the ESP8266 device

While the [ESP8266](http://esp8266.net/) is a cheap tiny device with constrained resources, it is a versatile project board well-suited for IoT uses.

* Low cost microcontroller (i.e. less than $5.0 USD)

* 80 MHz RISC processor based on the Tensilica L106 microcontroller

* 32 KiB instruction RAM

* Support for full-stack TCP/IP via WIFI radio.

* Arduino compatible

The board can be programmed using Arduino and leverage the vast open source device libraries that are readily available for that platform. The steps to programming the device are illustrated below.

![Steps to programming the ESP8266 device](https://cdn-images-1.medium.com/max/2242/1*6P-tgamoJNHnwO6EnCM8Ng.png)*Steps to programming the ESP8266 device*

We have already defined the IDL. Next let us see how we can compile the IDL into C code that can run on the ESP8266.

### Compiling the IDL to run on the ESP8266 with Nanopb

Using the standard protoc C plugin, to compile the IDL, would generate code that would not fit or work on such a tiny device like the ESP8266. Fortunately, there exists several open source *protoc plugin* projects that generate small footprint ANSI-C for constrained microcontroller devices. For this post, I use project [Nanopb](https://github.com/nanopb/nanopb) because it seems to be maintained and is compatible with the ESP8266 out of the box.

Once you have [setup your environment](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp#install-the-protocol-buffers-compiler) with the protocol buffers compiler and Nanopb, you can generate encoder and decoder C code for the protobuf message defined earlier as follows:

    protoc --plugin=protoc-gen-nanopb=\
        ~/nanopb/generator/protoc-gen-nanopb --nanopb_out=. temp.proto

The previous command uses protoc and the *Nanopb* compiler plugin to generate the C code from IDL file temp.proto. The command generates a C header file, temp.pb.h, and a C source file temp.pb.c. The generated header file contains a C struct that represents the IDL message defined earlier.

    **typedef** struct _pb_TempEvent {
        int32_t deviceId;
        int32_t eventId;
        **float** humidity;
        **float** tempCel;
        **float** heatIdxCel;
    } pb_TempEvent;

### Integrate the generated code with Arduino C

Next, add the generated C source files and the Nanopb protobuf C library to your Arduino project library. Then write the Arduino code that will run on the ESP8266. The following shows a snippet of the code which is used to read the temperature from the wired *DHT11* sensor (function loop()). Then, the data is encoded as a protobuf format, serialized and sent to the remote server using (function sendTemp() ).

    **#include** <temp.pb.h>
    
    **#include** <pb_common.h>
    **#include** <pb.h>
    **#include** <pb_encode.h>
    **#include** <pb_decode.h>
    
    **#include** <DHT.h>
    **#include **<ESP8266WiFi.h>

    WiFiClient client;

    **const** char* **ssid**     = "<SSID-Name>";
    **const** char* **password** = "<WIFI Pwd>";
    **const** char* **addr**     = "<Server-IP";
    **const** uint16_t **port**  = 10101;

    **void** setup(){...}

    **void** loop() {
    ...
      Serial.println("reading humidity/temp...");
      float hum = dht.readHumidity();
      float tmp = dht.readTemperature();
      float hiCel = dht.computeHeatIndex(tmp, hum, false);
        
      **pb_TempEvent** temp = pb_TempEvent_init_zero;
      temp.deviceId = 12;
      temp.eventId = 100;
      temp.humidity = hum;
      temp.tempCel = tmp;
      temp.heatIdxCel = hiCel;
      
      **sendTemp**(temp);
    }

    **void** sendTemp(**pb_TempEvent** e) {
      uint8_t buffer[128];
      pb_ostream_t stream = pb_ostream_from_buffer(buffer, ...);
      
      if (!**pb_encode**(&stream, **pb_TempEvent_fields**, &e)){
        Serial.println("failed to encode temp proto");
        return;
      }
      client.**write**(buffer, stream.bytes_written);
    }

Once the source code is complete, we can use the Arduino IDE to compile and upload the program to the ESP8266. When the device starts, it will attempt to connect to the Go server program at the configured WIFI network using the IP address and port specified.
> Read the detail walkthrough of the device Arduino code [here](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp#the-arduino-sketch-file).

## The backend server

When the device is turned on, it will immediately start to send temperature and humidity data to the Go server program. This program collects protobuf-encoded temperature data, decode it, and sends it to InfluxDB instance as a time series data.

### Compiling the IDL for the Go program

The server program must know how to decode the incoming protocol buffer encoded data. To do this, we will need to generate Go code from protocol buffers IDL filetemp.proto, created earlier, with the following command:

    protoc --go_out=. temp.proto

The previous command uses protoc and the Go protoc compiler plugin to generate Go source file temp.pb.go. The generated file contains a Go struct type, TempEvent, with fields that represent the message types defined in the IDL.

    **type** TempEvent struct {
     DeviceId         ***int32**   
     EventId          ***int32**  
     Humidity         ***float32** 
     TempCel          ***float32** 
     HeatIdxCel       ***float32** 
    }

### Integrating the generated code

The Go’s protocol buffers API comes with encoder and decoder functions that can be used to deserialize the protobuf-encoded binary data from the device into local Go values (of type TempEvent) as shown in the following snippet:

    **import** (**
        **temp "./pb" // pb generated package
        "github.com/golang/protobuf/proto"
    )

    **func** handleConnection(conn net.Conn) {
        ...
        buf := **make**([]byte, 1024)

        n, err := conn.Read(buf)
        **if** err != nil {
            log.Println(err)
            return
        }...

    **    var e temp.TempEvent**
        **if** err := **proto.Unmarshal**(buf[:n], &e); err != nil {
            log.Println("failed to unmarshal:", err)
            return
        }

        **if** err := **postEvent**(e); err != nil {...}
    }

### Posting the data to InfluxDB

For the data to be useful, it will be stored in InfluxDB as time series data. The next snippet shows a continuation of the server program which forwards the data to InfluxDB for storage.

    **import** influx "github.com/influxdata/influxdb/client/v2"

    **func** **postEvent**(e temp.TempEvent) error {
        **if** db != nil {
        ...
            tags := **map**[string]string{
                "deviceId": fmt.Sprintf("%d", e.GetDeviceId()),
                "eventId":  fmt.Sprintf("%d", e.GetDeviceId()),
            }

            fields := **map**[string]interface{}{
                "temp":      e.GetTempCel(),
                "humidity":  e.GetHumidity(),
                "heatIndex": e.GetHeatIdxCel(),
            }

            pt, err := influx.NewPoint("sensor-temp", tags, fields,   
                time.Now())
            bp.AddPoint(pt)

            **if** err := db.Write(bp); err != nil {
                return err
            }
         ...
        }
    }

## Running the services on Kubernetes

Now, we are ready to run the code and the services. There are many ways to do this. For instance, you can manually start InfluxDB, Grafana, and the Go program on your laptop for a quick test run (see instructions [here](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp#manual-deployment)).

Optionally, to make this whole project cloud native worthy, we can deploy all of the backend service programs on running Kubernetes cluster using the Helm package manager (yay more tech!).
> If you are not familiar with Kubernetes and would like to learn, start [here](https://kubernetes.io/) . Then, setup your local cluster using [minikube](https://kubernetes.io/docs/setup/minikube/).

### Deploy InfluxDB

Using [Helm](https://helm.sh/), the following command deploys an InfluxDB service on a running Kubernetes cluster:

    $> helm install --name dht11-db \
       --set config.http.auth-enabled=true \
       --set config.admin.enabled=true \
       --set env[0].name=INFLUXDB_DB,env[0].value=dht11 \
       --set env[1].name=INFLUXDB_ADMIN_USER,env[1].value=admin \
       --set env[2].name=INFLUXDB_ADMIN_PASSWORD,env[2].value=admin \
       --set env[3].name=INFLUXDB_USER,env[3].value=svcuser \
       --set env[4].name=INFLUXDB_USER_PASSWORD,env[4].value=svcuser \
    stable/influxdb

### Deploy Grafana

Similarly, Helm can be used to deploy a Grafana instance on your Kubernetes cluster with:

    $> helm install --name dht11-dashboard \
       --set adminUser=admin,adminPassword=admin \
    stable/grafana

Now, log into the Grafana portal and configure an [InfluxDB datasource](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp#log-into-grafana-portal) (depending on how you configure Grafana, you may have to use a kubectl port-forward command or a service end point to access the dashboard).

### Deploy server program

Next, we will use the kube run command to deploy a pod running the server program (from Docker image quay.io/vladimirvivien/esp8266-tempsvr):

    kubectl run esp8266-tempsvr \
      --port=10101 \
      --image=quay.io/vladimirvivien/esp8266-tempsvr \
      -- ./temp-server \
      -r http://dht11-db-influxdb.default:8086 -u svcuser -p svcuser

Finally, expose the program with a NodePort service:

    kubectl expose deployment/esp8266-tempsvr --type="NodePort" --port=10101 --target-port=10101

Next get the port on the host that has been exposed by Kubernetes service with:

    kubectl get service/esp8266-tempsvr \
      -o jsonpath="{.spec.ports[0].nodePort}"

If you are running a minikube cluster, setup a NAT port-forward rule in your Hypervisor, so that incoming traffic can be forwarded from your host’s port to the port exposed by Kubernetes service on your minkube VM.

If everything works and the device is capable of talking to the Go program running in Kubernetes, you should see that with the following:

    $> > kubectl logs pods/$(kubectl get pods -l run=esp8266-tempsvr -o jsonpath="{.items[0].metadata.name}")
    
    2018/07/29 11:03:01 Connected to  172.17.0.1:49852
    {DeviceID:12, EventID:100, Temp: 25.00, Humidity:58.00%, HeatIndex:25.07}
    2018/07/29 11:03:02 posting temp event to influxDB
    2018/07/29 11:03:02 INFO: closing connection

## Conclusion

There are numerous options when selecting a mean of communication for your IoT project. This write up discusses to use Protocol Buffers for efficient binary data encoding with IoT devices. It shows how to encode temperature data using an ESP8266 board with protocol buffer. The data is then sent to a backend Go program where it is saved in InfluxDB for visualization using Grafana running on Kubernetes!

Because of the large number of moving parts that make up this project, there is also a companion [GitHub repository](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp) with step-by-step detail for re-creating the project. Hope you enjoy it!

As always, if you find this writeup useful, please let me know by clicking on the clapping hands 👏 icon to recommend this post.

## References

Project Repository — [https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp](https://github.com/vladimirvivien/iot-dev/tree/master/esp8266/esp8266-dht11-temp)

ESP8266 — [http://esp8266.net/](http://esp8266.net/)

Arduino IDE — [https://www.arduino.cc/en/Main/Software](https://www.arduino.cc/en/Main/Software)

Protocol Buffers — [https://developers.google.com/protocol-buffers/](https://developers.google.com/protocol-buffers/)

Nanopb protoc plugin — [https://github.com/nanopb/nanopb](https://github.com/nanopb/nanopb)

InfluxDB — [https://www.influxdata.com/](https://www.influxdata.com/)

Grafana — [https://grafana.com/](https://grafana.com/)
