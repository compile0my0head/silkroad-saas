# SilkRoad — AI Social Media Automation SaaS

## Overview
SilkRoad is built for small online stores that sell through social channels.

The platform connects chat-based demand to structured operations:
- capture order intent from chat conversations
- convert it into real orders
- run campaign publishing on schedule

## Why SilkRoad?
Many small stores still process orders manually in chat threads, which is slow and error-prone.

SilkRoad reduces that manual work by combining workflow automation (`n8n`) with a multi-tenant backend that keeps each store isolated.

## System Flow
1. Customer sends a message on Facebook.
2. `n8n` processes the conversation.
3. Backend receives a structured order payload.
4. Order is saved and linked to the right store.
5. Admin reviews and handles the order in the dashboard.

Simple diagram:

Facebook -> n8n -> Backend (.NET) -> Database -> Frontend

## Tech Stack
- Backend: .NET 8 (Clean Architecture)
- Frontend: Angular
- Database: SQL Server
- Automation: n8n

## Architecture
The system follows Clean Architecture:
- Domain: core business model
- Application: services and use-case orchestration
- Infrastructure: EF Core, persistence, and integrations
- Presentation: HTTP API and middleware

Detailed architecture diagrams are available in [backend/ARCHITECTURE_DIAGRAMS.md](backend/ARCHITECTURE_DIAGRAMS.md).

## Key Features
- Multi-tenant store context with access validation
- Chatbot-driven order intake via `n8n`
- Campaign scheduling and automated publishing
- Facebook platform integration

## My Role
- Designed the backend architecture and service boundaries
- Implemented multi-tenant store scoping and validation
- Integrated chatbot order intake flow through `n8n`

## Project Structure
- `/backend` → API, domain, application, and infrastructure code  
- `/docs` → supporting documentation  

## Frontend Scope
Frontend exists (Angular) but is not yet included in this repository.

## Demo
Messenger chatbot order automation demo: https://drive.google.com/file/d/1LzeJ0ZqFYLsVtgQSlz43UZibLd_Y7e34/view?usp=sharing

Campaign scheduling and Facebook publishing demo: https://drive.google.com/file/d/1COTrgfU5QSWIP4TEqJpAygI-Q1qEeucZ/view?usp=sharing

## Status
In development