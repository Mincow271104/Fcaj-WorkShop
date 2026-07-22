---
title: "AWS Well-Architected: Hidden Costs in Cloud Architectures"
date: 2026-07-21
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
## Sharing on the AWS Well-Architected Framework: Hidden Costs in Cloud Architectures

## Introduction

This is the second blog post in my journey learning Amazon Web Services and building systems on the cloud. In this post, I dig deeper into why an AWS architecture must not only work, but also stay secure, reliable, performant, cost-efficient, and operable over time.

While researching, I read **“The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework”** on the AWS Architecture Blog (03/03/2026, Ryan Dsouza and Bradley Acar). The post stood out because it focuses on costs that never show up directly on an AWS bill but can still hurt a system badly — security incidents, downtime, data loss, or inefficient resource use.

![Figure 1 – Original blog post](/images/3-BlogsPosted/3.2-Blog2/hinh_1_bai_blog_goc.png)

Through Blog 2, I summarize the core ideas, share personal strengths and limitations of the article, and connect them to the AWS architecture our team is building for our project.

**Original post:**  
[The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)

---

## 1. Main Content of the Article

The AWS Well-Architected Framework helps builders understand the strengths and weaknesses of architectural decisions. It also helps evaluate architectures against AWS best practices and identify what needs improvement.

The framework is built on six pillars:

- **Operational Excellence**
- **Security**
- **Reliability**
- **Performance Efficiency**
- **Cost Optimization**
- **Sustainability**

![Figure 2 – Six pillars of the AWS Well-Architected Framework](/images/3-BlogsPosted/3.2-Blog2/hinh_2_sau_tru_cot.png)

*Figure 2. Six pillars of the AWS Well-Architected Framework. Source: Amazon Web Services.*

![Figure 2b – AWS Well-Architected Framework](https://res.cloudinary.com/dakqspssm/image/upload/v1784652986/well-architected-framework_er6nc7.png)

*Figure 2b. AWS Well-Architected Framework – six pillars supporting Business Outcomes.*

Reference:  
[The pillars of the AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)

In my view, these six pillars show that a good system cannot focus only on cost or performance. Security, reliability, operations, and sustainability must stay balanced during design.

The article focuses on three areas where a suboptimal architecture can create hidden costs: security, availability, and resource efficiency.

![Figure 3 – Three groups of hidden costs](/images/3-BlogsPosted/3.2-Blog2/hinh_3_ba_chi_phi_an.png)

### 1.1. Costs from security incidents

When IAM permissions are too broad, MFA is disabled, data is unencrypted, or monitoring is incomplete, organizations risk unauthorized access or security incidents. The impact is not only technical remediation cost — it can also damage reputation, business goals, and compliance posture.

The article recommends **least privilege**, multi-factor authentication, periodic policy reviews, encryption at rest and in transit, infrastructure protection, continuous monitoring, and an incident response plan.

### 1.2. Costs from system downtime

A system without redundancy can fail when a single component breaks. According to the article, downtime can mean lost productivity, lost revenue, missed SLAs, and unhappy customers. In some cases, organizations also face penalties or must hire external specialists to recover.

AWS Well-Architected recommends designing for fault tolerance, using redundancy and automatic failover, scaling automatically with demand, backing up data, and regularly testing recovery plans.

### 1.3. Costs from inefficient resource use

A common mistake is over-provisioning CPU, RAM, or storage to avoid slowdowns. Many workloads, however, are not always on 24/7 — some pause on weekends, run only a few days each month, or vary seasonally.

The article recommends **right-sizing**, removing unused resources, tracking costs regularly, and understanding pricing models such as On-Demand, Reserved Instances, Savings Plans, and Spot Instances.

---

## 2. Strengths of the Article (My Take)

What I value most is that the article does not treat cost optimization as simply picking cheaper AWS services. A short-term saving that weakens security or reliability can create much larger costs later.

The article also connects architectural decisions to business impact well. A wrong configuration is not only a technical failure — it can affect revenue, customer experience, SLAs, reputation, and compliance.

The recommended practices are concrete, including:

- Least privilege
- MFA
- Data encryption
- Monitoring
- Backup
- Disaster recovery
- Auto Scaling
- Right-sizing resources

I also like the view that architecture reviews are not personal audits to assign blame. AWS documentation describes the review process as consistent, blame-free, lightweight, and conversational.

Reference:  
[The review process](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-review-process.html)

---

## 3. Limitations of the Article (My Take)

The article is broad but does not include a complete sample architecture showing before-and-after optimization. Diagrams, cost figures, and remediation outcomes would make the ideas easier to visualize.

It also lists many recommendations without a clear starting order for students or small businesses. For beginners, applying every best practice at once can make the architecture unnecessarily complex.

From a personal view, student teams can start with basics:

1. Enable MFA.
2. Review IAM policies.
3. Block unnecessary public access on S3.
4. Set up AWS Budgets.
5. Enable logging and monitoring.
6. Verify backups.
7. Delete unused resources.

Also, because this is an official AWS post, the solutions mainly rely on AWS services and tools. It does not compare much with Azure or Google Cloud architecture approaches.

---

## 4. Connection to Our Team Project

Our project currently uses services such as:

- Amazon Cognito
- Amazon S3
- Amazon CloudFront
- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB

The reference architecture in the AWS Serverless Applications Lens uses similar services.

![Figure 4 – Reference architecture](/images/3-BlogsPosted/3.2-Blog2/hinh_4_kien_truc_do_an.png)

Reference architecture source:  
[Serverless Applications Lens – Web application](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/web-application.html)

After reading the article, I realized that “the system runs” does not mean “the architecture is good.” For avatar image storage on S3, the team should not only get upload and display working — we also need to ask:

- Is the S3 bucket unnecessarily public?
- Does the IAM role follow least privilege?
- Can a user modify or delete another user’s images?
- Is data encrypted, monitored, and logged appropriately?
- Is CloudFront cache configured correctly?
- Are upload file type and size limits enforced?
- Is there a lifecycle policy or cleanup for unused images?
- What happens if S3 upload succeeds but the DynamoDB update fails?

These are exactly the questions the Well-Architected Framework pushes designers to answer before production.

---

## 5. Conclusion

In my view, the greatest value of the AWS Well-Architected Framework is helping us shift from:

> **“Build a system that runs”**

to:

> **“Build a system that is secure, stable, efficient, and sustainable over time.”**

A good architecture does not require using as many AWS services as possible. What matters more is choosing the right services and understanding trade-offs among security, performance, reliability, and cost.

A Well-Architected Framework Review has three main stages:

1. **Prepare**
2. **Review**
3. **Improve**

![Figure 5 – WAFR process](/images/3-BlogsPosted/3.2-Blog2/hinh_5_quy_trinh_wafr.png)

Reference:  
[AWS Well-Architected Framework Review](https://docs.aws.amazon.com/wellarchitected/latest/userguide/wa-framework-review.html)

From this article, I take away that the most expensive problems often never appear on the AWS invoice first. They can hide in an overly broad IAM permission, a backup that was never tested, a service without recovery capability, or a resource that keeps running after it is no longer needed.

---

## References

1. [AWS Architecture Blog – The Hidden Price Tag](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)
2. [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
3. [The pillars of the framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)
4. [The review process](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-review-process.html)
5. [Serverless Applications Lens – Web application](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/web-application.html)
6. [AWS Well-Architected Framework Review](https://docs.aws.amazon.com/wellarchitected/latest/userguide/wa-framework-review.html)

> Note: Images in this post are illustrations synthesized by the author from official AWS documentation; they are not original screenshots from AWS materials.

*Source: AWS Architecture Blog*
*Original document: [The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)*