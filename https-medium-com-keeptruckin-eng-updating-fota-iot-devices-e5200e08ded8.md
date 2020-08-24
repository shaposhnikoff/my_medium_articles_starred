Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m124[39m }

# Updating Firmware Over-the-Air on KeepTruckin IoT Devices

At KeepTruckin, we deploy hundreds of thousands of IoT devices for trailer and equipment tracking in the freight industry. Upgrading firmware on these devices presented an interesting challenge. In this blog, we describe the highly scalable architecture we designed to update the firmware on these devices by leveraging AWS IoT static thing groups and continuous jobs.

## Background

In September 2019 KeepTruckin launched its second [IoT](https://en.wikipedia.org/wiki/Internet_of_things) product, the [Asset Gateway](https://keeptruckin.com/asset-tracking), for real-time trailer and equipment tracking (the first was the Vehicle Gateway, previously known as the ELD). The Asset Gateway enables KeepTruckin customers to track their assets (the trailers hauled), which can outnumber a fleet’s tractors by a ratio of 3-to-1.

![](https://cdn-images-1.medium.com/max/4800/1*05kmWP2j5ptu9XzoK8E2pA.jpeg)

The installed customer base for KeepTruckin IoT devices is growing. With the addition of Asset Gateway, we expect the growth to accelerate. Because any features these devices deliver are tied to the device firmware, a scalable approach to keeping this firmware updated is essential for us to introduce or improve features. And because these devices may be anywhere in the United States — attached to a truck, trailer, cement mixer, or any other moving or stationary equipment — we need to update the firmware in the least disruptive way to the customer’s operations.

Enter Firmware Over-The-Air (FOTA). FOTA is the service responsible for facilitating firmware updates on KeepTruckin IoT devices *without any manual intervention *and *without affecting customer operations at all*.

*(Note that we are using the term “firmware” in a generic sense, to represent the full stack of device software.)*

## Use Cases to Address
> # “A problem well put is half solved” — [John Dewey](https://en.wikipedia.org/wiki/John_Dewey)

### 1. Firmware Upload

The firmware is uploaded via KeepTruckin Admin Console to S3.

### 2. Firmware Signature

Firmware is signed with an intermediate certificate located on AWS Certificate Manager. The resulting signature is stored on S3.

### 3. Device Target Version

Admin Console facilitates setting a device’s *target version.*

### 4. Optimize Bandwidth Usage

There are two variants of firmware: major and minor. Both major and minor releases are tied to a minimum software version (*min_sw_version*). The minor releases are skipped for optimal bandwidth usage, when applicable.

To envision this, let’s say we have the firmware versions shown in Figure 1:

![Figure 1. Major and minor firmware versions](https://cdn-images-1.medium.com/max/2000/1*10I1KUsdsR9_3rwYopADxg.png)*Figure 1. Major and minor firmware versions*

As an example, let’s say a device is running **v64000**. The device’s update path varies depending on the *target version *it is set to:

* The update path for **v65000** is: 
v64000 → v65000

* The update path for **v66000** is: 
v64000 → v65000 → v66000

* The update path for **v66001** is: 
v64000 → v65000 → v66000 → v66001

* The update path for **v66002** is: 
v64000 → v65000 → v66000 → **v66002** 
(we *skip v66001* because its *min_sw_version *matches with v66002)

* The update path for **v66003** is: 
v64000 → v65000 → v66000 → **v66003** 
(we skip *v66002 and v66001* because their *min_sw_version *matches with v66003)

* The update path for **v67000** is: 
v64000 → v65000 → v66000 → **v66003** → v67000 
(we *skip v66002 and v66001* as mentioned in the previous step)

### 5. Downloads to the Device

The device securely downloads the firmware and signature from AWS S3 via a pre-signed URL. The device will retry the downloads if they fail for any reason, such as choppy network conditions.

### 6. Device on Shelf

Sometimes a device ends up sitting on the shelf for an extended period, during which multiple versions of firmware may be released. When such a device is finally connected to a power source, FOTA facilitates the device’s march up to its *target version.*

## Solution

Our tech stack for the solution consists of these components:

[AWS IoT](https://aws.amazon.com/iot/), [AWS Lambda](https://aws.amazon.com/lambda/), [AWS SQS](https://aws.amazon.com/sqs/), [AWS S3](https://aws.amazon.com/s3/), [AWS ACM](https://aws.amazon.com/certificate-manager/), and [Terraform](https://www.terraform.io/)

FOTA functionality is segregated into two smaller microservices:

* **FOTA-signer: **Responsible for signing the firmware.

* **FOTA-scheduler: **Responsible for creating AWS IoT artifacts for each firmware version uploaded on the Admin Console, crafting the device update path, and marching it to the right version.

Figure 2 depicts the solution for use cases #1 and #2:

![Figure 2. FOTA-signer and FOTA-scheduler workflow for use cases #1 and #2](https://cdn-images-1.medium.com/max/4200/1*eGYtUE1JpXFaEK1xIHns2A.png)*Figure 2. FOTA-signer and FOTA-scheduler workflow for use cases #1 and #2*

A couple of useful notes:

* There is a 1:1:1 relationship between a firmware version, static thing group, and continuous job.

* A continuous job contains a *job document* which has the pre-signed S3 URL for the firmware and the signature.

Figure 3 depicts the solution for use cases #3, #4, and #5:

![Figure 3. FOTA-scheduler workflow for use cases #3, #4, and #5](https://cdn-images-1.medium.com/max/3492/1*K7Z93XUaE68KasZatwURUg.png)*Figure 3. FOTA-scheduler workflow for use cases #3, #4, and #5*

A few points of clarification:

* <*thingName*> is the same as the device/identifier.

* Once a device is added to the static thing group, job execution is automatically queued for the same device in the continuous job.

Our IoT device is subscribed to $aws/things/<*thingName*>/jobs/notify-next*. *Once a job execution is queued for a device, the device receives a payload on the specified topic, containing firmware download details.

* Once the device installs the firmware, the device publishes to $aws/events/jobExecution/<*jobId*>/<succeeded/failed/timed_out>* *to update the status.

Figure 4 illustrates how FOTA-scheduler addresses use cases #5 and #6:

![Figure 4. FOTA-scheduler workflow for use case #5 and #6](https://cdn-images-1.medium.com/max/4634/1*UKgcVgJLM4j5ubkybn70AQ.png)*Figure 4. FOTA-scheduler workflow for use case #5 and #6*

Some points of clarification:

* Once a device marks the job execution to “succeeded”, the FOTA-scheduler removes the device from the static thing group. FOTA-scheduler makes another check to confirm that the device has reached the intended *target version*. If it has not, another search for the thing groups is initiated based on the updated shadow values, and the device is added to a different thing group. This is what helps the device to march to the targeted firmware.

* In the event of job execution failure on the device, FOTA retries a maximum of five times, adding the same device to the respective thing group.

## Additional Points to Consider

* We don’t leverage AWS IoT [Over The Air jobs](https://docs.aws.amazon.com/freertos/latest/userguide/freertos-ota-dev.html) because these jobs count towards the overall job limit, and we found this too low for our use case.

* We avoid using dynamic thing groups because AWS has set a hard [limit of 100](https://docs.aws.amazon.com/iot/latest/developerguide/limits-iot.html#fleet-indexing-limits). In our use case, the firmware version is tied to the device model (*n*) and the environment (*m*). Hence, releasing a single firmware version translates into (*n * m) *different thing groups, and this limits us to creating a maximum of (100 / (n * m)) different firmware versions.

* The S3 URL in the job document can be a pre-signed URL or a placeholder URL. The pre-signed URL is valid for a maximum of seven days, while the placeholder URL is valid for a maximum of one hour (once the device has requested the job document).

* AWS IoT has a [hard limit of 5 for the # of query parameters](https://docs.aws.amazon.com/iot/latest/developerguide/limits-iot.html#fleet-indexing-limits) in the dynamic thing group query. Had this not been the case, our use case would have benefitted from more query parameters for better filtering.

* AWS IoT has [throttling limits *per API](https://docs.aws.amazon.com/iot/latest/developerguide/limits-iot.html#throttling-limits)*. For bulk updates, design your application to handle throttling errors. SQS works well for us.

* The default batch size of the events that a lambda receives from the SQS is 10. While processing a batch, if one of the SQS events fails, the whole batch needs to be re-processed again (unless the application has the retry logic built-in, or step functions are leveraged).

## Wrap-Up

FOTA has been in production for a couple of months and has already processed ~130k device updates successfully! Kudos to the team at KeepTruckin whose efforts turned FOTA into a success (sorted lexicographically):

* Our *Embedded* Team: [Andrew Gieraltowski](https://www.linkedin.com/in/andrew-gieraltowski-9879b115b), [Joe Pulver](https://www.linkedin.com/in/joe-pulver-a567514), [Paras Shelawala](https://www.linkedin.com/in/paras-shelawala)

* Our *Devops/SRE* Team: [Chris Argeros](https://www.linkedin.com/in/chr-a), [Luis Zaldivar](https://www.linkedin.com/in/luis-zaldivar)

* Our *Platform* Team: [Filipe Martinho](https://www.linkedin.com/in/filipe-martinho-161b6557/), [Nicolas Vayias](https://www.linkedin.com/in/vayias), [Wael Nasreddine](https://www.linkedin.com/in/kalbasit)
> KeepTruckin is an AWS shop. With the help of top talent and leveraging the latest in cloud technologies, we can roll out new features at an impressive pace. This makes working at KeepTruckin an engaging and fun experience. If you are interested in learning more about the opportunities at KeepTruckin, check out our [Careers](https://keeptruckin.com/careers) page.
