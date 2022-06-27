#### Instrumentation Service
Service for logging instrumentation/analytics data from device as well as server. This not only includes Siri requests but provides a platform for all other 
services - MT/TTS.

**gRPC and Protobuf**: Service is gRPC based and data format is protobuf. The gRPC service will enable the use/adoption of Protocol Buffer wire format 
and the Protobuf IDL, those being descriptive/easy to read and portable across multiple platforms.
- Protobuf also enables backward compatibility.
- Moreover no further work would be required to enable C++ services, to begin logging UEI events or any type of analytics/instrumentation and hence 
would save the duplicate effort to support new features (e.g. privacy filtering) on multiple platforms. 

**Centralized Transport & Platform Independence:** Upon receipt of a message, entire processing (e.g. Proto to Avro Conversion, applying policies) 
related to transporting the instrumentation will be handled at the centralized place i.e. by SIS. The interface exposed by SIS can be used by the 
applications to Codegen a client in any supported programming language. 


##### Architecture



