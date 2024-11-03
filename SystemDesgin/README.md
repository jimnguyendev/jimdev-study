# System desgin

## What is "System Desgin"?

System Design = Problem solving + Compouter Science

System Design Interview != System Desgin in Reality

- System Desgin in Practice focus more on constraint resources (time, cost, humans) and unknown harmfull effects.

## Architectural Styles vs. Architectural Patterns vs. Design Patterns

- Phong cách kiến trúc (Architectural Styles)
  - Là cách tiếp cận tổng thể để thiết kế hệ thống
  - Định nghĩa cách các thành phần của hệ thống tương tác với nhau
  - Ví dụ: Kiến trúc hướng dịch vụ (Service-Oriented Architecture), Kiến trúc hướng sự kiện (Event-Driven Architecture)
- Mẫu kiến trúc (Architectural Patterns)
  - Là các giải pháp thiết kế cụ thể cho các vấn đề kiến trúc phổ biến
  - Ví dụ: Mẫu kiến trúc Model-View-Controller (MVC), Mẫu kiến trúc Microservices
  - Mẫu kiến trúc thường được sử dụng để giải quyết các vấn đề về cấu trúc và tổ chức hệ thống
- Mẫu thiết kế (Design Patterns)
  - Là các giải pháp thiết kế cụ thể cho các vấn đề thiết kế phổ biến
  - Ví dụ: Mẫu thiết kế Singleton, Mẫu thiết kế Factory
  - Mẫu thiết kế thường được sử dụng để giải quyết các vấn đề về thiết kế và lập trình hướng đối tượng

## You need to show off in the SD interview ?

- Problem Solving
  - How do you clarify the problem?
  - How do you analyze the problem and solutions?
- Solid fundamentals knowledge
  - Computer Science
- Communication skills
  - Proactive
  - Presentation
- Nice to have: Product mindset

> Principle 1: Understand the Problem First. Do not jump to solutions first!

- Clear assumptions

> Principle 2: Distinguish between opinion/assumptions and facts!

## Functional requirements vs Non functional requirements

- How to find key requirements ?
  - (1) The function solves customer's problem directly
  - (2) The function helps biz make money directly
  
- How to find key non-functional requirements ?
  - (1) What hurts he user experience ?
  - (2) What prevents the business growth ?

## Trade off
- What is "Trade off" ?
  - When you have multiple solutions but it is hard to choose a solution.
- What factor affect to the final decision ?
  - The order of non-functional requirements.

## How to Evaluate a Design ?

- Necessary Conditions
  - Đáp ứng đủ yêu cầu
  - Đáp ứng đc timeline, cost, ..
  - Được document và giải thích rõ ràng
- Điều kiện đủ
  - Strong evidences
    - Đặt các câu hỏi tại sao
  - Scalatibility
