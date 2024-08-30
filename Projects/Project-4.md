<h1> Microsoft Azure</h1>
<h1>Project Title: Lift and Shift: Migrating a Multi-Tier Web Application to Microsoft Azure</h1>

**Description**:

Successfully executed a **Lift and Shift** migration of a multi-tier web application from an on-premise environment to Microsoft Azure. This project involved transferring a complex stack, including a React.js frontend, Node.js backend, MySQL and MongoDB databases, NGINX as a web server, Apache Tomcat as an application server, Memcached for caching, and RabbitMQ for messaging. By replicating the existing infrastructure in Azure, we minimized application changes while maximizing the benefits of cloud scalability, security, and cost efficiency.

**Technologies & Tools**:

- **Frontend**: React.js, Azure Blob Storage, Azure CDN

- **Backend**: Node.js, Express, Azure VMs, Azure App Service

- **Database**: Azure Database for MySQL, Cosmos DB (for MongoDB API)

- **Web Server**: NGINX on Azure VMs

- **Application Server**: Apache Tomcat on Azure VMs

- **Caching**: Azure Cache for Redis (Memcached equivalent)

- **Message Queuing**: Azure Service Bus (RabbitMQ equivalent)

- **Infrastructure Management**: Azure Resource Manager (ARM) Templates, Terraform, Azure CLI

**Key Features**:

- **Seamless Migration**: Lift and Shift approach used to migrate the entire application stack to Azure, with minimal changes to the application code.

- **Azure Services Integration**: Utilized Azure VMs for compute resources, Azure Database for MySQL, Cosmos DB for MongoDB, Azure Cache for Redis, and Azure Service Bus for messaging.

- **Scalability and Performance**: Leveraged Azure Load Balancer, Azure Autoscale, and Azure CDN to enhance application performance and scalability.

- **Security and Compliance**: Implemented best practices for securing Azure resources using Virtual Networks (VNets), Network Security Groups (NSGs), Azure Identity and Access Management (IAM), and SSL/TLS encryption.

- **Cost Optimization**: Analyzed and optimized Azure resource utilization to ensure cost-effective operation.


**Project Implementation**:

1. **Preparation and Assessment**:

   - Conducted a thorough assessment of the existing on-premise infrastructure to identify components and dependencies for migration.
  
   - Created a migration plan that outlined the steps for replicating the on-premise environment in Azure, focusing on resource allocation and configuration.

2. **Infrastructure Setup on Azure**:
   
   - **Compute**: Deployed Azure VMs to host the web and application servers (NGINX, Apache Tomcat) and backend services (Node.js, Express).
  
   - **Database**: Migrated MySQL database to Azure Database for MySQL and MongoDB to Cosmos DB using the MongoDB API.
  
   - **Web Server**: Configured NGINX on Azure VMs to act as a reverse proxy and serve static content.
   
   - **Application Server**: Deployed Apache Tomcat on Azure VMs for running Java-based applications.
   
   - **Caching**: Set up Azure Cache for Redis (as a Memcached alternative) for caching frequently accessed data.
  
   - **Message Queuing**: Used Azure Service Bus to manage message queues, replacing the on-premise RabbitMQ.

3. **Data Migration**:
  
   - **Database Migration**: Migrated MySQL databases to Azure Database for MySQL using Azure Database Migration Service and MongoDB databases to Cosmos DB using native tools.
   
   - **File System Migration**: Transferred static assets and media files to Azure Blob Storage, using Azure CDN for global content delivery.

4. **Networking and Security**:
  
   - Configured Azure Virtual Networks (VNets) to isolate resources and manage network traffic securely.
   
   - Set up Network Security Groups (NSGs), Azure IAM roles, and managed identities to enforce security best practices.
   
   - Used Azure DNS for domain management and SSL/TLS certificates for secure HTTPS access.

5. **Testing and Optimization**:
   
   - Conducted extensive testing to ensure the migrated application functions correctly in the Azure environment.
  
   - Optimized Azure resource allocation for cost efficiency and performance.

6. **Go-Live and Monitoring**:
  
   - Executed the final cutover to the Azure environment, ensuring minimal downtime.
   
   - Implemented monitoring and alerting using Azure Monitor and Application Insights to track application health and performance.

