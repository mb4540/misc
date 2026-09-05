ArchitectureNotes


Attachment Screenshot 2026-07-28 at 8.27.25 AM.png added. None selected 

Skip to content
Using Gmail with screen readers
sedlock 
7 of many
Fw: SAG Architecture: Database - Lakebase PostgreSQL
Inbox
AI Overview
David forwarded an overview of Databricks Lakebase PostgreSQL functionality.
Lakebase adds OLTP support to Databricks, unifying operational data with analytics.
By Gemini; there may be mistakes. Learn more

David Sedlock
Mon, Jul 20, 5:58 PM (12 days ago)
to me, Lawrence




David Sedlock
Jemba9 Technology
469-486-1581 
dsedlock@jemba9.com




From: David Sedlock <DSedlock@jemba9.com>
Sent: Wednesday, July 8, 2026 5:00 PM
To: David Sedlock <DSedlock@jemba9.com>
Subject: SAG Architecture: Database - Lakebase PostgreSQL

Lakebase is a fully managed PostgreSQL database that is built directly into the Databricks Data Intelligence Platform.
PostgreSQL (often called Postgres) is one of the world's most widely used open-source relational databases. 
It enables you to run transactional applications (OLTP) and AI agents on the same platform where you perform analytics, machine learning, and data engineering.
With Lakebase, Databricks moves beyond being a lakehouse into a platform that can power both operational applications and AI agents while keeping data, governance, and intelligence together.  This is why Lakebase is considered one of the platform's most strategically significant additions.
Existing applications can usually connect using standard PostgreSQL drivers with little or no code changes.. 
It's commonly used for: Banking applications, SaaS products, ERP systems, CRM platforms, E-commerce, Mobile apps, Web application

PostgreSQL (often called Postgres) is one of the most powerful and widely used relational database management systems (RDBMS) in the world. It is open source, highly extensible, and known for its reliability, standards compliance, and advanced features. Companies such as Apple, Microsoft, Cisco, NASA, Reddit, Stripe, Shopify, and many financial institutions use PostgreSQL for mission-critical applications.

******************************************
Lakebase Postgres is one of the biggest architectural changes Databricks has made since introducing the Lakehouse.
In simple terms:
Lakebase is a fully managed PostgreSQL database that is built directly into the Databricks Data Intelligence Platform. It enables you to run transactional applications (OLTP) and AI agents on the same platform where you perform analytics, machine learning, and data engineering. (Databricks)
Why did Databricks build Lakebase?
Traditionally, enterprise architectures looked like this:
Customer App
       │
   PostgreSQL
       │
      ETL
       │
Databricks Lakehouse
       │
Analytics
       │
Machine Learning
       │
AI
This architecture has several drawbacks:
Duplicate data
ETL pipelines to maintain
Analytics lag behind operational data
AI works on stale information
Multiple security and governance models
With Lakebase, the architecture becomes:
Applications
AI Agents
Web APIs
Mobile Apps
        │
        ▼
   Lakebase (Postgres)
        │
        ▼
Unity Catalog
        │
        ▼
Delta Lake
        │
        ▼
Analytics
ML
AI
Dashboards
Everything lives within the Databricks ecosystem under a unified governance model. (Databricks)
What is PostgreSQL?
PostgreSQL (often called Postgres) is one of the world's most widely used open-source relational databases.
It's commonly used for:
Banking applications
SaaS products
ERP systems
CRM platforms
E-commerce
Mobile apps
Web applications
Lakebase is fully PostgreSQL compatible, so existing applications can usually connect using standard PostgreSQL drivers with little or no code changes. (Databricks)
What problem does Lakebase solve?
Historically, Databricks excelled at analytical (OLAP) workloads but wasn't designed to replace operational databases.

OLTP (Operational)
OLAP (Analytical)
Thousands of small transactions
Large analytical queries
Millisecond response
Seconds to minutes
Customer applications
BI, reporting, AI
PostgreSQL
Databricks SQL
Lakebase fills the OLTP gap, allowing Databricks to support both operational and analytical workloads. (PR Newswire)
What makes Lakebase different from Amazon RDS or Azure Database for PostgreSQL?
Managed PostgreSQL services from cloud providers focus primarily on running databases.
Lakebase adds tight integration with the rest of the Databricks platform.
Key capabilities include:
Native integration with Unity Catalog
Delta Lake synchronization
Automatic scaling (including scale-to-zero in autoscaling mode)
Instant branching and cloning for development
Point-in-time recovery
AI agent support
Unified governance and security
Serverless management with no database infrastructure to maintain (Databricks Documentation)
Where does Lakebase fit?
Business Applications
Customer Portals
Web APIs
AI Agents
Chatbots
        │
        ▼
───────────────
 Lakebase
(PostgreSQL)
───────────────
        │
        ▼
Unity Catalog
        │
        ▼
Delta Lake
        │
        ▼
Lakeflow
        │
        ▼
MLflow
Vector Search
AI Gateway
Agent Bricks
Genie
Think of Lakebase as the operational data layer that feeds and is governed alongside the rest of the Databricks ecosystem.
Why is Lakebase important for AI?
AI agents need more than a language model—they often need to maintain state and interact with applications.
Lakebase can be used to store:
User sessions
Shopping carts
Agent memory
Workflow state
Orders
Customer profiles
Inventory
Financial transactions
Application metadata
Because it's integrated with the lakehouse, AI systems can immediately combine operational data with analytical data without complex ETL pipelines. (Databricks)
Example: Retail
A customer places an order.
Without Lakebase
Website
    │
PostgreSQL
    │
Nightly ETL
    │
Databricks
    │
Analytics
The analytics platform may not see the new order until the ETL job runs.
With Lakebase
Website
     │
Lakebase
     │
Immediate availability
     │
Unity Catalog
     │
Dashboards
AI Agents
Forecasting
Recommendations
Operational systems, dashboards, and AI applications all work from current data.
When should you use Lakebase?
Lakebase is a good fit when you need:
Customer-facing applications
REST APIs
Transactional systems
AI agent memory
Real-time inventory
Order processing
Session management
Millisecond read/write performance
Operational workloads integrated with analytics
For large-scale analytical processing or historical reporting, Delta Lake remains the primary storage layer.
How does Lakebase fit into the Databricks vision?
Databricks has evolved its platform over time:
2013–2023: Analytics and data engineering
2023–2024: Generative AI, Vector Search, and model serving
2025: Agentic AI with Agent Bricks, AI Gateway, and Lakebase
2026: A unified platform for analytics, AI, and operational applications
With Lakebase, Databricks moves beyond being a lakehouse into a platform that can power both operational applications and AI agents while keeping data, governance, and intelligence together. This is why Lakebase is considered one of the platform's most strategically significant additions. (PR Newswire)





David Sedlock
Jemba9 Technology
469-486-1581 
dsedlock@jemba9.com





sedlock. Press tab to insert.