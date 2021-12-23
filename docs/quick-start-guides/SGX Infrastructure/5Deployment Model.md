# Deployment Model

Following deployment models are supported

* Enterprise Network Deployment Model

In this model, all services will be deployed on a single enterprise network

    Deploy all services on a SGX Compute Node

    Deploy SGX Agent and SKC Client on SGX Compute Node and all other services on another system


* Cloud Service Provider and Enterprise Network Model

In this model, some services are deployed on CSP network and other services on Enterprise Network.

    Deploy CMS, AAS, SCS, SGX Agent, SKC Client on Cloud Service Provider Network

    Deploy CMS, AAS, SCS and SQVS on Enterprise Network
    
Following diagram illustrates the CSP/Enterprise Network Model

 ![deploy_model](./images/isecl_deploy_model.PNG)


For most of the requirements, Enterprise Network Deployment model should suffice.
