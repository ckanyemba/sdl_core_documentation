## 4.2. Components View


![C4_Component](https://user-images.githubusercontent.com/95967301/164893437-e6fb093e-4e0d-40f7-ab7b-d0046f2cdbb6.svg)


The view is represented by module and subsystem diagrams that show the system's export and import relationships. The Components View diagram and its elements description please see below.

*Note*: UML notation for this Components View diagram is extended: both component and its interfaces are highlighted with the same color.

|||
##### Component View diagram
![Component View](./assets/ComponentView.png)
|||

***Elements description***

### Utility components:

#### Life Cycle
  - *Responsibility:*
    - Functional components manipulation
      -   creation
      -   destruction
      -   initialization
      -   start, stop
      -   binding
    - System and Utils-specifics initialization
    - *Relations*
    - Composes all available components
  - *Interfaces*
    -   Does not provide any external interfaces
  - *Behavior*
    - ***Life Cycle*** creates all available in system components according to configuration, binds components to components, and starts each component's internal routines.
  - *Constraints*
    - *N/A*

#### Config Profile
  - *Responsibility*
    -    Storing information about application configuration.
  - *Relations*
    - Used by ***Life Cycle*** for filling other components ***Settings ***
  - *Interfaces*
    - Provides ***Profile*** interface
  - *Behavior*
    - ***Config Profile*** parses configurable data storage and provides primitive types by section and name of configurable value.
  - *Constraints*
    - Configuration format - INI file.

#### Utils
  - *Responsibility*
    - Encapsulation system low-level functionality.
  - *Relations*
    -   Used by all components.
  - *Interfaces*
    - ***Logger*** macros-es and functions
    - Data and Time
    - Files
    - ***Thread*** and ***Timer***
    - ***Locks*** and ***ConditionalVariable*** classes
    - ***CustomString*** class for UTF8 string handling
  - *Behavior*
    - ***Utils*** behavior relates to system-specific API.
  - *Constraints*
    - *N/A*

### HMI layer components:

#### HMI Message Handler
  - *Responsibility*
    - Formatting message to and from unified protocol-API-independent format used by higher-level component.
    - Providing adapters for different transport types between SDL and HMI.
  - *Relations*
    - ***Application Manager***
    - ***Utils***
  - *Interfaces*
    - ***HMIMessageObserver*** interface for listening HMI messages notification
    - ***HMIMessageSender*** interface for sending Messages
    - ***HMIMessageAdapter*** interface for abstracting to-HMI transport
    - ***HMIMessageHandler*** interface for accumulating ***HMIMessageObserver, HMIMessageSender*** and ***HMIMessageAdapter***
  - *Behavior*
    - Transferring RPC Messages between business-layer and configured transport.
  - *Constraints*
    - Processes messages from a single instance of HMI only.
    - HMI-transport need to be statically configurable with build flags.

### Application layer components:

#### Application Manager
  - *Responsibility*
    - Storing and providing mobile-related information
    - Mobile application state manipulation
  - *Relations*
    - Uses ***Commands***
    - Uses ***RPC Service***
    - Uses ***MediaManager***
    - Requires ***PolicyHandler** and **PolicyHandlerObserver (Policy)***
    - Requires ***ConnectionHandler** and **ConnectionHandlerObserver (Connection Handler)***
    - Requires ***SessionObserver (Connection Handler)***
    - Requires ***SecurityManagerListener (Security Manager*** component***)***
  - *Interfaces*
    - Provides ***ApplicationManager*** interface
  - *Behavior*
    - The component implements business logic of the SDL.
  - *Constraints*
    - *N/A*

#### RPC Service
  - *Responsibility*
    - Send Mobile/HMI RPCs
    - Handle Mobile/HMI RPCs
  - *Relations*
    - Uses ***PluginManager***
    - Uses ***RequestController***
    - Requires ***ProtocolHandler** and **ProtocolObserver (Protocol Handler)***
    - Requires ***HMIMessageObserver** and **HMIMessageSender (HMI Message Handler)***
  - *Interfaces*
    - Provides ***RPCService*** interface
  - *Behavior*
    - The component implements RPC transferring functionality
  - *Constraints*
    - *N/A*

#### RPC plugin

  - *Responsibility*
    - Mobile and HMI RPC data verification according to business-requirements
    - Transferring Mobile RPC Requests to HMI subsystems (UI, VR, TTS and other available ones) and HMI to Mobile Responses and Notifications
  - *Relations*
    - Created by ***ApplicationManager***
    - Composed by **PluginManager***
    - Used by ***RequestController***
  - *Interfaces*
    - Provides ***RPCService*** interface
  - *Behavior*
    - Mobile Requests are spitted between responsible HMI interfaces and sent as separate HMI Requests or Notifications.
    - HMI Responses and notifications are verified according to business requirements and provided to Mobile.
  - *Constraints*
    - [FORD Mobile API Spec](https://github.com/smartdevicelink/sdl_core/blob/master/src/components/interfaces/MOBILE_API.xml)
    - [FORD HMI API Spec](https://github.com/smartdevicelink/sdl_core/blob/master/src/components/interfaces/HMI_API.xml)
    - Commands happy paths are depends on correct [HMI Behavior](https://github.com/smartdevicelink/sdl_hmi_integration_guidelines/blob/master/docs/Getting%20Started/index.md) implementation.

#### Request Controller
  - *Responsibility*
    - Pending requests handling
    - Optimization threads count for handling large quantity of pending RPCs
  - *Relations*
    - Composes ***Commands***
    - Composed by ***Application Manager***
  - *Interfaces*
    - Provides ***Request Controller*** interface
  - *Behavior*
    - ***Request Controller*** handles timeout of responses and notifications from HMI.
  - *Constraints*
    - Configurable count of threads usage.

#### App Launch
  - *Responsibility*
    - Launch known applications on devices.
  - *Relations*
    - Composed by ***Application Manager***
    - Use ***Resume Controller*** interface to get HMI level of saved application.
  - *Interfaces*
    - Provides ***App Launch Controller*** interface
  - *Behavior*
    - ***App Launch*** launch all known applications on newly connected device.
  - *Constraints*
    - Not work for Android apps.
    - Not work for apps connected via SDL protocol version lower than 4.

#### Plugin Manager
  - *Responsibility*
    - Loads all .so files from specific directory, checking if they’re exporting required methods
    - Stores information about plugin capabilities
    - Checks plugins capability to handle RPCs
  - *Relations*
    - Composed by ***Application Manager***
    - Composes ***Plugin***
  - *Interfaces*
    - Provides ***Plugin Manager*** interface
  - *Behavior*
    - Loads and manages plugins from specific directory.
  - *Constraints*
    - Able to load only RPC layer plugins

#### Plugin
  - *Responsibility*
    - Process Mobile/HMI RPC
  - *Relations*
    - Composed by ***Plugin Manager***
    - Uses ***Commands***
  - *Interfaces*
    - Provides ***Plugin*** interface
  - *Behavior*
    - Handle specific RPC and process its internal logic
  - *Constraints*
    - Able to implement only RPC layer functionality extension

#### Resumption
  - *Responsibility*
    - Restoring application data
    - Storing application and HMI-related data between shutdown cycles
  - *Relations*
    - Composed by ***Application Manager***
  - *Interfaces*
    - Provides ***Resume Controller*** interface
  - *Behavior*
    - ***Resumption*** backs up application and HMI-related data and restores it after SDL start-up according to business logics.
  - *Constraints*
    - *Configurable data storage type.*

#### Policy
  - *Responsibility*
    - Enabling advanced SDL functionality
    - SDL APIs protection from unauthorized application usage
    - Remotely manage SDL-enabled apps, including app-specific and device-specific access to system functionality
    - Maintain applications permissions on the system
  - *Relations*
    - Uses ***ApplicationManager*** interface for mobile application state manipulation
  - *Interfaces*
    - Provides ***PolicyManager*** interface for policy data manipulation
    - Provides ***PolicyListener*** interface for policy notification subscribing
  - *Behavior*
    - Receives data from Application manager
    - Parses data- Stores in local storage
    - Provides data via Application Manager to mobile device and HMI and vice-versa
  - *Constraints*
    - Needs to be a switchable components: dynamically by configuration file and statically by SDL build define.

#### Media Manager
  - *Responsibility*
    - Audio and Video data transferring to Media sub-system
    - Encapsulation binary data transferring transport
  - *Relations*
    - Used by ***Application Manager***
  - *Interfaces*
    - Provides ***MediaManager*** interface
  - *Behavior*
    - Media Manager transfers raw Audio and Video data through one of the Media-adapters.
  - *Constraints*
    - Configurable Media-adapter usage

#### RC RPC Plugin
  - *Responsibility*
    - Allows incorporating additional functionality to the core application by application extension.
    - Implements specific mobile RPC processing.
    - Implements specific HMI RPC processing.
  - *Relations*
    - Composed by ***Plugin***
    - Handles ***Application Manager*** by ***Application Manager*** interface
  - *Interfaces*
    - Provides ***Plugin*** interface
  - *Behavior*
    - Receives data from ApplicationManager
    - Parses data
    - Creates commands.
  - *Constraints*
    - N/A

#### SDL RPC Plugin
  - *Responsibility*
    - Implements regular mobile RPC processing.
    - Implements regular HMI RPC processing.
  - *Relations*
    - Composed by ***Plugin***
    - Handles ***Application Manager*** by ***Application Manager*** interface
  - *Interfaces*
    - Provides ***Plugin*** interface
  - *Behavior*
    - Receives data from ApplicationManager
    - Parses data
    - Creates commands.
  - *Constraints*
    - N/A

#### VehicleInfo Plugin
  - *Responsibility*
    - Allows incorporating additional functionality to the core application by application extension.
    - Implements specific mobile RPC processing.
    - Implements specific HMI RPC processing.
  - *Relations*
    - Composed by ***Plugin***
    - Handles ***Application Manager*** by ***Application Manager*** interface
  - *Interfaces*
    - Provides ***Plugin*** interface
  - *Behavior*
    - Receives data from ApplicationManager
    - Parses data
    - Creates commands.
  - *Constraints*
    - N/A

### Protocol layer components:

#### Protocol Handler
  - *Responsibility*
    - Control and business data distributing to appropriate sessions and service
    - Control message processing
    - Multi-frames assembling and disassembling
    - Malformed packets determination and filtering
  - *Relations*
    - Notifies ***ConnectionHandler*** about connection and session state change
    - Uses ***SecurityManager*** for encryption and decryption payload data
  - *Interfaces*
    - Provides ***ProtocolHandler*** interface for data sending and protocol layer manipulation
    - Provides ***ProtocolObserver*** notification for subscription on protocol events.
  - *Behavior*
    - Decodes income raw transport data and encodes outcome RPCs according to protocol specification.
  - *Constraints*
    - [SmartDeviceLink Protocol specification](https://github.com/smartdevicelink/protocol_spec/blob/master/README.md)

#### Connection Handler
  - *Responsibility*
    - Storing devices and connection information
    - Manage starting and ending of sessions
    - Providing device, connection and session information for protocol and business layer
    - Manipulation with devices, connections and sessions
    - Negotiation and monitoring the availability of device connections (heartbeat)
  - *Relations*
    - Requires ***ProtocolHandler*** for sending Control messages related to session life cycle
    - Requires ***TransportManager*** for forwarding business layer device and connection manipulations
  - *Interfaces*
    - Provides ***ConnectionHandller*** interface for connection manipulation
    - Provides ***SessionObserver*** interface for session information manipulation
  - *Behavior*
    - Connection Handler works as a proxy from business-layer to transport layer and provides additional information related to protocol sessions and services.
  - *Constraints*
    - [SmartDeviceLink Protocol specification](https://github.com/smartdevicelink/protocol_spec/blob/master/README.md)

#### Security Manager
  - *Responsibility*
    - Data encryption and decryption
    - TLS Handshake negotiation
    - TLS Library dependency encapsulation
 - *Relations*
   - Uses ***SessionObserver*** for setting Security information to sessions
   - Uses ***ProtocolHandler*** and ***ProtocolObserver*** for handling TLS handshake data
  - *Interfaces*
    - Provides ***SecurityManager*** interface for Security component
    - Provides ***SecurityManagerListener*** interface for notification handshake event
    - Provides ***SSLContext*** interface for data encryption and decryption
  - *Behavior*
    - ***Security Manager*** provides methods to establish encrypted connection to mobile.
  - *Constraints*
    - Needs to be a switchable components: dynamically by configuration file and statically by SDL build define.
    - [SmartDeviceLink Protocol specification](https://github.com/smartdevicelink/protocol_spec/blob/master/README.md)

### Transport layer components:

#### Transport Manager
  - *Responsibility*
    - Manages low-level connections from Mobile Applications
    - Transport devices and connections manipulation
    - Performs device discovery
    - Sending and receiving mobile messages
  - *Relations*
    - Composes ***TransportAdapters*** according to configuration
  - *Interfaces*
    - Provides ***TransportManager*** interface for devices and connections status manipulation
    - Provides ***TransportManagerListener*** interface for transport notification subscribing
  - *Behavior*
    - Accumulative class for all available in system devices and connections.
  - *Constraints*
    - *N/A*

#### Transport Adapter
 - *Responsibility*
   - Transport-specific API encapsulation
  - *Relations*
    - Composed by ***TransportManager***
  - *Interfaces*
    - Provides ***TransportAdapters*** interface
  - *Behavior*
    - Adopts transport searching, connecting, data transferring API for one ***TransportAdapters interface.***
  - *Constraints*
    - For Bluetooth BlueZ transport there are only 30 connections available due to RFCOMM channels limitations.
    - [Transport Manager Programming guide](../../Transport Manager Programming/index.md)
