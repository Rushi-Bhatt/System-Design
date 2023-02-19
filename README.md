# System-Design
Mobile System Design Template:

## Resources: 
* https://themobileinterview.com/cracking-the-mobile-system-design-interview/
* https://betterprogramming.pub/an-app-developers-guide-to-mobile-system-design-interviews-74cd552bd963
* https://medium.com/geekculture/system-design-interview-for-mobile-engineers-ce712d6ac2c1
* https://www.davidseek.com/fb/
* https://medium.com/@m3amer/ios-system-design-architecture-example-d55384da1449
* Examples: https://github.com/weeeBox/mobile-system-design

## Pointers: 
1) Make assertive statements, you are responsible for making the decision.  
2) Dictate the requirements rather than asking many questions, and if the interviewer disagrees then he/she will interject 
3) Start with smaller scope of functional requirement, not spend more than 7-10 mins in requirement phase  
4) Push back if the requirement doesn’t make sense: “This feature might have diminishing returns and very less probability/user impact so we can ignore for now”. Negotiate down the scope. 
5) If you don’t know about something -> Be open about it rather than speculating. You should have an opinion about which solution to pick but you just don’t have experience to form that opinion. 
6) Always check before doing each step if your interviewer likes that: 
* I am going to start with high level design diagram, is that okay? 
* I am going define some API definitions. is that okay?
* I am going to define some data models. is that okay? And so on… May be your interviewer doesn’t want you to focus on that area and you can skip that in that case. 
7) Add a backup option in case the App is very server driven and the functionality is essential. Like payment system.   

## Strategy: 
1) Understand the problem
2) Define the scope
3) Identify technical requirements
4) Propose a high-level design
5) Deep dive into one component
6) Wrap up


## Step 1: 
1) What are we being asked to design?
2) Who is the user, and how will they use our system?
3) What's the initial number of users? And the expected growth?
4) Are we being given an initial design/wireframes, or should we produce them as well?
5) Are we designing an MVP or final product?
6) Are we building this from scratch, or can we leverage any existing components? Any existing patterns/architecture we should follow?
7) How big is the team who will implement and maintain our system?
8) Are we expected to design just the mobile application or other parts of the overall system too (e.g. API)?: Client vs Client+ API vs Client+API+Backend
9) Is it iOS or Android only, or cross-platform? Shall we support smartphones, tablets, both?
10) Do we need to support emerging markets? – high caching, limit the number and frequency of user requests, control the app size
11) What number of users are we expecting?  - Exponential backoff and API rate limiting
12) How big is the engineering team? 

## Step 2: 
1) Functional requirement: use case and feature of the app

2) Non Functional requirements: 
* Availability
  * Offline mode? If yes, what kind of persistence mechanism? And how to handle the network coming back scenario
  * Record based storage: Local store to cache data like Core Data, Realm, SQLIte, Shared preferences/User defaults
  * File based storage: Images and other media. Do we need caching locally? 
* Accessibility: 
  * Make App accessible by proper contrast, atleast 44pt target, custom calling, dynamic font size
  * Resources: 
  	* https://medium.com/swiftcommmunity/accessibility-in-ios-hands-on-code-971a219290f8
  	* https://www.hackingwithswift.com/articles/91/checklist-how-to-make-your-ios-app-more-accessible
  	* https://www.youtube.com/playlist?list=PLIl2EzNYri0cLtSlZowttih25VnSvWITu
* Security: 
  * Storing sensitive data like access token, PII data (key chain or smart lock, but not in User defaults)
  * One good practice to securely store secrets (like API Key) is to utilize .xcconfig configuration file, and ignore it using .ignore so your repo won’t have it, Or you can use private CloudKit database 
  * Secure Communication: Make backend communication secure using HTTPS, TLS, Cert pinning
  * Exponential back off, rate limiting to prevent DDOS 
* Scalability
  * Deal with growing codebase and team size
  * Modularize the App, break and standardize the reusable UI
  * Solid principles
* Performance
  * Are there any UI intensive operation? Like Infinite list scrolling, heavy animation, complex transition -> you could prefetch data and create a buffer 
  * Asynchronous media load by different service so UI is slick 
  * Pagination for API response:  Offset vs cursor based vs keyset based. <-  bad idea to use offset based especially if new posts keep getting added. 
  * Response Zipping: For large HTTP responses, use gzip to compress the response. Gzip is supported out-of-the box in NSURLSession and automatically communicates to network servers that it can handle gzipped data. When the server sends down gzipped data, NSURLSession automatically de-compresses it; you don't have to do anything: https://clintmartin.net/2016/03/15/gzipping-ios-apps.html - Problem:  doesn’t work well with progress download calculations
  * Quality of service for each API: some APIs are higher priority and we can execute them using higher QOS (like fetching list of posts info should be higher QOS than downloading thumbnail for those posts)  
  * Caching  
  * Use CDN for images and static content, this will cache things and improve the performance
  * Throttling and debouncing using timers or app lifecycle state observing to reduce the number of calls 
* Testing: 
  * Explain the testing strategy
  * Use of dependency injection to make testing easier 
* Localization/I18N:
  * Make L10n/I18n support for your app

3) Assumption/ out of scope: Features that will be excluded from the task, but still important to the real project, i.e boundary of the problem, any constraints. 
* Authentication & Authorization
* Monitoring: Analytics, Crash reporting and logging
* Deployment: CI/CD pipeline with automated releases (Fastlane, Jenkins), Feature flags and experiments 


### Step 3: 
1) Client Architecture: MVC, MVP, MVVM, VIPER, RIB
2) Networking/Client-server communication:  
* Is REST API okay? Generally okay for unidirectional communication. Other common protocols are SOAP, GraphQL, RPC: https://www.youtube.com/watch?v=NFw0HznpLlM
* Is HTTP Okay? Or do we need live updates and low latency, i.e in other words, bidirectional communication? If so, how will you push the updates to the client? Push notification, polling, web socket, heartbeats or silent notifications
* Serialization: Protobuf vs JSON vs XML: Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data: https://sprinkle-twinkles.medium.com/what-is-protobuf-and-when-to-consider-it-over-xml-json-for-web-services-communication-d91158b57d4a.
* Authorization: Figure out some basic Auth libraries, Firebase, OAuth2.
* Do we need to support slow network? How would the behavior change? 
* Do we support background tasks? Do we maintain state if the app is terminated? 
* Reachability: Offline mode. More details given below. 

3) Communication between components: Delegate/protocols, Reactive patterns, or Closures 
4) Navigation: Using Router component 
5) Storage: Repositories, Network Data, Persisted Data
6) Caching: Caching and cache eviction policy (like LRU). More details given below. 
7) Helper service/Managers: Networking service, session service, Database service, Credential store
8) Third Party Libraries: sd_webImage, RX_Swift/PromiseKit, SnapKit, Alamofire, Adjust, SwiftLint, Firebase, etc. ( I personally avoid using those for AppSize issues)
* Issues: 
 	* Size increase 
 	* Privacy 
 	* Binary stability and minimum version support 
 	* Bugs and crashes in the library 
 	* Stopped support 


## Step 4:
1) Wireframe the main flow if required : Draw the main scenes as boxes, go over the flows using arrows, add details 
2) Define Data Entities: not the schema, but just important entities like Users, Posts, Comments etc, mentioning their most important attributes and relationships 
3) Design Endpoint: Just HTTP method, path (GET/POST/PUT/DELETE /post/:id/comments), I/P parameter, O/P Entity/Model.  If the response is too big to be fetched in one go, then we need to have the capability to fetch the data in pieces and fetch more when we need it. That where pagination and fragmentation can help. Offset, KeySet and Cursor based pagination are some types.  Example: GET — /places?lat={}&long={}&page={}&pageLimit={}
4) Layout High level design 


## Step 5: 
1) Choose the most interesting screen and draw its architecture 
2) Trace all dependencies
3) Walk over the user flow: Arrows and states like loading, data, error, no data
4) Most challenging part/ bottlenecks:

## Step 6:  
Quick recap
Follow up questions/stretch goals
Further refinement


Template:  https://systeminterview.com/drawing.php -> Download the template 

## More details about some components:

1) Data layer: 
* Repository Pattern between network and service: https://pyartez.github.io/architecture/repository-pattern-in-swift-and-combine.html
* Use the repository when using offline mode, because response from API and response from core data might be different, so rather than using Coddle or NSManagerObjects objects throughout the app, you can use Domain Objects. 
 
2) Persistence layer/caching: 
* caching policy: based on time, based on count, and these timestamp/count should come from the remote config/backend so in future we can customize the caching policy. For default value, we can investigate and see what makes sense (based on users scroll behavior, size of each data unit, users device memory  etc).  Example, for reddit with 100 character per post, user will see mostly 4 posts per screen, and lets say we want to handle for 5 page scroll behavior, so store 20 posts in cache. Always start with defining disk/memory usage: Start with “I don’t want to utilize more than X MB in cache”
* Image caching: Image caching will have its own image cache and image loader. Since images are stored in CDN, we can use few features from there: example
	* thumbnail vs regular image - based on use case 
	* download image based on the device resolution (i.e https://cdn.image.com/255?width=100&height=100) - In real large apps, generally server sends all different image urls to clients, and then client decides which one to use. So server can cache that particular response efficiently. 
	* Resolution based: low/medium/high based on my devices network connection 
* Cache Clearing Policy: LRU policy, but How would you sync it with actual cache that stores your response? i.e lets say you store all the posts for a user in cache, and images for that post in image cache. When you remove the post from cache, how will you remove respective images from image cache? Also, what if that image is being used by another post ?
* Generally bad idea to use imageURLs as keys for the image cache, they are very long and may vary based on locale in many cases, so better way is to have resourceID (BE generated) as key for caching 


3) Offline mode: 
* Conflict resolution: lets say we update the details in our local cache while offline, and that too on multiple devices, how to handle that? - ideally to propagate the last update to server, so example, for postID 10, if you like it from iPhone at timestamp 00:25 and if you dislike it from iPad at timeStamp 00:30, then once any of these device goes online, you update the server DB with postID, timestamp and like/dislike. And when the 2nd device goes online, check if that timestamp is the latest one, if not, ignore , else update the DB. It works but what if you change your device time clock ? Well in that case, better to leave the conflict resolution to server side, whatever device can go online first, wins and then the 2nd update will be ignored.  (Exp. Instagram doesn’t even update the offline likes). We can also use what Github does in the case of conflicts, and use three way merge or basically ask users to merge the conflict by showing popup.  https://hasura.io/blog/design-guide-to-offline-first-apps/

* Now these updates might be heavy considering how much local cache you have and how often these have changed, so once we go online, first try and fetch all latest reads/posts/threads and then after some time, try to update the server with these offline values (user don’t care about likes on old post, but rather want to see the new posts from server)
How to achieve synchronization: real time vs eventual, QOS, 


4) UI- Layer: 
* Design systems: https://github.com/ChiliLabs/ios_DesignSystemExample, https://www.ramshandilya.com/blog/design-system-intro/
* Configurator to configure cells : https://chililabs.io/blog/configuring-multiple-cells-with-generics-in-swift

5) Backend Components: 

* Load Balancer: Distribute the load on server to horizontally scale the architecture.  (Round robin, lease connection, hashing etc) - Nginx 
* Front end caching using CDN: Stores videos/images or static contents, servers are distributed all over the world to reduce latency, clients are served with nearest server - AmazonS3. Exp.. Localized static texts, or video trailers for recent movies. 
* Elastic Search: is used to support Search and analyitics APIs. Its distributed and RESTful. NO SQL db search
* Redis cache: Remote Dictionary Server, is a fast, open source, in-memory, key-value data store. All Redis data resides in memory, which enables low latency and high throughput data access. Unlike traditional databases, In-memory data stores don’t require a trip to disk, reducing engine latency to microseconds. Because of this, in-memory data stores can support an order of magnitude more operations and faster response times. Can also use Memcache. It also supports blocking mechanism to hold the reservation. 
* Database: 
    * RDBMS: Proper relational representation, with ACID property, transaction handling. If its read heavy, use master slave architecture (slave for reading, master for writing) and data sharding. 
    * NO SQL: store unstructured data like movie info, actors, crews, reviews, comments.  So use distributed NOSQL database - Cassandra 
* Async workers: Handle async tasks, uses Messaging queues. Mainly used for ticket generations, sending notifications, emails and SMS.  Tasks which are time consuming and should not be handled synchronously. 
    * Server sends a task to messaging queue, and a free worker will pick it up, and execute it. <- Python Celery workers
    * Kafka/RabbitMQ for distributed messaging queue. 
    * Third party services for SMS/Email
    * APNS for sending push notifications using Airship or something else. 
* ML/BI: Hadoop platform, used generally for recommendation system like movies, hotels etc.
* Log management: ELK (Elastic search, Logstash, Cabana) logging analytics and monitoring 
* Payment Gateways: PayPal, strip, square etc <—- read about payment services 
