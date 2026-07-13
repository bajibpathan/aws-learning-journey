# 🚀 Amazon S3 Performance Optimization

## 📖 What is Amazon S3 Performance?

Amazon S3 is designed to automatically scale and can support **thousands of requests per second**. As application traffic grows, S3 automatically adjusts to handle increased request rates without requiring manual provisioning.

AWS provides several techniques to optimize upload and download performance based on your workload.

---

# 1️⃣ Using Prefixes

## What?

Amazon S3 automatically scales request performance for each **prefix** within a bucket.

### Default Request Rate per Prefix

- **3,500** write requests per second
  - PUT
  - COPY
  - POST
  - DELETE

- **5,500** read requests per second
  - GET
  - HEAD

There is **no limit on the number of prefixes** within a bucket.

By distributing objects across multiple prefixes, applications can process requests in parallel and achieve significantly higher throughput.

### Example

```
logs/2026/
images/
videos/
documents/
backups/
```

Each prefix can scale independently.

---

# 2️⃣ Multipart Upload

## What?

Multipart Upload allows large objects to be uploaded in multiple parts simultaneously.

Instead of uploading one large file, the object is divided into smaller parts that are uploaded independently.

Once all parts are uploaded, Amazon S3 combines them into a single object.

### Characteristics

- Supports up to **10,000 parts**
- Each part:
  - **5 MB to 5 GB**
  - Last part can be smaller than 5 MB
- Failed parts can be retried individually
- Uploads can occur in parallel

### Best Use Cases

- Large files (recommended for objects larger than 100 MB)
- Large media files
- Database backups
- Big data workloads

---

# 3️⃣ S3 Transfer Acceleration

## What?

Amazon S3 Transfer Acceleration improves upload and download speeds for users located far from the target AWS Region.

Instead of uploading directly to the bucket, data is routed through the nearest **Amazon CloudFront Edge Location** using the AWS global network.

A dedicated Transfer Acceleration endpoint is used.

### Characteristics

- Uses AWS Global Network
- Uses CloudFront Edge Locations
- Faster long-distance uploads
- Additional charges apply

### Best Use Cases

- Global applications
- Remote users
- Large file uploads
- International customers

---

# 🎯 Why is Performance Optimization Important?

Performance optimization helps applications:

- Upload files faster
- Retrieve objects more quickly
- Improve user experience
- Handle increasing workloads
- Scale without infrastructure changes

---

# 🔒 Best Practices

- Use logical prefixes to distribute request traffic.
- Use Multipart Upload for large objects (generally over 100 MB).
- Enable Transfer Acceleration for geographically distributed users.
- Test performance before enabling Transfer Acceleration because additional charges apply.
- Monitor request metrics using Amazon CloudWatch.

---

# 💡 Key Takeaways

- Amazon S3 automatically scales to support thousands of requests per second.
- Prefixes improve throughput by distributing requests across multiple partitions.
- Multipart Upload improves performance and resiliency for large file uploads.
- Transfer Acceleration speeds up uploads by routing traffic through AWS Edge Locations.

---

# ❓ Frequently Asked Questions

### Q1. When should Multipart Upload be used?

Multipart Upload is recommended for large objects, especially those larger than **100 MB**, and is required for objects larger than **5 GB**.

---

### Q2. What is the maximum number of parts supported?

10,000 parts.

---

### Q3. Which AWS service does Transfer Acceleration use?

Amazon CloudFront Edge Locations.

---

### Q4. Does Transfer Acceleration improve uploads for users in the same AWS Region?

Usually, the benefit is minimal. It provides the greatest improvement for users who are geographically distant from the target Region.

---

### Q5. How can you test whether Transfer Acceleration improves performance?

AWS provides an official speed comparison tool:

https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html

---

# 📚 Related Topics

- Amazon S3
- Amazon S3 Storage Classes
- Amazon CloudFront
- Multipart Upload