---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# SESSION POLICIES IN AMAZON EKS POD IDENTITY

Amazon EKS Pod Identity has recently added the session policies feature, allowing you to narrow IAM permissions flexibly and precisely for each pod without needing to create many separate IAM roles. This is an important step forward that helps apply the principle of least privilege more effectively in large-scale Kubernetes environments.

Key points to know:

* A session policy is an inline IAM policy specified when creating or updating a Pod Identity association.
* Effective permissions = intersection between the IAM role permissions and the session policy → the session policy can only narrow permissions, not expand them.
* Helps avoid over-permissioning when reusing a single IAM role for multiple workloads with different needs.
* Supports both same-account and cross-account (via IAM role chaining).
* Significantly reduces the number of IAM roles that need to be managed, helping avoid hitting IAM quota limits in large clusters.
* Easily configured through the AWS Management Console, AWS CLI, or AWS SDK when creating an association between a Kubernetes ServiceAccount and an IAM role.

This feature is especially useful when you have many applications running on the same IAM role but need different permission restrictions (for example: one pod only reads a specific S3 bucket, another pod only calls certain APIs).

...Image...

...Link...

...Guide...