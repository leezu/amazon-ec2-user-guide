# Elastic Fabric Adapter<a name="efa"></a>

An Elastic Fabric Adapter \(EFA\) is a network device that you can attach to your Amazon EC2 instance to accelerate High Performance Computing \(HPC\) and machine learning applications\. EFA enables you to achieve the application performance of an on\-premises HPC cluster, with the scalability, flexibility, and elasticity provided by the AWS Cloud\.

EFA provides lower and more consistent latency and higher throughput than the TCP transport traditionally used in cloud\-based HPC systems\. It enhances the performance of inter\-instance communication that is critical for scaling HPC and machine learning applications\. It is optimized to work on the existing AWS network infrastructure and it can scale depending on application requirements\.

EFA integrates with Libfabric 1\.11\.1 and it supports Open MPI 4\.0\.5 and Intel MPI 2019 Update 7 for HPC applications, and Nvidia Collective Communications Library \(NCCL\) for machine learning applications\.

**Note**  
The OS\-bypass capabilities of EFAs are not supported on Windows instances\. If you attach an EFA to a Windows instance, the instance functions as an Elastic Network Adapter, without the added EFA capabilities\.

**Topics**
+ [EFA basics](#efa-basics)
+ [Supported interfaces and libraries](#efa-mpi)
+ [Supported instance types](#efa-instance-types)
+ [Supported AMIs](#efa-amis)
+ [EFA limitations](#efa-limits)
+ [Get started with EFA and MPI](efa-start.md)
+ [Get started with EFA and NCCL](efa-start-nccl.md)
+ [Work with EFA](efa-working-with.md)
+ [Monitor an EFA](efa-working-monitor.md)
+ [Verify the EFA installer using a checksum](efa-verify.md)

## EFA basics<a name="efa-basics"></a>

An EFA is an Elastic Network Adapter \(ENA\) with added capabilities\. It provides all of the functionality of an ENA, with an additional OS\-bypass functionality\. OS\-bypass is an access model that allows HPC and machine learning applications to communicate directly with the network interface hardware to provide low\-latency, reliable transport functionality\.

![\[Contrasting a traditional HPC software stack with one that uses an EFA.\]](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/efa_stack.png)

Traditionally, HPC applications use the Message Passing Interface \(MPI\) to interface with the system’s network transport\. In the AWS Cloud, this has meant that applications interface with MPI, which then uses the operating system's TCP/IP stack and the ENA device driver to enable network communication between instances\.

With an EFA, HPC applications use MPI or NCCL to interface with the *Libfabric* API\. The Libfabric API bypasses the operating system kernel and communicates directly with the EFA device to put packets on the network\. This reduces overhead and enables the HPC application to run more efficiently\.

**Note**  
Libfabric is a core component of the OpenFabrics Interfaces \(OFI\) framework, which defines and exports the user\-space API of OFI\. For more information, see the [Libfabric OpenFabrics](https://ofiwg.github.io/libfabric/) website\.

### Differences between EFAs and ENAs<a name="efa-differences"></a>

Elastic Network Adapters \(ENAs\) provide traditional IP networking features that are required to support VPC networking\. EFAs provide all of the same traditional IP networking features as ENAs, and they also support OS\-bypass capabilities\. OS\-bypass enables HPC and machine learning applications to bypass the operating system kernel and to communicate directly with the EFA device\.

## Supported interfaces and libraries<a name="efa-mpi"></a>

EFA supports the following interfaces and libraries:
+ Open MPI 4\.0\.5
+ Intel MPI 2019 Update 7
+ NVIDIA Collective Communications Library \(NCCL\) 2\.4\.2 and later

## Supported instance types<a name="efa-instance-types"></a>

The following instance types support EFAs:
+ General purpose: `m5dn.24xlarge` \| `m5n.24xlarge` \| `m5zn.12xlarge` \| `m5zn.metal`
+ Compute optimized: `c5n.18xlarge` \| `c5n.metal`  \| `c6gn.16xlarge`
+ Memory optimized: `r5dn.24xlarge` \| `r5dn.metal` \| `r5n.24xlarge` \| `r5n.metal` 
+ Storage optimized: `i3en.24xlarge` \| `i3en.metal`
+ Accelerated computing: `g4dn.metal` \| `inf1.24xlarge` \| `p3dn.24xlarge` \| `p4d.24xlarge`

The available instance types vary by Region\. To see the available instance types that support EFA in a Region, use the [describe\-instance\-types](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instance-types.html) command with the `--region` option and the appropriate Region code\.

```
aws ec2 describe-instance-types --region us-east-2 --filters Name=network-info.efa-supported,Values=true --query "InstanceTypes[*].[InstanceType]" --output text
```

The following is example output\.

```
g4dn.metal
i3en.24xlarge
r5n.24xlarge
c5n.18xlarge
m5n.24xlarge
inf1.24xlarge
m5dn.24xlarge
c5n.metal
p3dn.24xlarge
i3en.metal
r5dn.24xlarge
```

## Supported AMIs<a name="efa-amis"></a>

The following AMIs support EFA with Intel x86\-based instance types:
+ Amazon Linux and Amazon Linux 2
+ CentOS 7 and 8
+ RHEL 7\.6, 7\.7, 7\.8, 8\.2, and 8\.3
+ Ubuntu 16\.04, 18\.04, and 20\.04
+ SUSE Linux Enterprise 15 SP2
+ openSUSE Leap 15\.2

The following AMIs support EFA with Arm\-based \(Graviton 2\) instance types:
+ Amazon Linux 2
+ Ubuntu 18\.04 and 20\.04
+ openSUSE Leap 15\.2

## EFA limitations<a name="efa-limits"></a>

EFA has the following limitations:
+ `p4d.24xlarge` instances support up to four EFAs\. All other supported instance types support only one EFA per instance\.
+ EFA OS\-bypass traffic is limited to a single subnet\. In other words, EFA traffic cannot be sent from one subnet to another\. Normal IP traffic from the EFA can be sent from one subnet to another\.
+ EFA OS\-bypass traffic is not routable\. Normal IP traffic from the EFA remains routable\.
+ The EFA must be a member of a security group that allows all inbound and outbound traffic to and from the security group itself\.
+ EFA traffic between C6gn instances and other EFA\-enabled instances is not supported\.