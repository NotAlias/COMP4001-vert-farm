# UML modeling for vert-farm

## User flow

Based off this illustration.
![Paint user flow](./images/user-navigation.png)


```mermaid
flowchart
    authA((Start))
    authB(Login page)
    authC[/1.Device Address
        2.Web API Key
    /]
    authD[mDNS Discovery
        Service for
        Auto Device
        Address fill
    ]
    authE[/1.Web API Key/]
    authF[Authentication
        Via Websocket
        salted SHA256
        HMMAC
    ]
    authCheck{Auth
        Success?
    }

    dashMain(Main Dashboard:
        1.Current TDS/EC
        2.Current Temp
        3.AI Expert Feedback
        4.Last Data Graph
    )

    dashLog(Log Dashboard:
        1.TDS/EC log graph with range
        2.Temp log graph with range
        3.Export/Download data
    )

    settings(Settings:
        1.Device ID & mDNS domain
        2. WiFi Credentials
    )

    Nav{Navigation?}

    authA --> authB
    authB --> authC --> authF
    authB --> authD --> authE --> authF --> authCheck
    authCheck --> |No| authB
    authCheck --> |Yes| dashMain
    dashMain --> |Navigate| Nav --> |To main| dashMain
    dashLog --> |Navigate| Nav --> |To Log| dashLog
    settings --> |Navigate| Nav --> |To Settings| settings
    Nav --> |logout| authB
    Nav ---> |exit| Finish

    Finish((Finish))
```

## Water cycle

Based off this illustration of the water flow through the system.
![Water flow through system.](./images/water-cycle.png)

```mermaid
flowchart
    subgraph box[Flow group]
    direction TB
        node1[plant1]
        node2[plant2]
        node3[plant3]
    end

    subgraph mainTank[main tank]
        pump
    end

    waste[Waste water tank]
    treatment[Waste water treatment]
    sterile[Sterilization]
    sludge[Sludge]
    recovered[Recovered water]
    nutrient[Nutrient prep]

    box --return--> mainTank
    pump --pumps--> box
    mainTank --drain--> waste --> treatment --> sludge
    treatment --> sterile --> recovered -->nutrient --> mainTank
```

## Electronics

```mermaid
flowchart

cam[ov5640]
esp[esp32]
240v[Mains 240V]
lights[LED lights]
humid[humidity sensor]
ph[Ph Sensor]


```

### Cell

```mermaid
flowchart TD
powr[Mains Power Bus]
data[LAN Network]

switch[Network Switch]

data <--> switch

subgraph cell[Grow Cell]
    subgraph compute[Raspberry Pi]

    end

    subgraph camera[Cam Sensor]
        esp[ESP32-S3 Cam]
        ov[OV5640]
        esp<--SCCB (I2C)-->ov
        ov--DVP-->esp
    end

    dht11[Temp and 
        Humidity sensor
        DHT11]


    compute <--USB--> camera
    compute <-- one-Wire--> dht11
end

switch <--ethernet--> compute
```

## MQTT Sequence

From William
```mermaid
sequenceDiagram
    Growing cell->>MQTT Broker: CONNECT
    MQTT Broker->>Growing cell: CONNACK
    Growing cell->>MQTT Broker: SUBSCRIBE(Topic: ..., Qos: 1)
    loop Program loop
        alt Normal operation
            MQTT Broker->>Growing cell: PUBLISH(Topic: ..., Qos: 1)
            Growing cell->>MQTT Broker: PUBACK
        else issue detected
            Growing cell->>MQTT Broker: PUBLISH(Payload: state, Qos: 1)
            MQTT Broker->>Growing cell: PUBACK
        end
    end
```
