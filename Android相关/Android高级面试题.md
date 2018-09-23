
## Android Advanced Interview Questions

** First, the picture **

**Photo Gallery Comparison**

Http://stackoverflow.com/questions/29363321/picasso-vs-imageloader-vs-fresco-vs-glide
Http://www.trinea.cn/android/android-image-cache-compare/

Source code analysis of the photo library

Image frame cache implementation

LRUCache principle

Image loading principle

How do I do it myself?

****Glide source analysis ****

Http://www.lightskystreet.com/2015/10/12/glide_source_analysis/
Http://frodoking.github.io/2015/10/10/android-glide/
Http://ethanhua.cn/archives/243

What cache does Glide use?

How does the Glide memory cache control the size?

Talk about the understanding of Fresco?

**Fresco vs. Glide:**

Glide: Relatively lightweight, simple and elegant to use, support Gif dynamic map, suitable for those applications that are not very dependent on images.
Fresco: Use anonymous shared memory to save images, which is the Native heap. It effectively avoids OOM and is powerful, but the library is too large and suitable for use in apps that rely heavily on images.

Fresco's overall architecture is shown below:

![image](https://github.com/guoxiaoxing/android-open-framwork-analysis/raw/master/art/fresco/fresco_structure.png)

DraweeView: Inherited from ImageView, it simply reads some attribute values ​​of the xml file and does some initialization work. The layer management is handled by Hierarchy, and the layer data acquisition is responsible.
DraweeHierarchy: Consists of multiple layers of Drawable, each layer provides some functionality (for example: scaling, rounding).
DraweeController: Controls data acquisition and image loading, sends requests to the pipeline, receives corresponding events, and controls Hierarchy according to different events, receives user events from DraweeView, and then performs operations such as canceling network requests and reclaiming resources.
DraweeHolder: Coordinating the management of Hierarchy and DraweeHolder.
ImagePipeline: Fresco's core module for capturing images in a variety of ways (memory, disk, network, etc.).
Producer/Consumer: There are also many kinds of Producers, which are used to complete network data acquisition, cache data acquisition, image decoding and other work, and the results are consumed by Consumer.
IO/Data: This layer is the data layer, responsible for memory cache, disk cache, network cache and other IO related functions.
Throughout the entire Fresco architecture, DraweeView is the facade, interacting with the user, DraweeHierarchy is the view hierarchy, the management layer, and the DraweeController is the controller that manages the data. They form the troika of the entire Fresco framework. Of course, there is our behind-the-scenes hero Producer, all the dirty work is done, the best model 👍

Understanding the overall architecture of Fresco, we also have several key roles to understand the important role of this mine, as follows:

Supplier: Provides a specific type of object. There are many classes in the Fresco that end in Supplier.
SimpleDraweeView: We are familiar with this, it receives a URL and then calls the Controller to load the image. This class inherits from GenericDraweeView, GenericDraweeView inherits from DraweeView, and DraweeView is Fresco's top-level View class.
PipelineDraweeController: Responsible for the acquisition and loading of image data. It inherits from AbstractDraweeController and is built by PipelineDraweeControllerBuilder. AbstractDraweeController implements the DraweeController interface, and DraweeController is the data manager of Fresco, so the processing of image data is done by it.
GenericDraweeHierarchy: Responsible for layer management on SimpleDraweeView, composed of multiple layers of Drawable. Each layer of Drawable provides some functionality (for example: scaling, rounding). This class is built by GenericDraweeHierarchyBuilder, which will placeplaceImage, retryImage, failureImage, progressBarImage The xml files such as background, overlays, and pressedStateOverlay or the attribute information set in the Java code are passed to GenericDraweeHierarchy and processed by GenericDraweeHierarchy.
DraweeHolder: This class is a Holder class associated with SimpleDraweeView, which is managed by DraweeHolder. And DraweeHolder is used to unify the related Hierarchy and Controller.
DataSource: Similar to Futures in Java, it represents the source of data. Unlike Futures, it can have multiple results.
DataSubscriber: Receives the result returned by the DataSource.
ImagePipeline: used to retrieve the interface for getting images.
Producer: Load and process images, it has several implementations, such as: NetworkFetcherProducer, LocalAssetFetcherProducer, LocalFileFetchProducer. From the names of these classes we can know what they do. Producer is built by the Factory class of ProducerFactory, and all Producers are like Java's IO stream. They can be nested one level at a time, and only one result is obtained. This is a very elaborate design👍
Consumer: Used to receive the results produced by the Producer, which forms a producer and consumer model with the Producer.
Note: The names of the classes in the Fresco source code are relatively long, but they are all in accordance with certain command rules. For example, the class ending in Supplier implements the Supplier interface, which can provide a certain type of object (factory, generator, Builder, closure, etc.). The one that ends with Builder is of course the class that creates the object in constructor mode.

**How ​​to calculate the size of a picture, load the bitmap process (how to ensure no memory overflow), L2 cache, LRUCache algorithm. **

    Calculate the size of an image
    
    
    The calculation formula of the image occupied by the picture: the height of the picture * the width of the picture 
    * the size of the memory occupied by one pixel. Therefore, when calculating the size of the memory occupied by the
    picture, it is necessary to consider the directory where the picture is located and the density of the device. 
    These two factors actually affect the picture. High and wide, android will pull and compress the image.
    
    
    Load the bitmap process (how to ensure no memory overflow)
    
    
    Because Android has limited memory for images, if it loads a large number of megabytes, the memory overflows. 
    Bitmap will load all the pixels of the image (that is, the length x width) into the memory. If the resolution 
    of the image is too large, it will directly lead to the memory OOM. Only when the BitmapFactory loads the image
    , use BitmapFactory.Options to configure the related parameters to reduce the loading. Pixel.
    
    
    Detailed explanation of related parameters of BitmapFactory.Options
    
    
    (1) The value of .Options.inPreferredConfig is used to reduce memory consumption.
    
    For example: the default value of ARGB_8888 is changed to RGB_565, saving half of memory.
    
    
    (2). Set the Options.inSampleSize scaling to compress large images.
    
    
    (3). Set Options.inPurgeable and inInputShareable: Allow the system to reclaim memory in time.
    
    A:inPurgeable: When set to True, it means that it can be recycled when the system is out of memory. 
    When it is set to False, it means it cannot be recycled.
    
    B:inInputShareable: Set whether to copy deep, in combination with inPurgeable, this parameter is meaningless
    when inPurgeable is false.
    
    
    (4). Use decodeStream instead of other methods.
    
    decodeResource, setImageResource, setImageBitmap, etc.

** How is the LRUCache algorithm implemented? **

    There is a LinkedHashMap and maxSize inside, and the recently used object is stored in the LinkedHashMap 
    with a strong reference. The put and get methods are given. The total size of all the pictures in the cache
    is calculated each time the picture is put, compared with maxSize, greater than maxSize. The oldest added 
    image is removed; otherwise it is added less than maxSize.
    
    
    Previously, we would use memory caching technology, which is soft reference or weak reference. Starting 
    with Android 2.3 (APILevel 9), the garbage collector would prefer to recycle objects that hold soft or weak
    references, which makes soft references and Weak references are no longer reliable.
    
Write a picture browser and say your thoughts

**Bitmap compression strategy**

    How to load a Bitmap:
    BitmapFactory four methods:
    decodeFile(file system)
    decodeResourece( resource )
    decodeStream (input stream) 
    decodeByteArray(bytes)
    BitmapFactory.options parameter
    inSampleSize The sampling rate, which scales the height and width of the image and scales with a minimum 
    ratio (typically an exponent of 2). Usually, the width ratio of the width and height is calculated according
    to the actual size of the picture width/height and the required width and height. But you should take the smallest
    zoom ratio, to avoid scaling the image is too small, can not be covered in the specified control, need to stretch
    and cause blur.
    inJustDecodeBounds Get the width and height information of the image, and give the inSampleSize parameter to select
    the zoom ratio. By inJustDecodeBounds = true, then loading the image can achieve only parsing the width and height
    of the image, and does not actually load the image, so this operation is lightweight. When the width and height 
    information is obtained, the zoom ratio is calculated, and then the image is loaded by inJustDecodeBounds = false
    and the image is reloaded, the scaled image can be loaded.
    Process for efficiently loading Bitmaps
    Set the inJustDecodeBounds parameter of BitmapFactory.Options to true and load the image
    Extract the original width and height information of the image from BitmapFactory.Options, corresponding to the 
    outWidth and outHeight parameters
    Calculate the sampling rate inSampleSize according to the sampling rate rule and the size of the target view.
    Set the inJustDecodeBounds of BitmapFactory.Options to false to reload the image

**Bitmap processing: **

When using ImageView, the pixels of the image may be larger than ImageView. In this case, the image can be compressed
by BitmapFactory.Option, and inSampleSize is reduced by 2^(inSampleSize-1) times.

BitMap cache:

1. Use LruCache for memory caching.

2. Use DiskLruCache for hard disk caching.

3. Implement an ImageLoader process: synchronous asynchronous loading, image compression, memory hard disk caching, 
network pull

    1. Synchronous loading only creates one thread and then loads the images in order
    2. Asynchronous loading uses a thread pool, so that existing load tasks are in different threads
    3. In order not to open too many asynchronous tasks, only open the image loading when the list is still

**Image loading library related, how to handle large images, such as a 30M big picture, how to prevent OOM**

    
**[How to elegantly display a big picture of Bitmap] (http://blog.csdn.net/guolin_blog/article/details/9316683)**


** Second, network and security mechanisms**

**Android: Comparison of mainstream web requests for open source libraries (Android-Async-Http, Volley, OkHttp, Retrofit)**

Https://www.jianshu.com/p/050c6db5af5a

**How ​​to consider the security of data transmission**

    If the application does not have any security measures against the transmitted data, the attacker sets 
    the DNS server in the phishing network. This server can obtain user information or act as an intermediary to 
    exchange data with the original server. In SSL/TLS communication, the client judges whether the server is trusted
    by digital certificate, and uses the public key of the certificate to perform encrypted communication with the 
    server.

How to access the network to encrypt
1: symmetric encryption (DES, AES) and asymmetric (RSA public and private keys). (the public and private keys of the merchant in Alipay)
2: MD5 (algorithm)
3: Base64

How do you design your own network request framework?

Okhttp source

Volley vs. OkHttp:

Volley: Support for HTTPS. Cache, asynchronous request, does not support synchronous requests. The protocol type is Http/1.0, Http/1.1, the network transmission uses HttpUrlConnection/HttpClient, and the data is read and written using IO.
OkHttp: Support for HTTPS. Cache, asynchronous request, synchronization request. The protocol types are Http/1.0, Http/1.1, SPDY, Http/2.0, WebSocket. The network transmission uses the encapsulated Socket, and the data is read and written using NIO (Okio).
The SPDY protocol is similar to HTTP, but is designed to reduce the load time and security of web pages. The SPDY protocol reduces load time by compressing, multiplexing, and prioritizing.

The subsystem hierarchy diagram of Okhttp is as follows:

![image](https://github.com/guoxiaoxing/android-open-framwork-analysis/raw/master/art/okhttp/okhttp_structure.png)

Network configuration layer: Use the Builder mode to configure various parameters, such as timeout, interceptor, etc. These parameters will be distributed to each required subsystem by Okhttp.
Redirection layer: Responsible for redirection.
Header splicing layer: responsible for translating user-constructed requests into requests sent to the server, converting the response returned by the server into a user-friendly response.
HTTP cache layer: responsible for reading the cache and updating the cache.
Connection layer: The connection layer is a relatively complex level. It implements network protocol, internal interceptor, security authentication, connection and connection pool, etc., but this layer has not yet initiated a real connection, it just made a connection. The processing of some parameters.
Data Response Layer: The data responsible for reading the response from the server.
In the entire Okhttp system, we also need to understand the following key roles:

OkHttpClient: The client of the communication, used to uniformly manage the initiation request and the resolution response.
Call: Call is an interface, which is an abstract description of the HTTP request. The concrete implementation class is RealCall, which is created by CallFactory.
Request: Request, encapsulate the specific information of the request, such as: url, header, and so on.
RequestBody: Request body, used to submit request information such as streams and forms.
Response: The response of the HTTP request, and obtain response information, for example, response header.
ResponseBody: The response body of the HTTP request will be closed after being read once, so we will respond with repeated calls to responseBody.string() to get the result of the request.
Interceptor: Interceptor is a request interceptor that intercepts and processes requests. It unifies network requests, caching, transparent compression, etc. Each function is an Interceptor, and all Interceptors are finally connected into an Interceptor.Chain. A typical chain of responsibility model is implemented.
StreamAllocation: Used to control the resource allocation and release of Connections and Streas.
RouteSelector: Select route and auto reconnect.
RouteDatabase: Records the route blacklist that failed to connect.

Network request cache processing, how okhttp handles network cache

Load a 10M picture from the network, say the following considerations

Image cache, exception recovery, quality compression

How to verify the legality of the certificate?

How does the client determine that the message it sent is received by the server?

Talk about your understanding of WebSocket

The difference between WebSocket and socket

Talk about your understanding of Android signatures.

Please explain Android as a signature mechanism?

Video encrypted transmission

How is the app sandboxed, why do you want to do this?

The rights management system (how is the underlying permission to grant)?

Ways to improve app security

** Third, the database **

Database framework comparison and source code analysis

Database optimization

Database data migration problem

** Fourth, plug-in, modular, component, hot fix, incremental update, Gradle**

**How ​​to implement plug-in related technology, how to implement hot patching technology, and what is the 
difference between plug-in and **

Http://www.liuguangli.win/archives/366
Http://www.liuguangli.win/archives/387
Http://www.liuguangli.win/archives/452

    Same point:
    
    
    Both the ClassClassLoader and the DexClassLoader can be used to load new functional classes that use ClassLoader.
    
    
    difference:
    
    
    Because the hot fix is ​​to fix the bug, you need to replace the bug with the same name with the same name. 
    You must first load the new class instead of the bug class, so do two more things: when the original app is 
    packaged, block the related The class is marked with the CLASS_ISPREVERIFIED flag, and the dexElements that 
    indirectly reference the BaseDexClassLoader object are dynamically changed during the hot fix, so that the 
    bug class can be replaced first, and the system does not load the old bug class.
    
    
    Plug-in is just a new functional class or resource file, so it doesn't involve the task of preloading the old
    class. It avoids blocking related classes from playing the CLASS_ISPREVERIFIED flag and dynamically changing 
    the BaseDexClassLoader object in the hot fix. Quoted dexElements.
    
    So plug-in is simpler than hot-fixing, and hot-repair is based on plug-in.
    
**Understanding plugins and hot fixes, what is the difference between them and how do you understand them? **

Plug-in: Plug-in is reflected in the function split, it extracts a function independently, independently developed
, independently tested, and then inserted into the main application. In turn, the size of the main application is
less.
Hot fix: Hot fix is ​​reflected in bug fixes. It does not require re-release and re-installation to fix known bugs.
Use PathClassLoader and DexClassLoader to load the class with the same name as the bug class, replace the bug class,
and then fix the bug. The principle is to prevent the class from being marked with CLASS_ISPREVERIFIED when the app 
is packaged, and then dynamically change the BaseDexClassLoader object indirect reference during hot fix. The 
dexElements, replace the old class.

At present, the hot fix framework is mainly divided into two categories:

Sophix: Modify method pointers.
Tinker: Modify the dex array element.

**Hot patch**

    Reason: Because the storage method id in a dvm uses the short type, the method in dex cannot exceed 65,536.
    Principle: The compiled class file is split into two dex, bypassing the limit of the number of dex methods 
    and checking during installation, and dynamically loading the second dex file at runtime. Use Dexclassloader.

**dynamic loading (also known as plugin technology)**

    Dynamic loading mainly solves 3 technical problems:
    1, use the ClassLoader to load the class.
    2, resource access.
    3, life cycle management.
    
**The benefits of modularity**

Https://www.jianshu.com/p/376ea8a19a17

The principle of Android componentization, as well as some of the problems that componentization usually uses;


Understanding of hot fixes and plugins

Why use plug-in, plug-in framework comparison, combing plug-in architecture

Plug-in principle analysis

Modular implementation (benefit, cause)

Project componentization

Describe what happened after clicking the build button of Android Studio

Gradile familiar, automatic packaging knows what?

How to speed up Gradle's compilation speed

** V. Architecture design and design mode**

Architecture design

![image](http://www.jackywang.tech/AndroidInterview-QA/picture/architucture.png)

http://www.tianmaying.com/tutorial/AndroidMVC

Talk about your understanding of Android design patterns

MVC MVP MVVM principle and difference

What design patterns do you know?

Design patterns commonly used in projects

Handwritten producer/consumer model

Adapter mode, decorator mode, appearance mode similarities and differences?

Some open source frameworks are used to introduce an internal implementation process that has seen the source code.

Talk about the understanding of RxJava

The function and principle of RxJava

The advantages and disadvantages of RxJava compared to the asynchronous operations usually used

Talk about the role of EventBus, the way to achieve, instead of EventBus

Design an App overall architecture from 0, how to do it?

Say an application that you think is currently hot and design (eg: live app, P2P finance, small video, etc.)

Talk about the understanding of java state machine

How should Fragment be decoupled if used in Adapter?

Binder mechanism and the underlying implementation

How is this for the application update? (Answer: grayscale, forced update, sub-regional update)?

Implement a Json parser (can improve speed by regularity)

Statistical start time, standard

** Sixth, performance optimization**


How to perform performance analysis and optimization for Android apps?

Ddms and traceView

How performance optimization analyzes systrace

How to analyze memory leaks with IDE

Java multi-threaded performance problems, how to solve

Start page white screen and black screen solution?

How to solve if the startup is too slow

How to ensure that the application does not start?

App launch crash exception capture

Custom View Considerations

Render frame rate, memory

Now the download speed is very slow, try to analyze the reasons from the perspective of the network
protocol, and optimize (hint: the 5 layers of the network can be involved).

Https request slow solution (hint: DNS, carry data, direct access to IP)

How to maintain application stability

Performance comparison between RecyclerView and ListView

ListView optimization

RecycleView optimization

View rendering

How does Bitmap handle large images, such as a large 30M image, how to prevent OOM

The difference between the four references in java and the usage scenarios

Strong reference is set to null, will it be recycled?

How to handle App startup process optimization

** Seven, NDK, jni, Binder, AIDL, process communication related **

**What is the full name of AIDL? How does it work? What types of data can I process?**

Http://blog.csdn.net/singwhatiwanna/article/details/17041691

AIDL (Android Interface Definition Language) is an IDL language used to generate code for interprocess communication (IPC) between two processes on an Android device. If you want to call an operation of another process (such as a Service) object in a process (such as an Activity), you can use AIDL to generate serializable parameters. The AIDL IPC mechanism is interface oriented, like COM or Corba, but more lightweight. It uses a proxy class to pass data between the client and the implementation.

    AIDL's full name Android Interface Definition Language is an interface description language.
    The compiler can generate a piece of code through the aidl file to achieve the purpose of cross-
    border access to the internal communication process of two processes through a predefined interface
    . AIDL The IPC mechanism is similar to COM or CORBA and is interface based, but it is lightweight. 
    It uses a proxy class to pass values ​​between the client and the implementation layer. If you want
    to use AIDL, you need to do two things: 1. Introduce the relevant class of AIDL. 2. Call the class
    generated by aidl.In theory, parameters can pass primitive data types and Strings, as well as Bundle derived classes,
    but in Eclipse, the current ADT does not support Bundles as arguments.
    The specific implementation steps are as follows:
    1. Create an AIDL file, define an interface in this file, which defines methods and properties that
    are accessible to the client.
    2, compile the AIDL file, with Ant, you may need to manually, using the Eclipse plugin, you can automatically
    produce java files and compile according to the adil file, no human intervention.
    3. In the Java file, implement the interface defined in AIDL. The compiler will generate a JAVA 
    interface according to the AIDL interface. This interface has an internal abstract class called Stub 
    that inherits several methods needed to extend the interface and implement remote calls. Next, you need
    to implement a few custom    interfaces yourself.
    4. Provide the interface ITaskBinder to the client. If the service is written, extend the Service
    and override the onBind() method to return an instance of the class that implements the above interface.
    5, on the server side callback client function. The premise is that when the client gets the IBinder
    interface, you have to register the callback function, only then, the server side knows which function to call
    The AIDL syntax is simple and can be used to declare an interface with one or more methods, as 
    well as pass parameters and return values. Due to the need for remote calls, these parameters and 
    return values ​​are not of any type. Here are some of the data types supported by AIDL:
    
    Simple Java programming language type (int, boolean, etc.) that does not require an import declaration
    String, CharSequence does not require special declaration
    List, Map and Parcelables types, the data members contained in these types can only 
    be simple data types, String and other types than supported.
    (Also: I didn't try Parcelables, compile under Eclipse+ADT, but maybe I will support it later).
    There are several principles when implementing an interface:
    1. Throwing exceptions should not be returned to the caller. It is not advisable to throw exception
    handling across processes.
    2. The IPC call is synchronous. If you know that an IPC service takes more than a few milliseconds
    to complete, you should avoid calling it in the main thread of the Activity. That is, the IPC call
    will suspend the application and cause the interface to lose response. This situation should be 
    considered a single thread to handle.
    3. Static properties cannot be declared in the AIDL interface.
    IPC call steps:
    Declare a variable of the interface type defined in the .aidl file.
    Implement a ServiceConnection.
    Call ApplicationContext.bindService() and pass it in the ServiceConnection implementation.
    In the ServiceConnection.onServiceConnected() implementation, you will receive an IBinder 
    instance (the called Service).
    YourInterfaceName.Stub.asInterface((IBinder)service) converts the argument to the YourInterface
    type.
    Call the method defined in the interface. You always have to detect a DeadObjectException, 
    which is thrown when the connection is broken. It will only be thrown by remote methods.
    Disconnect, call ApplicationContext.unbindService() in the interface instance
    Aidl is mainly to help us complete the process of packaging data and unpacking, and call the 
    transact process, and the data packet used for delivery is called parcel.
    
    AIDL: xxx.aidl->xxx.java, register service
    
    Use aidl to define the method interface that needs to be called
    Implement these methods
    Call these methods

Said the process of binder directing and deserialization, and the process of use

**Inter-Process-Communication on Android works when communicating across processes**

Cross-process communication mainly depends on Binder

Https://blog.csdn.net/carson_ho/article/details/73560642

**Android IPC: Binder principle. **

IPC:

![image](https://user-gold-cdn.xitu.io/2017/10/10/a1cd0604f7807e215047053498e6daad?imageslim)

    Communication, using the kernel memory space shared between processes to complete the underlying 
    communication work, the client side and the server side process often use ioctl and other methods to 
    interact with the kernel space driver.
    Binder principle:
    Binder communication adopts C/S architecture, including Client, Server, ServiceManager and binder 
    drivers. ServiceManager is used to manage various services in the system.
The architecture diagram is as follows:

![image](https://raw.githubusercontent.com/wangkuiwu/android_applets/master/os/pic/binder/binder_frame.jpg)

    Binder four roles:
    Client process: the process that uses the service
    Server process: the process that provides the service
    The ServiceManager process: converts the Binder name of the character type to a reference to the Binder
    in the Client, so that the Client can obtain a reference to the Binder entity in the Server through the
    Binder name.
    Binder driver: the establishment of Binder communication between processes, the transfer of Binder 
    between processes, the management of Binder reference counting, the transmission and interaction of 
    data packets between processes, and a series of underlying support.
    Binder operating mechanism:
    Registration Service: Server Register Service in ServiceManager
    Get the service: Client gets the corresponding Service from ServiceManager
    Using the service: The client establishes a communication path with the Server process in which the 
    Service is located according to the obtained Service information to interact with the Server.
    
Android Binder is used for process communication. Android applications and system services are running in separate processes, and their communication depends on Binder.

Why choose Binder? Before discussing this issue, we know that Android is also based on the Linux kernel. The existing communication methods of Linux are as follows:

Pipeline: allocate a page size memory when creating, the size of the buffer area is limited;
Message Queuing: Information is copied twice, additional CPU consumption; communication that is not suitable for frequent or informative;
Shared memory: no need to copy, the shared buffer is directly attached to the process virtual address space, and the speed is fast; but the synchronization problem between processes cannot be realized by the operating system, and each process must be solved by using a synchronization tool;
Socket: As a more general interface, the transmission efficiency is low, mainly used for communication without a machine or across networks;
Semaphore: Often used as a lock mechanism to prevent a process from accessing a shared resource, other processes also access the resource. Therefore, it is mainly used as a means of synchronization between processes and between different threads within the same process. 6. Signal: Not applicable to information exchange, more suitable for process interrupt control, such as illegal memory access, killing a process, etc.;
Since there is an existing IPC method, why redesign a Binder mechanism? Mainly due to the consideration of the above three aspects:

High performance: From the number of data copies, Binder only needs to make a memory copy, and the pipeline, message queue, and Socket need to be used twice. The shared memory does not need to be copied. The performance of Binder is second only to shared memory.
Stability: The performance of shared memory is better than Binder. Why not use shared memory? Because shared memory needs to handle concurrent synchronization problems, control is responsible, and it is prone to deadlock and resource competition, and the stability is poor. Binder is based on the C/S architecture, and the client and server are independent of each other and have good stability.
Security: We know that Android assigns a UID to each application, which is used as an important indicator of the authentication process. Android also relies on this UID for rights management, including fixed permissions before 6.0 and dynamic permissions after 6.0. The UID/PID can be filled in the data packet by the user. This tag is completely controlled in the user space, and is not placed in the kernel space, so there is a possibility of malicious tampering, so the Binder is more secure.


**Binder mechanism**

Https://www.zhihu.com/question/39440766
Http://gityuan.com/2016/09/04/binder-start-service/
http://baronzhang.com/blog/Android/%E5%86%99%E7%BB%99-Android-%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8% 8B%E5%B8%88%E7%9A%84-Binder-%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90/
Https://blog.csdn.net/prike/article/details/70195608
Http://blog4jimmy.com/2018/01/356.html

Lao Luo:
Https://blog.csdn.net/luoshengyang/article/details/6618363

    In the Android system, each application runs in a separate process, which also ensures that one of the 
    programs has an exception and does not affect the normal operation of the other application. In many cases,
    our activity will deal with the service of various systems. Obviously, the activity and system service we 
    write in the program are definitely not the same process, but how do they communicate with each other?


    So Binder is one of the ways to implement interprocess communication (IPC) in android.


1). First, Binder is divided into two processes: Client and Server.


    Note that Client and Server are relative. Who sent the message, who is the Client, who receives the message,
    who is the Server.
    
    
    For example, two processes A and B use Binder communication, process A sends a message to process B, then 
    A is Binder Client, B is Binder Server; process B sends a message to process A, then B is Binder Client, 
    A is Binder Server - in fact, although it is simple, but still not very rigorous, we understand this first.


2). Secondly, let's look at the following picture (excerpted from the blog of Tian Weishu), which basically explains the deconstruction of Binder's composition:

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd565735a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

The IPC in the figure is the meaning of interprocess communication.

The ServiceManager in the figure is responsible for registering the Binder Server into a container.


    Some people compare ServiceManager to a telephone office and store the landline phone number of each home
    . It is still quite appropriate. Zhang San called Li Si, dialed the telephone number, and then transferred
    to the telephone office. The operator of the telephone office checked the address of the telephone number.
    Because Li Si’s telephone number was registered with the telephone office before, it can be dialed. If you 
    do not register, you will be prompted that the number does not exist.
    
    
    In contrast to the Android Binder mechanism, against the above picture, Zhang San is the Binder Client, Li
    Si is the Binder Server, and the telephone office is the ServiceManager. The operator of the 
    telephone office has done a lot of things in this process, corresponding to the Binder driver in the figure.


3). Next, let's look at the process of Binder communication, or take a picture from the Tianwei blog:

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd6cf0739?imageslim)

Note: The SM in the figure is also the ServiceManager.


    We see that the Client wants to call the Server's add method directly, it is not possible, because they are
    in different processes, this time you need Binder to help.
    
    
    The first is that Server is registered in the SM container.
    
    Secondly, if the client wants to call the add method of the server, it needs to obtain the Server object 
    first, but the SM does not return the real Server object to the Client, but returns a proxy object of the
    Server to the Client, which is the Proxy.
    
    Then, the Client calls the add method of the Proxy, and the SM will help him to call the add method of the
    Server and return the result to the Client.
    
    
    The above 3 steps, Binder driver has a lot of power, but we do not need to know the underlying implementation
    of the Binder driver, involving C + + code - to spend more time to do more meaningful things.

2. Why does android choose Binder to achieve interprocess communication?

    1). Reliability. On mobile devices, Client-Server-based communication is usually used to implement internal communication between the Internet and devices. At present, Linux supports IPC including traditional pipeline, System V IPC, namely message queue/shared memory/semaphore, and only sockets support the client-server communication method in the socket. The Android system provides developers with a rich interface for interprocess communication, media playback, sensors, and wireless transmission.
    
    
    These features are managed by different servers. Development only cares about establishing the communication between the client and server of the application and then using the service. There is no doubt that if a set of protocols is set up on the underlying to implement Client-Server communication, the complexity of the system is increased. To achieve this complex environment on a mobile phone with limited resources, reliability is difficult to guarantee.
    
    
    2). Transmission performance. The socket is mainly used for inter-process communication across the network and communication between processes on the local machine, but the transmission efficiency is low and the overhead is large. The message queue and pipe adopt the store-and-forward method, that is, the data is first copied from the sender buffer to a buffer opened by the kernel, and then copied from the kernel buffer to the receiver buffer, and the process has at least two copies. Although shared memory does not require copying, the control is complicated. Compare the number of data copies of various IPC methods. Shared memory: 0 times. Binder: 1 time. Socket/pipe/message queue: 2 times.
    
    
    3). Security. Android is an open platform, so it's important to keep your application secure. Android assigns a UID/PID to each installed application, where the UID of the process can be used to identify the process identity. Traditionally, users can only fill in UID/PID in the data packet, which is unreliable and easy to be exploited by malicious programs. And we require the kernel to add a reliable UID.
    
    
    Therefore, for reliability, transmission, and security. Android has established a new set of interprocess communication methods.

Binder principle:

    1. When the Activity and Service communicate, Binder is used.
        1. When belonging to the same process, we can inherit Binder and then operate the Service in the Activity.
        2. When not belonging to the same process, then use AIDL to let the system create a Binder for us, 
        and then operate the remote Service in the Activity.
    2. The Binder generated by the system:
        1. Stub class: id of the interface method, there is the identity of the Binder, there is asInterface 
        (IBinder) (Let us get the interface that implements Binder in the Activity, the implementation of the 
        interface is in the Service, return the Stub when the same process, otherwise return Proxy), there is 
        onTransact () this method is to allow the Proxy to remotely call the Activity in the different processes
        to achieve the Activity operation Service
        2.Proxy class is a proxy, in the Activity side, which is: IBinder mRemote (this is the remote Binder), 
        the implementation of the two interfaces is only the proxy or the actual operation in the remote onTransact ().
    3. At the end of the Binder is a copy, the end can be operated by the other end, because the Binder body can
    operate the local thing when it is defined. So you can pass the Binder on the Activity side, let the Service
    side operate on it as a Listener. You can use the RemoteCallbackList container to install the Listener to prevent
    the Listener from having problems caused by serialization.
    4. When the Activity side is called to the far end, the current thread will hang and will wake up when the method
    is processed.
    5. If an AIDL is too expensive to use a Service, you can use the Binder pool to create an AIDL. The method is to
    return IBinder, and then return the specific AIDL according to the parameters passed in the method.
    6. IPC methods are: Bundle (incoming when Intent is started, but one-time), file sharing (for SharedPreference 
    is a special case, because it will have a cache in memory), using Messenger (the underlying is also used AIDL, 
    which side to operate the same, on which side to define Messenger), AIDL, ContentProvider (in the process of 
    inheriting the implementation of a ContentProvider, in the addition, deletion and change method call the SQLite
    of the process, query in other processes), Socket

Please introduce NDK

What is the NDK library?

Have you used jni?

How to register native functions in jni, there are several ways to register


How does Java call c, c++ language?

**Java calls C++**

Declaring native methods in Java (that is, local methods that need to be called)
Compile the above Java source file javac (get the .class file) 3. Export JNI header files (.h files) via the javah command
Native methods that are declared in Java using native code that Java needs to interact with
Compile the .so library file
Execute a Java program through Java commands, and finally implement Java to call native code
C++ calls Java

Search for the ClassMethod class from the classpath and return the Class object of the class.
Get the default constructor ID of the class.
Find the ID of the instance method.
Create an instance of this class.
Call the instance method of the object.
    JNIEXPORT void JNICALL Java_com_study_jnilearn_AccessMethod_callJavaInstaceMethod  
    (JNIEnv *env, jclass cls)  
    {  
        Jclass clazz = NULL;  
        Jobject jobj = NULL;  
        jmethodID mid_construct = NULL;  
        jmethodID mid_instance = NULL;  
        Jstring str_arg = NULL;  
        // 1. Search for the ClassMethod class from the classpath and return the Class object of the class.  
        Clazz = (*env)->FindClass(env, "com/study/jnilearn/ClassMethod");  
        If (clazz == NULL) {  
            Printf("The class 'com.study.jnilearn.ClassMethod' could not be found");  
            Return;  
        }  
    
        // 2, get the default constructor ID of the class  
        Mid_construct = (*env)->GetMethodID(env,clazz, "<init>","()V");  
        If (mid_construct == NULL) {  
            Printf ("The default constructor could not be found");  
            Return;  
        }  
    
        // 3, find the ID of the instance method  
        Mid_instance = (*env)->GetMethodID(env, clazz, "callInstanceMethod", "(Ljava/lang/String;I)V");  
        If (mid_instance == NULL) {  
    
            Return;  
        }  
    
        // 4, create an instance of the class  
        Jobj = (*env)->NewObject(env,clazz,mid_construct);  
        If (jobj == NULL) {  
            Printf ("callInstanceMethod method not found in the com.study.jnilearn.ClassMethod class");  
            Return;  
        }  
    
        // 5, call the instance method of the object  
        Str_arg = (*env)->NewStringUTF(env, "I am an instance method");  
        (*env)->CallVoidMethod(env,jobj,mid_instance,str_arg,200);  
    
        // delete local references  
        (*env)->DeleteLocalRef(env,clazz);  
        (*env)->DeleteLocalRef(env,jobj);  
        (*env)->DeleteLocalRef(env,str_arg);  
    }  
    
** Eight, Android Framework related **

**Android important term explanation**

1.ActivityManagerServices, referred to as AMS, server object, responsible for the life cycle of all activities in the system

2.ActivityThread, the real entry of the App. After the App is opened, main() is called to start running, and the message loop queue is started. This is the legendary UI thread or the main thread. Together with ActivityManagerServices, complete the management of the Activity

3.ApplicationThread, used to achieve the interaction between ActivityManagerServie and ActivityThread. When ActivityManagerSevice needs to manage the life week of the Activity in the relevant Application, it communicates with ActivityThrad through the proxy object of ApplicationThread.

4.ApplicationThreadProxy, is the agent of ApplicationThread on the server side, responsible for communication with the application Thread of the client. AMS communicates with ActivityThread through the proxy.

5.Instrumentation, each application has only one Instrumetation object, and each Activity has a reference to the object. Instrumentation can be understood as the administrator of the application process. When Activityhread wants to create or pause an Activity, it needs to be performed by Instruentation. Specific operations.

6.ActivityStack, Activity in the AMS stack management, used to record the relationship between the start of the activity, status information. Use ActivtyStack to decide if you need to start a new process.

7.ActivityRecord, ActivityStack management object, each Acivity corresponds to an ActivityRecord in the AMS, to record the Activity status and other management information. In fact, it is an image of the Activit object on the server side.

8.TaskRecord, the concept of a "task" abstracted by AMS is the stack of ActivityRecord, and a "Task" contains several ActivitRecords. AMS uses TaskRecord to ensure the order of activity startup and exit. If you know the four launchModes of the Activity, then you should be familiar with this concept.
    
**Understanding Window and WindowManager**

1.Window is used to display View and receive various events. Window has three types: Application Window (each Activity corresponds to a Window), Child Widow (cannot exist alone, attached to a specific Window), System window (oast and status bar)

2.Window hierarchical level, application Window in 1-99, child Window in 1000-999, system Window in 2000-2999. WindowManager provides three functions to increase and change View.

3.Window is an abstract concept: each Window corresponds to a ViewViewRootImpl, Window establishes a relationship with View through ViewRootImpl, View is the entity that exists in Window, and can only access Window through WindowManage.

The implementation of WindowManager is WindowManagerImpl, which then delegates WindowManagerGlobal to operate on Window. There are four Lists respectively storing the corresponding View, ViewRootImpl, WindowMange.LayoutParams and the View being deleted.

5.Window entity is the WindowMangerService that exists at the far end, so adding or deleting the Window at the local end is to modify the above List and then redraw the View through ViewRootImpl, and modify the Window at the far end through WindowSession (each one).

6.Activity creates Window:Activity will create Wndow in attach() and set its callback (onAttachedToWindow(), dispatchTochEvent()). The Activity's Window is created by the Policy class to create PhoneWndow. Then call the setContentView of the honeWindow via Activity#setContentView().

**The startup process of the next four components, the way in which the four components are started and destroyed. **

**Android Framework layer has not understood, talk about the process of adding Window window; **

**ActivityThread works ** 

**Optimized upgrade point for Android dalvik virtual machine and Art virtual machine**

**Android2 virtual machine difference (a 5.0 before, after a 5.0)**

**ART and Dalvik difference**

The application on the art starts fast and runs fast, but it consumes more storage space and has a long installation time. In general, the effect of ART is “space for time”.

ART: Ahead of Time
Dalvik: Just in Time

What is Dalvik: Dalvik is Google's own Java virtual machine designed for the Android platform. The Dalvik virtual machine is one of the core components of the Android mobile device platform developed by Google and other vendors. It can support the running of Java applications that have been converted to .dex (Dalvik Executable) format. The .dex format is designed for Dalvik applications. A compression format designed for systems with limited memory and processor speed. Dalvik is optimized to allow multiple instances of virtual machines to run simultaneously in limited memory, and each Dalvik application is executed as a separate Linux process. A separate process prevents all programs from being closed when the virtual machine crashes.

What is ART: The Android operating system is mature, and Google's Android team is turning its attention to some of the underlying components, one of which is the Dalvik runtime that is responsible for running the application. Google developers have spent two years developing faster and more efficient alternative power-saving ART runtimes. ART stands for Android Runtime, and its way of handling application execution is completely different from Dalvik, which relies on a Just-In-Time (JIT) compiler to interpret bytecode. The developer's compiled application code needs to run on the user's device through an interpreter. This mechanism is not efficient, but makes it easier for applications to run on different hardware and architectures. ART completely changed the practice of pre-compiling bytecode into machine language when the application was installed. This mechanism is called Ahead-Of-Time (AOT) compilation. After the process of removing the interpreted code, the application execution will be more efficient and start faster.

ART advantages:

Significant improvement in system performance
Apps get faster, run faster, experience smoother, and feel more timely.
Longer battery life
Support for lower hardware

ART disadvantages:
Larger storage space may increase by 10%-20%
Longer application installation time

**Basic understanding of Dalvik, ART virtual machines**

Https://blog.csdn.net/jason0539/article/details/50440669

http://www.jackywang.tech/2017/08/21/%E5%85%B3%E4%BA%8EDalvik%EF%BC%8C%E6%88%91%E4%BB%AC%E8%AF %A5%E7%9F%A5%E9%81%93%E4%BA%9B%E4%BB%80%E4%B9%88%EF%BC%9F/

**How ​​is the app sandboxed in Android? Why do you want to do this**

** Rights Management System**

Https://juejin.im/entry/57a99fba5bbb500064418fc0

**Talk about the apk packaging process; **

Android package file APK is divided into two parts: code and resources, so the packaging aspect is divided into two aspects of resource packaging and code packaging. This article analyzes the principle of compilation and packaging of resources and code.

The overall packaging process of the APK is shown below:

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/vm/apk_package_flow.png)

Specifically:

The AAPT tool is used to package resource files (including AndroidManifest.xml, layout files, various xml resources, etc.) to generate R.java files.
The AIDL file is processed by the AIDL tool to generate the corresponding Java file.
Compile the project source code through Javac tools and generate the Class file.
Convert all Class files into DEX files through DX tool. This process mainly completes the conversion of Java bytecode into Dalvik bytecode, compressing constant pool and clearing redundant information.
Package the resource file and DEX file to generate the APK file through the ApkBuilder tool.
Sign the generated APK file with KeyStore.
If it is the official version of the APK, it will also use the ZipAlign tool for alignment. The alignment process is to offset the starting distance of all resource file examples in the APK file by an integer multiple of 4 bytes, so that the APK is accessed through memory mapping. The file will be faster.

**Introduction to the Android application startup process**

The whole application startup process has to perform many steps, but overall, it is mainly divided into the following five stages:

1. The Launcher notifies the ActityManagerService through the Binder interprocess communication mechanism, which starts an Activity;

II.: ActivityManagerService informs the Launcher to enter the Paused state through the Binder interprocess mechanism;

3. Launcher informs the ActityManagerService through the Binder inter-process communication mechanism, it is ready to enter the Paused state, so the ActivityManagerService creates a new process, which is used to start an ActivityThread instance, and the Activity to be started is run in the ActivityThread instance;
    
IV. ActivityThread passes an ApplicationThread type Binder object to the ActivityManagerService through the Binder interprocess communication mechanism, so that the ActivityManagerService can communicate with it through the Binder object.
    
Five: ActivityManagerService informs ActivityThread through the Binder process communication mechanism, now everything is ready, it can actually perform the activity start operation.

**App startup process, starting from clicking on the desktop**

After clicking the application icon, it will launch the LauncherActivity of the application. If the process where the LancerActivity is located is not created, a new process will be created. The overall process is the startup process of an Activity.

The activity startup flow chart (zoom in to view) is as follows:

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/component/activity_start_flow.png)

The main roles involved in the entire process are:

Instrumentation: Monitors application-related interactions with the system.
AMS: Component Management Dispatch Center, do nothing, but manage everything.
ActivityStarter: The controller started by Activity, handles the impact of Intent and Flag on Activity startup, specifically: 1 Find the Activity that meets the startup conditions, if there are multiple, let the user choose; 2 Verify the validity of the startup parameters; 3 Returns the int parameter, indicating whether the Activity started successfully.
ActivityStackSupervisior: The role of this class can be seen from its name, which is used to manage the task stack.
ActivityStack: used to manage the Activity in the task stack.
ActivityThread: The person who finally works is the internal class of ActivityThread. The operations of Activity, Service, and BroadcastReceiver start, switch, and schedule are all completed in this class.
Note: Here is a separate mention of ActivityStackSupervisior, which is a class that is only available in the high version. It is used to manage multiple ActivityStacks. In the early version, only one ActivityStack corresponds to the mobile phone screen. Later, after the high version supports multiple screens, there are multiple ActivityStacks. So, the ActivityStackSupervisior is introduced to manage multiple ActivityStacks.

The entire process involves four processes:

The caller process, if the application is launched on the desktop, is the Launcher application process.
The System Server process where the ActivityManagerService is located, which runs the system service component.
Zygote process, the process is mainly used to fork new processes.
The newly started application process, which is used to host the application running process, it is also the application's main thread (the newly created process is the main thread), processing component life cycle, interface drawing and other related matters.
With the above understanding, the whole process can be summarized as follows:

Click the desktop application icon, and the Launcher process will send the Activity (MainActivity) request to the AMS in Binder mode.
After receiving the start request, the AMS delivers the ActivityStarter to process the Intent and Flag information, and then hands it to the ActivityStackSupervisior/ActivityStack to process the Activity into the stack. At the same time, the Zygote process is called to the new process in Socket mode.
After receiving the new process creation request, Zygote forks the new process.
Create an ActivityThread object in the new process, the newly created process is the main thread of the application, open the Looper message loop in the main thread, and start processing to create the Activity.
ActivityThread uses the ClassLoader to load the Activity, create an Activity instance, and callback the Activity's onCreate() method. This completes the activation of the Activity.

![image](http://img.mp.itc.cn/upload/20170329/ca9567ce3bf04c4abdb4d124cebfee76_th.jpeg)

Http://www.sohu.com/a/130814934_675634

**What happened when an app was installed on your phone**

Http://www.androidchina.net/6667.html

**Android system startup process, App startup process** 
    
From the desktop click to the activity startup process

1.Launcher thread captures the click event of onclick, calls LauncherActivitySafely, further calls Launcher.startActivty, and finally calls the startActivity of the parent class Activity.

2.Activity and ActivityManagerService interaction, introduce Instrmentation, hand the startup request to Instrumentation, call Insrumentation.execStartActivity.

3. Call ActivityManagerService's startActivity method to do process switching here (see the source code for the specific process).

4. Open the Activity and call the onCreate method.
    
**The difference between JVM and Dalvik virtual machine**

JVM: .java -> javac -> .class -> jar -> .jar
    
Architecture: The architecture of the heap and stack.

DVM: .java -> javac -> .class -> dx.bat -> .dex

Architecture: Register (a block of cache on the cpu)

**Zygote startup process**

In the Android system, zygote is the name of a process. Android is based on Linux System. When your phone is booted, a process called "init" will be started after the Linux kernel is loaded. In the Linux System, all processes are forked by the init process, and our zygote process is no exception.

    
**Android view drawing mechanism and loading process, please elaborate the whole process**

1.ViewRootImpl will call performTraversals(), which will use performMeasure(), performLayout, performDraw() internally.

2.performMeasure() will call the measur()-->onMeasure() of the outermost ViewGroup, and the onMeasure() of the ViewGroup is the abstract side, but it provides measureChildren(), which will traverse the sub-Vie and then call it in a loop. MeasureChild() will get the MeasureSpec of this View using getChildMeasueSpec()+ LayoutSpec+ of the parent View's MeasureSpec+ child View, then call the measur() of the child View to the view's onMeasure()-->setMeasureDimension(getDeaultSize(), getDefaultSize()), getDefaultSize() returns the measured value of measureSpec by default, so inheriting View to customize wrap_content needs to be rewritten.

3.performLayout() will call the layout(,t,r,b) of the outermost ViewGroup. This View uses setFrame() to set the four vertex positions of this View. In the onLayout (abstract method) to determine the position of the child View, such as LinearLayout will traverse the child View, loop call setChildFrame--> child View.layout ().

4.performDraw() will call the draw() of the outermost ViewGroup: it will call background.draw() (draw background), onDraw() (self), dispatchDraw() (draw View), onDrawScrollars() (Draw a decoration).

5.MeasureSpec consists of 2 bits SpecMode (UNSPECIFIED, EXACTLY (should exact value and match_parent), AT_MOST (corresponding to warp_content) and 30-bit SpecSize to form an int. DecorView's MeasureSpe is determined by the window size and its LayoutParams. Other Views are controlled by the parent ViewMeasureSpec and this View's LayoutParams decision. ViewGroup has getChildMeasureSpec() to get the MeasureSpec of the child View.

6. Three ways to get the width and height after measure():

- Get the call in Activity#onWindowFocusChange()
- view.post(Runnable) posts the fetched code to the end of the message queue.
- ViewTreeObservable.
    
**Activity of the activty please explain in detail: **

1.Activity finally to startActivityForResult() (mMainhread.getApplicationThread() passed an Applicationhread to check APT)
->Instrumentation#execStartActivity() and checkStartctivityResult() (This is to determine whether Activty is started successfully after starting the Activity. For example, if it is not registered in AM, it will report an error)
->ActivityManagerNative.getDefault().startActivit() (similar to AIDL, implements IAM, which is actually implemented by remote AMS statActivity())
->ActivityStackSupervisor#startActivityMayWait()
->ActivityStack#resumeTopActivityInnerLocked
-> ActivityStackSupervisor#realStartActivityLocked (call APT's scheduleLaunchActivity here, also AID, but the application thread is called at the remote end)
->ApplicationThread#scheduleLaunchActivity() (a thread of this process, used as a service to accept the AMSclient side)
->ActivityThread#handleLaunchActivity() (receives the message of class H, the ApplicationThread thread sends a LAUNCH_ACTIVTY message to H)
-> Finally, in the ActivityThread#performLaunchActivity (), the start of the Activity has completed the following things:

2. Get the component information of the Activty to be started from the incoming ActivityClientRecord

3. Create a class loader, using Instrumentation#newActivity (load Activity object)

4. Call the LoadedApk.makeApplication method to try to create an Appliction, which will not be created repeatedly due to the singleton.

5. Create a Context implementation class ContextImpl object, and complete the data initialization and Context through Activty#attach (), because Ativity is the bridge class of Context,
The last is to create and associate the window, let the event received by the Window pass to the Ativity, in the window creation process will call the ViewRootImpl prformTraversals () to initialize the View.

6.Instrumentation#callActivityOnCreate()->Activit#performCreate()->Activity#onCreate().onCreate() will call PhoneWindow's stContentView() via Activity#setContentView()
Update the interface.    

**OSGI**

**Describe what happened after clicking the build button of Android Studio**

**Generally what happens when an app is installed on your phone;**

The installation process for the APK is as follows:

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/package/apk_install_structure.png)

Copy the APK to the /data/app directory, extract and scan the installation package.
The resource manager parses the resource files in the APK.
Parse the AndroidManifest file and create the corresponding application data directory in the /data/data/ directory.
The dex file is then optimized and saved in the dalvik-cache directory.
Register the four component information parsed from the AndroidManifest file into the PackageManagerService.
After the installation is complete, send the broadcast.

**How ​​Inter-Process-Communication works on cross-process communication on Android;**

** Rights Management System (the underlying permissions are how to grant)**

What are the differences between **adb install and pms scan? **

**How ​​was a picture found after calling R.id in the app? **

**What is the technology of Android rights management? **

**Please describe the boot process and shutdown process? **

**Android ++ Smart pointer related use introduction? **

**What are the related operations of PowerManagerService? What are the processes for the system to turn off the screen? **

**How ​​does AMS manage Activity**

**Hook and instrumentation technology**

**What is familiar with the source code of the android api layer? explain**

**What do you know about Dalvik and ART virtual machines? **

**Virtual machine principle, how to design a virtual machine (memory management, class loading, parental delegation)**

**Ubuntu compiles Android **

**What is the system startup process? (Hint: Zygote Process –> SystemServer Process –> Various System Services –> Application Process)**    
    
** Nine, other high frequency interview questions**

**Event delivery mechanism**

![image](https://upload-images.jianshu.io/upload_images/2911038-5349d6ebb32372da)

1). The essence of the Android event distribution mechanism is to solve: the click event is sent by an object, through which objects, which object is finally reached and finally processed. The objects here are Activity, ViewGroup, View.

2). Event distribution order in Android: Activity(Window) ->ViewGroup -> View.

3). The event distribution process is assisted by three parties: dispatchTouchEvent(), onInterceptTouchEvent(), and onTouchEvent().

Set the Button button to respond to the click event event delivery: (as shown below)

The layout is as follows:

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfaed36b0e?imageslim)
    
The outermost layer: Activy A, contains two sub-views: ViewGroupB, View C

Middle layer: ViewGroup B, containing a child View: View C

Innermost layer: View C

Suppose the user first touches a point on the ViewC on the screen (the yellow area in the figure), then the Action_DOWN event is generated at that point, then the user moves the finger and finally leaves the screen.

Button click event:

The DOWN event is passed to C's onTouchEvent method, which returns tre to handle the event;

Since C is processing this event, the DOWN event will no longer be uploaded to B and A's onTouchEvent();

The other events in the event column (Move, Up) will also be passed to C's onToucEvent();

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd00ab478?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![image](https://upload-images.jianshu.io/upload_images/2911038-5349d6ebb32372da)

(Remember the order of delivery of this diagram, it can be drawn when interviewing, it is very detailed)

Please write more than four design patterns that you know (such as where to use the observer mode in Android, singleton mode related), and introduce the implementation principle
          
Whether the Android sub-thread can update the UI, if you can please specify the details.

**Is the principle of broadcast transmission and reception understood? **

- Inherit the BroadcastReceiver and override the onReceive() method.
- Register a broadcast to the ActivityManagerService via the Binder mechanism.
- Send a broadcast to the ActivityMangerService via the Binder mechanism.
- ActivityManagerService finds the BroadcastReceiver of the broadcast (IntentFilter/Permission) that meets the corresponding conditions, and sends the broadcast to the message queue where the BroadcastReceiver is located.
- After the BroadcastReceiver message queue gets the broadcast, call back its onReceive() method.

**View drawing process**

View drawing process: OnMeasure () -> OnLayout () -> OnDraw

The main work of each step:

OnMeasure():

Measure the view size. The measur method is called recursively from the top parent view to the child View, and the measure method calls back the OnMeasure.

OnLayout():

Determine the location of the view and make the page layout. The process of recursively calling the view.layout method from the top parent view to the child view, that is, the parent view according to the layout size and layout parameters obtained by the previous step easure child View, the child view is in the appropriate position.

OnDraw():

Draw a view: ViewRoot creates a Canvas object and then calls OnDrw(). Six steps:

1. Draw the background of the view;

2. Save the layer of the canvas (Layer);

3. Draw the contents of the View;

4, draw a View subview, if not there is no need;

5. Restore layer (Layer);

6. Draw a scroll bar.

What is the difference between onTouch and onTouchEvent in event distribution, and how do I use it?

What are the event dispatch related callback methods for View and ViewGroup respectively?

View refresh mechanism

HttpUrlConnection and okhttp relationship

Understanding of Bitmap objects

ActivityThread, AMS, WMS works

What is the difference between AstncTask+HttpClient and AsyncHttpClient?

View drawing process

Http://www.codekk.com/blogs/detail/54cfab086c4761e5001b253f
Https://www.jianshu.com/p/5a71014e7b1b

Custom control principle

How does a custom View provide an interface to get View properties?

Implement WAP mode networking in Android code

AsyncTask mechanism

AsyncTask principle and deficiency

How to cancel AsyncTask?

LeakCanary implementation principle

Http://blog.csdn.net/cloud_huan/article/details/53081120

The essence of memory leaks

Unable to recycle useless objects

What do you think is the difference between Rxjava's thread pool and your own task management framework?

Handling Ordered Arrays Why Reference StackOverflow Faster than Unordered Arrays

Integer class is not thread safe, why

Android animation framework implementation principle

Differences between Android versions of the API

**Requestlayout, onlayout, onDraw, DrawChild difference and contact**

requestLayout() method: Causes the measure() procedure and the layout() procedure to be called. Will judge whether you need ondraw according to the flag

onLayout () method (if the View is a ViewGroup object, you need to implement this method, layout each subview)

Call the onDraw() method to draw the view itself (each View needs to override this method, ViewGroup does not need to implement this method)

drawChild() to call back the draw() method of each subview

The difference between invalidate and postInvalidate and its use

Difference between Activity-Window-View

**How ​​to optimize custom View**

In order to speed up your view, you need to minimize unnecessary code for frequently called methods. Starting with onDraw, you need to pay special attention to things that should not be allocated memory here, because it will cause GC, which will cause the card to be stuck. The action of allocating memory during initialization or animation gaps. Don't do memory allocation while the animation is executing.

You also need to reduce the number of times onDraw is called as much as possible. Most of the time, onDraw is called because invalidate() is called. So try to reduce the number of times invaildate() is called. If possible, try to call the invalidate() method with 4 arguments instead of invalidate() with no arguments. An invalidate with no arguments forces a redraw of the entire view.

Another very time consuming operation is requesting layout. Executing requestLayout() at any time will cause the Android UI system to traverse the entire View hierarchy to calculate the size of each view. If a conflicting value is found, it will need to be recalculated several times. In addition, you need to keep the level of the View as flat as possible, which is very helpful for improving efficiency.

If you have a complex UI, you should consider writing a custom ViewGroup to perform his layout operations. Unlike the built-in view, a custom view allows the program to measure only that part, which avoids traversing the entire view's hierarchy to calculate the size. This PieChart example shows how to inherit ViewGroup as part of a custom view. PieChart has child views, but it never measures them. Instead, set their size directly according to his own layout rules.

How does the low version SDK implement the high version api?


Calculate the nesting level of a view

Image loading principle

Statistical start time, standard

How to maintain application stability

SpareArray principle

Performance optimization, how to ensure that the application does not start

SP is process synchronization? Is there any way to synchronize?

Inter-thread operation List

The life cycle of the process and Application;

The difference between recycleview listview, performance

Database data migration problem

Project componentization

Why Android System Designs ContentProvider, Process Sharing and Thread Safety Issues

Android related optimization (such as memory optimization, network optimization, layout optimization, power optimization, business optimization)

EventBus role, implementation, EventBus implementation principle, instead of EventBus

How to know the size of the view when encapsulating the view

The drop-down status bar does not affect the life cycle of the activity. If you make a network request on the onStop, how to restore it when onResume

View rendering

Logical address and physical address, why use logical address

RecycleView use, principle, RecycleView optimization

How to wake up other processes in the app


**Soft keyboard top layout**

**What is the use of ViewHolder? **

**Can Gradle's Flavor configure the sourceset? **

** Thread pool core thread number is generally defined, why? **

**How ​​is the underlying movement of the View in the screen?

**setContentView is doing 啥**

**How ​​to reuse the memory requested by Bitmap when decoding, release the timing**

**Note how to implement a findViewById**

** Tell me about your understanding of Context**

** Start BActivity by A, A is the intra-distribution mode, B is the standard mode, then start A or kill B again, talk about the life cycle of A, B, why **

**Discriminate the usage of Animation and Animator, and outline its principles**

**How ​​to load the NDK library? How to register the native function in jni, how many registration methods? **

**What are the connections and differences between processes and threads in the operating system? Under what circumstances will the system switch between user mode and good kernel state. **

**What are the possible reasons for the Android APP flashback? Please briefly describe the analysis process for each case. **

**Listview and recyclerview should be processed when they are loaded and loaded separately**

** How to ensure int self-increase security without locks**

**How ​​to automate the deployment of the packaged package process**

** How does the upper layer encapsulate the AIDL when the WeChat Alipay payment is called?

**How ​​to implement a push, Aurora Push Principle**

**Image Frame Selection**

**Image loading principle**

**Statistical startup time**

**How ​​to maintain the stability of the application**

**Runtime permission, how to give a preset app default permissions to it, do not authorize. **

**How ​​to take the C and V of the Activity in the case of MVC**

** Differences and advantages and disadvantages between various network frameworks, the reason why the network framework replaces evolution**

**The differences and advantages and disadvantages of the image caching framework. Is there a better image loading framework than Glide? **

** There are no problems caused by the encapsulation of Base class, BaseActivity and BaseFragment in the project framework, and the solution **

**Why not recommend soft references, is the garbage collection mechanism on the dvm the same as jvm? **

**L6UCache delete condition, what does LRU mean **

**Start page cache design white screen problem**

**How ​​to load the network picture? How does Glide determine that the image is loaded**

**Support for multiple Views in the project framework**

**The principle of Arouter**

**Componentization principle, implementation of routing in componentization**

**When the application communicates with the system, when does the Socket use Binder**?

**Debug and Release APK difference **

**New Google's Room Architecture**

**SplashActivity initializes the parameters of MainActivity, Splash is not initialized, AMS directly starts MainActivity how to do **

**Design a multi-thread, can read at the same time, can not write when reading, can not read when writing (read-write lock) **

**Android signature mechanism, what is included in the APK**

**Click on Launcher to click on WeChat Pay to start WeChat **

** No permission to locate, specific model positioning failed, how to solve **

**Gradle life cycle**

** When does ACTION_CANCEL trigger, touch the button and then slide it to the outside to raise the click event, and slide it back in + + to lift it up **

**How ​​to deal with sliding conflicts in nested views**

**The principle of heat repair, the framework is familiar **

**Gradget packaging process is familiar **

**Any questioning session: In fact, you can ask questions encountered in the previous interview: For example, when multiple modules are developed, different responsible persons may introduce duplicate resources, the same string, the same icon, etc. but the file names are not the same. How to deal with it? **

**Canvas's underlying mechanism, drawing framework, hardware acceleration is what is the principle, how is the canvas lock buffer?

**surfaceview, suface, surfacetexure, etc., and the underlying principles**

** android file storage, evolution of permissions control for each version of the storage location, external storage, internal storage**

**What are the pits of the upper business activity and fragment? ? Some pits and optimization experience on the page display**

**Open source framework for network requests: Introduction to OKHttp, wrote an interceptor?**

**Do you have a unified management of the data layer? How is the data cache done? Does the http request provide unified management? **

**What model is useful, what is the logic in the Activity layer? How to separate **

**How ​​do you manage the lifecycle if you use some decoupling strategy? **

**What is the way to improve the compilation speed? **

**Do you have unified management of the threads in the application? **

**Jni's algorithm is provided by the main thread? Do you want to ask the service class?

**What is the edge detection? Do you have any knowledge about deep learning? **

**What is the performance analysis of the app after going online?**

**Inter-process communication method? What are the components of Binder? **

** want to change the height of the listview, how to do **

**Fragment lazy loading implementation, parameter passing and saving** 

**ViewPager's cache implementation**

**How ​​to implement memory cache and disk cache in Android. **

The memory cache is based on the LruCache implementation, and the disk cache is based on the DiskLruCache implementation. Both classes are implemented based on the Lru algorithm and LinkedHashMap.

The LRU algorithm can be described in one sentence as follows:

    LRU is the abbreviation of Least Recently Used. The algorithm has not been used for a long time. As its 
    name suggests, its core principle is that if a data has not been used in the recent period, then the possibility
    of being accessed in the future is also If it is small, such data items will be eliminated first.

The principle of LruCache is to use LinkedHashMap to hold strong references to objects and to eliminate objects according to Lru algorithm. Specifically, let's assume that we access the data from the end of the table and delete the data in the header. When the accessed data item exists in the linked list, the data item is moved to the end of the table, otherwise a new data item is created at the end of the table. When the linked list capacity exceeds a certain threshold, the data of the header is removed.

Why would you choose LinkedHashMap?

This is related to the characteristics of LinkedHashMap. The constructor of LinkedHashMap has a boolean parameter accessOrder. When it is true, the LinkedHashMap will sort the elements in order of access order, otherwise the elements will be sorted in the order of insertion.

DiskLruCache is similar to LruCache in that it only adds a journal file to manage disk files and welcomes them, as shown below:

    libcore.io.DiskLruCache
    1
    1
    1
    
    DIRTY 1517126350519
    CLEAN 1517126350519 5325928
    REMOVE 1517126350519
Note: The cache directory here is the application's cache directory /data/data/pckagename/cache. The unrooted 
mobile phone can enter the directory or copy the entire directory by the following command:

    //Enter the /data/data/pckagename/cache directory
    Adb shell
    Run-as com.your.packagename 
    Cp /data/data/com.your.packagename/
    
    / / Copy the /data/data/pckagename directory
    Adb backup -noapk com.your.packagename
Let's analyze the contents of this file:

The first line: libcore.io.DiskLruCache, fixed string.
The second line: 1, DiskLruCache source version number.
The third line: 1, the version number of the App, passed in through the open () method.
The fourth line: 1, each key corresponds to several files, generally 1.
Fifth line: blank line
Line 6 and subsequent lines: Cache operation records.
The sixth line and subsequent lines represent the cache operation record. For the operation record, we need to 
understand the following three points:

DIRTY indicates that an entry is being written. There are two cases of writing. If it succeeds, it will write a line of CLEAN records. If it fails, it will add a row of REMOVE records. Note that records with only DIRTY status alone are illegal.
A REMOVE record is also written when the remove(key) method is called manually.
READ is a record indicating that there is a read.
The length of the file is also recorded after CLEAN. Note that there may be multiple keys corresponding to one file, so there will be multiple numbers.

** What is the difference between RecyclerView and ListView? Partial refresh? How to avoid sliding jams in the multiple type scene when the former is used. How to achieve lazy loading, how to optimize the sliding experience. **

**Open Issue: If you increase the startup speed, design a lazy loading framework or sdk method and attention. **

**Scroller has a method, how to use it. **

**webwiew understand? How to achieve communication with javascript? Communication between the two parties. @JavascriptInterface is in? The version has a bug, in addition to this there are other programs that call the android method? **

**The effect of thread sleep on messages**

**If you use Handler postdelayed two messages in the current thread, a delay of 5s, a delay of 10s, and then let the current thread sleep for 5 seconds, how will the execution time of the above message change? **

A: Execute as usual

Extension: sleep time <=5 has no effect on two messages, 5<sleep time <=10 has an effect on the first message, the first message will be delayed until sleep, and sleep time >10 for both times The effect will be delayed until after sleep.

**Logical address and physical address, why use logical address? **

** Android process memory allocation, can you allocate a fixed amount of memory? **

** How to ensure that a background service is not killed? (Same question: How to ensure that the service is not killed in the background?) What is the way to save power? **

First, onStartCommand method, return START_STICKY

START_STICKY After the service process is killed after running onStartCommand, it will remain in the start state, but will not retain those incoming intents. Soon after, the service will try to recreate it again, because it will remain in the start state, and it will guarantee to call onstartCommand after the service is created. If no start command is passed to the service, then a null intent will be obtained.

START_NOT_STICKY After the service process is killed after running onStartCommand, no new intent is passed to it. The Service will move out of the start state and will not be recreated until a new explicit method (startService) call. Because if no undetermined intent is passed then the service will not start, that is, during the onstartCommand will not receive any null intent.

START_REDELIVER_INTENT After the service process is killed after running onStartCommand, the system will start the service again and pass in the last intent to onstartCommand. The intent is not passed until stopSelf(int) is called. If there is an unprocessed intent after being killed, the service will start automatically after being killed. Therefore onstartCommand will not receive any null intents.

Second, improve the service process priority

The process in Android is managed. When the system process space is tight, the process will be automatically recycled according to the priority. Android divides the process into six levels, which are in descending order of priority:

Foreground process ( FOREGROUND_APP)
Visual process (VISIBLE_APP)
Secondary service process (SECONDARY_SERVER)
Background process (HIDDEN_APP)
Content Provisioning Node (CONTENT_PROVIDER)
Empty process (EMPTY_APP)
When the service is running in a low-memory environment, some existing processes will be killed. Therefore, the priority of the process will be important, you can use startForeground to put the service to the foreground state. This will be less likely to be killed in low memory.

Third, restart the service in the onDestroy method

Service +broadcast mode, when the service goes ondestory, send a custom broadcast, when the broadcast is received, restart the service;

Fourth, Application plus Persistent attribute

Fifth, the monitoring system broadcast determines the Service status

Listen through and capture some of the system's broadcasts, such as: phone restart, interface wake-up, application state changes, etc., and then determine if our service is still alive, don't forget to add permissions.


**What is the startup mode of Activity? The stack is ABC, first want to go directly to A, BC are cleaned up, there are several ways to do it? The result of these several methods is that there are several instances of A? **

**What tools can I see the Activity stack information? Multiple stacks, there are ways to get the list of activities of each stack separately**

**What are the commands that are familiar with? Do you know how to start an Activity with a command?**
    
**Android performance optimization method**

Http://www.trinea.cn/android/performance/



    Layout optimization: Minimize the hierarchy of layout files
    Minimize the hierarchy of layout files
    Remove useless controls and hierarchies in the layout, and selectively use a higher performance ViewGroup
    Using tags, ViewStub, and layout reuse can reduce the level of layout (ViewStub provides on-demand loading,
    loading ViewStub layouts into memory when needed, improving initialization efficiency)
    Avoid over drawing
    
    Drawing optimization: View's onDraw() method avoids performing a lot of operations.
    Do not create new layout objects in onDraw().
    OnDraw() does not do time-consuming tasks, and a large number of loops preempt the cpu time slice, 
    causing the drawing of the View to be unsmooth.
    
    Memory leak optimization
    Avoid writing memory leak codes
    Identify potential memory leaks through analysis tools (MAT, LeakCannary) and resolve them.
    Causes of a memory leak:
    Collection class leak
    Singleton/static variables cause memory leaks
    Anonymous inner class/non-static inner class causes memory leak
    Resource not closed
    
    Optimization of response speed: avoiding time-consuming operations on the main thread
    
    ListView / RecyclerView optimization:
    Use ViewHolder mode to increase efficiency
    Asynchronous loading: time-consuming operations placed on asynchronous threads
    Stop loading and page loading when sliding
    
    Thread optimization: use thread pool to avoid a large number of Threads in the program.
    
    Other performance optimizations:
    Don't create objects too much.
    Don't use the enumeration class too much, the enumeration takes up more memory than the integer.
    Constants are decorated with static final.
    Use Android-specific data structures.
    Appropriate use of soft and weak references.
    Use memory cache and disk cache.
    Try to use static inner classes to avoid potential memory leaks due to internal classes.

**Application scenarios for soft references and weak references in Android. **

Java reference type classification:

![image](https://user-gold-cdn.xitu.io/2017/10/10/29c884389e96babb2759b95014628aae?imageslim)

    In the development of Android applications, in order to prevent memory overflow, when dealing with some 
    objects that occupy large memory and have a long declaration period, you can apply soft reference and w
    eak reference technology as much as possible.
    A soft/weak reference can be used in conjunction with a reference queue (ReferenceQueue). If the object 
    referenced by the soft reference is reclaimed by the garbage collector, the Java virtual machine will 
    add the soft reference to the reference queue associated with it. Use this queue to know the list of 
    soft/weak referenced objects being reclaimed, thereby clearing expired soft/weak references for the buffer.
    If you just want to avoid OOM exceptions, you can use soft references. If you are more concerned about 
    the performance of your application and want to reclaim some objects that take up a lot of memory as 
    soon as possible, you can use weak references.
    It is possible to judge whether to select a soft reference or a weak reference depending on whether the
    object is frequently used. If the object may be used frequently, try to use soft references. If the 
    object is more likely to be unused, you can use weak references.


    
**Android long connection, how to deal with the heartbeat mechanism. **

    Long connection: After the connection is established, the connection is not actively disconnected. 
    The two parties send data to each other, and do not actively disconnect after the transmission is 
    completed. After that, the data that needs to be sent continues to be sent through this connection.
    Heartbeat package: In fact, it is mainly to prevent NAT timeout. The client sends a data actively 
    after a period of time to detect whether the connection is disconnected.
    The server processes the heartbeat packet: If the client heartbeat interval is fixed, then if the 
    server does not receive the heartbeat after the connection is idle for more than this time, the server
    can be considered to be dropped and the connection is closed. If the client heartbeat changes dynamically,
    a maximum value should be set. If the maximum value is exceeded, the other party is considered to be offline.
    There is also a case where the server actively sends a message to the client through the TCP connection and 
    a write timeout occurs, and the other party can directly think that the other party is offline.

**CrashHandler implementation principle**

    The information for getting the app crash is saved locally and sent to the server the next time the app is opened.

**Dynamic rights adaptation scheme, concept of permission group**


**Start a program, you can click on the icon in the main interface to enter, you can also jump from a program, what is the difference between the two? **

    Is because the launcher (the main interface is also an app), found that there is a setting in this program as
    
    <category android:name="android.intent.category.LAUNCHER" />
    Activity,
    So this launcher will put the icon and put it on the main interface. When the user clicks on the icon, an 
    Intent is issued:
    
    Intent intent = mActivity.getPackageManager().getLaunchIntentForPackage(packageName);
    mActivity.startActivity(intent);   
    Jump over to jump to any allowed page, such as a program can be downloaded, then the actual downloaded page 
    may not be the home page (and possibly the home page), then construct an Intent, startActivity.
    The action in this intent may have multiple views, and download is possible. The system will select the programs
    or pages that can be opened for your Intent based on the functions registered by the third-party program to the
    system. So the only point
    The difference is that the action of the intent initiated from the click of the icon is relatively singular, and
    jumps from the program or starts with more styles. The essence is the same.


**FC (Force Close)**

When will it appear?

Error
OOM, memory overflow
StackOverFlowError
Runtime, such as a null pointer exception
The solution

Pay attention to the use and management of memory
Use the Thread.UncaughtExceptionHandler interface

**Interface optimization** 

    Too much overlapping background (overdraw)
    
    This problem is actually the easiest to solve. The suggestion is to check the background you set in the 
    layout and code. Some backgrounds are hidden underneath. It can never be displayed. This unnecessary background
    must be removed because it is very likely Will seriously affect the performance of the app. If you use the
    background of the selector, set the color of the normal state to "@android:color/transparent", which can also
    solve the problem.
    
    Too many overlapping views
    
    The first recommendation is to use ViewStub to load some less common layouts. It's a lightweight and default
    view that is invisible. It can dynamically load a layout, as long as you use this overlapping View. , postponed 
    loading time.
    
    The second suggestion is: If you use a combination like Viewpager+Fragment or have multiple Fragments on an 
    interface, you need to control the display and hiding of the Fragment. Try to use the dynamic Inflation view,
    which is better than SetVisibility.
    
    Complex layout level
    
    There are more suggestions here. First, we recommend using the layout tool Hierarchy Viewer provided by Android 
    to check and optimize the layout. The first suggestion is that if the nested linear layout deepens the layout
    hierarchy, you can use a relative layout instead. The second suggestion is to use labels to merge layouts. The
    third recommendation is to use the label to reuse the layout, and extracting the common layout can make the 
    logic of the layout clearer. Remember, the ultimate goal of these suggestions is to make your layout wider 
    and shallower in the Hierarchy Viewer than narrow and deep.
    
    Summary: You can consider using merge and include, ViewStub. Try to make the layout shallow, and use the 
    RelactivityLayout as little as possible for the root layout, because RelactivityLayout needs to measure 2 times each time.

**Memory Optimization** 

    The core idea: to reduce memory usage, can not be new, not new, can allocate less allocation. Because
    allocating more memory means more GCs are generated, each time the GC is triggered, it takes up CPU time and 
    affects performance.
    
    Collection optimization: Android provides a series of optimized data collection tool classes, such as
    SparseArray, SparseBooleanArray, LongSparseArray, which can make our program more efficient. The HashMap 
    utility class is relatively inefficient because it requires an object entry for each key-value pair, and 
    SparseArray avoids the time when the underlying data type is converted to the object data type.
    Bitmap optimization: When reading a Bitmap image, don't load the unwanted resolution. You can compress 
    images and other operations.
    Try to avoid using the dependency injection framework.
    Avoid creating unnecessary objects: String splicing uses StringBuffer, StringBuilder.
    Do not create objects in the onDraw method.
    Override onTrimMemory to release the memory based on the parameters passed in.
    Use static final to optimize member variables.
    
** Several points for mobile data to optimize network data**

    Connection multiplexing: saves connection setup time, such as enabling keep-alive.
    For Android, HttpURLConnection and HttpClient both have keep-alive enabled by default. Just 2.2 before 
    HttpURLConnection has a bug affecting the connection pool, specifically visible: Android HttpURLConnection
    and HttpClient selection
    
    Request merge: Combine multiple requests into one request, the more common is the CSS Image Sprites in 
    the web page. If there are too many requests within a page, you can also consider doing a certain request merge.
    
    Reduce the size of the request data: for post requests, the body can be gzipped, and the header can also
    do data compression (but only supports http 2.0).
    The body of the returned data can also be gzipped, and the body data volume can be reduced to about 30%.
    (It is also possible to consider the volume of the key data of the returned json data, especially for the
    case where the return data format does not change much, and the data returned by Alipay chat is used)
    According to the user's current network quality to determine what quality pictures to download (more commercial)

**Android system architecture**

![image](https://upload-images.jianshu.io/upload_images/2893137-1047c70c15c1589b.png?imageMogr2/auto-orient)

    The android system architecture, like its operating system, uses a layered architecture. From the architecture
    diagram, android is divided into four layers, from the upper layer to the lower layer are the application layer, 
    the application framework layer, the system runtime layer and the linux core layer.
    　　Application 
    　　Android will be released with the same series of core application packages, including email client, SMS short
      message program, calendar, map, browser, contact manager and so on. All applications are written in the Java 
      language.
    　　2. Application framework 
    　　Developers also have full access to the API framework used by the core application. The application's 
      architectural design simplifies component reuse; any application can publish its function blocks and any 
      other application can use its published function blocks (although the framework's security restrictions 
      are followed). Similarly, the application reuse mechanism also allows users to easily replace program components.
    　　Hidden behind each application is a series of services and systems, including;
    　　* Rich and extensible views (Views) that can be used to build applications, including lists, grids, text boxes,
      buttons, and even embeddable web browsing Device.
    　　* Content Providers allow applications to access data from another application (such as a contact database) or 
      share their own data
    　　* Resource Manager provides access to non-code resources such as local strings, graphics, and layout files.
    　　* Notification Manager allows the application to display custom prompts in the status bar.
    　　* Activity Manager is used to manage the application lifecycle and provide common navigation fallback capabilities.
    　　For more details and how to write an application from scratch, please refer to how to write an Android app.
    3. System runtime 
    　　1) Library
    　　Android includes some C/C++ libraries that can be used by different components in the Android system. They 
      serve developers through the Android application framework. Here are some core libraries:
    　　System C library - A standard C system library ( libc ) inherited from BSD that is specifically tailored for
      embedded linux-based devices.
    　　 Media Library - Based on PacketVideo OpenCORE; this library supports a variety of commonly used audio and 
       video formats for playback and recording, as well as support for still image files. The encoding formats 
       include MPEG4, H.264, MP3, AAC, AMR, JPG, PNG.
    　　 Surface Manager - Manages the display subsystem and provides seamless integration of 2D and 3D layers 
       for multiple applications.
    　　 LibWebCore - A new web browser engine that supports Android browsers and an embeddable web view.
    　　SGL - the underlying 2D graphics engine
    　　 3D libraries - based on OpenGL ES 1.0 APIs; the library can use hardware 3D acceleration (if available)
       or use highly optimized 3D soft acceleration.
    　　 FreeType - Bitmap and vector font display.
    　　* SQLite - A lightweight, relational database engine that is available for all applications.
    　　2) Android runtime
    　　Android includes a core library that provides most of the functionality of the JAVA programming 
      language core library.
    　　Every Android application runs in its own process and has a separate Dalvik virtual machine 
      instance. Dalvik is designed as a device that can run multiple virtual systems efficiently and efficiently
      at the same time. The Dalvik virtual machine executes (.dex) the Dalvik executable, which is optimized for
      small memory usage. At the same time, the virtual machine is register-based. All classes are compiled by the 
      JAVA compiler and then converted to the .dex format by the virtual machine through the "dx" tool in the SDK.
    　　The Dalvik virtual machine relies on some features of the Linux kernel, such as the threading mechanism and 
      the underlying memory management mechanism.
    　　4.Linux kernel 
    Android's core system services rely on the Linux 2.6 kernel such as security, memory management, process management,
    network protocol stack and driver model. The Linux kernel also serves as an abstraction layer between the hardware 
    and software stacks.
    
![image](https://raw.githubusercontent.com/BeesAndroid/BeesAndroid/master/art/android_system_structure.png)

It is divided into four layers from top to bottom:

Android application framework layer
Java system framework layer
C++ system framework layer
Linux kernel layer

Android view drawing mechanism and loading process, please elaborate on the whole process

Please refer to the loading process of activty in detail (not the life cycle)

Android uses automatic garbage collection mechanism, please talk about the principle of Android memory management

Say the principle and difference of Android virtual machine and java virtual machine 

**[The difference between the map and flatmap operators in RxJava and the underlying implementation] (https://www.jianshu.com/p/af13a8278a05)**

**[RecyclerView is different from ListView caching mechanism] (https://segmentfault.com/a/1190000007331249)**

**ButterKnife principle**

ButterKnife has little impact on performance, because instead of using reflection, it uses the Annotation Processing Tool (APT), the annotation processor, and the tools used in javac to compile and parse Java annotations. Executed during the compilation phase, its principle is to read the Java source code, parse the annotations, and then generate new Java code. The newly generated Java code is finally compiled into Java bytecode, and the annotation parser cannot change the Java class that is read in, such as the inability to add or remove Java methods.

**Activity/Window/View difference, fragment features**

    Activity is like a craftsman (control unit), Window like a window (bearing model), View like a window (display view) 
    LayoutInflater like scissors, Xml configuration like a window drawing.
    
    Call attach in Activity to create a Window
    The created window is its subclass PhoneWindow, creating PhoneWindow in attach
    Call setContentView(R.layout.xxx) in the Activity
    Which is actually the call to getWindow().setContentView()
    Call the setContentView method in PhoneWindow
    Create a ParentView:
    As a subclass of ViewGroup, it is actually a created DecorView (as a subclass of FramLayout)
    Fill the specified R.layout.xxx
    Filled by the layout filler [where the parent refers to the DecorView]
    Call to ViewGroup
    Call ViewAll's removeAllView() to remove all views first.
    Add a new view: addView()

**Event delivery mechanism**

    When the finger touches the screen, the system will call the corresponding View's onTouchEvent and pass 
    in a series of actions.
    
    The execution order of dispatchTouchEvent is:
    
    First trigger ACTIVITY's dispatchTouchEvent, then trigger ACTIVITY's onInterceptTouchEvent.
    Then trigger the dispatchTouchEvent of LAYOUT, then trigger the onInterceptTouchEvent of LAYOUT
    This explains that you must call super.dispatchTouchEvent() when rewriting the ViewGroup;
    
    (1) dispatchTouchEvent:
    
    This method is generally used to initially process events, because the action is dispatched from this, 
    so super.dispatchTouchEvent is usually called. This will continue to call onInterceptTouchEvent, 
    which is then determined by onInterceptTouchEvent.
    
    (2) onInterceptTouchEvent:
    
    If the return value is true, the event will be passed to its own onTouchEvent(); if the return
    value is false, it will be passed to the next View's dispatchTouchEvent();
    
    (3) onTouchEvent():
    
    If the return value is true, the event is consumed by itself, and subsequent actions let it process; if the 
    return value is false, it does not consume the event itself, and returns to the other parent's 
    onTouchEvent to be processed.
    
    Pseudo code of the three method relationships: If the current View intercepts the event, it is handed over 
    to its own onTouchEvent to handle, otherwise it is thrown to the child View to continue the same process.
    
    Public boolean dispatchTouchEvent(MotionEvent ev)
    {
        Boolean consume = false;
        If(onInterceptTouchEvent(ev))
        {
            Consume = onTouchEvent(ev);
        }
        Else
        {
            Consume = child.dispatchTouchEvent(ev);
        }
        Return consume;
    }
    The delivery of onTouchEvent:
    
    When there are multiple levels of View, this action will continue until the parent level is
    allowed until the deepest View is encountered. So the touch event first calls the onTouchEvent
    of the lowest level View. If the view's onTouchEvent receives a touch action and does the corresponding
    processing, there are two return methods return true and return false; return true will tell the system
    the current View Need to deal with this touch event, the future system issued ACTION_MOVE, ACTION_UP still
    need to continue to listen and receive, and this action has been processed, the parent layer View is impossible 
    to trigger onTouchEvent. So each action can only have one onTouchEvent interface that returns true. If it returns 
    false, it will notify the system that the current View does not care about this touch event. At this time,
    the action will be passed to the parent, calling the onTouchEvent of the parent View. But after this 
    touch event, any action is issued, the View is not accepted, onTouchEvent will not be triggered in this
    touch event, that is, once View returns false, then ACTION_MOVE, ACTION_UP and other ACTION will not 
    Pass in this View, but the action of the next touch event will still be passed in.
    
    Parent layer's onInterceptTouchEvent
    
    As mentioned earlier, the underlying View can receive this event with a precondition: if the parent 
    layer allows it. Assuming that the parent level's dispatch method is not changed, the parent's 
    onInterceptTouchEvent method is called before the system calls the underlying onTouchEvent. Whether 
    onInterceptTouchEvent means that the parent layer has intercepted the touch event, and the subsequent action is also You 
    don't have to ask onInterceptTouchEvent, the onInterceptTouchEvent will not be called again after the action
    that is emitted after this touch event until the next touch event. If onInterceptTouchEvent returns false,
    then this action will be sent to the deeper View, and each subsequent action will ask the parent layer's 
    onInterceptTouchEvent to not need to intercept this touch event. Only the ViewGroup has the onInterceptTouchEvent
    method, because an ordinary View is definitely the deepest View, and the touch can be passed to the 
    last station. It will definitely call the View's onTouchEvent().
    
    The underlying View's getParent().requestDisallowInterceptTouchEvent(true)
    
    For the underlying View, there is a way to prevent the parent layer's View from getting the touch event, 
    which is to call the getParent().requestDisallowInterceptTouchEvent(true) method. Once the underlying 
    View receives the touch action and then calls this method, the parent layer View will not call onInterceptTouchEvent
    anymore, nor can it intercept future actions (if the parent layer ViewGroup and the lowest level 
    View need to intercept different focus, or touch of different gestures, Can't use this to write dead).
    
Http://gityuan.com/2015/09/19/android-touch/
Https://www.jianshu.com/p/84b2e0038080
http://hanhailong.com/2015/09/24/Android-%E4%B8%89%E5%BC%A0%E5%9B%BE%E6%90%9E%E5%AE%9ATouch%E4%BA %8B%E4%BB%B6%E4%BC%A0%E9%80%92%E6%9C%BA%E5%88%B6/

**Scroller principle**

    Scroller execution three core methods in the process
    
    mScroller.startScroll()
    mScroller.computeScrollOffset()
    view.computeScroll()
    1. Make some initialization preparations for the slide in mScroller.startScroll(), such as: starting 
    coordinates, sliding distance and direction and duration (with default values), animation start time, etc.
    
    2. The mScroller.computeScrollOffset() method mainly calculates the current coordinate point based on the 
    time that has elapsed. Since the animation time is set in mScroller.startScroll(), it is easy to get 
    the position of the current moment in the computeScrollOffset() method based on the elapsed time and 
    save it in the variables mCurrX and mCurrY. In addition to this, the method can also determine whether 
    the animation has ended.

**Java and JavaScript interaction in Android**

    webView.addJavaScriptInterface(new Object(){xxx}, "xxx");
    1
    Answer: You can use the WebView control to execute JavaScript scripts and execute Java code in
    JavaScript. To have the WebView control execute JavaScript, you need to call the WebSettings.setJavaScriptEnabled 
    method with the following code:
    
    WebView webView = (WebView)findViewById(R.id.webview);
    WebSettings webSettings = webView.getSettings();
    / / Set WebView support JavaScript
    webSettings.setJavaScriptEnabled(true);
    webView.setWebChromeClient(new WebChromeClient());
    
    JavaScript calls Java methods that need to use the WebView.addJavascriptInterface method to set
    the Java method of the JavaScript call. The code is as follows:
    
    webView.addJavascriptInterface(new Object()
    {
        / / JavaScript call method
        Public String process(String value)
        {
            / / Processing code
            Return result;
        }
    }, "demo"); //demo is the object name of the Java object mapped to JavaScript
    
    The process method can be called using the following JavaScript code, the code is as follows:
    
    <script language="javascript">
        Function search()
        {
            / / Call the searchWord method
            result.innerHTML = "<font color='red'>" + window.demo.process('data') + "</font>";
        }

**The most essential difference between SurfaceView and View**

    SurfaceView is able to redraw the picture in a new separate thread, and the view must update the
    picture in the main thread of the UI.
    
    Updating the screen in the main thread of the UI can cause problems. For example, if you update too long, your
    main UI thread will be blocked by the function you are drawing. Then you will not be able to respond to messages
    such as buttons and touch screens. When using SurfaceView, it will not block your UI main thread because
    it updates the screen in a new thread. But this also brings another problem, that is, event synchronization. 
    For example, if you touch the screen for a while, you need thread processing in SurfaceView. Generally, 
    you need to have an event queue design to save touchevent, which is a bit more complicated because it
    involves thread safety.

**Android program runtime permissions and file system permissions** 

1, Linux file system permissions. Different users have different read and write execution rights for files. In the android system, system and application are separate, the data in the system is unchangeable.

2, Android has 3 kinds of permissions, process permissions UserID, signature, application declaration permissions. Each time the installation, the system assigns a unique userID according to the package name application. Different user IDs are run in different processes. The memory between processes is independent and cannot be accessed by each other unless a specific Binder mechanism is adopted.

Android provides a mechanism that allows two apks to break the barriers mentioned earlier.

In the AndroidManifest.xml, the shared userId attribute is used to assign the same userID to different packages. By doing so, the two packages can be treated as the same program, and the system will assign the same UserID to the two programs. Of course, for security reasons, the two packages need to have the same signature, otherwise there is no point in not verifying.
    
**How ​​to make the program start automatically** 

Define a Braodcastreceiver, action is BOOT - COMPLETE, start the program after receiving the broadcast.

**[How to ensure that the background service is not killed] (https://segmentfault.com/a/1190000006251859#articleHeader1)**

**[Difference between deep copy and shallow copy] (http://www.cnblogs.com/chenssy/p/3308489.html)**

**[The default implementation of [clone() is deep copy or shallow copy? How do I make clone() implement deep copy? ](http://blog.csdn.net/zhangjg_blog/article/details/18369201)**

How to implement a circular ImageView;

How to implement the slideback deletion of RecyclerView by yourself;

How to make the current label always in the middle of the screen in TabLayout;

TextView calls the internal execution flow of the setText method;

Activity startup mode, allowReparent features and stack affinity

How to control the View display of another process

Two-finger zoom drag big picture

Client network security implementation

Surface screen adaptation

Can you dynamically add the same layout?

Handwritten rxjava traversal array

Aop thought

Implement a library, complete the real-time reporting and delayed reporting of logs, and consider which aspects

The implementation principle of RecyclerView's ItemTouchHelper

7.0 8.0 p features and compatibility

The four properties of Bitmap, how to load a large image (inJustDecodeBounds).

How to detect the execution time of a piece of code

How does TabLayout set the width of the indicator package content?

Maximum online bug handling

How MVP manages the life cycle of Presenter and when to cancel network requests

Webview performance optimization

**Why is WebView loading slower? **

This is because in the client, before loading the H5 page, you need to initialize the WebView. After the WebView is fully initialized, the subsequent interface loading process is blocked.

The optimization method is carried out around two points:

Preload the WebView.
Request H5 page data while loading WebView.
So the common method is:

Global WebView.
Client proxy page request. After the WebView is initialized, it requests data from the client.
The asset stores the offline package.
There are other optimizations besides this:

The script executes slowly, allowing the script to run last without blocking page parsing.
DNS and links are slow, allowing clients to reuse domain names and links.
React framework code execution is slow, you can split this part of the code and parse it in advance.

How to repair the interface

How to deal with smooth sliding

50fps Is there any way to increase it to 60fps?

How to achieve right sliding finish activity

How to achieve the rounded corner effect of the interface at the whole system level (that is, all APP open interfaces will be rounded corners, I admit, I was forced at the time)

The view with a width and height of 20dp is different for different mobile phones. The difference is %20, %30 or 2,3 times.

What is included in the APK, what is the packaging process;

Android interface refresh principle

** Can a non-UI thread update the UI?**

    can
    When accessing the UI, ViewRootImpl will call the checkThread method to check which thread is currently 
    accessing the UI. If it is not a UI thread, an exception will be thrown.
    When the onCreate method is executed, ViewRootImpl has not been created yet. It is impossible to check 
    the current thread. The creation of ViewRootImpl is after the onResume method callback.
    Void checkThread() {
        If (mThread != Thread.currentThread()) {
            Throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }
    A non-UI thread can refresh the UI, provided that it has its own ViewRoot, ie the thread that updates 
    the UI is the same as the ViewRoot, or the UI is updated before checkThread() is executed.

**Resolve the method of ScrollView nested ListView and GridView conflict**

Https://blog.csdn.net/btt2013/article/details/53447649

**Custom View Optimization Strategy**

    In order to speed up your view, you need to minimize unnecessary code for frequently called methods.
    Starting with onDraw, you need to pay special attention to things that should not be allocated memory
    here, because it will cause GC, which will cause the card to be stuck. The action of allocating memory
    during initialization or animation gaps. Don't do memory allocation while the animation is executing.
    You also need to reduce the number of times onDraw is called as much as possible. Most of the time,
    onDraw is called because invalidate() is called. So try to reduce the number of times invaildate() 
    is called. If possible, try to call the invalidate() method with 4 arguments instead of invalidate()
    with no arguments. An invalidate with no arguments forces a redraw of the entire view.
    Another very time consuming operation is requesting layout. Executing requestLayout() at any time
    will cause the Android UI system to traverse the entire View hierarchy to calculate the size of each
    view. If a conflicting value is found, it will need to be recalculated several times. In addition, 
    you need to keep the level of the View as flat as possible, which is very helpful for improving efficiency.
    If you have a complex UI, you should consider writing a custom ViewGroup to perform his layout operations. 
    Unlike the built-in view, a custom view allows the program to measure only that part, which avoids 
    traversing the entire view's hierarchy to calculate the size. This PieChart example shows how to
    inherit ViewGroup as part of a custom view. PieChart has child views, but it never measures them.
    Instead, set their size directly according to his own layout rules.

What is the difference between Gradle's api and implementation in Android Studio 3.0;

Has the message push been done, pushing the arrival rate problem;

How to solve git conflicts

Design a music player interface, how do you implement it, how to use those classes, how to design, how to define interfaces, how to interact with the background, how to cache and download, how to optimize (15 minutes)


What is the structure of the app, and why is this, what are the advantages and disadvantages?

Android Native and JS
There are several ways to communicate, and what frameworks are used?

Have you done any unit testing, talk about the familiar unit testing framework;

Nested sliding implementation principle 

Fragment in the life cycle of ViewPager, the life cycle of Fragment changes when sliding the page of ViewPager;

Gradle packaging;

The benefits of AOP IOC and its application in Android development;

Kotlin features, what is the difference between Java and Java;

The difference between @module and @component in the Dagger2 framework;

What are the benefits of Presenter defined as an interface in the MVP architecture;

Jenkins continues to integrate;

The use of Java dynamic proxy, what is the use of InvocationHandler;

Why is Google rolling out Fragment, what are the benefits and uses? Can you use View instead of it?

Have you learned about Android security, decompiling, packing, etc.


The click event is blocked, but I want to pass it to the View below. How do I do this?

Implementation of the WeChat main page

The principle of small red dot on WeChat

Implement the pop and push interface requirements for stack:

1. Implement with a basic array

2. Consider the paradigm

3. Consider the next synchronization problem

4. Consider the expansion problem

Introduce the previous app architecture and communication

Is there a rich text for push messages?

Is there a limit to the size of the apk package? How to reduce the package size?

Have you used or written any tools in your work? Scripts, plugins, etc.

For example, multi-person collaborative development may have a separate copy of some of the same resources, and there is no way to automatically detect such duplications.

How to customize the view to consider the model adaptation;

How the custom View provides an interface to get the View property;

Which event dispatching related callback methods are available for View and ViewGroup respectively;

The role and principle of annotation



Say what the cold start and hot start are, the difference, how to optimize, use the scene, and so on.


