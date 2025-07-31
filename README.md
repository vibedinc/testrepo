### HOW TO CONFIGURE AND DEPLOY AN OPERATIONAL DIGITAL THREAD
> This guide is intended for qualified personnel who have a basic understanding of IoT and 3D application development.
<img width="1200" alt="Optix Unity Connector Header" src="https://github.com/dtosmartexperiences/202-IoT-Feeds/assets/158075065/198b1a1d-bc36-4f75-a75c-3727043df193">

ThreadManager is an Optix<>Unity plugin extension and streaming service created by Rockwell Automation.

A hosted service that can scale to millions of users, ThreadManager is built on a client-server model. ThreadManager synchronizes live IT and OT production data across various 3D visualization devices. ThreadManager was created to reduce production downtime and drive ongoing process improvements by delivering critical information to the right people, at the right time. Whether you are accessing from a tablet or AR HMD, ThreadManager has you covered!

ThreadManager empowers smart connected workers with a collection of the following core packaged capabilities:
| Description | Demonstration |
| ------------- | ------------- |
| Monitor with Predictive Twin | 
| Locate with Fault Wayfinder | 
| Tune with Data Guage | 
| Resolve with Popup HUD | 

---

### ThreadManager API Architecture
<img width="3840" alt="Architecture Diagram" src="https://github.com/user-attachments/assets/08b96120-352d-4802-9f6f-16af6adeaf14">
ThreadManager is a REST API that Rockwell Automation enables customers to stream embedded PLC data to 3D visualization in real-time, making it faster to identify faults and resolve production downtime. The lower-level Connected Worker + Connected Zone APIs manage the lower level connections to the Optix server and chosen datastore. ThreadManager is a layer built on top of Hotspot and Zone APIs.

The ThreadManager Cloud backend is made up of the following subsystems:
- Composer DB is a matcher service that connects specific controllers data streams to target visualization clients
- Agents are synchronized app containers that receive real-time contextual data streams from the Composer.

Just like all software, ThreadManager is built on the shoulders of incredible people! Here's a list of the required library dependencies:
<div>
<img src="https://github.com/devicons/devicon/blob/master/icons/unity/unity-original.svg" title="Unity" alt="Unity" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/azure/azure-original.svg" title="Azure" alt="Azure" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/chrome/chrome-original.svg" title="Chrome" alt="Chrome" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/visualstudio/visualstudio-original.svg" title="Visual Studio" alt="VisualStudio" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/kubernetes/kubernetes-original.svg" title="Kubernetes" alt="Kubernetes" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/javascript/javascript-original.svg" title="JavaScript" alt="Chrome" width="40" height="40"/> 
<img src="https://github.com/devicons/devicon/blob/master/icons/json/json-original.svg" title="JSON" **alt="JSON" width="40" height="40"/>
</div>

---

### PART 1 OF 3) EXTEND OPTIX IIOT GATEWAY
> Supported Optix Runtime: v1.3.3 or newer

#### Step 1) Create the Ethernet/IP CommDrivers
The CommDriver is responsible for managing the secure protocol connection with the embedded controller.


#### Step 2) Create the Ethernet/IP Station
NOTE: Stations include tags and types folde

rs

3. Update the RestAPIClient NetLogic apiURL script with your target datastore:

<html>
<head>
<title>Example: https://65b3e661770d43aba47aa090.mockapi.io/ftapi/test/OptixtoUnity </title>
</head>

4. Update the title to match the name of your website.

#### Step 3) Import the Controller ACD File


#### Step 4) Configure Routing to Controller I.P


#### Step 5) Copy and Paste the following Code into a 'UnityConnector' NetLogic Script

#region Using directives using System; using System.Timers; using UAManagedCore; using OpcUa = UAManagedCore.OpcUa; using FTOptix.HMIProject; using FTOptix.NetLogic; using FTOptix.UI; using FTOptix.Retentivity; using FTOptix.NativeUI; using FTOptix.CoreBase; using FTOptix.Core; using System.Net.Http.Headers; using System.Net.Http; using System.Threading.Tasks; using FTOptix.WebUI; using FTOptix.RAEtherNetIP; using Newtonsoft; using static System.Net.WebRequestMethods; using System.Diagnostics.Contracts; using FTOptix.System; using FTOptix.Alarm; #endregion  public class UnityConnector : BaseNetLogic {      readonly struct HTTPResponse     {         public HTTPResponse(string payload, int code)         {             Payload = payload;             Code = code;         }          public string Payload { get; }         public int Code { get; }     };      // Define standarized data structure for data serialization     public class Packet     {         // Define JSON Object Structure         public string timeStamp { get; set; }         public string assetName { get; set; }         public string onlineStatus { get; set; }         public string assetValues { get; set; }         public string optixDelivered { get; set; }         public string unityReceived { get; set; }         public string DataPacket { get; set; }     }      private long GetTimeout()     {         var timeoutVariable = LogicObject.Get<IUAVariable>("Timeout");         if (timeoutVariable == null)             throw new Exception($"Missing Timeout variable under the NetLogic {LogicObject.BrowseName}");          return timeoutVariable.Value;     }      private string GetUserAgent()     {         var userAgentVariable = LogicObject.Get<IUAVariable>("UserAgent");         if (userAgentVariable == null)             throw new Exception($"Missing UserAgent variable under the NetLogic {LogicObject.BrowseName}");          return userAgentVariable.Value;     }      private bool IsSupportedScheme(string scheme)     {         return scheme == "http" || scheme == "https";     }      private bool IsSecureScheme(string scheme)     {         return scheme == "https";     }      private HttpRequestMessage BuildGetMessage(Uri url, string userAgent, string bearerToken)     {         HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, url);          if (!string.IsNullOrWhiteSpace(userAgent))             request.Headers.UserAgent.Add(new ProductInfoHeaderValue(new ProductHeaderValue(userAgent)));          if (!string.IsNullOrWhiteSpace(bearerToken))             request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", bearerToken);          return request;     }      private HttpRequestMessage BuildPostMessage(Uri url, string body, string contentType, string userAgent, string bearerToken)     {         HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Post, url);          if (!string.IsNullOrWhiteSpace(body))             request.Content = new StringContent(body, System.Text.Encoding.UTF8, contentType);          if (!string.IsNullOrWhiteSpace(userAgent))             request.Headers.UserAgent.Add(new ProductInfoHeaderValue(new ProductHeaderValue(userAgent)));          if (!string.IsNullOrWhiteSpace(bearerToken))             request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", bearerToken);          return request;     }      private HttpRequestMessage BuildPutMessage(Uri url, string body, string contentType, string userAgent, string bearerToken)     {         HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Put, url);          if (!string.IsNullOrWhiteSpace(body))             request.Content = new StringContent(body, System.Text.Encoding.UTF8, contentType);          if (!string.IsNullOrWhiteSpace(userAgent))             request.Headers.UserAgent.Add(new ProductInfoHeaderValue(new ProductHeaderValue(userAgent)));          if (!string.IsNullOrWhiteSpace(bearerToken))             request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", bearerToken);          return request;     }      private async Task<HTTPResponse> PerformRequest(HttpRequestMessage request, TimeSpan timeout)     {         HttpClient client = new HttpClient();         client.Timeout = timeout;          using HttpResponseMessage httpResponse = await client.SendAsync(request);         string responseBody = await httpResponse.Content.ReadAsStringAsync();          return new HTTPResponse(responseBody, (int)httpResponse.StatusCode);     }      private HttpRequestMessage BuildMessage(HttpMethod verb, Uri url, string requestBody, string bearerToken, string contentType)     {         TimeSpan timeout = TimeSpan.FromMilliseconds(GetTimeout());         string userAgent = GetUserAgent();          if (string.IsNullOrWhiteSpace(contentType))             contentType = "application/json";          if (!IsSupportedScheme(url.Scheme))             throw new Exception($"The URI scheme {url.Scheme} is not supported");          if (!IsSecureScheme(url.Scheme) && !string.IsNullOrWhiteSpace(bearerToken))             Log.Warning("Possible sending of unencrypted confidential information");          if (verb == HttpMethod.Get)             return BuildGetMessage(url, userAgent, bearerToken);         if (verb == HttpMethod.Post)             return BuildPostMessage(url, requestBody, contentType, userAgent, bearerToken);         if (verb == HttpMethod.Put)             return BuildPutMessage(url, requestBody, contentType, userAgent, bearerToken);          throw new Exception($"Unsupported verb {verb}");     }      [ExportMethod]     public void Get(string apiUrl, string queryString, string bearerToken, out string response, out int code)     {         TimeSpan timeout = TimeSpan.FromMilliseconds(GetTimeout());         UriBuilder uriBuilder = new UriBuilder(apiUrl);         uriBuilder.Query = queryString;          var requestMessage = BuildMessage(HttpMethod.Get, uriBuilder.Uri, "", bearerToken, "");         var requestTask = PerformRequest(requestMessage, timeout);         var httpResponse = requestTask.Result;          (response, code) = (httpResponse.Payload, httpResponse.Code);     }      [ExportMethod]     public void Post(string apiUrl, string requestBody, string bearerToken, string contentType, out string response, out int code)     {         TimeSpan timeout = TimeSpan.FromMilliseconds(GetTimeout());          // Add the IP of your chosed DataStore (Azure / GCP / Oracle / Etc)         UriBuilder uriBuilder = new UriBuilder("<https://65b3e661770d43aba47aa090.mockapi.io/ftapi/test/OptixtoUnity>");          // Dynamic value to monitor live embedded controller changes          string trackedTagVal = LogicObject.GetVariable("trackedTagVal").Value;          // Containerize data packets for encoding         Packet JSON = new Packet()         {             timeStamp = new DateTimeOffset(DateTime.UtcNow).ToUnixTimeSeconds().ToString(),             assetName = "test",             onlineStatus = "true",             assetValues = trackedTagVal,             optixDelivered = "true",             unityReceived = "false",             DataPacket = "test"         };          requestBody = Newtonsoft.Json.JsonConvert.SerializeObject(JSON);          var requestMessage = BuildMessage(HttpMethod.Post, uriBuilder.Uri, requestBody, bearerToken, contentType);         var requestTask = PerformRequest(requestMessage, timeout);         var httpResponse = requestTask.Result;          (response, code) = (httpResponse.Payload, httpResponse.Code);      }      [ExportMethod]     public void Put(string apiUrl, string requestBody, string bearerToken, string contentType, out string response, out int code)     {         TimeSpan timeout = TimeSpan.FromMilliseconds(GetTimeout());         UriBuilder uriBuilder = new UriBuilder(apiUrl);          var requestMessage = BuildMessage(HttpMethod.Put, uriBuilder.Uri, requestBody, bearerToken, contentType);         var requestTask = PerformRequest(requestMessage, timeout);         var httpResponse = requestTask.Result;          (response, code) = (httpResponse.Payload, httpResponse.Code);     }  } 
5.2 Add a 'trackedTagVal' string to the 'UnityConnector' NetLogic Script

---

### PART 2 OF 3) CUSTOMIZE UNITY TEMPLATE SCENE
> Quickstart your development with our project template (Supported Runtimes: Unity v2022.3.21f1 or later)

The Facility of the Future scene iPad sample uses the Vuforia Area, Model Target, and Ground Placement features for localization and camera tracking. The sample scenes are located in the SampleResources folder along with other assets and resources in this sample.

ThreadManager uses a datastore to track all state in a smart facility. The datastore itself is made up of model objects. Every model in the datastore, including child models, can have an owner with which it is associated.

How an Immersive Facility hierarchy is structured:
<img width="834" alt="ThreadManager Structure" src="https://github.com/dtosmartexperiences/ThreadManager-iPad-Sample/assets/158075065/933289ae-d720-45b0-a029-1c79703c427f">

The scene Thread-ManagerScene shows the basic setup for development on iPad, combining OpenXR with Vuforia components.
All other scenes are loaded additively to add target-specific functionality.
Scene Graph

#### Step 1) Import the IFDK Unity Package
Download the Unity Asset Package from GitHub page and import it with the Package Manager. The package will include all dependencies during the import.

a. In Unity, select Assets -> Import Package -> Custom Package and open the ThreadManager Unity Asset Package.
b. In Unity, select Window -> Package Manager -> My Assets and select the ThreadManager iPad Sample.

1. Import Vuforia Engine into the project using the Package Manager
2. // A dynamic system for querying data trees through JSON deserialization
3. In the XR Plug-in Management Standalone tab, enable Initialize XR on Startup, OpenXR, and ARKit.

#### Step 2) Add Procedure Navigation Paths
Use Wayfinders to guide technicians to target locations across the facility

1. Add Area Target Scan


2. Drag and drop the 'Wayfinder' prefab into the canvas view
NOTE: The wayfinder texture can be modified by replacing the WayfinderTexture in Assets > IFDK > Textures.

#### Step 3) Customize HMI Panel Popups
Use the Anywhere HMIs to add brand identity

1. Drag and drop the 'Work Zone' prefab into the canvas view
2. Import your company logo
3. Use the Theme to add your brand color scheme (Style Sheet)
4. Configure the UITextFields with your TagIP
NOTE: Matcher service is time-series based query.


#### Step 4) Design Spatial Operating Procedure
Use the Sequence Creator to create 3D animated spatial work instruction NOTE: Matcher service is time-series based query.

2. Connect Window Text Fields with Fix API (*Coming Soon)
NOTE: To enable Vuforia Engine, remember to add your App License Key to the Vuforia Configuration component which can be found in Assets > Resources > VuforiaConfiguration.

---

### PART 3 OF 3) PUBLISH THREAD STREAM
> Access a real-time view of your asset lifecylce (Supported Devices: Apple iPad Pro & Microsoft HoloLens 2)



#### Step 1) Build and Deploy to Target Client
1.1 In the XR Plug-in Management Standalone tab, enable Initialize XR on Startup, OpenXR, and ARKit.
1.2 To allow the project to configure correctly in Play Mode, the ThreadManager sample must be played from the Facility-of-the-Future scene. The scene can be found in SampleResources/Scenes.

#### Step 2) Validate Connection in Play Mode
2.1. In the XR Plug-in Management Standalone tab, enable Initialize XR on Startup, OpenXR, and ARKit.
NOTE: If you only wish to test the ThreadManager iPad sample in Play Mode, the XR Plug-in Management options need to be adjusted for the Standalone build target.

#### Step 3) Build and Deploy to Target Client



---
#### ADDITIONAL RESOURCES
- If you find any issue with the repository, please submit your ticket here
- Stay up to date on lastest improvements and upcoming releases
- Join the recurring ThreadManager Affinity Call for open discussion, help, and updates.
