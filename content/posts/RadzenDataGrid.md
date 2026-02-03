+++
date = '2026-02-03T14:33:39-05:00'
draft = false
title = 'Using Radzen Data Grid'
wight = 3
tags = ['design', 'Radzen']
categories = ['architecture', 'Blazor']
+++

## What this document is **not** about!

This document is not a tutorial on using the Radzen Data Grid. The vendor has [documented](https://blazor.radzen.com/datagrid?theme=material3) their product. 

## What is this document?

I often find myself facing a dilemma: should I directly query my database from my front-end application, or should I instead rely on an external service to fetch the data needed to populate my DataGrid? This decision can significantly affect the application's architecture, performance, and security.

Hence, I started a chat with an AI agent and this is the summary of my discussion.

## **Architecture Summary**

### **1. Data Access Strategy**

For this application, we have opted for an **External API Service** (Backend-for-Frontend) rather than direct database access from the frontend.

- **Security:** Keeps database credentials and connection strings on the server.

- **Decoupling:** Allows the database schema to change without breaking the frontend.

- **Performance:** Enables server-side data shaping, ensuring only the necessary 11 columns are sent over the network.

### **2. Data Retrieval & Batching**

To populate the **Holdings DataGrid** (approx. 300 to 2,000 rows), the system will use a **single-call batch strategy**.

- **Avoid "Chatty" APIs:** Instead of calling separate endpoints for pricing, ratings, and index status, the API will aggregate all data into a single **DTO (Data Transfer Object)**.

- **Table Joins:** The server will perform a 4-table join (Holdings, Pricing, Ratings, and Index Components) using **EF Core IQueryable** to generate a flat result set.

- **Payload Efficiency:** With ~2,000 records containing 5-6 numeric fields and short strings, the payload remains under **1 MB**, which is well within the "Green Zone" for modern web performance.

### **3. Handling Computed Values (Price Slopes)**

- **Current Strategy:** Since the dataset is small (up to 2,000 rows), these can be calculated at runtime within the API's LINQ query.

- **Future Proofing:** If calculations become more complex, these values can be pre-computed by a background process and stored in a database table for immediate retrieval.

### **4. Frontend Integration (Radzen DataGrid)**

The Radzen DataGrid will bind to a local `List<PortfolioRowDto>` received from the API.

- **In-Memory Operations:** For 2,000 rows, filtering, sorting, and grouping should be performed **client-side** for an "instant" user experience.

- **UI Performance:** To maintain a smooth 60fps scroll as the portfolio grows, **Virtualization** (`AllowVirtualization="true"`) should be enabled. This ensures the browser only renders the rows visible in the current viewport.

| **Component**  | **300 Rows (Current)** | **2,000 Rows (Target)**       | **10,000+ Rows (Limit)** |
| -------------- | ---------------------- | ----------------------------- | ------------------------ |
| **API Calls**  | Single Batch           | Single Batch                  | Server-side Paging       |
| **Data Logic** | Runtime Calculation    | Runtime/Pre-compute           | Pre-computed Only        |
| **Grid Mode**  | Standard               | **Virtualization Enabled**    | Server-side LoadData     |
| **User Feel**  | Instant                | Instant (with Virtualization) | Dependent on Latency     |
