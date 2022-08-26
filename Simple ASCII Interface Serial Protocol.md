# Simple ASCII Interface

This interface is designed to be a simple way to read and write parameters from and to microcontrollers using an ASCII serial terminal.
All communication is initialted by the serial terminal.



## Writing a Parameter to a Microcontroller

A write request consists of a topic followed by a payload. The payload is separated from the topic by a space. A newline character (\n) indicates the end of the payload. Topics may not contain any spaces, however payloads may. Neither topic nor payload may contain newline characters.

### Request:

        topic     payload
     _____|_____   __|__
    /           \ /     \
    parameter_xyz 1234567
                 T       T
               space  newline


Once the microcontroller has processed the request, it will respond to the write request with the topic followed by a payload. This payload may be different than the received payload due to application limits. In our example we sent the numeber 1234567 to parameter_xyz, but lets say that parameter_xyz is limited to a maximum value of 1200000. The response payload would be 1200000. If the request is within application limits, the response should be an echo of the write request.

### Response:

        topic     payload
     _____|_____   __|__
    /           \ /     \
    parameter_xyz 1200000
                 T       T
               space  newline




## Reading a Parameter from a Microcontroller

A read request consists of a topic followed by a newline character (\n).

### Request:

        topic
     _____|_____
    /           \
    parameter_abc
                 T
              newline


The microcontroller's response will echo the topic and append the requested data in a payload.

### Response:

        topic     payload
     _____|_____   __|__
    /           \ /     \
    parameter_xyz 45.9873
                 T       T
               space  newline




## Topic Style

MQTT style topics are recommended. SCPI(ish) style topics can be used as well.



## Dealing with Bad Requests

A microcontroller may or may not respond to messages it has no parameter for. If multiple microcontrollers are connected in a multidrop configuration, data collisions will occur if the microcontrollers attempt to respond to messages not addressed to them. However in a single drop configuration, echoing the topic of such a message will indicate that communication is working and that the specified parameter does not exist on the microcontroller.

There is no specification on the format of the payload, other than it may not contain newline characters. The payload can be integers, numbers with a decimal point, or text strings. It is recommended that the microcontroller reject a value sent to a parameter of the wrong datatype, and respond with the data already in that parameter.



## The "?" Topic

The "?" topic is used to get information about the device: what it does, what topics it has, the datatypes of the topics, and descriptions of each topic. This is not a required topic, but it is recommended. The microcontroller's response to this topic is permitted to contain newline characters; this is the only exception to this rule.



## Arduino Example:

    float voltage = 0;
    float current = 0;
    boolean output = false;
    
    void setup() {
      Serial.begin(9600);
      Serial.setTimeout(10);
      while(!Serial);
    }
    
    void loop() {
      if (Serial.available()) {
        String topic = Serial.readStringUntil('\n');
        topic.trim();
        int index = topic.indexOf(' ');
        if (index != -1) {
          String payload = topic.substring(index + 1);
          topic.remove(index);
          if      (topic == "voltage")  voltage = payload.toFloat();
          else if (topic == "current")  current = payload.toFloat();
          else if (topic == "output")   output  = payload.toInt();
        }
        Serial.print(topic);
        Serial.print(' ');
        if (topic == "?") {
          Serial.println(F("Power Supply"));
          Serial.println(F("TOPIC\tTYPE\tDESCRIPTION"));
          Serial.println(F("voltage\tfloat\tControls power supply voltage"));
          Serial.println(F("current\tfloat\tControls power supply current limit"));
          Serial.println(F("output\tbool\tEnables or disables power supply output"));
        }
        else if (topic == "voltage")  Serial.print(voltage);
        else if (topic == "current")  Serial.print(current);
        else if (topic == "output")   Serial.print(output);
        Serial.println();
      }
    }

