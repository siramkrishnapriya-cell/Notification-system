# 🔔 Real-Time Event-Driven Notification System

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![WebSockets](https://img.shields.io/badge/WebSockets-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![AWS SNS](https://img.shields.io/badge/AWS_SNS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![AWS SQS](https://img.shields.io/badge/AWS_SQS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React.js-61DAFB?style=for-the-badge&logo=react&logoColor=black)

> A **fault-tolerant, horizontally scalable** distributed notification system built on an event-driven architecture — WebSockets for real-time client delivery, AWS SNS/SQS for async decoupled message routing.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Key Features](#key-features)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [How It Works](#how-it-works)
- [Fault Tolerance Design](#fault-tolerance-design)
- [Project Structure](#project-structure)
- [Author](#author)

---

## Overview

This system solves a core distributed systems problem: **reliably delivering real-time notifications to thousands of clients without losing a single message**, even under traffic spikes.

The architecture separates concerns cleanly:
- **WebSockets** handle persistent, low-latency connections to browser clients
- **AWS SNS** fans out events to multiple subscriber queues simultaneously
- **AWS SQS** buffers and decouples producers from consumers
- **MongoDB** stores notification state and delivery history

---

## Architecture
┌─────────────┐       WebSocket        ┌──────────────────┐
│   React.js  │ ◄──────────────────── │  WebSocket Gate  │
│   Client    │                         │  (Node.js)       │
└─────────────┘                         └────────┬─────────┘
│ Publish
┌────────▼─────────┐
│    AWS SNS Topic  │
└────────┬─────────┘
│ Fan-out
┌───────────────────▼──────────────────┐
│                                        │
┌────────▼────────┐                   ┌──────────▼───────┐
│   SQS Queue A   │                   │   SQS Queue B    │
│  (Email Worker) │                   │  (Push Worker)   │
└────────┬────────┘                   └──────────┬───────┘
│                                        │
┌────────▼────────┐                   ┌──────────▼───────┐
│  Worker Service │                   │  Worker Service  │
│  (Stateless)    │                   │  (Stateless)     │
└────────┬────────┘                   └──────────┬───────┘
│                                        │
└──────────────┬─────────────────────────┘
│
┌────────▼────────┐
│    MongoDB       │
│  (State + Logs) │
└─────────────────┘
---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js |
| Real-Time Transport | WebSockets |
| Backend | Node.js, Express.js |
| Message Broker | AWS SNS (pub/sub), AWS SQS (queuing) |
| Database | MongoDB |
| Observability | AWS CloudWatch |
| Infrastructure | AWS EC2, Load Balancer |

---

## Key Features

- ⚡ **Persistent WebSocket connections** — real-time bi-directional delivery with minimal latency
- 🔁 **SNS fan-out** — one event published to multiple SQS subscriber queues simultaneously
- 🛡️ **Zero message loss** — SQS buffering + exponential backoff retries + Dead-Letter Queues (DLQs)
- 📊 **CloudWatch observability** — alarms on queue depth, error rates, and p99 latency
- 📈 **Horizontal scaling** — stateless service layer behind a load balancer
- 🗂️ **MongoDB delivery receipts** — per-user notification history with indexed queries

---

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB (local or Atlas)
- AWS account with SNS, SQS, and CloudWatch access
- AWS CLI configured (`aws configure`)

### Installation

```bash
# Clone the repo
git clone https://github.com/siramkrishnapriya-cell/Notification-System.git
cd Notification-System

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env

# Start the server
npm run dev

Environment Variables
PORT=5000
MONGO_URI=mongodb://localhost:27017/notifications

AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key

SNS_TOPIC_ARN=arn:aws:sns:ap-south-1:xxxx:notifications
SQS_QUEUE_URL=https://sqs.ap-south-1.amazonaws.com/https://notification-queue
DLQ_URL=https://sqs.ap-south-1.amazonaws.com/https://notification-dlq

How It Works

	1.	Client connects via WebSocket to the gateway server
	2.	Event is triggered (e.g., new message, system alert)
	3.	Producer publishes the event to an AWS SNS topic
	4.	SNS fans out the message to all subscribed SQS queues
	5.	Worker services poll SQS queues and process notifications asynchronously
	6.	Processed notification pushed to client via WebSocket and saved to MongoDB
	7.	Failed messages after max retries routed to the Dead-Letter Queue
	8.	CloudWatch monitors DLQ depth and triggers alarms if messages accumulate

Fault Tolerance Design
|Failure Scenario                |Mechanism                                                   |
|--------------------------------|------------------------------------------------------------|
|Consumer crashes mid-processing |SQS visibility timeout — message becomes re-processable     |
|Traffic spike overwhelms workers|SQS buffers indefinitely; workers process at their own rate |
|Persistent processing failure   |Exponential backoff retries → DLQ after max attempts        |
|Worker instance goes down       |Stateless design — load balancer routes to healthy instances|
|Silent failures in production   |CloudWatch alarms alert on DLQ depth and error rates        |

Project Structure
Notification-System/
├── client/                  # React.js frontend
│   └── src/
│       ├── components/
│       └── hooks/useWebSocket.js
├── server/
│   ├── gateway/             # WebSocket server
│   ├── producers/           # SNS publishers
│   ├── workers/             # SQS consumers
│   ├── models/              # MongoDB schemas
│   └── middleware/
├── infra/                   # AWS setup scripts
├── .env.example
└── README.md

Author
Siram Krishna Priya
📧 siramkrishnapriya@gmail.com
