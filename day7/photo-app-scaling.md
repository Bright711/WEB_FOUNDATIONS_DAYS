# Day 7 Assignment: Scaling Plan for SnapShare

## 1. Assumptions

SnapShare is a photo-sharing application where users upload photos and view photos posted by people they follow.

The following assumptions are used:

- Total registered users: 10 million.
- 10% of registered users are active daily.
- Each daily active user uploads one photo per day.
- Each daily active user views 50 feed pages per day.
- Average original photo size: 2 MB.
- Each photo generates one thumbnail of 50 KB.
- One year contains 365 days.
- There are 86,400 seconds in one day.
- Users are assumed to be active and upload throughout the day for average-rate calculations.
- Photos are retained for at least one year, with no deletions.
- Peak feed traffic is five times the average traffic.
- Storage calculations use decimal units (1 TB = 1,000,000 MB).

### Daily Active Users (DAU)

DAU = Total registered users × Daily activity percentage

DAU = 10,000,000 × 10%

**DAU = 1,000,000 users**

Therefore, SnapShare has approximately 1 million daily active users.

## 2. Traffic and Storage Calculations

### A. Photo uploads per second

Each active user uploads one photo daily.

Daily uploads = 1,000,000 × 1

Daily uploads = 1,000,000 photos

Average uploads per second:

1,000,000 ÷ 86,400 = 11.57

**Average uploads = approximately 12 photos per second.**

### B. Feed views per second

Each active user views 50 feed pages daily.

Daily feed views = 1,000,000 × 50

Daily feed views = 50,000,000

Average feed views per second:

50,000,000 ÷ 86,400 = 578.70

**Average feed views = approximately 579 requests per second.**

Peak feed views per second:

578.70 × 5 = 2,893.50

**Peak feed views = approximately 2,894 requests per second.**

### C. Photo storage per year

Each original photo requires 2 MB of storage.

Daily original photo storage:

1,000,000 × 2 MB = 2,000,000 MB = 2 TB

Annual original photo storage:

2 TB × 365 = 730 TB

Each thumbnail requires 50 KB of storage.

Daily thumbnail storage:

1,000,000 × 50 KB = 50,000,000 KB = 50 GB

Annual thumbnail storage:

50 GB × 365 = 18.25 TB

Total annual storage:

730 TB + 18.25 TB = 748.25 TB

**Total estimated storage per year = 748.25 TB.**

This estimate excludes backups, replicas, metadata, and additional storage overhead.

## 3. Is SnapShare Read-Heavy or Write-Heavy?

SnapShare is a **read-heavy system**.

This is because users generate 50 million feed views daily compared to only 1 million photo uploads daily.

This means the application performs approximately 50 feed-view requests for every photo upload.

The architecture should therefore prioritize fast content delivery, caching, database read replicas, and CDN distribution.

The system should also scale its read capacity independently from its upload capacity to handle increasing traffic efficiently.

## 4. Why Photos Should Not Be Stored Inside the Database

Photos should not be stored directly inside the database because image files are large and would consume significant database storage.

Storing large binary files in the database can increase backup sizes, slow down database operations, and make database maintenance more expensive.

Instead, original photos and thumbnails should be stored in **object storage**, such as Amazon S3 or Google Cloud Storage.

The database should store photo metadata, including:

- Photo ID
- User ID
- Photo URL or object-storage key
- Caption
- Upload timestamp
- Thumbnail URL or object-storage key

This approach separates application data from large media files and makes storage more scalable.

## 5. SnapShare Architecture Diagram

```text
                    USERS
                      |
                      v
                 +---------+
                 |   CDN   |
                 +---------+
                      |
                      v
              +----------------+
              | Load Balancer  |
              +----------------+
                      |
          +-----------+-----------+
          |                       |
          v                       v
    +-------------+         +-------------+
    | App Server 1|         | App Server 2|
    +-------------+         +-------------+
          |                       |
          +-----------+-----------+
                      |
             +--------+--------+
             |                 |
             v                 v
       +-----------+     +-------------+
       |   Cache   |     |  Database   |
       |  (Redis)  |     |   Primary   |
       +-----------+     +-------------+
                               |
                               v
                         +-------------+
                         | Read Replica|
                         +-------------+

       PHOTO UPLOAD PROCESS

    App Servers
         |
         v
    +----------------+
    | Object Storage |
    | Original Photos|
    +----------------+
         |
         v
    +----------------+
    | Message Queue  |
    +----------------+
         |
         v
    +----------------+
    | Thumbnail      |
    | Worker         |
    +----------------+
         |
         v
    +----------------+
    | Object Storage |
    | Thumbnails     |
    +----------------+
```

## 6. Explanation of Each Component

1. **CDN (Content Delivery Network):** Delivers photos and thumbnails from geographically distributed servers, reducing latency and traffic reaching the main application.

2. **Load Balancer:** Distributes incoming requests across multiple application servers to prevent one server from becoming overloaded.

3. **Application Servers:** Handle user authentication, photo uploads, feed requests, following relationships, and business logic.

4. **Cache (Redis):** Stores frequently accessed feed data and metadata in memory to reduce repeated database queries and improve response times.

5. **Primary Database:** Stores permanent application data such as users, relationships, photo metadata, and captions.

6. **Read Replica:** Handles read queries independently from the primary database, reducing read pressure on the primary database.

7. **Object Storage:** Stores original photos and generated thumbnails separately from the database, allowing large media files to scale efficiently.

8. **Message Queue:** Holds thumbnail-generation jobs so photo uploads do not have to wait for image processing to finish.

9. **Thumbnail Worker:** Processes queued jobs by resizing uploaded photos and generating smaller thumbnail images before saving them in object storage.

## 7. Step-by-Step Photo Upload Flow

1. A user selects a photo and submits it through the SnapShare application.

2. The application server authenticates the user and validates the photo's file type and size.

3. The application generates a unique photo ID and obtains a secure upload URL for object storage.

4. The original photo is uploaded directly to object storage using the secure upload URL.

5. After a successful upload, the application saves the photo metadata and storage key in the primary database.

6. The application publishes a thumbnail-generation job to the message queue.

7. A background worker retrieves the job from the queue and downloads the original photo from object storage.

8. The worker resizes the original photo to create a 50 KB thumbnail.

9. The generated thumbnail is uploaded to object storage.

10. The worker updates the photo metadata with the thumbnail's storage key and marks the processing job as complete.

11. The photo becomes available in the user's feed, and the CDN can deliver the original image and thumbnail to other users.

This asynchronous approach allows users to upload photos without waiting for thumbnail processing to finish.

## 8. Architecture Trade-offs

### Trade-off 1: Caching vs. Data Freshness

Caching improves performance because frequently requested feed data can be retrieved from memory instead of repeatedly querying the database.

However, cached information may become outdated when users upload new photos or follow new people.

**Solution:** Use cache expiration policies and invalidate or update relevant cache entries when important changes occur.

### Trade-off 2: Asynchronous Processing vs. Immediate Availability

Using a message queue allows uploads to complete without waiting for thumbnail generation, improving the user experience and allowing background workers to scale independently.

However, thumbnails may not be immediately available because workers process jobs asynchronously.

**Solution:** Display a temporary placeholder while thumbnail generation is in progress and retry failed jobs.

### Trade-off 3: Database Read Replicas vs. Data Consistency

Read replicas improve scalability by distributing read requests away from the primary database.

However, replication can introduce delays, meaning recently uploaded photo metadata might not immediately appear in a read replica.

**Solution:** Read recently created or updated records from the primary database when immediate consistency is required.

### Trade-off 4: Object Storage and CDN vs. Cost

Object storage and CDNs make photo delivery scalable and fast.

However, storing hundreds of terabytes of photos and delivering large amounts of media traffic can become expensive.

**Solution:** Use image compression, appropriate thumbnail sizes, CDN caching policies, and lifecycle rules for older content.

## 9. Conclusion

SnapShare requires a scalable architecture because it supports 1 million daily active users, approximately 579 average feed requests per second, and nearly 2,894 peak feed requests per second.

The estimated annual photo storage requirement is 748.25 TB.

Since SnapShare is read-heavy, the system should prioritize CDN delivery, caching, load balancing, and database read replicas.

Using object storage for photos and asynchronous thumbnail generation through a message queue will also help the application handle growth efficiently without overloading the database or application servers.

The proposed architecture provides a foundation for scaling SnapShare as its user base and traffic increase.