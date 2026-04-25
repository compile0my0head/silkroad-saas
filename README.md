# SilkRoad — AI Social Media Automation SaaS

## Overview

SilkRoad is a backend system designed to automate how small online stores handle orders coming from social media.

Instead of manually managing conversations in chat threads, the system transforms unstructured messages into structured orders and integrates them into a controlled workflow.

---

## Why SilkRoad?

Many small stores still process orders manually in chat threads, which leads to:

* lost or duplicated orders
* slow response times
* lack of tracking and organization

SilkRoad addresses this by introducing a structured backend pipeline that connects chat interactions to persistent order management.

---

## System Flow

1. Customer sends a message via Facebook Messenger
2. n8n processes the conversation and extracts intent
3. Backend receives a structured order payload
4. Order is validated and assigned to the correct store (multi-tenant)
5. Admin reviews and manages the order through the system

**Flow:**

```
Facebook Messenger → n8n → Backend (.NET API) → SQL Server → Admin Dashboard
```

---

## Tech Stack

* Backend: .NET 8 (Clean Architecture)
* Frontend: Angular (not included in this repo)
* Database: SQL Server
* Automation: n8n

---

## Architecture

The backend follows a layered architecture focused on separation of concerns:

* **Domain** — business entities and core rules
* **Application** — services handling use cases and orchestration
* **Infrastructure** — EF Core, persistence, and external integrations
* **Presentation** — HTTP API, middleware, and request pipeline

The system also enforces **store-level isolation** using request-scoped context (StoreId), ensuring strict multi-tenancy boundaries.

Detailed diagrams are available in:
`backend/ARCHITECTURE_DIAGRAMS.md`

---

## Key Features

* Multi-tenant store system with request-level isolation
* Chatbot-driven order intake via n8n integration
* Campaign scheduling and automated publishing
* Facebook platform integration

---

## Engineering Focus

This project focuses on solving backend design challenges such as:

* maintaining strict data isolation between tenants
* handling external workflow integration (n8n) without coupling
* structuring services for scalability and clarity
* ensuring consistent request flow using middleware and filters

---

## My Role

I built SilkRoad end-to-end as a solo project, covering both backend and frontend.

My primary focus was backend engineering, including:

* Designing the system architecture (Clean Architecture)
* Implementing multi-tenant store isolation and request scoping
* Building core services for orders, campaigns, and integrations
* Integrating external automation workflows (n8n) for chatbot-driven order intake

On the frontend side (Angular), I developed interfaces to support:

* order management and review
* campaign creation and scheduling
* store-level operations

The frontend is not included in this repository, as this version focuses on backend design and architecture.


---

## Project Structure

* `/backend` → API, domain, application, infrastructure
* `/docs` → architecture and system design

---

## Demo

* Messenger chatbot order automation:
  https://drive.google.com/file/d/1LzeJ0ZqFYLsVtgQSlz43UZibLd_Y7e34/view

* Campaign scheduling & Facebook publishing:
  https://drive.google.com/file/d/1COTrgfU5QSWIP4TEqJpAygI-Q1qEeucZ/view

---

## Status

In development
