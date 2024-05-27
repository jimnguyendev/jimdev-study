# Kubernetes Architecture

Kubernetes is a portable, extensible, open source platform for managing containerized workloads and services.

> "Portable" trong ngữ cảnh của Kubernetes nghĩa là có khả năng di chuyển, chạy hoặc triển khai trên nhiều môi trường khác nhau mà không cần phải thay đổi quá nhiều. Trong trường hợp của Kubernetes, nó có khả năng di động và có thể hoạt động trên nhiều hạ tầng khác nhau mà không yêu cầu sự điều chỉnh lớn.

[![Lịch sử triển khai phần mềm](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)](https://kubernetes.io/docs/concepts/overview/#:~:text=Storage%20orchestration%20Kubernetes%20allows%20you,public%20cloud%20providers%2C%20and%20more.)

[![Kiến trúc k8s](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)](https://kubernetes.io/docs/concepts/overview/components/)

![Kiến trúc k8s](https://www.aquasec.com/wp-content/uploads/2020/11/Kubernetes-101-Architecture-Diagram.jpg)
_<center>High level Kubernetes architecture diagram showing a cluster with a master and two worker nodes</center>_

## Kubernetes cung cấp:

- Service discovery and load balancing
- Storage orchestration
- Automated rollouts and rollbacks
- Automatic bin packing
- Self-healing
- Secret and configuration management
- Batch execution
- Horizontal scaling
- IPv4/IPv6 dual-stack
- Designed for extensibility 
