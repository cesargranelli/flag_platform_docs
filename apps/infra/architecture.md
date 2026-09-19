# Architecture

## Overview

The Flag Platform infrastructure is designed to run on Oracle Cloud Infrastructure (OCI) Always Free tier, providing a cost-effective solution for development and testing.

## Components

### 1. Compute (VM Ampere A1)

- **Shape**: VM.Standard.A1.Flex
- **OCPU**: 1
- **Memory**: 4 GB
- **OS**: Ubuntu 24.04 LTS
- **Purpose**: Host Docker containers for backend and frontend

### 2. Database (Autonomous Database)

- **Type**: Autonomous Transaction Processing (ATP)
- **Version**: Oracle Database 23ai
- **CPU**: 1 OCPU
- **Storage**: 1 TB
- **Purpose**: Primary data store

### 3. Networking

- **VCN**: 10.0.0.0/16
- **Public Subnet**: 10.0.1.0/24
- **Internet Gateway**: Enabled
- **Security Lists**: SSH (22), HTTP (80), HTTPS (443), API (8080)

### 4. Application

- **Backend**: Spring Boot (Java 21)
- **Frontend**: Flutter Web (Dart)
- **Reverse Proxy**: Nginx
- **Containerization**: Docker

## Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    OCI Always Free                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  VM Ampere A1 (ARM) - 1 OCPU, 4 GB RAM             │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │  Docker: Backend (Spring Boot)              │   │   │
│  │  │  - Port 8080                                │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │  Docker: Frontend (Flutter Web + Nginx)     │   │   │
│  │  │  - Port 80/443                              │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Autonomous Database (ATP)                          │   │
│  │  - 1 OCPU, 1 TB storage                            │   │
│  │  - Oracle Database 23ai                             │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Cost

All resources are within OCI Always Free tier:

- **Compute**: $0.00/month
- **Database**: $0.00/month
- **Storage**: $0.00/month
- **Network**: $0.00/month

**Total: $0.00/month**
