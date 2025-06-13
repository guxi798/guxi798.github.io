layout: page
title: "Be Cautious with AWS EFS Pricing Model"
permalink: docs/

We recently completed the initial development and deployment of a bioinformatic pipeline with AWS ECS and EFS, and entered into the testing phase. Being cost conscious, I also kept an eye on the AWS cost associated with the tests. 
Unexpected, EFS turned out to be quite costly, with around ~$40 per run, which can add up to $80 per study.  

I did some investigations and realized that the default mode for EFS is `Elastic Throughput`, also recommended by AWS. This mode charges extra for read and write operations (per GB), on top of storage. As we consumes large sequence databases 
(several hundreds GB) repeatedly, the default mode quickly mounts up on our budget.  
