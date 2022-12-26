# Designing VM based workloads on Azure 
Things that architects need to know when designing VM based workloads on Azure. The details include
- The various disk based offerings on Azure
- When to use which
- What are the key features to be mindful of when discussing performance requirements (Latency, Iops, Throughput)
- Matching reliability requirements with the available features
- Understanding the security features available
- How to match these factors against the requirement of the workloads
- Test the performance of the disks for Iops and throughput to validate the design decisions

## Performance
### Disk Types and Characteristics
https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types

### Choosing an Azure Disk - Decision Tree Approach

![image](https://user-images.githubusercontent.com/13979783/209492637-b9f3d96f-ed63-4fa5-8d51-9cbf67eaa53a.png)


### Understanding Bursting

Reference to Official Docs: https://learn.microsoft.com/en-us/azure/virtual-machines/disk-bursting  
#### Comparison of Credit-based bursting with On-demand bursting

| Credit-based   bursting | On-demand   bursting                                                                                          | Changing   performance tier                                         | Changing   performance tier                                                            |   |
|-------------------------|---------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------------------|---|
| Scenarios               | Ideal for   short-term scaling (30 minutes or less).                                                          | Ideal for   short-term scaling(Not time restricted).                | Ideal if your   workload would otherwise continually be running in burst.              |   |
| Cost                    | Free                                                                                                          | Cost   is variable, see the Billing section for details.            | The   cost of each performance tier is fixed, see Managed Disks   pricing for details. |   |
| Availability            | Only available   for premium SSD managed disks 512 GiB and smaller, and standard SSDs 1024 GiB   and smaller. | Only available   for premium SSD managed disks larger than 512 GiB. | Available to   all premium SSD sizes.                                                  |   |
| Enablement              | Enabled by   default on eligible disks.                                                                       | Must be enabled   by user.                                          | User must   manually change their tier.                                                |   |

#### Credit based bursting summarized  

![image](https://user-images.githubusercontent.com/13979783/209492788-a922edf2-4529-4448-9769-aafbaedb1ced.png)

#### On-demand bursting summarized  

![image](https://user-images.githubusercontent.com/13979783/209492862-11b19ae2-3f84-4097-a334-516b472a16dc.png)  

### Azure Disk Performance Scaling Solutions
With so many offerings and the multitude of factors in comparing the options, the usual question that customer would have is Which performance solution should I choose? Architects can use the factors specified in the following table to ask very specific questions before designing the IAAS components

![image](https://user-images.githubusercontent.com/13979783/209493381-928109d2-0571-4a14-bbb5-48148939a928.png)

### Disk Related Metrics
The metrics listed in the following official docs page has all the metrics necessary to understand the performance of the disks and also the VM they are attached to. Added newly are the bursting related metrics that include the bursting that has happened w.r.t the iops & throughput and also the bursting credits that have been utilized (if credit based bursting has been used and on-demand has not been configured)
Reference: https://learn.microsoft.com/en-us/azure/virtual-machines/disks-metrics

### Importance of choosing the right VM 
Excerpt from the MS docs  
"Azure virtual machines have input/output operations per second (IOPS) and throughput performance limits based on the virtual machine type and size. OS disks and data disks can be attached to virtual machines. The disks have their own IOPS and throughput limits.
**Your application's performance gets capped when it requests more IOPS or throughput than what is allotted for the virtual machines or attached disks. When capped, the application experiences suboptimal performance.** This can lead to negative consequences like increased latency"  
The following article talks about some of the super important basics including
- Disk IO capping
- VM IO capping
- Importance of using host based cache
- When to use read-only cache and 
- When to use read-write cache
Reference: https://learn.microsoft.com/en-us/azure/virtual-machines/disks-performance
Additional reference: Choosing the right VM SKUs in the context of SQL workloads- https://sqlperformance.com/2019/11/azure/importance-vm-size 

### Benchmarking Disks
Designing the disks would not be complete if the design decisions have not been tested. The following article provides instructions on how benchmarking can be done and what tools to use  
https://learn.microsoft.com/en-us/azure/virtual-machines/disks-benchmarks

### Performance testing tools
- [Diskspd for Windows](https://github.com/Microsoft/diskspd/wiki/)
- [FIO for linux](http://freecode.com/projects/fio)

## Reliability  (Availability)

### Architecting for High-Availability
When choosing the distribution and redundancy options for high availability, it is important to note that the managed disks attached to the VM are also distributed according to the chosen option  
#### Single Instance, Availability sets and Availability zones- SLA guarantees and the disk placement 
![image](https://user-images.githubusercontent.com/13979783/209507454-268ad85e-c47d-4d8a-90cf-37f4f9c300c6.png)

#### Zone-redundancy in Standard and Premium SKUs
![image](https://user-images.githubusercontent.com/13979783/209507619-6ff277fe-a698-4c9c-b263-fe8733600ac5.png)

## Security
### Encryption of Managed Disks
There are about 4 encryption options that can be used to encrypt Azure managed disks. The following table draws out the available features and the differences when using each of the options. Also, certain options cannot be used in conjunction with the others. For instance, ADE (Azure disk encryption) cannot be used with SSE (Server side encryption) with CMK (Customer Managed Keys) enabled through DES (Disk Encryption Sets)  
These constraints will have to be taken into account when designing the Security of Managed Disks on Azure
Reference: https://learn.microsoft.com/en-us/azure/virtual-machines/disk-encryption-overview
![image](https://user-images.githubusercontent.com/13979783/209510740-9ce085ba-2bc9-449f-b219-b9c034c93cdc.png)

#### Double Encrytion
Excerpt from [MS documentation](https://learn.microsoft.com/en-us/azure/security/fundamentals/double-encryption)  
Microsoft’s approach to enabling two layers of encryption for data at rest is:
- Encryption at rest using customer-managed keys. You provide your own key for data encryption at rest. You can bring your own keys to your Key Vault (BYOK – Bring Your Own Key), or generate new keys in Azure Key Vault to encrypt the desired resources.
- Infrastructure encryption using platform-managed keys. By default, data is automatically encrypted at rest using platform-managed encryption keys.

## Main Video References
- [Deep dive on Azure Managed Disks](https://www.youtube.com/watch?v=2nPZyLmciN4&list=WL&index=7&ab_channel=JohnSavill%27sTechnicalTraining)
- [Azure Disk Storage-Tips for Performance & Scale](https://www.youtube.com/watch?v=i2Xm2HNy92o&ab_channel=MicrosoftAzure)
- [Premium SSD v2 quick peek](https://www.youtube.com/watch?v=GFfztuVtjpI)
- [Understanding disk IO and utilizing disk bursting](https://www.youtube.com/watch?v=a5CISpwo5I8&list=PLQL1JGGe-t0s0D5Vl6VRfc4HqeRgMiTav&index=59&t=146s)
- [Microsoft's playlist on Azure Disk Storage](https://www.youtube.com/watch?v=66kAKOsOSVM&list=PLLasX02E8BPCdLJT8GAWkYCFxzm3evzt0)
- [Disk Storage Security Features](https://www.youtube.com/watch?v=ZnXWa0lFdYg&list=PLLasX02E8BPCdLJT8GAWkYCFxzm3evzt0&index=7&ab_channel=MicrosoftAzure)
- [Disk Encryption Deep Dive](https://www.youtube.com/watch?v=EOXgzTqceok&list=PLQL1JGGe-t0udypvJ4EzDuX1fJTowmDTn&index=33&t=627s&ab_channel=JohnSavill%27sTechnicalTraining)

## Important Note:
The images used in this page are from the videos posted in the Microsoft Azure youtube channel and aren't mine. Links to all of the videos are made available in the section above. Also, the images and the corresponding information are used here just to serve as pointers to architects designing IAAS workloads on Azure and are done with no intention of duplicating the information available on the official MS docs.


  
